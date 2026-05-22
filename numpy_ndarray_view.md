# `ndarray.view(type=SubArray)` binds the subclass constructor

Source: [`__init__.pyi`][src]

Kind $\ast \to \ast \to \ast$.

## Definition

```python
from typing import Origin, overload

class ndarray[
    ShapeT_co: tuple[int, ...] = tuple[Any, ...],
    DTypeT_co: dtype[Any] = dtype[Any],
]:
    @overload  # no args
    def view[DTypeT: dtype](self,) -> Self: ...
    @overload  # dtype
    def view[DTypeT: dtype](
        self,
        dtype: DTypeT,
    ) -> Origin[Self][ShapeT_co, DTypeT]: ...
    @overload  # type
    def view[ArrayT: ndarray](
        self,
        type: type[ArrayT],
    ) -> Origin[ArrayT][ShapeT_co, DTypeT_co]: ...
    @overload  # dtype + type
    def view[DTypeT: dtype, ArrayT: ndarray](
        self,
        dtype: DTypeT,
        type: type[ArrayT],
    ) -> Origin[ArrayT][ShapeT_co, DTypeT]: ...
```

Note that in reality there should be fallback overloads in case `dtype` isn't
statically known (e.g. `dtype="d"` and even `dtype=object` are valid).

## Call sites

```python
import numpy as np

vec_i8: np.ndarray[tuple[int], np.dtype[np.int8]] = ...

vec_i8_masked = vec_i8.view(type=np.ma.MaskedArray)
reveal_type(vec_i8_masked)  # MaskedArray[tuple[int], np.dtype[int8]]
```

Today this isn't possible, causing false positives when acessing `MaskedArray`-specific
attributes such as `vec_i8_masked.mask`.

[src]: https://github.com/numpy/numpy/blob/bf8007a5/numpy/__init__.pyi#L3951-L3952
