# PR #1842 Review — `/reschedule/batch` endpoint

**Author:** Raj Mehta
**Branch:** raj/aurora-1298-batch-endpoint → main
**Files changed:** 4
**Verdict:** Request Changes — Do NOT merge (5 independent critical blockers)

---

## Critical Issues (Must Fix — Block Merge)

### 1. `handler.py:13` — `db` is undefined; service crashes on every request

`db.query(...)` is called but `db` is never imported, injected, or instantiated anywhere
in this file. Every call to `handle_batch` raises `NameError: name 'db' is not defined`
at runtime. Complete functional failure.

**Fix:** Use FastAPI dependency injection:
```python
from sqlalchemy.orm import Session
from .database import get_db

def handle_batch(batch: BatchRequest, db: Session):
    ...
```

---

### 2. `scheduler_client.py:3-4` — `# TODO: handle errors` shipped to production

gRPC stubs raise `grpc.RpcError` on any transport failure, timeout, or service-side error.
The call on line 4 is completely unwrapped. A single bad job in a batch of 1000 causes
the entire batch to fail with a 500, losing all results computed up to that point.

**Fix:** Per-item try/except inside the loop:
```python
import grpc

try:
    response = scheduler.reschedule(job_id=job.job_id, requested_at=job.requested_at)
except grpc.RpcError as e:
    logger.error("gRPC reschedule failed", extra={
        "job_id": job.job_id, "grpc_code": e.code(), "grpc_details": e.details()
    })
    results.append({"job_id": job.job_id, "status": "error", "reason": "scheduler_unavailable"})
    continue
```

---

### 3. `handler.py:24-27` — AURORA-1247 (Kafka double-fire) not resolved; shipped as-is

Team wiki explicitly documents `job.rescheduled` sometimes double-fires and idempotency
was an open question never revisited. This PR ships `publish_event("job.rescheduled", ...)`
with no idempotency key. On a revenue-impacting critical path, a double-fired event can
cause duplicate scheduling, duplicate billing, or duplicate state transitions downstream.

**Fix:** Add idempotency key to every Kafka publish:
```python
publish_event("job.rescheduled", {
    "job_id": job.job_id,
    "new_schedule": response.new_schedule_id,
    "idempotency_key": f"{job.job_id}:reschedule:{job.requested_at}",
})
```

---

### 4. `api.py:2` + `handler.py` — Blocking synchronous code inside `async def`

`async def reschedule_batch` (api.py:2) calls synchronous `handle_batch` which makes
up to 1000 blocking gRPC calls sequentially. Blocking I/O in the async event loop starves
all other in-flight requests for the full duration of the batch. Will cause cascading
latency under any real load.

**Fix:** Use `asyncio.gather` with an async gRPC client (grpc.aio) or offload to thread pool:
```python
await asyncio.get_event_loop().run_in_executor(None, handle_batch, req, db)
```

---

### 5. `tests/test_batch.py:1-4` — The one test does not pass as written

`db` is undefined in scope, no mocks for `scheduler` or `kafka`. The test raises
`NameError` before any assertion runs. This is not a test; it is a placeholder.

---

## Major Issues (Should Fix Before Enabling in Production)

### 6. `handler.py:11-15` — N+1 query: one `SELECT` per job in a loop of 1000

One `db.query(SELECT * FROM schedules WHERE job_id = %s)` fires inside the loop
for every job. 1000 jobs = 1000 round-trips to PostgreSQL.

**Fix:** Batch fetch before the loop:
```python
job_ids = [job.job_id for job in batch.jobs]
schedules = {
    row["job_id"]: row
    for row in db.query("SELECT job_id, ... FROM schedules WHERE job_id = ANY(%s)", job_ids)
}
```

---

### 7. `handler.py:23-30` — No transactional guarantee between gRPC success → Kafka publish

If Kafka publish fails after `scheduler.reschedule()` succeeds, the reschedule is
confirmed in the scheduler but no `job.rescheduled` event fires. Customer sees a
reschedule that never triggers a notification. No rollback, no at-least-once guarantee.

---

### 8. `handler.py:5` — Module-level singleton `NewSchedulerV2Client()` is not injectable

`scheduler = NewSchedulerV2Client()` at module load time means:
- Tests cannot mock without monkey-patching the module
- gRPC channel created at import time — fails if service is down at startup
- No connection lifecycle management

**Fix:** Use `Depends(get_scheduler_client)` in the FastAPI route signature.

---

### 9. `api.py:5` — Wrong HTTP status code for disabled feature flag

`raise HTTPException(403, "Not enabled")` returns 403 Forbidden. A disabled feature
flag is not an authorization failure. Use `501 Not Implemented`. 403 will trigger
false authorization alerts in monitoring.

---

### 10. `handler.py:8-31` — Zero logging on a revenue-critical path

No `job_id`, no `request_id`, no timing, no error details anywhere in `handle_batch`.
Completely dark in production. Impossible to debug failures or build alerting.

---

## Minor Issues

| File | Line | Issue |
|------|------|-------|
| `scheduler_client.py` | 2 | No type hints on `reschedule(self, job_id, requested_at)` |
| `handler.py` | 14 | `SELECT *` — select only needed columns |
| `handler.py` | 24-27 | Kafka payload missing `requested_at` — consumers can't deduplicate |
| `handler.py` | 8 | Missing return type hint on `handle_batch` |

---

## Testing Checklist

- [ ] Happy path unit test
- [ ] gRPC failure path (per-item exception)
- [ ] Empty batch input
- [ ] Batch at max size (1000 items)
- [ ] Partial batch failure (some succeed, some fail)
- [ ] Kafka publish failure after scheduler success
- [ ] Feature flag disabled path (correct HTTP status)
- [ ] `db`, `scheduler`, `kafka` all properly mocked

## Security Checklist

- [ ] Batch size max enforced in `BatchRequest` model (not confirmed in PR)
- [ ] Authorization check on `/reschedule/batch` endpoint (not visible — confirm middleware)
- [ ] Rate limiting at gateway level (not visible)
- [ ] Parameterized queries confirmed (uses `%s` — correct, but `db` object must be DBAPI2 compliant)

## Performance Checklist

- [ ] N+1 queries — FAIL (must batch)
- [ ] Blocking I/O in async context — FAIL (must fix)
- [ ] gRPC channel reuse — unconfirmed
- [ ] Connection pooling for `db` — unconfirmed

---

## Open Questions for Author

1. Is `handle_batch` intended all-or-nothing or best-effort per item? Must be documented.
2. What is the gRPC call timeout? With 1000 sequential calls, what is the worst-case duration?
3. How is the 15% OldSchedulerV1 traffic handled? Can V1 jobs appear in a batch request?
4. When `response.success` is false, is there an error code on the response object that should be surfaced to callers?
5. Has a decision been made on AURORA-1247, or is this shipping unresolved?
