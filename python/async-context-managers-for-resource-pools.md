# Async Context Managers for Resource Pools

Using `async with` for database connection pools and HTTP session management.

## Pattern

```python
import asyncio
from contextlib import asynccontextmanager

class AsyncResourcePool:
    def __init__(self, max_size=10):
        self._semaphore = asyncio.Semaphore(max_size)
        self._resources = []

    async def acquire(self):
        await self._semaphore.acquire()
        resource = await self._create_resource()
        self._resources.append(resource)
        return resource

    async def release(self, resource):
        await self._destroy_resource(resource)
        self._resources.remove(resource)
        self._semaphore.release()

    async def _create_resource(self):
        # Simulate connection
        await asyncio.sleep(0.01)
        return {"id": id(object()), "conn": "active"}

    async def _destroy_resource(self, resource):
        resource["conn"] = "closed"

@asynccontextmanager
async def pooled(pool):
    resource = await pool.acquire()
    try:
        yield resource
    finally:
        await pool.release(resource)

# Usage
async def main():
    pool = AsyncResourcePool(max_size=5)
    async with pooled(pool) as conn:
        print(f"Using {conn}")

asyncio.run(main())
```

## Benefits

- Automatic cleanup on exceptions
- Bounded concurrency without manual counting
- Composable with other async context managers
