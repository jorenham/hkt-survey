# Higher-Kinded Types in Python: a survey of real-world use cases

Use cases for higher-kinded typing (HKT) in Python, distilled from real code,
one per file, as requested in [python/typing#548 (comment)][ivan-request].
Each file includes the expected type-checker behavior **at call sites**, since
inference of type constructors is the hard part.

## Notation

Examples use the notation from [python/typing#548 (comment)][proposal]: a
PEP 695 type parameter bound to a generic class/protocol is a higher-kinded
variable -- a type constructor that is applied by subscription.

A bare class can't be that bound: under PEP 696, `T: dict` already *means*
`T: dict[Any, Any]` (a fully applied type), so it can't denote the
constructor. We write the bound as `Origin[dict]` instead -- the unapplied
constructor, matching [`typing.get_origin`][get-origin]
(`get_origin(dict[int, str]) is dict`); cf. [returns' `Kind`][returns-hkt].
`Origin[X]` "unsubscripts" `X` to its constructor; the variable is then
applied with `T[...]`. A variable with no family bound (e.g.
`def f[F](x: F[int]) -> F[str]`) is still higher-kinded, recognized by its
subscripted use.

```python
def map_async[V, R, T: Origin[Awaitable]](fn: Callable[[V], R], aval: T[V]) -> T[R]: ...
```

Ordinary types have [kind][kind] $\ast$; `T` here is a type operator of kind
$\ast \to \ast$, and binary constructors have kind
$\ast \to \ast \to \ast$, etc. At a call site the checker solves a
higher-order unification of the parameter against the argument type,

```math
\begin{aligned}
T[V] \;&\sim\; \texttt{asyncio.Task}[\texttt{int}] \\
       &\Longrightarrow\quad T = \texttt{asyncio.Task} \\
       &\phantom{\Longrightarrow}\quad V = \texttt{int}
\end{aligned}
```

so `map_async(str, t)` with `t: asyncio.Task[int]` has type
`asyncio.Task[R]` where $R$ is fixed by `fn`. No new surface syntax is
implied; the point is the semantics.

## Index

| Example                                                                | Library                       | Refs                                          |
| ---------------------------------------------------------------------- | ----------------------------- | --------------------------------------------- |
| [functor_map_async.md](functor_map_async.md)                           | stdlib (`asyncio`)            | [typing#548][proposal]                        |
| [bidict_inverse.md](bidict_inverse.md)                                 | bidict                        | [typing#548][bidict-issue], [src][bidict-src] |
| [numpy_ndarray_subclass.md](numpy_ndarray_subclass.md)                 | NumPy                         | numpy/numpy#29807, numpy/numpy#20072          |
| [scipy_sparse_getitem.md](scipy_sparse_getitem.md)                     | scipy-stubs                   | [src][scipy-dok]                              |
| [scipy_transformed_distribution.md](scipy_transformed_distribution.md) | scipy-stubs                   | [src][scipy-dist]                             |
| [gevent_group_greenlet.md](gevent_group_greenlet.md)                   | gevent (typeshed)             | [src][gevent-src]                             |
| [tensorflow_gradient_tape.md](tensorflow_gradient_tape.md)             | tensorflow (typeshed)         | [src][tf-src]                                 |
| [dict_fromkeys.md](dict_fromkeys.md)                                   | stdlib (`dict`/`OrderedDict`) | python/typeshed#6485, python/typeshed#3800    |
| [returns_monad_bind.md](returns_monad_bind.md)                         | returns                       | [docs][returns-hkt]                           |

[kind]: https://en.wikipedia.org/wiki/Kind_(type_theory)
[get-origin]: https://docs.python.org/3/library/typing.html#typing.get_origin
[returns-hkt]: https://returns.readthedocs.io/en/latest/pages/hkt.html
[ivan-request]: https://github.com/python/typing/issues/548#issuecomment-4467756659
[proposal]: https://github.com/python/typing/issues/548#issuecomment-1898377873
[bidict-issue]: https://github.com/python/typing/issues/548#issuecomment-621195693
[bidict-src]: https://github.com/jab/bidict/blob/84bceb12/bidict/_abc.py#L42-L43
[scipy-dok]: https://github.com/scipy/scipy-stubs/blob/4d51a695/scipy-stubs/sparse/_dok.pyi#L533-L537
[scipy-dist]: https://github.com/scipy/scipy-stubs/blob/4d51a695/scipy-stubs/stats/_distribution_infrastructure.pyi#L1330-L1340
[gevent-src]: https://github.com/python/typeshed/blob/ee212088/stubs/gevent/gevent/pool.pyi#L166-L199
[tf-src]: https://github.com/python/typeshed/blob/ee212088/stubs/tensorflow/tensorflow/autodiff.pyi#L26-L58
