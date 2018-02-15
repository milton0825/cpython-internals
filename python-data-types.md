## Python data types

Sequence data types
* String: a sequence of characters. It is immutable.
* Tuple: a sequence of objects. It is immutable.
* List: a sequence of objects. It is mutable.

PyStringObject
```c
typedef struct {
    PyObject_VAR_HEAD
    long ob_shash;
    int ob_sstate;
    char ob_sval[1];

    /* Invariants:
     *     ob_sval contains space for 'ob_size+1' elements.
     *     ob_sval[ob_size] == 0.
     *     ob_shash is the hash of the string or -1 if not computed yet.
     *     ob_sstate != 0 iff the string object is in stringobject.c's
     *       'interned' dictionary; in this case the two references
     *       from 'interned' to this object are *not counted* in ob_refcnt.
     */
} PyStringObject;

```

```py
>>> x = "asdmvl;kam a;lskmd;lckamga;si as;lkdmf"
>>> y = "asdmvl;kam a;lskmd;lckamga;si as;lkdmf"
>>> x is y
False
```

## References
* https://github.com/python/cpython/blob/master/Objects/abstract.c
