# Runtime-Checkable Protocol for Structural Subtyping

## Problem

Need to check if an object implements an interface without inheritance.

## Solution

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Renderable(Protocol):
    def render(self) -> str: ...

class Widget:
    def render(self) -> str:
        return "<widget/>"

# Works without inheriting from Renderable
assert isinstance(Widget(), Renderable)  # True
```

## Key Points

- `@runtime_checkable` enables `isinstance()` checks at runtime
- Only checks method existence, not signatures (at runtime)
- Static type checkers (mypy, pyright) validate full signatures
- Useful for plugin systems and duck-typed APIs
- Performance: isinstance check is O(n) where n = protocol methods

## Caveat

Runtime checks only verify method existence, not return types or full behavior.
