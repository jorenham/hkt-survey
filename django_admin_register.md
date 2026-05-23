# `django.contrib.admin.register`

Sources:

- jorenham/hkt-survey#9
- typeddjango/django-stubs#3387
- [`django-stubs/contrib/admin/decorators.pyi`][src]

Kind: `K` is $\ast \to \ast$ (`ModelAdmin[M]`).

## Motivation

Today the decorator binds the admin via a `TypeVarbound` with `bound=ModelAdmin[Any])`:
the admin subclass is preserved, but the model parameter is erased, so this is silently
accepted:

```python
@admin.register(SomeOtherModel)
class QuoteAdmin(ModelAdmin[Quote]):
    pass
```

## Definition

```python
from typing import Origin, Protocol, type_check_only

from django.contrib.admin import AdminSite, ModelAdmin
from django.db.models import Model

@type_check_only
class _RegisterDecorator[M: Model](Protocol):
    def __call__[K: Origin[ModelAdmin]](
        self,
        admin: type[K[M]],
        /,
    ) -> type[K[M]]: ...

def register[M: Model](
    *models: type[M],
    site: AdminSite | None = None,
) -> _RegisterDecorator[M]: ...
```

Note that `M` is scoped to `register`, while `K` (the kind) is scoped to the returned
decorator.

## Call sites

```python
class Quote(Model): ...
class Invoice(Model): ...

# M@register = Quote; M@_RegisterDecorator = Quote => accepted
@register(Quote)
class QuoteAdmin(ModelAdmin[Quote]):
    ...

# M@register = Invoice; M@_RegisterDecorator = Quote => rejected
@register(Invoice)
class QuinvoiceAdmin(ModelAdmin[Quote]):  # E
    ...
```

[src]: https://github.com/typeddjango/django-stubs/blob/75ec8092/django-stubs/contrib/admin/decorators.pyi#L63-L65
