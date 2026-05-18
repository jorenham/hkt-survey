# Functor-style map over an arbitrary Awaitable

Source: [python/typing#548 (comment)][proposal]

Kind: `T` is $\ast \to \ast$ (the `Awaitable` operator).

## Definition

```python
from collections.abc import Awaitable, Callable
from typing import Origin

# T is the Awaitable type operator, applied by subscription.
def map_async[V, R, T: Origin[Awaitable]](
    fn: Callable[[V], R],
    aval: T[V],
    /,
) -> T[R]: ...
```

## Call sites

```python
task: asyncio.Task[int]

# from aval = Task[int]: infer T = asyncio.Task, V = int; fn: int -> str
reveal_type(map_async(str, task))  # asyncio.Task[str]

# aval is not T[V] for any Awaitable type constructor T
map_async(str, 3)  # E: int is not an Awaitable[V]

# fn domain must match the inner type V of aval (here V = int)
map_async(str.upper, task)  # E: (str) -> str not applicable to V=int
```

[proposal]: https://github.com/python/typing/issues/548#issuecomment-1898377873
