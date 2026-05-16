# asyncio.TaskGroup Structured Concurrency (3.11+)

## Pattern

```python
import asyncio

async def main():
    async with asyncio.TaskGroup() as tg:
        t1 = tg.create_task(fetch("https://api.example.com/a"))
        t2 = tg.create_task(fetch("https://api.example.com/b"))
    
    # All tasks guaranteed complete here
    results = [t1.result(), t2.result()]
```

## Error Behavior

- If ANY task raises, ALL other tasks are cancelled
- All exceptions collected into `ExceptionGroup`
- No orphaned tasks possible (structured concurrency)

## Handling ExceptionGroup

```python
try:
    async with asyncio.TaskGroup() as tg:
        tg.create_task(might_fail())
except* ValueError as eg:
    for exc in eg.exceptions:
        print(f"ValueError: {exc}")
except* TypeError as eg:
    for exc in eg.exceptions:
        print(f"TypeError: {exc}")
```

## vs gather()

| Feature | TaskGroup | gather() |
|---------|-----------|----------|
| Cancel on error | Automatic | Only with return_exceptions=False |
| Exception type | ExceptionGroup | First exception OR list |
| Add tasks dynamically | Yes | No (fixed at call time) |
