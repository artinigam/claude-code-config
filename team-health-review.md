# Team Aurora — Health Snapshot Review

**Sprint 47, Day 7 of 10**
**Sprint goal:** Ship `/reschedule/batch` endpoint behind feature flag.

---

## Headline Signal: Deploy Frequency Collapse

```
Deploys per week: 6 → 4 → 2 → 1 → 0
```

This is the single most important number in this snapshot. Everything else is a symptom.
A team that has stopped deploying has stopped getting feedback from production. The sprint
burndown, the open PRs, the SLO breaches — all of these follow from this.

**Root cause read:** Something in the last 4 weeks made shipping feel risky or expensive.
Candidates: broken CI, fear of the known AURORA-1247 bug, no clear ownership of on-call,
or accumulated tech debt that makes every change feel dangerous.

**First-week action:** Identify the exact moment deploys dropped and what changed. Ask
the team directly: "What would need to be true for you to feel safe deploying today?"

---

## Burndown

```
Day 1:  20 pts remaining
Day 3:  17 pts
Day 4:  18 pts  ← +3 pts added mid-sprint (AURORA-1318, VP request)
Day 7:  18 pts  ← TODAY (zero progress in 3 days)
```

**Diagnosis:** Sprint goal is at serious risk with 3 days left. Zero velocity for 3 days
means the team is either blocked, context-switching, or both. The mid-sprint scope addition
(AURORA-1318) absorbed points without a tradeoff conversation — something was not negotiated.

---

## Open Ticket Analysis

| Ticket | Owner | Status | Issue |
|--------|-------|--------|-------|
| AURORA-1298 | Raj | In Progress | No commits in 2 days — likely blocked or overloaded |
| AURORA-1247 | Priya | In Review | PR open 4 days, unresolved comment — review cycle stalled |
| AURORA-1318 | Anil | In Progress | VP-injected mid-sprint, no tradeoff made |
| AURORA-1102 | Raj | Blocked | **No blocker named** — this is the most urgent conversation |

**Pattern:** Raj owns both the sprint goal (AURORA-1298) and the carry-over blocker
(AURORA-1102). Two high-stakes items, one engineer, no visible progress in 2 days.

---

## PR Throughput

| Engineer | S45 | S46 | S47 so far |
|----------|-----|-----|------------|
| Priya | 5 | 6 | 4 |
| Raj | 4 | 4 | **1** |
| Anil | 5 | 4 | 3 |
| Devansh | 6 | 5 | 4 |

**Raj is the outlier.** His throughput dropped from 4/sprint to 1 mid-sprint while
owning both the sprint goal and a blocked carry-over. This is not a performance issue —
it's a load and blocker issue. He needs an unblocking conversation today, not a
performance conversation next week.

---

## Service Signals

| Signal | Value | Target | Status |
|--------|-------|--------|--------|
| Reschedule success rate | 99.2% | 99.5% | BREACHED |
| p99 latency | 920ms | 800ms | BREACHED |
| EligibilityDSL alerts | 156 fires (30d) | — | Alert fatigue |
| p99 latency alerts | 42 fires, mostly weekends, unacknowledged | — | On-call dark |
| Last incident | AURORA-1247 duplicate events, Sev-3, 6 days ago | — | Unresolved bug still live |

**Key read on alerts:** 156 EligibilityDSL alerts that are "nearly all just over the
threshold" means the threshold is miscalibrated, not that there are 156 real incidents.
This is alert fatigue — the team has learned to ignore this alert. When the team ignores
alerts, real incidents get missed. The 42 unacknowledged p99 latency alerts on weekends
confirm this: nobody is looking.

---

## Root Causes (Named)

1. **Unmanaged blocker (AURORA-1102):** Raj has a blocked ticket with no blocker named.
   This is the EM's first call on Day 1.

2. **Stalled review cycle (AURORA-1247):** PR open 4 days with an unresolved comment.
   Either the reviewer hasn't had time, or there's a disagreement nobody is naming.
   Unblock today.

3. **Scope injection without tradeoff (AURORA-1318):** A VP request was absorbed mid-sprint
   with no visible negotiation. This added 3 points to an already at-risk sprint. The team
   needs a process for handling mid-sprint requests that doesn't silently erode commitments.

4. **Alert fatigue:** 156 miscalibrated alerts have trained the team to ignore the alert
   channel. Real incidents (p99 latency, success rate SLO breach) are going unacknowledged
   on weekends.

5. **Deploy fear:** The collapse in deploy frequency (6 → 0) suggests the team has lost
   confidence in their ability to ship safely. AURORA-1247 (known double-fire bug) is
   likely a contributor — shipping feels risky when a known bug is unresolved.

---

## First-Week Actions

| Day | Action | Why |
|-----|--------|-----|
| Day 1 | 1:1 with Raj — name the AURORA-1102 blocker explicitly | Unnamed blockers don't get resolved |
| Day 1 | Unblock AURORA-1247 PR — resolve the open comment or make a decision | 4-day old PRs kill momentum |
| Day 1 | Push back on AURORA-1318 scope with VP — what was the tradeoff? | Protect the sprint goal or explicitly descope |
| Day 2 | Team sync on deploy frequency — "what would make you feel safe deploying today?" | Diagnose the deploy fear root cause |
| Day 3 | Recalibrate EligibilityDSL alert threshold — move it to where it's actually actionable | Kill alert fatigue before it hides a real incident |
| Week 1 | Establish PR review SLA (e.g., first review within 24 hours) | 4-day open PRs are a process gap, not a people gap |
| Week 1 | Make AURORA-1247 (idempotency) a P0 for Sprint 48 | It's blocking safe deploys and the V2 design |

---

## What Good Looks Like by End of Week 1

- AURORA-1102 blocker named and owned
- AURORA-1247 PR merged or unblocked
- Deploy frequency discussion held — team has a shared view of why it collapsed
- Alert threshold recalibrated — on-call has < 10 actionable alerts/week
- Sprint 48 planning starts with AURORA-1247 idempotency fix as first item
