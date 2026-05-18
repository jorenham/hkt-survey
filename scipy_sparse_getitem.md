# `scipy.sparse.dok_matrix.__getitem__`

Source: [scipy-stubs/sparse/_dok.pyi][src]

Kind: `T` is $\ast \to \ast$ (`_spbase[ScalarT]`).

## Definition

```python
from typing import Origin, overload

import numpy as np

# T is recovered from `self`; the scalar overload does not need it.
# Overload resolution must interact with type-constructor inference.
class _spbase[ScalarT]:
    @overload
    def __getitem__[T: Origin[_spbase]](self: T[ScalarT], key: int, /) -> T[ScalarT]: ...
    @overload
    def __getitem__(self, key: tuple[int, int], /) -> ScalarT: ...


class dok_matrix[ScalarT](_spbase[ScalarT]): ...
```

## Call sites

```python
m: dok_matrix[np.float64]

# overload 1: infer T = dok_matrix from `self`
reveal_type(m[2])  # dok_matrix[np.float64]
# overload 2: T unused, falls through to the scalar element
reveal_type(m[2, 3])  # np.float64
```

[src]: https://github.com/scipy/scipy-stubs/blob/4d51a695/scipy-stubs/sparse/_dok.pyi#L521-L534
