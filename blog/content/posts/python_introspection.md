---
title: "Introspection in python 🕵🏽"
date: 2020-08-05
# weight: 1
# aliases: ["/first"]
tags: ["introspection"]
categories: ["python"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Introspection in python"
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/<path_to_repo>/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

Introspection in python is very interesting and it can help debug in various situation.

Introspecting type information. 

We can use type function to check the type of python object. In the example below we assign 7 to variable val. And as we see type of val is class int. We can further check what is the type of int and surprise surprise it is of class type.


```python
val = 7
print(type(val))
print(type(int))

```

    <class 'int'>
    <class 'type'>


We can do a similar introspection with a class object. We can see that the instance of class a is of type a. Whereas the type of class is again `type`. Similarly, with my_func you can see that even a function is a class. And if we check the type of class function then it is of type `type`. So all the built-in objects in python are of type `type`.


```python
class a:
    def __repr__(self):
        return "apple"
b = a()
print(b)
print(type(b))
print(type(a))

def my_func():
    return "hello"

print(type(my_func))
print(type(type(my_func)))

```

    apple
    <class '__main__.a'>
    <class 'type'>
    <class 'function'>
    <class 'type'>


Another handy function is `isinstance()` and `issubclass()`. And here is how we can use 


```python
print(isinstance(b,a))
print(issubclass(a, type))
```

    True
    False


These two functions are self-explanatory. `isinstance()` just checks whether the first argument is an instance of the second argument. Whereas `issubclass()` checks whether the first argument is the subclass of the second argument. In the example above we can see that although the class object `a` is a type of `type` it is not the subclass of `type`.

## Attributes
I find function `dir` useful to introspect the python object. This built-in function will list the attributes of python objects. For variable `val` which of type int we can see its built in attributes using `dir`.

```python
val = 7
print(dir(val))
print(abs(val))
```

    ['__abs__', '__add__', '__and__', '__bool__', '__ceil__', '__class__', '__delattr__', '__dir__', '__divmod__', '__doc__', '__eq__', '__float__', '__floor__', '__floordiv__', '__format__', '__ge__', '__getattribute__', '__getnewargs__', '__gt__', '__hash__', '__index__', '__init__', '__init_subclass__', '__int__', '__invert__', '__le__', '__lshift__', '__lt__', '__mod__', '__mul__', '__ne__', '__neg__', '__new__', '__or__', '__pos__', '__pow__', '__radd__', '__rand__', '__rdivmod__', '__reduce__', '__reduce_ex__', '__repr__', '__rfloordiv__', '__rlshift__', '__rmod__', '__rmul__', '__ror__', '__round__', '__rpow__', '__rrshift__', '__rshift__', '__rsub__', '__rtruediv__', '__rxor__', '__setattr__', '__sizeof__', '__str__', '__sub__', '__subclasshook__', '__truediv__', '__trunc__', '__xor__', 'as_integer_ratio', 'bit_length', 'conjugate', 'denominator', 'from_bytes', 'imag', 'numerator', 'real', 'to_bytes']
    7


We can check whether the objects attributes are callable or not by first getting the attribute using function `getattr()` and then using the function `callable()` on the result. See the documentation of function [`to_bytes`](https://docs.python.org/3/library/stdtypes.html) for more if you are interested on what it does.


```python
print(callable(getattr(val, 'to_bytes')))
getattr(val, 'to_bytes')

```
    True
    <function int.to_bytes(length, byteorder, *, signed=False)>



So we see attribute `to_bytes` is callable. And we can convert our variable to bytes.


```python
val.to_bytes(2,byteorder='big')
```
    b'\x00\x07'



## Introspecting namespace
We can introspect all the global variables available by simply using function `globals()`


```python
globals()
```




    {'__name__': '__main__',
     '__builtin__': <module 'builtins' (built-in)>,
     '__builtins__': <module 'builtins' (built-in)>,
     '_ih': ['', 'globals()'],
     '_oh': {},
     '_dh': ['/'],
     'In': ['', 'globals()'],
     'Out': {},
     'get_ipython': <bound method InteractiveShell.get_ipython of <pyolite.interpreter.Interpreter object at 0x2453298>>,
     'exit': <IPython.core.autocall.ExitAutocall at 0x2fe7698>,
     'quit': <IPython.core.autocall.ExitAutocall at 0x2fe7698>,
     '_': '',
     '__': '',
     '___': '',
     '_i': '',
     '_ii': '',
     '_iii': '',
     '_i1': 'globals()'}



These are the global variable available to me(might be different for you). Now if I define new variable it will be available from globals too.


```python
new_int = 2
globals().get('new_int')
```
    2



## Let's inspect 🔎
Another python library useful for introspect is `inspect`.


```python
import inspect
```


```python
def addition(a:int, b:int) -> int:
    """Adds two integers"""
    c = a + b
    return c
```


```python
inspect.isfunction(addition)
```

    True




```python
inspect.signature(addition)
```
    <Signature (a: int, b: int) -> int>




```python
inspect.getdoc(addition)
```
    'Adds two integers'



Inspect has lots of interesting use cases see the [doc for more](https://docs.python.org/3/library/inspect.html)

## Final words
I have shown some of the ways you can introspect objects in python. In most day-to-day use you might never have to check types and instances and it is not recommended either. With the principle of it's easier to ask forgiveness than permission raising exception is better than checking types and attributes. However, this might be useful for debugging or in general better understanding of how python works.





