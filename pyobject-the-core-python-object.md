## PyObject: the core Python object
In Python, everything is an object. `dir` shows the built-in fields and functions of the object. For example, you can use \_\_add\_\_, \_\_div\_\_ on object 123.
```py
>>> dir(123)
['__abs__', '__add__', '__and__', '__class__', '__cmp__', '__coerce__', '__delattr__', '__div__', '__divmod__', '__doc__', '__float__', '__floordiv__', '__format__', '__getattribute__', '__getnewargs__', '__hash__', '__hex__', '__index__', '__init__', '__int__', '__invert__', '__long__', '__lshift__', '__mod__', '__mul__', '__neg__', '__new__', '__nonzero__', '__oct__', '__or__', '__pos__', '__pow__', '__radd__', '__rand__', '__rdiv__', '__rdivmod__', '__reduce__', '__reduce_ex__', '__repr__', '__rfloordiv__', '__rlshift__', '__rmod__', '__rmul__', '__ror__', '__rpow__', '__rrshift__', '__rshift__', '__rsub__', '__rtruediv__', '__rxor__', '__setattr__', '__sizeof__', '__str__', '__sub__', '__subclasshook__', '__truediv__', '__trunc__', '__xor__', 'bit_length', 'conjugate', 'denominator', 'imag', 'numerator', 'real']
```

`id` method shows the unique id that represents the object. In cpython interpreter, it is just the address in memory. Once an object is 
alocated at an address, it never moves.
```py
>>> x = 1
>>> id(x)
140369635362760
```

Every object in Python is a subclass of PyObject. It has two fields: reference count and type.
```c
typedef struct _object {
    Py_ssize_t ob_refcnt;
    struct _typeobject *ob_type;
} PyObject;
```

PyVarObject is a subclass of PyObject. Although C does not support inheritance out of the box, we still can use C structural subtype.
The first two elements of PyVarObject is reference count and object type. If we cast a PyVarObject pyVarObject to PyObject through 
`(PyObject *) pyVarObject`, it will only look at the first two fields as if it is a PyObject.
```c
typedef struct {
    PyObject ob_base;
    Py_ssize_t ob_size; /* Number of items in variable part */
} PyVarObject;
```

PyLongObject is a subclass of PyVarObject. It is also a subclass of PyObject.
```c
struct _longobject {
    PyVarObject ob_base;
    digit ob_digit[1];
} PyLongObject;
```

When instantiating a new object, the Python interpreter allocates memory based on the object type.
```c
PyObject *
_PyObject_New(PyTypeObject *tp)
{
    PyObject *op;
    op = (PyObject *) PyObject_MALLOC(_PyObject_SIZE(tp));
    if (op == NULL)
        return PyErr_NoMemory();
    return PyObject_INIT(op, tp);
}
```

`str` is a convenient function that returns a string representation of an object.
```py
>>> str([1,2,3])
'[1, 2, 3]'
```

Let's take a closer look at the internal of `str' method. 
```c
PyObject *
PyObject_Str(PyObject *v)
{
    PyObject *res;
    ...
    res = (*Py_TYPE(v)->tp_str)(v); ## Converts pointer to type and calls tp_str method
                                    ## Different objects such as list and dict has its own implementation of tp_str to convert
                                    ## the object to string.
    ...
    return res;
}
```

## References:
https://docs.python.org/2/reference/datamodel.html
https://docs.python.org/2/c-api/object.html
https://github.com/python/cpython/blob/master/Include/object.h
https://github.com/python/cpython/blob/master/Objects/object.c
