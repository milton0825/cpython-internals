## Python data types

Sequence data types
* String: a sequence of characters. It is immutable.
* Tuple: a sequence of objects. It is immutable.
* List: a sequence of objects. It is mutable.

PyStringObject is the c implementation of string in Python.
```c
typedef struct {
    PyObject_VAR_HEAD
    long ob_shash;
    int ob_sstate;
    char ob_sval[1];

    /* Invariants:
     *     ob_sval contains space for 'ob_size+1' elements.
     *     ob_sval[ob_size] == 0.
     *     ob_shash is the hash of the string or -1 if not computed yet.
     *     ob_sstate != 0 iff the string object is in stringobject.c's
     *       'interned' dictionary; in this case the two references
     *       from 'interned' to this object are *not counted* in ob_refcnt.
     */
} PyStringObject;

```

Let's take a look at how string comparison works. The interpreter will receive a COMPARE_OP instruction bytecode to compare two objects.
```c
PyObject * PyEval_EvalFrameEx(PyFrameObject *f, int throwflag)
    case COMPARE_OP:
        w = POP();
        v = TOP();
        x = cmp_outcome(oparg, v, w);
        PyObject_RichCompare(v, w, op);
            richcmpfunc frich = RICHCOMPARE(v->ob_type);
```

In rich compare function, it will find the function pointer to the comparison methods based on the type of the object and execute the
comparison method.
```c
static PyObject * cmp_outcome(int op, register PyObject *v, register PyObject *w)
    PyObject_RichCompare(v, w, op);

PyObject * PyObject_RichCompare(PyObject *v, PyObject *w, int op
    PyObject *res;
    richcmpfunc frich = RICHCOMPARE(v->ob_type);
    res = (*frich)(v, w, op);
    
#define RICHCOMPARE(t) (PyType_HasFeature((t), Py_TPFLAGS_HAVE_RICHCOMPARE) ? (t)->tp_richcompare : NULL)
```

If we are comparing two string objects, it will find the `string_richcompare` method to do the comparison.
```c
PyTypeObject PyString_Type = {
    ...
    (richcmpfunc)string_richcompare,            /* tp_richcompare */
    ...
};

static PyObject* string_richcompare(PyStringObject *a, PyStringObject *b, int op)

```


## References
* https://github.com/python/cpython/blob/master/Objects/abstract.c
