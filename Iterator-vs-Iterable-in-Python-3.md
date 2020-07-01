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

In other words, `Iterator` is going to be more common on type annotations. 