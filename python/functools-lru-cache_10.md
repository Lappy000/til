# functools.lru_cache for Memoization

```python
from functools import lru_cache

@lru_cache(maxsize=128)
def fib(n):
    if n < 2: return n
    return fib(n-1) + fib(n-2)

# Cache info
fib.cache_info()  # CacheInfo(hits=0, misses=10, maxsize=128, currsize=10)

# Clear cache
fib.cache_clear()

# Python 3.9+ unlimited cache
@lru_cache(maxsize=None)
def expensive(x): ...
```
