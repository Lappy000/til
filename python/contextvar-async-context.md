# ContextVar for Async-Safe State

## Problem
Global/thread-local state breaks with asyncio (shared thread).

## Solution

```python
from contextvars import ContextVar

_ctx: ContextVar[RequestContext] = ContextVar("request_ctx")

# In middleware
token = _ctx.set(RequestContext(request_id="abc"))
try:
    response = await handler(request)
finally:
    _ctx.reset(token)

# Anywhere in async code
ctx = _ctx.get()
logger.info(f"[{ctx.request_id}] processing")
```

## Properties
- Each async task gets own copy
- Works with asyncio.create_task() (child inherits)
- Thread-safe
- Zero-cost when not accessed
