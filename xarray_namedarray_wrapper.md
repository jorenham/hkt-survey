# `xarray.NamedArray`: a wrapper generic over the wrapped array type

Source: [xarray/namedarray/core.py][src]

xarray wraps arbitrary duck arrays (NumPy, CuPy, sparse, ...). At runtime, methods keep both the wrapper's subclass and the wrapped array type, while e.g. `astype` changes the dtype. Ideally the wrapper would be generic in the wrapped array type itself.

Kind: `Origin[Self]` and `A` are both $\ast \to \ast \to \ast$, nested: the inner constructor is recovered from a type argument of `self`.

## Definition

```python
from typing import Any, Origin, Protocol, Self

class duckarray[ShapeT_co, DTypeT_co](Protocol):
    @property
    def shape(self) -> ShapeT_co: ...
    @property
    def dtype(self) -> DTypeT_co: ...

class NamedArray[ArrayT: duckarray[Any, Any], DimT]:
    @property
    def data(self) -> ArrayT: ...
    def __neg__(self) -> Self: ...  # works today
    def astype[A: Origin[duckarray], S, D, *Ts: _](
        self: Origin[Self][A[S, Any], *Ts],
        dtype: D,
    ) -> Origin[Self][A[S, D], *Ts]: ...

class Variable[ArrayT: duckarray[Any, Any], DimT](NamedArray[ArrayT, DimT]): ...
```

## Call sites

```python
import numpy as np

x = Variable(("x",), np.zeros(3))
reveal_type(x)  # Variable[np.ndarray[tuple[int], np.dtype[np.float64]], str]
reveal_type(x.astype(np.dtype(np.int32)))  # Variable[np.ndarray[tuple[int], np.dtype[np.int32]], str]

y: Variable[sparse.COO, str]  # not generic, kind *: nothing to re-apply
reveal_type(y.astype(np.dtype(np.int32)))  # Variable[sparse.COO, str]
```

Wrapped array types differ in arity and may order their parameters differently from `duckarray[ShapeT, DTypeT]` (see #8), so re-applying `A[S, D]` by position isn't always right.

[src]: https://github.com/pydata/xarray/blob/357efad180e54c3e9c419c5d441e68f06e2006eb/xarray/namedarray/core.py#L313-L351
