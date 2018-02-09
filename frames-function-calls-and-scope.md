
https://github.com/python/cpython/blob/master/Python/ceval.c
The cpython interpreter 

```c
for (;;) {
  switch (opcode) {
    TARGET(LOAD_CONST) {
        PyObject *value = GETITEM(consts, oparg);
        Py_INCREF(value);
        PUSH(value);
        FAST_DISPATCH();
    }
    PREDICTED(STORE_FAST);
    TARGET(STORE_FAST) {
        PyObject *value = POP();
        SETLOCAL(oparg, value);
        FAST_DISPATCH();
    }
    .
    .
    .
  }
}
```

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
