## PyObject: the core Python object
In Python, everything is an object. Each object hs an identity, a type and a value. An object's identity and type never changes once it has been created. Objects whose value can change are said to be mutable; objects whose value is unchangeable once they are created are called immutable. An object has a reference count that is increased or decreased when a pointer to the object is copied or deleted. When the reference count reaches zero, the object can be garbage-collected.

`dir` shows the built-in fields and functions of the object. For example, you can use \_\_add\_\_, \_\_div\_\_ on object 123.
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
#define PyObject_HEAD                   \
    _PyObject_HEAD_EXTRA                \
    Py_ssize_t ob_refcnt;               \
    struct _typeobject *ob_type;

typedef struct _object {
    PyObject_HEAD
} PyObject;
```

PyVarObject is a subclass of PyObject. Although C does not support inheritance out of the box, we still can use C structural subtype.
The first two elements of PyVarObject is reference count and object type. If we cast a PyVarObject pyVarObject to PyObject through 
`(PyObject *) pyVarObject`, it will only look at the first two fields as if it is a PyObject.
```c
/* PyObject_VAR_HEAD defines the initial segment of all variable-size
 * container objects.  These end with a declaration of an array with 1
 * element, but enough space is malloc'ed so that the array actually
 * has room for ob_size elements.  Note that ob_size is an element count,
 * not necessarily a byte count.
 */
#define PyObject_VAR_HEAD               \
    PyObject_HEAD                       \
    Py_ssize_t ob_size; /* Number of items in variable part */

typedef struct {
    PyObject_VAR_HEAD
} PyVarObject;
```

PyIntObject is a subclass of PyObject.
```c
typedef struct {
    PyObject_HEAD
    long ob_ival;
} PyIntObject;
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
* https://docs.python.org/2/reference/datamodel.html
* https://docs.python.org/2/c-api/object.html
* https://github.com/python/cpython/blob/master/Include/object.h
* https://github.com/python/cpython/blob/master/Objects/object.c
