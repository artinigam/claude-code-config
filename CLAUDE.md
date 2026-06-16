# CLAUDE.md — ReschedulingEngine (Team Aurora)

## Context

Backend Python service — FastAPI, gRPC, Kafka, PostgreSQL, MongoDB.
Zero Touch critical path. Any outage directly impacts revenue.
Stack: Python 3.11, pytest, Temporal for orchestration, Statsig for feature flags.
Team: 6-engineer pod (Team Aurora). Bangalore-based.

## What you own vs. what you ask me first

**Proceed autonomously:**
- Writing/editing code, tests, and configs within the repo
- Running read-only commands (grep, find, git log, git diff)
- Generating drafts, plans, and design notes

**Always ask before:**
- Any `git push`, deployment, or production operation
- Deleting files or dropping database objects
- Calling external APIs that mutate state
- Toggling feature flags above 10% traffic
- Schema migrations on tables with live traffic

## Code standards (actionable, not aspirational)

- All functions must have type hints — no bare `def foo(x)`
- No `SELECT *` — always name the columns you need
- Parameterized queries only — no f-string or %-interpolated SQL
- Every new endpoint needs a unit test for the happy path + at least one failure path
- Structured logging on all critical path operations: include `job_id`, `request_id`, latency
- No `TODO` comments merged to main — convert to tickets before merge
- No module-level singletons for external clients — use dependency injection
- Async endpoints must not make blocking I/O calls; use `asyncio.gather` or `run_in_executor`

## Idempotency rules

- Any operation that publishes a Kafka event must include an idempotency key
- Idempotency key format: `{entity_id}:{operation}:{requested_at_epoch}`
- Downstream consumers are responsible for deduplication on this key

## Secrets

Never put credentials, API keys, or tokens in any file.
Use environment variables. Deploy script: `./deploy.sh` — credentials injected by CI, never passed inline.

## Push back on me when

- I'm about to write synchronous blocking code inside an async context
- I'm proposing a storage migration without a rollback plan
- A function is doing more than one thing (violates SRP)
- I'm about to merge code with a `# TODO: handle errors` comment
- Error handling returns a generic 500 with no per-item failure semantics in a batch operation
- A new endpoint has no authorization check

## Off limits

- Never run `git push` or `./deploy.sh` without explicit instruction
- Never generate or store secrets inline
- Never disable or skip tests to make CI pass
- Never use `SELECT *` on the `reschedule_events` table — it is on the critical path
