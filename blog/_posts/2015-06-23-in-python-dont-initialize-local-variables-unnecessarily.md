---
title: "In Python, don't initialize local variables unnecessarily"
date: "2015-06-23"
---

A common pattern:

```
def foo():
    x = some value # but do we need this? (short answer: no)
    if something:
        # ... do stuff ...
        x = 'bla'
    else:
        x = 'blo'

```

The variable `x` is being initialized before the if/else, but the intention of the programmer is that its value will actually be determined by the if/else itself. If somebody later comes around and mistakenly removes one of the assignments (inside 'if' or 'else'), no runtime error will occur and `x` will remain initialized to a probably wrong value.

**Leaving out the initialization is better** - in that case, forgetting to set `x` in one of the branches will cause an `UnboundLocalError`:

```
>>> def foo():
...     if False:
...         x = 0
...     return x
... 
>>> foo()
Traceback (most recent call last):
  File "", line 1, in 
  File "", line 4, in foo
UnboundLocalError: local variable 'x' referenced before assignment

```

Errors are good! (when they flag buggy code)

Now, what if we **also** have an `x` declared in the global scope? Because of [how Python handles variable scope](http://www.python-course.eu/python3_global_vs_local_variables.php), the error will still happen (which is good).

```
>>> x = 1
>>> def foo():
...     if False:
...         x = 0
...     return x
... 
>>> foo()
Traceback (most recent call last):
..
UnboundLocalError: local variable 'x' referenced before assignment

```

**Summary**: In Python, don't initialize variables until you know what value to assign to them.
