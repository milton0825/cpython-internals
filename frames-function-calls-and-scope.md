## Frames function calls and scope

Create test.py
```py
x = 10

def foo(x):
    y = x * 2
    return bar(y)

def bar(x):
    y = x / 2
    return y

z = foo(x)
```
You can copy and paste the above code and trace it in http://www.pythontutor.com/.

```bash
$ python -m dis test.py
  1           0 LOAD_CONST               0 (10)
              3 STORE_NAME               0 (x)

  3           6 LOAD_CONST               1 (<code object foo at 0x1070a2630, file "test.py", line 3>)
              9 MAKE_FUNCTION            0
             12 STORE_NAME               1 (foo)

  7          15 LOAD_CONST               2 (<code object bar at 0x1070a2430, file "test.py", line 7>)
             18 MAKE_FUNCTION            0
             21 STORE_NAME               2 (bar)

 11          24 LOAD_NAME                1 (foo)
             27 LOAD_NAME                0 (x)
             30 CALL_FUNCTION            1
             33 STORE_NAME               3 (z)
             36 LOAD_CONST               3 (None)
             39 RETURN_VALUE
```
Code object is the code inside the function. 


The reason why we need to run MAKE_FUNCTION at run time. Function object contains code and a pointer that points to its environment.
At run time we bind function

```py$
python
>>> import dis
>>> import test
>>> dis.dis(test.foo)
  4           0 LOAD_FAST                0 (x)
              3 LOAD_CONST               1 (2)
              6 BINARY_MULTIPLY
              7 STORE_FAST               1 (y)

  5          10 LOAD_GLOBAL              0 (bar)
             13 LOAD_FAST                1 (y)
             16 CALL_FUNCTION            1
             19 RETURN_VALUE
>>> dis.dis(test.bar)
  8           0 LOAD_FAST                0 (x)
              3 LOAD_CONST               1 (2)
              6 BINARY_DIVIDE
              7 STORE_FAST               1 (y)

  9          10 LOAD_FAST                1 (y)
             13 RETURN_VALUE


```

In the code object, we have a list of instruction opcodes that we need to run.
```c
typedef struct {
  PyObject *co_code;          /* instruction opcodes */
} PyCodeObject;
```

The frame object has a pointer that points at the previous frame so that we can return to it after current frame ends. It also has
the code object so that we can run the code.
```c
typedef struct _frame {
  struct _frame *f_back;      /* previous frame, or NULL */
  PyCodeObject *f_code;       /* code segment */
  PyObject *f_builtins;       /* builtin symbol table (PyDictObject) */
  PyObject *f_globals;        /* global symbol table (PyDictObject) */
  PyObject *f_locals;         /* local symbol table (any mapping) */
  PyObject **f_valuestack;    /* points after the last local */
} PyFrameObject;

```

```
PyEval_EvalFrameEx
  call_function
    f = _PyFrame_New_NoTrack(tstate, co, globals, locals)
    retval = PyEval_EvalFrameEx(f,0);
  
```


https://github.com/python/cpython/blob/master/Include/code.h
https://github.com/python/cpython/blob/master/Include/frameobject.h
