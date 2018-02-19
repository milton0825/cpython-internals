## Iterators
An iterator allows you to iterate through all the elements in a collection in a specific order. From the example below, we created a list with elements 1, 2, and 3. We created a iterator object `i` and used `i.next()` to iterate through all the objects in `x`. When we reach the end of the list, a StopIteration exception is raised.

```py
>>> x = [1, 2, 3]
>>> i = x.__iter__()
>>> i
<listiterator object at 0x10d912950>
>>> i.next()
1
>>> i.next()
2
>>> i.next()
3
>>> i.next()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

Let's create a Python script test.py with code:
```py
x = ['a', 'b', 'c']
for e in x:
    print e
```

Let's disassemble test.py and look at the instruction code. `GET_ITER` creates an iterator object over collection x. `FOR_ITER` creates a loop to iterate through all the elements. When ever we finished an iteration, `JUMP_ABSOLUTE` will jump back to `FOR_ITER` to proceed on the next iteration. After we iterate through all the elements in x, we will jump to `POP_BLOCK` and exit.
```bash
$ python -m dis test.py
  1           0 LOAD_CONST               0 ('a')
              3 LOAD_CONST               1 ('b')
              6 LOAD_CONST               2 ('c')
              9 BUILD_LIST               3
             12 STORE_NAME               0 (x)

  2          15 SETUP_LOOP              19 (to 37)
             18 LOAD_NAME                0 (x)
             21 GET_ITER                             
        >>   22 FOR_ITER                11 (to 36)
             25 STORE_NAME               1 (e)

  3          28 LOAD_NAME                1 (e)
             31 PRINT_ITEM
             32 PRINT_NEWLINE
             33 JUMP_ABSOLUTE           22
        >>   36 POP_BLOCK
        >>   37 LOAD_CONST               3 (None)
             40 RETURN_VALUE
```

This is the cpython internal of how `GET_ITER` and `FOR_ITER` works. `GET_ITER` instruction gets the interator of object v. Since `GET_ITER` usually followed by `FOR_ITER`, it jumps to `FOR_ITER` to skip the main interpreter loop for optimization. In `FOR_ITER`, it grabs the `tp_iternext` function pointer from the type object of v and calls it.
```c
PyObject * PyEval_EvalFrameEx(PyFrameObject *f, int throwflag)
    case GET_ITER:
        v = TOP();
        x = PyObject_GetIter(v);
        ...
        PREDICT(FOR_ITER); ## GET_ITER often follows by FOR_ITER
        break;
    case FOR_ITER:
        v = TOP();
        x = (*v->ob_type->tp_iternext)(v); /* x = v.next() */
        ...
        if (PyErr_Occurred()) {
            if (!PyErr_ExceptionMatches(
                            PyExc_StopIteration))
                break;
            PyErr_Clear();
        }
        ...
```

This is the cpython internal of seqiterobject. `it_index` records the current position and `it_seq` points at the object we are iterating through.
```c
typedef struct {
    PyObject_HEAD
    long      it_index;
    PyObject *it_seq; /* Set to NULL when iterator is exhausted */
} seqiterobject;
```

We can make a custom object iterable as long as the class implements `__iter__` and `next`.
```py
class Counter:
    def __init__(self, low, high):
        self.current = low
        self.high = high

    def __iter__(self):
        return self

    def next(self):
        if self.current > self.high:
            raise StopIteration
        else:
            self.current += 1
            return self.current - 1

for c in Counter(5, 10):
    print c
```
