## Code objects
Code objects is created during the compilation stage and cached in memory.

```c
typedef struct {
    PyObject_HEAD
    int co_argcount;		/* #arguments, except *args */
    int co_nlocals;		/* #local variables */
    int co_stacksize;		/* #entries needed for evaluation stack */
    int co_flags;		/* CO_..., see below */
    PyObject *co_code;		/* instruction opcodes */
    PyObject *co_consts;	/* list (constants used) */
    PyObject *co_names;		/* list of strings (names used) */
    PyObject *co_varnames;	/* tuple of strings (local variable names) */
    PyObject *co_freevars;	/* tuple of strings (free variable names) */
    PyObject *co_cellvars;      /* tuple of strings (cell variable names) */
    /* The rest doesn't count for hash/cmp */
    PyObject *co_filename;	/* string (where it was loaded from) */
    PyObject *co_name;		/* string (name, for reference) */
    int co_firstlineno;		/* first source line number */
    PyObject *co_lnotab;	/* string (encoding addr<->lineno mapping) See
				   Objects/lnotab_notes.txt for details. */
    void *co_zombieframe;     /* for optimization only (see frameobject.c) */
    PyObject *co_weakreflist;   /* to support weakrefs to code objects */
} PyCodeObject;
```


```python
>>> def foo(x, y):
...     z = x + y
...     return z
...
>>> foo
<function foo at 0x10fc96c80>
>>> bar = foo
>>> bar
<function foo at 0x10fc96c80>
>>> dir(foo)
['__call__', '__class__', '__closure__', '__code__', '__defaults__', '__delattr__', '__dict__', '__doc__', '__format__', '__get__', '__getattribute__', '__globals__', '__hash__', '__init__', '__module__', '__name__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'func_closure', 'func_code', 'func_defaults', 'func_dict', 'func_doc', 'func_globals', 'func_name']
>>> foo.func_globals
{'bar': <function foo at 0x10fc96c80>, '__builtins__': <module '__builtin__' (built-in)>, '__package__': None, '__name__': '__main__', 'foo': <function foo at 0x10fc96c80>, '__doc__': None}
>>> foo.func_code
<code object foo at 0x10fc82d30, file "<stdin>", line 1>
>>> dir(foo.func_code)
['__class__', '__cmp__', '__delattr__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__le__', '__lt__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'co_argcount', 'co_cellvars', 'co_code', 'co_consts', 'co_filename', 'co_firstlineno', 'co_flags', 'co_freevars', 'co_lnotab', 'co_name', 'co_names', 'co_nlocals', 'co_stacksize', 'co_varnames']
>>> foo.func_code.co_argcount
2
>>> foo.func_code.co_varnames
('x', 'y', 'z')
```


## Function objects
Function objects is not created until you execute that line of code. During runtime we link the function object with the corresponding code object

```c
/* Function objects and code objects should not be confused with each other:
 *
 * Function objects are created by the execution of the 'def' statement.
 * They reference a code object in their func_code attribute, which is a
 * purely syntactic object, i.e. nothing more than a compiled version of some
 * source code lines.  There is one code object per source code "fragment",
 * but each code object can be referenced by zero or many function objects
 * depending only on how many times the 'def' statement in the source was
 * executed so far.
 */

typedef struct {
    PyObject_HEAD
    PyObject *func_code;	/* A code object */
    PyObject *func_globals;	/* A dictionary (other mappings won't do) */
    PyObject *func_defaults;	/* NULL or a tuple */
    PyObject *func_closure;	/* NULL or a tuple of cell objects */
    PyObject *func_doc;		/* The __doc__ attribute, can be anything */
    PyObject *func_name;	/* The __name__ attribute, a string object */
    PyObject *func_dict;	/* The __dict__ attribute, a dict or NULL */
    PyObject *func_weakreflist;	/* List of weak references */
    PyObject *func_module;	/* The __module__ attribute, can be anything */

    /* Invariant:
     *     func_closure contains the bindings for func_code->co_freevars, so
     *     PyTuple_Size(func_closure) == PyCode_GetNumFree(func_code)
     *     (func_closure may be NULL if PyCode_GetNumFree(func_code) == 0).
     */
} PyFunctionObject;
```

Maps Python name for example func_closure to actually grab the func_closure field from C code. Allows you to access the 
```c
static PyMemberDef func_memberlist[] = {
    {"func_closure",  T_OBJECT,     OFF(func_closure),
     RESTRICTED|READONLY},
    {"__closure__",  T_OBJECT,      OFF(func_closure),
     RESTRICTED|READONLY},
    {"func_doc",      T_OBJECT,     OFF(func_doc), PY_WRITE_RESTRICTED},
    {"__doc__",       T_OBJECT,     OFF(func_doc), PY_WRITE_RESTRICTED},
    {"func_globals",  T_OBJECT,     OFF(func_globals),
     RESTRICTED|READONLY},
    {"__globals__",  T_OBJECT,      OFF(func_globals),
     RESTRICTED|READONLY},
    {"__module__",    T_OBJECT,     OFF(func_module), PY_WRITE_RESTRICTED},
    {NULL}  /* Sentinel */
};
```

```c
PyObject * PyEval_EvalFrameEx(PyFrameObject *f, int throwflag)
	case CALL_FUNCTION:
		static PyObject * call_function(PyObject ***pp_stack, int oparg #ifdef WITH_TSC , uint64* pintr0, uint64* pintr1 #endif)
			static PyObject * fast_function(PyObject *func, PyObject ***pp_stack, int n, int na, int nk)
				PyCodeObject *co = (PyCodeObject *)PyFunction_GET_CODE(func);
   		 		PyObject *globals = PyFunction_GET_GLOBALS(func);
    			PyObject *argdefs = PyFunction_GET_DEFAULTS(func);
				PyFrameObject *f = PyFrame_New(tstate, co, globals, NULL);
				retval = PyEval_EvalFrameEx(f,0);
			
			
			
```

## Closures
