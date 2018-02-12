## Frames, function calls and scope

Create a Python script test.py with code:
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

Let's disassemble the byte code of the above python script. We can find it first load and store the global variable x. At line 3,
it loads the code object foo, makes it a function and bind it to its name. At line 11, it loads the function foo and the input variable
to value stack and invoke function through CALL_FUNCTION.
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

Let's disassemble the byte code of method foo and bar. 
```py
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

The code object contains a list of instruction opcodes compiled from the script.
```c
typedef struct {
  PyObject *co_code;          /* instruction opcodes */
} PyCodeObject;
```

The frame object has a pointer that points to the previous frame so that program can return to previous frame after current frame ends.
It also has the code object so that we can run the code.
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

This is the main logic about how Python interpreter handles function calls. PyEval_EvalFrameEx is a method that handles execution of
a frame. If program need to call another function in the current frame, it will create a new frame for the next level and invoke
PyEval_EvalFrameEx with the next frame.
```
PyEval_EvalFrameEx
  call_function
    f = _PyFrame_New_NoTrack(tstate, co, globals, locals)
    retval = PyEval_EvalFrameEx(f,0);
  
```

## References
https://github.com/python/cpython/blob/master/Include/code.h
https://github.com/python/cpython/blob/master/Include/frameobject.h
