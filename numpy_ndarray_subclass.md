# ndarray subclass survives slicing and arithmetic

Sources:

- numpy/numpy#29807 (slice -> ndarray, not Self)
- numpy/numpy#20072 (a + b loses the subclass)
- current numpy stubs: [`ndarray.__getitem__`][np-getitem], [`ndarray.__add__`][np-add] (overload spaghetti that drops the subclass)

Kind: `T` is $\ast \to \ast \to \ast$ (`ndarray[ShapeT, DTypeT]`),
recovered from `self`.

## Definition

```python
from typing import Origin

class dtype[ScalarT]: ...
class float64: ...

# T is recovered from `self` as a type constructor and reapplied unchanged.
# Note: T binds to a *non-generic* subclass, while ShapeT/DTypeT come from
# the generic base; the subclass is not itself generic over them.
class ndarray[ShapeT, DTypeT]:
    # -- snip --
    @overload
    def __getitem__[T: Origin[ndarray]](
        self: T[ShapeT, DTypeT],
        i: slice,
        /,
    ) -> T[ShapeT, DTypeT]: ...
    # -- snip --

    # -- snip --
    @overload
    def __add__[T: Origin[ndarray]](
        self: T[ShapeT, DTypeT],
        rhs: T[ShapeT, DTypeT],
        /,
    ) -> T[ShapeT, DTypeT]: ...
    # -- snip --


class C(ndarray[tuple[int, ...], dtype[float64]]):
    # no overrides of __getitem__/__add__
    def foo(self) -> None: ...
```

## Call sites

```python
c: C

# infer T = C from `self`; ShapeT/DTypeT fixed by C's base
reveal_type(c[:2])  # C
c[:2].foo()  # ok (today: E: ndarray has no attribute "foo")

reveal_type(c + c)  # C
```

[np-getitem]: https://github.com/numpy/numpy/blob/e54990c7/numpy/__init__.pyi#L2146-L2156
[np-add]: https://github.com/numpy/numpy/blob/e54990c7/numpy/__init__.pyi#L4244-L4290
