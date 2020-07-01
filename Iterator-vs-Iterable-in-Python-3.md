Some thoughts on the difference between `Iterable` and `Iterator`.

Basically, this should be enough to understand the difference:

```
# newtover/iters.py
from typing import Iterable, Iterator


def it1(items: Iterable[str]) -> None:
    print(items.__iter__)
    print(items.__next__)


def it2(items: Iterator[str]) -> None:
    print(items.__iter__)
    print(items.__next__)


if __name__ == '__main__':
    print(Iterable, Iterable.mro())
    print(Iterator, Iterator.mro())

```

```
$ python newtover/iters.py
typing.Iterable [typing.Iterable, <class 'collections.abc.Iterable'>, typing.Generic, <class 'object'>]
typing.Iterator [typing.Iterator, <class 'collections.abc.Iterator'>, typing.Iterable, <class 'collections.abc.Iterable'>, typing.Generic, <class 'object'>]
```

```
$ mypy --strict newtover/iters.py
newtover/iters.py:6: error: "Iterable[str]" has no attribute "__next__"
Found 1 error in 1 file (checked 1 source file)
```

That is `Iterable` is something that has `__iter__()` method, which produces an iterator, where `Iterator` is something that has `__next__()` method and is `Iterable` as well (which usually returns `self`).

---
Both `Iterable` and `Iterator` are usually good for type arguments within the function body:

```
from typing import Iterable, Iterator
from itertools import islice


def it3(items1: Iterator[str], items2: Iterable[str]) -> None:
    for item in items1:
        pass

    for item in items2:
        pass

    a = islice(items1, 10)
    b = islice(items2, 10)

```

```
$ mypy --strict newtover/iters.py
Success: no issues found in 1 source file
```
---
But things become more interesting with returned values:

```
from typing import Iterable, Iterator
from itertools import islice


def it4(size: int) -> Iterable[int]:
    if size <= 0:
        return []
    return range(5)


def it5(size: int) -> Iterator[int]:
    if size <= 0:
        return []
    return range(5)

```

```
$ mypy --strict newtover/iters.py
newtover/iters.py:13: error: Incompatible return value type (got "List[<nothing>]", expected "Iterator[int]")
newtover/iters.py:13: note: 'list' is missing following 'Iterator' protocol member:
newtover/iters.py:13: note:     __next__
newtover/iters.py:14: error: Incompatible return value type (got "range", expected "Iterator[int]")
newtover/iters.py:14: note: 'range' is missing following 'Iterator' protocol member:
newtover/iters.py:14: note:     __next__
Found 2 errors in 1 file (checked 1 source file)
```

Which means that a `list`, a `set` or `dict.keys()`, `range` have not a `__next__` method and are not `Iterator` instances.

---

And if you annotate return values as `Iterable`:

```
from typing import Iterable, Iterator


def it4(size: int) -> Iterable[int]:
    if size <= 0:
        return []
    return range(5)


def it6(items: Iterator[int]) -> None:
    for item in items:
        pass


def do1() -> None:
    res = it4(10)
    it6(res)  # it6(iter(res)) will fix the mypy error

```
`mypy` will annoy you:

```
$ mypy --strict newtover/iters.py
newtover/iters.py:17: error: Argument 1 to "it6" has incompatible type "Iterable[int]"; expected "Iterator[int]"
newtover/iters.py:17: note: 'Iterable' is missing following 'Iterator' protocol member:
newtover/iters.py:17: note:     __next__
Found 1 error in 1 file (checked 1 source file)
```
---
In other words, `Iterable` is going to be more common in your type annotations than `Iterator`.

```
from typing import Iterable


def it4(size: int) -> Iterable[int]:
    if size <= 0:
        return []
    return range(5)


# items is now Iterable
def it6(items: Iterable[int]) -> None:
    for item in items:
        pass


def do1() -> None:
    res = it4(10)
    it6(res)
```

```
$ mypy --strict newtover/iters.py
Success: no issues found in 1 source file
```

---
Certainly, there is nothing new here, but this is a confusing case.