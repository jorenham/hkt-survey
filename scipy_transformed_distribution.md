# `scipy.stats` distribution transformations

Source: [scipy-stubs/stats/_distribution_infrastructure.pyi][src]

Kind: `D` is $\ast \to \ast$ (a distribution over its shape).

## Definition

```python
from typing import Origin

class Distribution[ShapeT]: ...
class Normal[ShapeT](Distribution[ShapeT]): ...

# D (the wrapped distribution's constructor) is carried as a class
# parameter, so the wrapper remembers *which* distribution it transforms.
class MonotonicTransformedDistribution[D: Origin[Distribution], ShapeT](Distribution[ShapeT]): ...

# real scipy.stats transforms (exp, log, ...): each preserves D and shape
def exp[D: Origin[Distribution], S](X: D[S], /) -> MonotonicTransformedDistribution[D, S]: ...
def log[D: Origin[Distribution], S](X: D[S], /) -> MonotonicTransformedDistribution[D, S]: ...
```

## Call sites

```python
n: Normal[tuple[int]]

# infer D = Normal, S = tuple[int] from the single argument X
reveal_type(exp(n))  # MonotonicTransformedDistribution[Normal, tuple[int]]

# nesting: D = MonotonicTransformedDistribution[Normal, tuple[int]]
en = exp(n)
reveal_type(log(en))
# MonotonicTransformedDistribution[MonotonicTransformedDistribution[Normal, tuple[int]], tuple[int]]

exp(42)  # E: int is not D[S] for any Distribution D
```

[src]: https://github.com/scipy/scipy-stubs/blob/4d51a695/scipy-stubs/stats/_distribution_infrastructure.pyi#L1564-L1585
