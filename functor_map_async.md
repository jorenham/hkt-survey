# Functor-style map over an `asyncio.Future`

Based on: [python/typing#548 (comment)][proposal]

Kind: `K` is $\ast \to \ast$ (the `asyncio.Future` operator).

## Definition

```python
import asyncio
from typing import Any, Callable, Origin

def map_async[InT, OutT, K: Origin[asyncio.Future[Any]]](
    function: Callable[[InT], OutT],
    future: K[InT],
) -> K[OutT]:
    out: asyncio.Future[OutT] = asyncio.get_running_loop().create_future()

    def on_done(in_: asyncio.Future[InT], /) -> None:
        out.set_result(function(in_.result()))

    future.add_done_callback(on_done)

    return out
```

## Call sites

```python
fut: asyncio.Future[int] = ...
tsk: asyncio.Task[int] = ...  # asyncio.Task[T] <: asyncio.Future[T]

reveal_type(map_async(str, fut))  # asyncio.Future[str]
reveal_type(map_async(str, tsk))  # asyncio.Task[str]

map_async(str.upper, fut)  # E: (str) -> str not applicable to InT=int
```

[proposal]: https://github.com/python/typing/issues/548#issuecomment-1898377873
