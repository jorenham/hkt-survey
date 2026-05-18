# `builtins.dict.fromkeys`

Sources:

- python/typeshed#6485 ("no other solution possible until we have higher-kinded TypeVars")
- python/typeshed#3800
- [current typeshed signature][stdlib] (overloads + a TODO: "not expressible in the current type system")

Generic-self fails: the value type is a free parameter of the bound, so
`TypeVar(bound='dict[_KT, _S]')` leaves `_S` unbound. The constructor must be
recovered from `cls` and reapplied with *both* params chosen at the call.

Recover it from `cls` typed as `type[T]` -- the *unapplied* class object --
not `type[T[Any, Any]]`: the key slot is not covariant, so pinning it to `Any`
is unsound, and it would erase the `K` we must infer. A subclass may also
re-parametrize the base (`Counter` below fixes the value type), so the binary
`T[K, V]` must be reconciled with the subclass's own arity.

Kind: `T` is $\ast \to \ast \to \ast$ (`dict[K, V]`), recovered from `cls`.

## Definition

```python
from typing import Iterable, Origin

# T (the unapplied constructor) is recovered from the class object `cls`.
class dict[KT, VT]:
    @classmethod
    def fromkeys[T: Origin[dict], K, V](
        cls: type[T],
        iterable: Iterable[K],
        value: V,
        /
    ) -> T[K, V]: ...

class OrderedDict[KT, VT](dict[KT, VT]):
    def move_to_end(self, key: KT, /) -> None: ...
```

## Call sites

```python
# infer T = OrderedDict from cls, K = str, V = int from the arguments
reveal_type(OrderedDict.fromkeys(["a", "b"], 0))  # OrderedDict[str, int]
OrderedDict.fromkeys(["a"], 0).move_to_end("a")  # ok (today: dict, E)

class Counter[KT](dict[KT, int]): ...

# Counter is *unary* (value fixed to int): the binary return T[K, V] must be
# reconciled with Counter's own arity, not applied blindly
reveal_type(Counter.fromkeys("ab", 1))  # Counter[str]
Counter.fromkeys("ab", "x")  # E: value str conflicts with Counter's fixed int

# instance call resolves the constructor the same way
od: OrderedDict[bytes, float]
reveal_type(od.fromkeys([b"x"], 1.0))  # OrderedDict[bytes, float]
```

[stdlib]: https://github.com/python/typeshed/blob/ee212088/stdlib/builtins.pyi#L1322-L1329
