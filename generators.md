## Generators

From the previous chapter [Iterators](https://github.com/milton0825/cpython-internals/blob/master/iterators.md), we learned how to create a custom class to support iterator. The class needs to implement two methods: `__iter__` and `next` to support iterator.
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

for c in Counter(5, 10):
    print c
```

We can also 
```py
def Counter(low, high):
    current = low
    while current < high:
        yield current
        current += 1
        
for c in Counter(5, 10):
    print c
```

`os.walk` also uses generator.

what is generator. what is yield

```py
>>> import test
>>> import dis
>>> dis.dis(test.Counter)
  2           0 LOAD_FAST                0 (low)
              3 STORE_FAST               2 (current)

  3           6 SETUP_LOOP              31 (to 40)
        >>    9 LOAD_FAST                2 (current)
             12 LOAD_FAST                1 (high)
             15 COMPARE_OP               0 (<)
             18 POP_JUMP_IF_FALSE       39

  4          21 LOAD_FAST                2 (current)
             24 YIELD_VALUE
             25 POP_TOP

  5          26 LOAD_FAST                2 (current)
             29 LOAD_CONST               1 (1)
             32 INPLACE_ADD
             33 STORE_FAST               2 (current)
             36 JUMP_ABSOLUTE            9
        >>   39 POP_BLOCK
        >>   40 LOAD_CONST               0 (None)
             43 RETURN_VALUE
```
