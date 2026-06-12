# collections.Counter Patterns

```python
from collections import Counter

# Most common elements
c = Counter('abracadabra')
c.most_common(3)  # [('a', 5), ('b', 2), ('r', 2)]

# Arithmetic
c1 = Counter(a=3, b=1)
c2 = Counter(a=1, b=2)
c1 + c2   # Counter({'a': 4, 'b': 3})
c1 - c2   # Counter({'a': 2})  # keeps only positive

# Bulk update
c.update('aaa')
c.subtract('aa')  # can go negative
```
