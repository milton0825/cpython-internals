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

```c
PyObject * PyEval_EvalFrameEx(PyFrameObject *f, int throwflag)
    case GET_ITER:
        v = TOP();
        x = PyObject_GetIter(v);
            PyObject * PyObject_GetIter(PyObject *o)
            {
                PyTypeObject *t = o->ob_type;
                getiterfunc f = NULL;
                if (PyType_HasFeature(t, Py_TPFLAGS_HAVE_ITER)) ## listobject does not have Py_TPFLAGS_HAVE_ITER=true
                    f = t->tp_iter;
                if (f == NULL) {
                    if (PySequence_Check(o))
                        return PySeqIter_New(o);
                            PyObject * PySeqIter_New(PyObject *seq)
                            {
                                seqiterobject *it;

                                if (!PySequence_Check(seq)) {
                                    PyErr_BadInternalCall();
                                    return NULL;
                                }
                                it = PyObject_GC_New(seqiterobject, &PySeqIter_Type);
                                if (it == NULL)
                                    return NULL;
                                it->it_index = 0;
                                Py_INCREF(seq);
                                it->it_seq = seq;
                                _PyObject_GC_TRACK(it);
                                return (PyObject *)it;
                            }
                        
                        
                        
                    return type_error("'%.200s' object is not iterable", o);
                }
                else {
                    PyObject *res = (*f)(o);
                    if (res != NULL && !PyIter_Check(res)) {
                        PyErr_Format(PyExc_TypeError,
                                     "iter() returned non-iterator "
                                     "of type '%.100s'",
                                     res->ob_type->tp_name);
                        Py_DECREF(res);
                        res = NULL;
                    }
                    return res;
                }
            }
        
        
        Py_DECREF(v);
        if (x != NULL) {
            SET_TOP(x);
            PREDICT(FOR_ITER); ## GET_ITER often follows by FOR_ITER
            continue;
        }
        break;
    case FOR_ITER:
        /* before: [iter]; after: [iter, iter()] *or* [] */
        v = TOP();
        x = (*v->ob_type->tp_iternext)(v); /* x = v.next() */
        if (x != NULL) {
            PUSH(x);
            PREDICT(STORE_FAST);
            PREDICT(UNPACK_SEQUENCE);
            continue;
        }
        if (PyErr_Occurred()) {
            if (!PyErr_ExceptionMatches(
                            PyExc_StopIteration))
                break;
            PyErr_Clear();
        }
        /* iterator ended normally */
        x = v = POP();
        Py_DECREF(v);
        JUMPBY(oparg);
        continue;

```

```c
typedef struct {
    PyObject_HEAD
    long      it_index;
    PyObject *it_seq; /* Set to NULL when iterator is exhausted */
} seqiterobject;
```

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
