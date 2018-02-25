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
