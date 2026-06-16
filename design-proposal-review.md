# ReschedulingEngine V2 — Design Proposal Review

**Author:** Raj Mehta
**Status:** Draft for design review
**Reviewer:** New EM (you)

---

## Critical Issues — Push Back Hard (Block Until Resolved)

### 1. Storage section — PostgreSQL → MongoDB migration with no real justification

The proposal states:
> "Document model fits event data better than relational. Sharding gives us horizontal scale."

`reschedule_events` is an append-only audit log. That is a relational pattern:
ordered, immutable rows, queried by `job_id` and timestamp range. MongoDB's document
model adds no value here. The data isn't nested or schemaless.

**Questions to ask the author:**
- What specific query patterns does MongoDB unlock that PostgreSQL can't serve?
- What is the migration plan for existing rows in `reschedule_events`?
- What is the rollback plan if Atlas has an outage during rollout?
- Have you benchmarked this? Latency goal is 800ms → 500ms — is storage actually the bottleneck?

**Stance:** Do not approve the storage switch without a concrete benchmark and migration plan.
This is a high-risk change to a revenue-critical table with no evidence it solves the actual problem.

---

### 2. Current state notes + Goals section — AURORA-1247 (double-fire) absent from the design

The team wiki explicitly states:
> "Known issue: `job.rescheduled` sometimes double-fires (AURORA-1247)."
> "Idempotency is 'handled by the scheduler' — listed as an open question at the time and not revisited."

The V2 design proposes adding a new `POST /reschedule/batch` endpoint that publishes
`job.rescheduled` events, yet idempotency does not appear anywhere in the design.

This is not a minor omission. Shipping a new high-volume batch endpoint that inherits
a known double-fire bug on the critical path is not acceptable.

**Questions to ask the author:**
- What is the idempotency strategy for `job.rescheduled` events?
- Will the batch endpoint use idempotency keys? How will consumers deduplicate?
- Is AURORA-1247 a prerequisite for this design to go forward?

**Stance:** Idempotency strategy must be in the design before implementation starts.

---

### 3. Error handling section — One line for a batch of 1000 jobs

The entire error handling section reads:
> "If SchedulerV2 returns an error, return error to client."

For a single-job endpoint this is thin but workable. For a batch of 1000 jobs it is not a design:
- What happens to the 600 jobs that already succeeded when job 601 fails?
- Is this all-or-nothing or best-effort per item?
- What is the retry contract? Can callers safely resubmit the full batch?
- What is the rollback strategy if the DB write succeeds but Kafka publish fails?

**Questions to ask the author:**
- Define the batch failure contract: all-or-nothing vs. partial success with per-item results.
- What triggers a retry, and how does idempotency make retries safe?
- What happens in the window between a successful gRPC call and a failed Kafka publish?

---

### 4. Flow section (steps 1-5) — No atomicity between DB write → gRPC → Kafka

The proposed flow is:
1. Client posts request
2. Service writes to MongoDB
3. Service calls SchedulerV2 RPC
4. Publishes `job.rescheduled` to Kafka
5. NotificationService consumes

Steps 2, 3, and 4 are three separate operations with no transaction boundary.
If step 4 (Kafka) fails after step 3 (gRPC) succeeds, the reschedule is confirmed
in the scheduler but no notification ever fires. The customer is rescheduled but
never told.

**Stance:** The design must define how these three steps are made consistent, or
explicitly document the failure mode and its acceptable business impact.

---

### 5. Current state — 15% OldSchedulerV1 traffic unaddressed in rollout plan

Team wiki states:
> "The legacy adapter still routes ~15% of traffic through OldSchedulerV1 — cleanup PR never finished."

One stated goal is "Finish the SchedulerV2 migration." The rollout plan (5% → 25% → 100%)
does not explain:
- Does the feature flag gate only V2 traffic, or all traffic including the 15%?
- What happens to in-flight V1 jobs when the flag flips to 25%?
- Is the V1 cleanup a prerequisite for this design, or does V2 run alongside V1?

---

## Major Issues — Ask to Justify

### 6. Batch endpoint — 1000 synchronous gRPC calls per request is not a plan

`POST /reschedule/batch` accepts up to 1000 jobs. The design does not state whether
these are processed sequentially or concurrently.

- Sequential: 1000 × ~30ms gRPC call = 30 seconds per batch request. Misses the <500ms goal by 60x.
- Concurrent: What is the concurrency limit? What is the timeout? How does the service
  avoid overwhelming SchedulerV2?

**Ask:** Show the math. What is the expected p99 for a 1000-job batch? What is the
timeout strategy and the downstream impact on SchedulerV2?

---

### 7. Rollout plan — No success criteria or abort triggers

Rollout: 5% week 1 → 25% week 2 → 100% week 3.

There are no defined:
- Success metrics to observe between each step
- Error rate or latency thresholds that trigger a rollback
- Rollback procedure at each stage

**Ask:** What does "success at 5%" look like? What metric causes you to pause or rollback?

---

### 8. "Open" section — Intentionally left blank

The design explicitly marks the Open Questions section as:
> "(intentionally left blank for review)"

This means either:
- The author believes there are no open questions (unlikely given the issues above), or
- The author is deferring all unknowns to the reviewer

Neither is acceptable before implementation starts. The author should enumerate what
they know is unresolved, not leave it for reviewers to discover.

---

## What to Approve

- The goal of supporting batch rescheduling is valid and well-motivated.
- Feature flag rollout approach is the right pattern.
- Finishing the SchedulerV2 migration as part of this work is the right time to do it.
- The p99 latency goal (800ms → 500ms) is measurable and specific.

---

## Summary Stance

Approve direction, not implementation. Send back for:
1. Idempotency strategy for `job.rescheduled` (required before any code is written)
2. Batch failure contract definition (all-or-nothing vs. partial success)
3. Atomicity/consistency model across DB write → gRPC → Kafka
4. MongoDB migration benchmarks and rollback plan (or drop the migration)
5. Rollout success criteria and abort thresholds
6. V1 legacy traffic handling during rollout
