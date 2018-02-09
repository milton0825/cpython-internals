

First of all, create a Python script with name test.py and with the following code.

test.py

```py
x = 1
y = 2
z = x + y
print z
```

Start the python interpreter in interactive mode:

```py
$ python  
[GCC 4.2.1 Compatible Apple LLVM 8.0.0 (clang-800.0.34)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
```

```py
## This command compiles test.py into a code object.
>>> c = compile(open('test.py').read(), 'test.py', 'exec')
>>> c
<code object <module> at 0x104f069b0, file "test.py", line 1>
```

```py
## This command shows the byte code that test.py compile to.
>>> c.co_code
'd\x00\x00Z\x00\x00d\x01\x00Z\x01\x00e\x00\x00e\x01\x00\x17Z\x02\x00e\x02\x00GHd\x02\x00S'
```

```py
## Split the bytes into a list.
>>> [byte for byte in c.co_code]
['d', '\x00', '\x00', 'Z', '\x00', '\x00', 'd', '\x01', '\x00', 'Z', '\x01', '\x00', 'e', '\x00', '\x00', 'e', '\x01', '\x00', '\x17', 'Z', '\x02', '\x00', 'e', '\x02', '\x00', 'G', 'H', 'd', '\x02', '\x00', 'S']
```

```py
## Shows each byte in ordinary.
>>> [ord(byte) for byte in c.co_code]
[100, 0, 0, 90, 0, 0, 100, 1, 0, 90, 1, 0, 101, 0, 0, 101, 1, 0, 23, 90, 2, 0, 101, 2, 0, 71, 72, 100, 2, 0, 83]
```

```py
## This command loads the module in 
## https://github.com/python/cpython/blob/master/Lib/dis.py and disassemble test.py
## You can check https://github.com/python/cpython/blob/master/Include/opcode.h to see all the instruction opcodes.
$ python -m dis test.py

  1           0 LOAD_CONST               0 (1)
              3 STORE_NAME               0 (x)

  2           6 LOAD_CONST               1 (2)
              9 STORE_NAME               1 (y)

  3          12 LOAD_NAME                0 (x)
             15 LOAD_NAME                1 (y)
             18 BINARY_ADD
             19 STORE_NAME               2 (z)

  4          22 LOAD_NAME                2 (z)
             25 PRINT_ITEM
             26 PRINT_NEWLINE
             27 LOAD_CONST               2 (None)
             30 RETURN_VALUE
```

Here we can see that the Python script is first compiled into instruction opcodes. When the Python interpreter runs, it will execute based on the instruction opcode and the value stack. Whenever the instruction involves manipulating data, the data will first be pushed to the value stack. Let's look at the example above. LOAD_CONST will push 1 to the value stack. Then STORE_NAME will pop 1 from the value stack and allocate a memory space to save it.

Here is the main cpython interpreter loop:
```c
initialize value stack

for (;;) {
  find opcode
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
https://github.com/python/cpython/blob/master/Python/ceval.c
