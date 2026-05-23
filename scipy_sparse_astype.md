# `scipy.sparse._spbase.astype`

Source: [scipy-stubs/sparse/_base.pyi][src]

Today this is spelled as 14 per-subclass overloads, plus a `Self`-typed "same dtype"
overload and a `DTypeLike` catch-all. `Self` does not help:
`astype` *changes* the scalar parameter, so the return is not `Self`, it's "the same
constructor as `self`, re-applied with a new scalar".

The subclasses also disagree on arity: some pin the shape parameter, others leave it
open. Like the `dict`/`Counter` case in [dict_fromkeys.md](dict_fromkeys.md), the
higher-kinded variable must be reconciled with each subclass's own arity.

Kind: `Origin[Self]` is $\ast \to \ast$ or $\ast \to \ast \to \ast$, recovered from
`self` and re-applied with `ScalarT` swapped.

## Definition

```python
from typing import Any, Origin, Self, overload
import numpy as np

# type variable bounds and defaults omitted; see [src]
class _spbase[ScalarT_co, ShapeT_co: tuple[int, ...]]:
    @overload  # known dtype
    def astype[ScalarT, *Ts: _](
        self: Origin[Self][Any, *Ts],
        dtype: np.dtype[ScalarT] | type[ScalarT],
    ) -> Origin[Self][ScalarT, *Ts]: ...
    @overload  # dtype not statically known ("d", builtins.int, etc.)
    def astype[*Ts: _](
        self: Origin[Self][Any, *Ts],
        dtype: np.typing.DTypeLike,
    ) -> Origin[Self][Any, *Ts]: ...

class bsr_array[ScalarT_co](_spbase[ScalarT_co, tuple[int, int]]): ...
class csr_array[ScalarT_co, ShapeT_co: tuple[int, ...]](_spbase[ScalarT_co, ShapeT_co]): ...
# 12 other subclasses omitted
```

Splitting each `*Ts` overload into per-arity versions (one for unary subclasses, one
for binary) would also work, but is less general: the subclasses aren't `final`/sealed,
so a downstream subclass could introduce a new arity that neither version covers. The
`: _` on `*Ts` means "bound inferred from the constructor": `*Ts` picks up whatever
bounds the tail parameters of the resolved `Origin[Self]` carry.

## Call sites

```python
# unary constructors
bsr: bsr_array[np.float64]
reveal_type(bsr.astype(np.int32))  # bsr_array[np.int32]
reveal_type(bsr.astype("d"))  # bsr_array[Any]

# binary constructors
type _2D = tuple[int, int]
csr_2d: csr_array[np.float64, _2D]
reveal_type(csr_2d.astype(np.int32))  # csr_array[np.int32, _2D]
reveal_type(csr_2d.astype("d"))  # csr_array[Any, _2D]
```

[src]: https://github.com/scipy/scipy-stubs/blob/4d51a695/scipy-stubs/sparse/_base.pyi#L806-L866
