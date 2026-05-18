# A greenlet group generic in its greenlet type constructor

Source: [typeshed: stubs/gevent/gevent/pool.pyi][src]

The stub's own TODO: *"We would need higher-kinded TypeVars if we wanted to
give up neither"* -- i.e. be generic in `Greenlet` *and* keep spawn/apply
return types specific, while still mixing greenlets in one group.

Kind: `G` is $\ast \to \ast$ (the `Greenlet` operator over its result).

## Definition

```python
from collections.abc import Callable
from typing import Origin

class Greenlet[R]: ...

# G is a type constructor; the group is generic in it, so spawn/apply
# stay precise without pinning every greenlet to one type.
class Group[G: Origin[Greenlet]]:
    def add(self, greenlet: G[object]) -> None: ...
    def spawn[R](self, func: Callable[[], R], /) -> G[R]: ...
    def apply[R](self, func: Callable[[], R], /) -> R: ...

class MyGreenlet[R](Greenlet[R]):
    def link(self) -> None: ...
```

## Call sites

```python
def make_int() -> int: ...
def make_str() -> str: ...

g: Group[MyGreenlet]

# infer R = int from func; result is the group's own constructor G[int]
reveal_type(g.spawn(make_int))  # MyGreenlet[int]

g.spawn(make_int).link()  # ok (today: Greenlet has no attribute "link")

reveal_type(g.apply(make_str))  # str

# group only accepts its own greenlet constructor
g.add(Greenlet[object]())  # E: Greenlet is not a MyGreenlet[object]
```

[src]: https://github.com/python/typeshed/blob/ee212088/stubs/gevent/gevent/pool.pyi#L166-L199
