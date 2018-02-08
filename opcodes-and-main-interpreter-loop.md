

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
>>> c = compile('test.py', 'test.py', 'exec')
>>> c
<code object <module> at 0x109785930, file "test.py", line 1>
```

```py
## This command shows the byte code that test.py compile to.
>>> c.co_code
'e\x00\x00j\x01\x00\x01d\x00\x00S'
```

```py
## Split the bytes into a list.
>>> [byte for byte in c.co_code]
['e', '\x00', '\x00', 'j', '\x01', '\x00', '\x01', 'd', '\x00', '\x00', 'S']
```

```py
## Shows each byte in ordinary.
>>> [ord(byte) for byte in c.co_code]
[101, 0, 0, 106, 1, 0, 1, 100, 0, 0, 83]
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

Here we can see that the Python script is first compiled into instruction opcodes. When the Python interpreter runs, it will execute based on the instruction opcode and the value stack. Whenever the instruction involves manipulating data, the data will first be pushed to the value stack. Let's look at the example above. LOAD_CONST will push 1 to the value stack. Then STORE_NAME will pop 1 from the value stack and allocate a memory space to save it._

stack\_pointer in [https://github.com/python/cpython/blob/master/Python/ceval.c](https://github.com/python/cpython/blob/master/Python/ceval.c) is the value stack

[https://github.com/python/cpython/blob/master/Python/ceval.c](https://github.com/python/cpython/blob/master/Python/ceval.c)-&gt; main interpreter function

  


