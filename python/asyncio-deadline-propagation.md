# Propagate Async Deadlines with `asyncio.timeout_at`

Python 3.11's `asyncio.timeout()` is handy, but `timeout_at()` is better when one request fans out into several awaits: every helper shares the same absolute deadline.

```python
import asyncio
from contextvars import ContextVar

request_deadline: ContextVar[float] = ContextVar("request_deadline")

def remaining() -> float:
    return max(0.0, request_deadline.get() - asyncio.get_running_loop().time())

async def call_service(name: str, delay: float) -> str:
    # Each nested operation consumes from the original request budget.
    async with asyncio.timeout_at(request_deadline.get()):
        await asyncio.sleep(delay)
        return f"{name} ok with {remaining():.2f}s left"

async def handle_request(budget_seconds: float) -> list[str]:
    loop = asyncio.get_running_loop()
    token = request_deadline.set(loop.time() + budget_seconds)
    try:
        async with asyncio.TaskGroup() as tg:
            a = tg.create_task(call_service("profile", 0.2))
            b = tg.create_task(call_service("billing", 0.4))
        return [a.result(), b.result()]
    finally:
        request_deadline.reset(token)

print(asyncio.run(handle_request(1.0)))
```

FastAPI middleware can set the same `ContextVar` once per request, then routers, DB wrappers, and HTTP clients can call `remaining()` for per-call timeouts.

```python
# timeout = httpx.Timeout(remaining())
# rows = await asyncio.wait_for(query(), timeout=remaining())
```

This avoids the common bug where three helpers each get a fresh 1-second timeout and the endpoint actually runs for 3+ seconds.
