## Code objects
A code object is CPython's internal representation of a piece of runnable Python code, such as a function, a module, a class body, or a generator expression. Code objects are created during the compilation stage and cached in memory.
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

## Function objects
Function objects contains every thing the interpreter needs to run a functional code: runnable bytecode and execution environment. Function objects are created and binded to the corresponding code objects at runtime.

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

To make a function call in current frame, the interpreter would find the code object and the executable environment globals and create a new frame. Then execute the new frame.
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

`func_memberlist` maps Python name for example func_closure to actually grab the func_closure field from C code. It allows us to expose members of our C instance structure to Python.
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

## Closures
Closures are function objects that have access to a local variable from an enclosing environment which has finished its execution. In the following example, b1 has access to a local variable x=10, which is local to function foo and finished its execution.
```py
x = 1000
def foo(x):
    def bar(y):
        print x + y
    return bar

b1 = foo(10)
b2 = foo(20)
```

```py
>>> dis.dis(test)
Disassembly of b1:
  5           0 LOAD_DEREF               0 (x)
              3 LOAD_FAST                0 (y)
              6 BINARY_ADD
              7 PRINT_ITEM
              8 PRINT_NEWLINE
              9 LOAD_CONST               0 (None)
             12 RETURN_VALUE

Disassembly of b2:
  5           0 LOAD_DEREF               0 (x)
              3 LOAD_FAST                0 (y)
              6 BINARY_ADD
              7 PRINT_ITEM
              8 PRINT_NEWLINE
              9 LOAD_CONST               0 (None)
             12 RETURN_VALUE

Disassembly of foo:
  4           0 LOAD_CLOSURE             0 (x)
              3 BUILD_TUPLE              1
              6 LOAD_CONST               1 (<code object bar at 0x1080739b0, file "test.py", line 4>)
              9 MAKE_CLOSURE             0
             12 STORE_FAST               1 (bar)

  6          15 LOAD_FAST                1 (bar)
             18 RETURN_VALUE
>>> test.b1 == test.b2
False
>>> test.b1.func_code == test.b2.func_code
True
>>> test.b1.func_code is test.b2.func_code
True
>>> test.b1.func_closure
(<cell at 0x10809d8a0: int object at 0x7fb1f3d057c0>,)
>>> test.b1.func_closure[0].cell_contents
10
>>> test.b2.func_closure[0].cell_contents
20
```





