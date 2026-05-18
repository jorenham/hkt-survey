# Structure-preserving gradient: the container shape of `sources` is kept

Source: [typeshed: stubs/tensorflow/tensorflow/autodiff.pyi][src]

The stub's own comment: *"Higher kinded types would be nice here and these
overloads are a way to simulate some of them."* Four overloads enumerate the
container shapes; one HKT signature replaces them.

Kind: `F` is $\ast \to \ast$; `dict[str, _]` is a partial application.

## Definition

```python
from collections.abc import Sequence

class Tensor: ...
class Gradients: ...

# F is an *unbounded* unary type constructor inferred from `sources`
# (no family, so no Origin bound); `dict[str, _]` is a partial
# application of dict to its value parameter.
class GradientTape:
    def gradient[F](self, target: Tensor, sources: F[Tensor]) -> F[Gradients]: ...
```

## Call sites

```python
tape: GradientTape
t: Tensor

# F = identity-ish (the bare element); F[Gradients] = Gradients
reveal_type(tape.gradient(t, t))  # Gradients

# F = list
xs: list[Tensor]
reveal_type(tape.gradient(t, xs))  # list[Gradients]

# F = dict[str, _]  (partial application; key type stays str)
d: dict[str, Tensor]
reveal_type(tape.gradient(t, d))  # dict[str, Gradients]

# F = tuple[_, _]  (homogeneous nested container)
nested: Sequence[list[Tensor]]
reveal_type(tape.gradient(t, nested))  # Sequence[list[Gradients]]

# sources whose element is not a Tensor cannot bind F[Tensor]
tape.gradient(t, [1, 2, 3])  # E: list[int] is not F[Tensor]
```

[src]: https://github.com/python/typeshed/blob/ee212088/stubs/tensorflow/tensorflow/autodiff.pyi#L26-L58
