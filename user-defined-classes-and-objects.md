## User-defined classes and objects

In C++ or Java, fields and methods in a class are pre-defined; in Python, fields and methods can be added to a class at runtime. For example, we can add a field `low` in `Counter` class at runtime. We can also add a new method `jump` at runtime.
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

Counter.__dict__['jump'] = Counter.__dict__['next']
c = Counter(5, 7)
c.low = -5
c.jump()
```

To build a class, the interpreter will load the class name, load all base classes, and make a function call to get the methods dictionary.
```bash
$ python -m dis test.py
  1           0 LOAD_CONST               0 ('Counter')
              3 LOAD_CONST               5 (())           
              6 LOAD_CONST               1 (<code object Counter at 0x110466530, file "test.py", line 1>)
              9 MAKE_FUNCTION            0                
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

With the methods dictionary, tuple of base classes and class name on value stack, the interpreter build a class object.
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
        SET_TOP(x);
        Py_DECREF(u);
        Py_DECREF(v);
        Py_DECREF(w);
        break;
    ...
}
```

This is the cpython internal representation of a class. It stores its base classes, the dictionary of all its class methods and its class name.
```c
typedef struct {
    PyObject_HEAD
    PyObject	*cl_bases;	/* A tuple of class objects */
    PyObject	*cl_dict;	/* A dictionary */
    PyObject	*cl_name;	/* A string */
    /* The following three are functions or NULL */
    PyObject	*cl_getattr;
    PyObject	*cl_setattr;
    PyObject	*cl_delattr;
    PyObject    *cl_weakreflist; /* List of weak references */
} PyClassObject;
```

This is the cpython internal representation of an object. Instance object has a pointer to the class object, a dictionary of all its fileds.
```c
typedef struct {
    PyObject_HEAD
    PyClassObject *in_class;	/* The class object */
    PyObject	  *in_dict;	/* A dictionary */
    PyObject	  *in_weakreflist; /* List of weak references */
} PyInstanceObject;
```

```c
typedef struct {
    PyObject_HEAD
    PyObject *im_func;   /* The callable object implementing the method */
    PyObject *im_self;   /* The instance it is bound to, or NULL */
    PyObject *im_class;  /* The class that asked for the method */
    PyObject *im_weakreflist; /* List of weak references */
} PyMethodObject;
```

```c
PyObject * PyEval_EvalFrameEx(PyFrameObject *f, int throwflag) 
{
    ...
    case CALL_FUNCTION:
    {
        x = call_function(&sp, oparg, &intr0, &intr1);
        |
            static PyObject *call_function(PyObject ***pp_stack, int oparg #ifdef WITH_TSC, uint64* pintr0, uint64* pintr1 #endif)
            {
                x = do_call(func, pp_stack, na, nk);
                |
                static PyObject * do_call(PyObject *func, PyObject ***pp_stack, int na, int nk)
                {
                    result = PyObject_Call(func, callargs, kwdict)
                    |
                    PyObject * PyObject_Call(PyObject *func, PyObject *arg, PyObject *kw)
                    {
                        call = func->ob_type->tp_call;
                        |
                        PyObject * PyInstance_New(PyObject *klass, PyObject *arg, PyObject *kw)
                        |
                        result = (*call)(func, arg, kw);
                    }
                    |
                }
                |
            }
        |
    }
    ...
}
```

```c
PyTypeObject PyClass_Type = {
    ...
    PyInstance_New,                             /* tp_call */
    ...
};
```
