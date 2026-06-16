# asyncio.gather vs TaskGroup (Python 3.11+)

Python 3.11 introduced `asyncio.TaskGroup` as a safer alternative to `asyncio.gather`.

## Old Way: asyncio.gather

```python
import asyncio

async def fetch(url, delay):
    await asyncio.sleep(delay)
    return f"Data from {url}"

async def main():
    # gather runs all at once, returns results in order
    results = await asyncio.gather(
        fetch("api1", 1),
        fetch("api2", 2),
        fetch("api3", 0.5),
    )
    print(results)

asyncio.run(main())
```

### Problem: One failure doesn't cancel others

```python
async def main():
    try:
        results = await asyncio.gather(
            fetch("api1", 1),
            failing_task(),      # raises after 0.1s
            fetch("api3", 0.5),  # still runs to completion!
        )
    except Exception as e:
        print(f"Got error: {e}")
        # Other tasks are NOT cancelled automatically
```

## New Way: TaskGroup

```python
async def main():
    async with asyncio.TaskGroup() as tg:
        t1 = tg.create_task(fetch("api1", 1))
        t2 = tg.create_task(fetch("api2", 2))
        t3 = tg.create_task(fetch("api3", 0.5))
    # All tasks complete when exiting the context manager
    print(t1.result(), t2.result(), t3.result())

asyncio.run(main())
```

### TaskGroup cancels all tasks on failure

```python
async def main():
    try:
        async with asyncio.TaskGroup() as tg:
            t1 = tg.create_task(fetch("api1", 1))
            t2 = tg.create_task(failing_task())   # raises
            t3 = tg.create_task(fetch("api3", 5)) # cancelled!
    except* ValueError as eg:
        print(f"Errors: {eg.exceptions}")
```

## Comparison

| Feature | gather | TaskGroup |
|---------|--------|-----------|
| Python | 3.7+ | 3.11+ |
| Error handling | First exception, others continue | All cancelled, ExceptionGroup |
| Dynamic tasks | No | Yes (add in callbacks) |
| Structured concurrency | No | Yes |
| Return exceptions | `return_exceptions=True` | No (use try/except per task) |

## When to Use Each

```python
# Use gather for independent fire-and-forget
results = await asyncio.gather(*tasks, return_exceptions=True)

# Use TaskGroup for structured, safe concurrent operations
async with asyncio.TaskGroup() as tg:
    for item in items:
        tg.create_task(process(item))
```
