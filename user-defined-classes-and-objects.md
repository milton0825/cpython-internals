## User-defined classes and objects

In C++ or Java, fields in a class are pre-defined; in Python, fields can be added to a class at runtime. For example, we can add a field `low` in `Counter` class at runtime.
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

c = Counter(5, 7)
c.low = -5
```

```bash
$ python -m dis test.py
  1           0 LOAD_CONST               0 ('Counter')
              3 LOAD_CONST               5 (())           ## Load base class
              6 LOAD_CONST               1 (<code object Counter at 0x110466530, file "test.py", line 1>)
              9 MAKE_FUNCTION            0                ## makes the methods dictionary
             12 CALL_FUNCTION            0
             15 BUILD_CLASS
             16 STORE_NAME               0 (Counter)

 16          19 LOAD_NAME                0 (Counter)
             22 LOAD_CONST               2 (5)
             25 LOAD_CONST               3 (7)
             28 CALL_FUNCTION            2
             31 STORE_NAME               1 (c)
             34 LOAD_CONST               4 (None)
             37 RETURN_VALUE
```

```c
PyObject * PyEval_EvalFrameEx(PyFrameObject *f, int throwflag) 
{
    ...
    case BUILD_CLASS:
        u = TOP();        /* Methods dictionary */
        v = SECOND();     /* Tuple of base classes */
        w = THIRD();      /* Class name */
        STACKADJ(-2);
        x = build_class(u, v, w);
            |
            static PyObject * build_class(PyObject *methods, PyObject *bases, PyObject *name)
            {
                ...
                result = PyObject_CallFunctionObjArgs(metaclass, name, bases, methods, NULL);
                ...
                return result;
            }
            |
        SET_TOP(x);
        Py_DECREF(u);
        Py_DECREF(v);
        Py_DECREF(w);
        break;
    ...
}
```
