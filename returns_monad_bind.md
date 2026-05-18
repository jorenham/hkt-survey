# Bind over any monad, preserving the concrete container

Source: [returns: Higher-Kinded Types][returns-hkt]

`returns` cannot write a function generic over *any* user container without
HKT: a `Bindable`-typed parameter loses the concrete `Maybe`/`IO` on return.
It emulates the fix with `Kind1` + `SupportsKindN` + `@kinded`; here the bound
is the `Bindable` typeclass itself.

Kind: `T` is $\ast \to \ast$ (the `Bindable` protocol).

## Definition

```python
from collections.abc import Callable
from typing import Origin, Protocol

class Bindable[V](Protocol):
    def bind[W](self, f: Callable[[V], Bindable[W]], /) -> Bindable[W]: ...

class Maybe[V](Bindable[V]):
    def bind[W](self, f: Callable[[V], Maybe[W]], /) -> Maybe[W]: ...
    @classmethod
    def from_value(cls, v: V, /) -> Maybe[V]: ...

class IO[V](Bindable[V]):
    def bind[W](self, f: Callable[[V], IO[W]], /) -> IO[W]: ...
    @classmethod
    def from_value(cls, v: V, /) -> IO[V]: ...

# Kleisli composition over *any* Bindable, kept concrete.
def then[T: Origin[Bindable], V, W](
    c: T[V],
    f: Callable[[V], T[W]],
    /,
) -> T[W]: ...
```

## Call sites

```python
def to_str_maybe(x: int) -> Maybe[str]: ...
def succ_io(x: int) -> IO[int]: ...
def to_str_io(x: int) -> IO[str]: ...

m = Maybe.from_value(1)
# infer T = Maybe, V = int, W = str; both c and f's result pin T
reveal_type(then(m, to_str_maybe))  # Maybe[str]

i = IO.from_value(1)
reveal_type(then(i, succ_io))  # IO[int]
# without HKT the best obtainable type is Bindable[int]

# the callback must return the *same* constructor T
then(m, to_str_io)  # E: IO[str] is not Maybe[W]
```

[returns-hkt]: https://returns.readthedocs.io/en/latest/pages/hkt.html
