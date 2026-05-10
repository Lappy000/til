# asyncio.gather() Error Handling: Don't Lose Exceptions Silently

`asyncio.gather()` has a subtle default behavior that silently loses exceptions. Here's how to handle it properly.

## The Trap

By default, if one task in `gather()` raises an exception, the other tasks **keep running** but the exception propagates immediately, and you lose their results:

```python
import asyncio

async def fetch_user(user_id: int) -> dict:
    await asyncio.sleep(0.1)
    if user_id == 2:
        raise ValueError(f"User {user_id} not found")
    return {"id": user_id, "name": f"User {user_id}"}

async def main():
    # This raises immediately on the first failure
    results = await asyncio.gather(
        fetch_user(1),
        fetch_user(2),  # This will raise
        fetch_user(3),
    )
    # Never reaches here
    print(results)

asyncio.run(main())
# ValueError: User 2 not found
```

## Fix 1: return_exceptions=True

Returns exceptions as values instead of raising them:

```python
async def main():
    results = await asyncio.gather(
        fetch_user(1),
        fetch_user(2),
        fetch_user(3),
        return_exceptions=True,
    )

    for i, result in enumerate(results):
        if isinstance(result, Exception):
            print(f"Task {i} failed: {result}")
        else:
            print(f"Task {i} succeeded: {result}")

# Task 0 succeeded: {'id': 1, 'name': 'User 1'}
# Task 1 failed: User 2 not found
# Task 2 succeeded: {'id': 3, 'name': 'User 3'}
```

## Fix 2: TaskGroup (Python 3.11+, Preferred)

`TaskGroup` is the modern replacement. It cancels siblings on failure and gives you proper exception groups:

```python
async def main():
    try:
        async with asyncio.TaskGroup() as tg:
            task1 = tg.create_task(fetch_user(1))
            task2 = tg.create_task(fetch_user(2))
            task3 = tg.create_task(fetch_user(3))
    except* ValueError as eg:
        for exc in eg.exceptions:
            print(f"Caught: {exc}")
    else:
        print(task1.result(), task2.result(), task3.result())
```

**Critical difference:** `TaskGroup` cancels ALL remaining tasks when one fails. `gather(return_exceptions=True)` lets them all complete.

## Fix 3: Wrapper Pattern for Partial Failures

When you want all tasks to complete AND handle errors per-task:

```python
from dataclasses import dataclass
from typing import TypeVar, Generic

T = TypeVar("T")

@dataclass
class Result(Generic[T]):
    value: T | None = None
    error: Exception | None = None

    @property
    def ok(self) -> bool:
        return self.error is None

async def safe_execute(coro) -> Result:
    """Wrap a coroutine to always return a Result."""
    try:
        value = await coro
        return Result(value=value)
    except Exception as e:
        return Result(error=e)

async def main():
    results = await asyncio.gather(
        safe_execute(fetch_user(1)),
        safe_execute(fetch_user(2)),
        safe_execute(fetch_user(3)),
    )

    succeeded = [r.value for r in results if r.ok]
    failed = [r.error for r in results if not r.ok]
    print(f"Succeeded: {len(succeeded)}, Failed: {len(failed)}")
    # Succeeded: 2, Failed: 1
```

## When to Use What

| Scenario | Approach |
|----------|----------|
| All must succeed or all fail | `TaskGroup` (3.11+) |
| Want all results, even partial | `gather(return_exceptions=True)` |
| Need per-task error handling | Wrapper pattern with `safe_execute` |
| Fire and forget | `gather()` default (but don't) |

## Key Takeaway

Never use bare `asyncio.gather()` in production without either `return_exceptions=True` or wrapping in error handlers. On Python 3.11+, prefer `TaskGroup` for the cancel-on-failure semantic. Use the `Result` wrapper pattern when you need all tasks to complete regardless of individual failures.