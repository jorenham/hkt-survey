# `bidict.BidirectionalMapping`

Sources:

- [python/typing#548 (comment)][discussion]
- [bidict/_abc.py][code]

Kind: `T` is $\ast \to \ast \to \ast$, recovered from `self` and reapplied
with the parameters swapped.

## Definition

```python
from collections.abc import Mapping
from typing import Origin

class BidirectionalMapping[KT, VT](Mapping[KT, VT]):
    @property
    def inverse[T: Origin[BidirectionalMapping]](self: T[KT, VT]) -> T[VT, KT]: ...

class bidict[KT, VT](BidirectionalMapping[KT, VT]): ...
```

## Call sites

```python
b: bidict[int, str]

# infer T = bidict from `self`; result is T[VT, KT]
reveal_type(b.inverse)  # bidict[str, int]

# constructor resolves again on the HKT result, not widened to the base
reveal_type(b.inverse.inverse)  # bidict[int, str]
```

[discussion]: https://github.com/python/typing/issues/548#issuecomment-621195693
[code]: https://github.com/jab/bidict/blob/84bceb12/bidict/_abc.py#L42-L43
