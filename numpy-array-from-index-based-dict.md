## The problem definition
Suppose, you have a hash index stored as python dict:

    In [10]: hash_index = {abs(i-4): i * 10 for i in range(5)}
    In [11]: hash_index
    Out[11]: {0: 40, 1: 30, 2: 20, 3: 10, 4: 0}

and now you want to build a numpy array from the index. The keys are the positions in the array, the values are the values to put in the corresponding positions. 

At least for me, it wasn't clear what is the best way to do this.

## Solution 1
The most obvious approach is to create a list and pass in to the constructor of array:

    In [12]: np.array([d1[x] for x in range(len(d1))])
    Out[12]: array([40, 30, 20, 10,  0])

The main disadvantage of the approach is that you need to create an intermediate throw-away list just to pass the contents of the dictionary to the constructor. In case of really long dictionaries that may cause memory consumption spikes beyond the limits of what you have (for example, if your code sits within a container).

## Solution 2

There is a [numpy.fromfunction](https://docs.scipy.org/doc/numpy-1.13.0/reference/generated/numpy.fromfunction.html#numpy.fromfunction) function that looks like a good candidate:

> Construct an array by executing a function over each coordinate.

But when you run the code, it looks like the function is executed once with indexes as a parameter, instead of being called once for each coordinate:

    In [13]: np.fromfunction(lambda x: d1[x], (len(d1),))
    ---------------------------------------------------------------------------
    TypeError                                 Traceback (most recent call last)
    <ipython-input-62-9cd3a9d9eec3> in <module>()
    ----> 1 np.fromfunction(lambda x: d1[x], (len(d1),))
    
    ~/miniconda3/lib/python3.6/site-packages/numpy/core/numeric.py in fromfunction(function, shape, **kwargs)
       1912     dtype = kwargs.pop('dtype', float)
       1913     args = indices(shape, dtype=dtype)
    -> 1914     return function(*args, **kwargs)
       1915
       1916
    
    <ipython-input-62-9cd3a9d9eec3> in <lambda>(x)
    ----> 1 np.fromfunction(lambda x: d1[x], (len(d1),))
    
    TypeError: unhashable type: 'numpy.ndarray'

The solution is to [vectorize](https://docs.scipy.org/doc/numpy-1.13.0/reference/generated/numpy.vectorize.html) the function, but on my opinion, this now involves too much magic for the trivial intention:

    In [14]: np.fromfunction(np.vectorize(lambda x: d1[x]), (len(d1),))
    Out[14]: array([40, 30, 20, 10,  0])

## Solution 3

And there is one more approach. The numpy.array supports [sequence protocol](https://docs.python.org/3/library/functions.html?highlight=__getitem__#iter) for iteration. To use this we need to write a small wrapper, but the resulting solution now looks very clean, on my opinion:

    class DictIterator:
        def __init__(self, dict_):
            self._dict = dict_
    
        def __len__(self):
            return len(self._dict)

        def __getitem__(self, index):
            try:
                return self._dict[index]
            except KeyError:
                raise IndexError(repr(index))

and the usage:

    In [15]: np.array(DictIterator(d1))
    Out[15]: array([40, 30, 20, 10,  0])

Moreover, the same wrapper can be used for the lists and numpy arrays as well:

    In [16]: np.array(DictIterator(_))
    Out[16]: array([40, 30, 20, 10,  0])
