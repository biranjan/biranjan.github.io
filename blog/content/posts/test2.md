---
title: "Adventure with python decorator 🚣"
date: 2020-09-15T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["decorator"]
categories: ["python"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Dive into python decorator 🏊‍♀️"
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

Decorator can be intimidating at first and if you have passed that phase then it might be difficult to find the proper use case, here I will try to cover the basics of decorator and end with some use case that might be useful.

<!--more-->

#### Starting with basics 📖

Probably everyone knows that in python functions are just like any other objects, so you can pass them around as arguments and do introspection. 

So this makes it possible to use decorator in python. Decorator is a function that takes function as argument and returns a function. As you will see below decorator is not limited to just function, decorator can be class which takes another class as argument as long as the class object is callable (i.e implements __call__ method) we will see example on this in a while but let see first simple example of decorator. 

```python
def greet_decorator(func):
  def wrapper(name):
    print("Before")
    func(name)
    print("After")
  return wrapper

@greet_decorator
def greet(name):
  print(f"Hi {name}")

greet("John")
```

```
{{< code_output >}}
BEFORE
Hi John
AFTER
```

Ok so that was a very basic example. Lets try to come up with slightly more useful application of decorator. Lets assume you are building an application and you need input from a web request or any service that do not give you a deterministic output. In other words the object that you are querying may or may not give the output that you are looking for, consider this function **roll_dice**.

```python
import random
def roll_dice():
  val = random.randint(1,10)
  if val % 2 != 0:
    raise ValueError(val)
  return val

```

The function **roll_dice** has equal chance of returning value or a value error. But what if you will only like it to return valid value not an error. Let's use decorator to make it happen.

```python
import random 

def retry(func):
  """ Decorator function that retries when there's a value error """
  def wrapper(*args, **kwargs):
    """ wrapper function that modifies the func """
    while True:
      try:
        return func(*args, **kwargs)
      except ValueError as v:
        print(f"Retrying after value error: {v}")
  return wrapper

@retry
def roll_dice(min_num, max_num):
  """ Returns random int between min and max num """
  val = random.randint(min_num,maxnum)
  if val % 2 != 0:
    raise ValueError(val)
  return val

roll_dice(1,100)  
```

```
{{< code_output >}}
Retrying after value error: 1
Retrying after value error: 5
Retrying after value error: 3
2
```

#### Brief Detour 🚧
So lets look at the decorator function **retry** it takes function as input then passes it to its wrapper function aptly named wrapper, wrapper function takes \*args and \*kwargs as input. This allows decorator function to accept any arguments in this case *min_num and max_num*. Inside the wrapper function you can see there is a while loop that only returns when the function roll_dice return without a value error.

Although the decorator function works fine but here is slight problem. If you inspect the new `roll_dice` function this is what you get.

```python
# check for name
print(roll_dice.__name__)
# check the docstring 
print(roll_dice.__doc__)
```

```
{{< code_output >}}
wrapper
wrapper function that modifys the func 
```

So you see the problem once we decorate the `roll_dice` function with retry decorator the signature of the function `roll_dice` has been replaced by wrapper function. This might cause some problem if other part of your program relies on the function signature. But fear not we can fix this by another decorator. Lets see the more correct implementation. Below you can see `wraps` decorator from **functools** library will solve this problem.

```python
import random
import functools

def retry(func):
  """ Decorator function that retries when there's a value error"""
  @functools.wraps(func)
  def wrapper(*args, **kwargs):
    """ wrapper function that modifys the func """
    while True:
      try:
        return func(*args, **kwargs)
      except ValueError as v:
        print(f"Retrying after value error: {v}")
  return wrapper

@retry
def roll_dice(min_num, max_num):
  """ Returns random int between min and max num """
  val = random.randint(min_num, max_num)
  if val % 2 != 0:
    raise ValueError(val)
  return val
```

```
{{< code_output >}}
roll_dice2
 Returns random int between min and max num 
```

#### Next Adventure 🤸
So we made retry decorator and also used `functools.wraps` which preserved all the metadata of original function that it decorates. But what if we want to limit the number of retries. Theoretically the retry decorator the way it is could be retrying forever never returning anything. In order to do so we need someway for decorator function to accept its own argument.


```python
def retry(max_retries):
  def decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
      for _ in range(max_retries):
        try:
          return func(*args, **kwargs)
        except ValueError as v:
          print(f"Retrying after value error: {v}")
      print(f"Max retry reached: Tried {max_retries} times")
    return wrapper
  return decorator

@retry(max_retries=4)
def roll_dice(min_num, max_num):
  """ Returns random int between min and max num """
  val = random.randint(min_num, max_num)
  if val % 2 != 0:
    raise ValueError(val)
  return val
```

```
{{< code_output >}}
Retrying after value error: 13
Retrying after value error: 11
Retrying after value error: 85
Retrying after value error: 83
Max retry reached: Tried 4 times
```

So as you can see that in order to accept argument we have to add one more layer of function to our decorator.

### Final show case 💥
So in very beginning of the post I mentioned that decorator is not just limited to function. Decorator in itself can be a class and can also accept class as its input as long as it has `__call__` implementation.
As our final show case I am going to make a decorator that kind a works like builtin python decorator [`lru_cache`](https://docs.python.org/3/library/functools.html).
The basic idea is that decorator function will store the output of the inner function and the next time instead of executing the inner function it imply returns the stored input, which will result in visible improvement in execution time of the original function. I hope the code will make it more clear if its not clear yet.

```python
import time
import functools

class Cache:
  def __init__(self, func):
    functools.update_wrapper(self, func)
    self.func = func
    self._cache = {}
    
  def __call__(self,host):
    if host not in self._cache:
      self._cache[host] = self.func(host)
    return self._cache[host]
  
@Cache
def long_request(host):
  print("processing request")
  time.sleep(10)
  print("finished processing request")
  return("final value")

# first request
long_request("google.com")    
```

```
{{< code_output >}}
processing request
finished processing request
'final value'
```

```python
# second request
long_request("google.com")
'final value'
```

So above you can see the decorator implementation on class object. Class `Cache` is initialized with func argument and cache. Private variable cache stores the output of decorated function. And you can see the implementation of call method in `__call__` this is where our logic to modify original function happens. Finally you can see that the first call to decorated function `long_request` takes long time however subsequent operation happens instantaneously.

If you don't believe that second call returns at no time and you want proof then we can create one more bonus decorator `time_it`, which simply records the execution time of decorated function.
```python
import time
import functools

class Cache:
  def __init__(self, func):
    functools.update_wrapper(self, func)
    self.func = func
    self._cache = {}
    
  def __call__(self,host):
    if host not in self._cache:
      self._cache[host] = self.func(host)
    return self._cache[host]
  
def time_it(func):
  @functools.wraps(func)
  def wrapper(*args, **kwargs):
    start = time.time()
    try:
      return func(*args, **kwargs)
    finally:
      end_ = time.time() - start
      print(f"Total execution time of function {func.__name__}: {round(end_,2)} s")
  return wrapper

@time_it  
@Cache
def long_request(host):
  print("processing request")
  time.sleep(10)
  print("finished processing request")
  return("final value")

# fist try
long_request("google.com")  
```

```
{{< code_output >}}
processing request
finished processing request
Total execution time long_request: 10.01 s
'final value'
```

```python
# second try 
long_request("google.com")  
```

```
{{< code_output >}}
Total execution time long_request: 0.0 s
'final value'
```

So now you can see another decorator `time_it` stacked on top of the `long_function` which keeps track of the time took to process the decorated function in this case `long_function`.

---

So this concludes our brief tour of python decorator. From here on you can choose your own adventure and create you own decorator to solve your problem. Thank you for reading and I hope that this gave you some insight into decorator 🤞.