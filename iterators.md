## Iterators
An iterator allows you to iterate through all the elements in a collection in a specific order.

```py
>>> x = [1, 2, 3]
>>> i = x.__iter__()
>>> i
<listiterator object at 0x10d912950>
>>> i.next()
1
>>> i.next()
2
>>> i.next()
3
>>> i.next()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

```py
x = ['a', 'b', 'c']
for e in x:
    print e
```
```bash
$ python -m dis test.py
  1           0 LOAD_CONST               0 ('a')
              3 LOAD_CONST               1 ('b')
              6 LOAD_CONST               2 ('c')
              9 BUILD_LIST               3
             12 STORE_NAME               0 (x)

  2          15 SETUP_LOOP              19 (to 37)
             18 LOAD_NAME                0 (x)
             21 GET_ITER                             
        >>   22 FOR_ITER                11 (to 36)
             25 STORE_NAME               1 (e)

  3          28 LOAD_NAME                1 (e)
             31 PRINT_ITEM
             32 PRINT_NEWLINE
             33 JUMP_ABSOLUTE           22
        >>   36 POP_BLOCK
        >>   37 LOAD_CONST               3 (None)
             40 RETURN_VALUE
```
