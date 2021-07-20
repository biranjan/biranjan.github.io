---
title: "Using python generator to count word frequency in text file 🗒️"
date: 2020-09-15T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["generator", "map-reduce"]
categories: ["python"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: false
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Using generator in map reduce way"
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

In this post I will use python generator to count the word frequency from a big text file and try use as little memory as possible.
<!--more-->

In python generator function are function that uses keyword `yield` to return the final result instead of standard `return` keyworod. Generator function returns generator object. They are similar object like lists but they are lazy, in other words it doesn't produce the final object until you iterate over it or use `next` over the object. Its lot easier to show it in code then to define it in words, here is an example below.


```python
# regulalr function
def sequence(n):  
  my_list = []
  for i in range(n):
    my_list.append(i)
  return my_list
 
  
val = sequence(2)
print(val)
```

    [0, 1]



```python
# generator function
def sequence(n):  
  for i in range(n):
    yield i
    
val2 = sequence(2)
print(val2)

  

```

    <generator object sequence at 0x7f1cbc43f5f0>


As you can see function that uses `yield` keyword doesn't return a generator object. This makes it lazy. In order to get the value we can simply loop over it.


```python
def do_something_imp(val):
  print(val)
  
for i in val2:
  do_something_imp(i)
```

    0
    1


But here is a slight caveat to generator obj if you want to call it twice let's see what happens


```python

# first call  
for i in val2:
  do_something_imp(i)
  
# second call
for i in val2:
  do_something_imp(i)
  

```

    0
    1


It only gets printed once. This brings us to the reason why we use generator. Its highly memory efficient it doesn't store its value in memroy once you loop over it its done and its empty. Obviously if you need to use the generator object twice then you can turn it into the list like this `[val for val in gen_obj]` but then you will have all the value in memory and you might not get full benefit out of your generator.

## Time  for some action
Now I have defined the obligatory definition and example of basic generator its time to put in use and see how generator can be useful.  
In this example we will read the text file and list the most used words in the text file. I will do this with generator and without generator and we can see the pros and cons of each approach.  
The text file that I am going to use is *The Complete Works of William Shakespeare* which can be found [here](https://ocw.mit.edu/ans7870/6/6.006/s08/lecturenotes/files/t8.shakespeare.txt).  
First I am going to count the occurrence of each word and list the most used word in the text without using generator. Here we go.



```python
from collections import OrderedDict
from urllib.request import urlopen 

def download_file(url):
  with urlopen(url) as response:
    file = response.read()
  return file

def count_words(file):
  normalised_doc = [word.lower() if word.isalpha() else "" for word in file.decode("utf-8").split()]
  frequencies = {}
  for word in normalised_doc:
    if word != "":
      frequencies[word] = frequencies.get(word,0)+1
  sorted_dict = OrderedDict(sorted(frequencies.items(), key=lambda freq:freq[1], reverse=True))  
  return sorted_dict

def main_func():
    file = download_file("https://ocw.mit.edu/ans7870/6/6.006/s08/lecturenotes/files/t8.shakespeare.txt")
    freq_dict = count_words(file)
    # get top 10
    top_ten_items = list(freq_dict.items())[:10]
    print(top_ten_items)

main_func()
```

    [('the', 27549), ('and', 26037), ('i', 19540), ('to', 18700), ('of', 18010), ('a', 14383), ('my', 12455), ('in', 10671), ('you', 10630), ('that', 10487)]


So we opened the file from url and loaded into the memory using function `download_file` then we converted the text file into a giant list called `normalised_doc` inside the function `count_words`, we then converted list into dictionary containing the key as word and value as the number of occurrences in the file. Then we simply sorted the dictionary using the value so that highest frequency word are at the top of the dictionary. We used `OrderedDict` to make sure that dictionary sorted order retains its order every time we request a value from it.  
Now lets solve this using generator. But before I do that I need to explain that we can produce generator object not just from function but also from comprehension. Here is a quick example of it.


```python
# regular list comprehension
a = [i for i in range(2)]
print(a)
# generator from comprehension
b = (i for i in range(2))
print(b)
```

    [0, 1]
    <generator object <genexpr> at 0x7f0e96c18350>


By simply wrapping the comprehension expression inside the round bracket `()` we produce generator object.  
With that explained we can see the generator way of doing this.


```python
# generator.py
from collections import OrderedDict
from urllib.request import urlopen 
from functools import reduce


def gen_line(url):
   texts = urlopen(url)
   for line in texts:
     yield line
    
def count_words(doc):
  normalised_doc = (word.lower() if word.isalpha() else "" for word in doc.decode("utf-8").split())
  frequencies = {}
  for word in normalised_doc:
    if word != "":
      frequencies[word] = frequencies.get(word,0) + 1
  return frequencies


def combine_counts(d1, d2):
  d = d1.copy()
  for word, count in d2.items():
    d[word] = d.get(word,0) + count
  return d  


def main_func():
    file = gen_line("https://ocw.mit.edu/ans7870/6/6.006/s08/lecturenotes/files/t8.shakespeare.txt")
    counts = map(count_words, file)
    total_counts = reduce(combine_counts, counts)
    d = OrderedDict(sorted(total_counts.items(), key=lambda freq:freq[1], reverse=True))  
    top_ten_items = list(d.items())[:10]
    print(top_ten_items)
    
main_func()
```

    [('the', 27549), ('and', 26037), ('i', 19540), ('to', 18700), ('of', 18010), ('a', 14383), ('my', 12455), ('in', 10671), ('you', 10630), ('that', 10487)]


So this implementation is bit involved. Lets go line by line. The first function `gen_line` is similar to download file however instead of reading the whole file in memory `gen_line` yields each line. See below, function `gen_line` return generator and when you loop over it, it returns line in the order they appear in text. Without storing anything in memory.


```python
file = gen_line("https://ocw.mit.edu/ans7870/6/6.006/s08/lecturenotes/files/t8.shakespeare.txt")
print(file)
for line in file:
  print(line)
  break
```

    <generator object gen_line at 0x7f0e97533b30>
    b'This is the 100th Etext file presented by Project Gutenberg, and\n'


Function `count_words` remains the same. However instead of directly using function `count_word` we instead use python builtin function `map`. Which applies the function `count_word` to the generator object returned by function `gen_line`, essentially each line generated by `gen_line` function goes through `count_words` and produces list of dictionary which contains words and its frequency on that particular line. If its not clear then lets see the output of map below. And by the way map also returns generator so it is still lazily evaluated which means we haven't used any memory nor we have done any computation.



```python
file = gen_line("https://ocw.mit.edu/ans7870/6/6.006/s08/lecturenotes/files/t8.shakespeare.txt")
counts = map(count_words, file)

for i in counts:
  print(i)
  break

```

    {'this': 1, 'is': 1, 'the': 1, 'etext': 1, 'file': 1, 'presented': 1, 'by': 1, 'project': 1, 'and': 1}


So with the application of map over count_words(which in turn takes file object as its input) it returns a dictionary of words and its frequency at that given line. And obviously this is not what we want. So we need to merge all the dictionary while summing the value, that way we can count the words and its frequency throughout the text file not just in each line. Thats why we have function `combine_counts` which merges two dictionary by summing value if there is a common key. If you would like to know how reduce function work here is nice [tutorial](https://realpython.com/python-reduce-function/).  
Here we have delayed the calculation and loading any value in memory up until the point where we call `reduce` function. This helps us to keep it memory efficient. See the result of memory profile between these two approach.


```python
Filename: generator.py

Line #    Mem usage    Increment  Occurences   Line Contents
============================================================
    27     39.0 MiB     39.0 MiB           1   @profile(stream = fp)
    28                                         def main_func():
    29     39.0 MiB      0.0 MiB           1       file = gen_line("https://ocw.mit.edu/ans7870/6/6.006/s08/lecturenotes/files/t8.shakespeare.txt")
    30     39.0 MiB      0.0 MiB           1       counts = map(count_words, file)
    31     49.7 MiB     10.8 MiB           1       total_counts = reduce(combine_counts, counts)
    32     55.9 MiB      6.2 MiB      119017       d = OrderedDict(sorted(total_counts.items(), key=lambda freq:freq[1], reverse=True))  
    33     56.2 MiB      0.2 MiB           1       top_ten_items = list(d.items())[:10]
    34     56.2 MiB      0.0 MiB           1       print(top_ten_items)  
```


```python
Filename: general_way.py

Line #    Mem usage    Increment  Occurences   Line Contents
============================================================
    25     39.0 MiB     39.0 MiB           1   @profile
    26                                         def main_func():
    27     46.5 MiB      7.5 MiB           1       file = download_file("https://ocw.mit.edu/ans7870/6/6.006/s08/lecturenotes/files/t8.shakespeare.txt")
    28    121.0 MiB     74.4 MiB           1       freq_dict = count_words(file)
    29                                             # get top 10
    30    121.0 MiB      0.0 MiB           1       top_ten_items = list(freq_dict.items())[:10]
    31    121.0 MiB      0.0 MiB           1       print(top_ten_items)

```

So here we can say that we save quite a lot of memory using generator. But if you have followed along then you must have noticed that my implementation of generator for producing word frequency was very time consuming compared to without generator. And the problem here is not with generator but my implementation of it. In particular the function `combine_counts` in a line where I am copying dictionary `d = d1.copy()`. Since we have thousands of dictionaries returned from map function, when we copy dictionary thousand of times this cost us on time. To solve this we have to modify our function a bit and slightly take a hit on our accuracy of counting. Here is the faster implementation.


```python
def gen_line(url):
	texts = urlopen(url)
	while True:
		piece = texts.read(1024*10)
		if piece:
			yield piece
		else:
			return

def count_words(doc):
  normalised_doc = (word.lower() if word.isalpha() else "" for word in doc.decode("utf-8").split())
  frequencies = {}
  
  for word in normalised_doc:
    if word != "":
      frequencies[word] = frequencies.get(word,0) + 1
  return frequencies


def combine_counts(d1, d2):
  d = d1.copy()
  for word, count in d2.items():
    d[word] = d.get(word,0) + count
  return d  

def main_func():
    file = gen_line("https://ocw.mit.edu/ans7870/6/6.006/s08/lecturenotes/files/t8.shakespeare.txt")
    counts = map(count_words, file)
    # total_counts = {key: counts.get([key],0) + counts.get([key],0) for key in counts}
    total_counts = reduce(combine_counts, counts)
    d = OrderedDict(sorted(total_counts.items(), key=lambda freq:freq[1], reverse=True))  
    top_ten_items = list(d.items())[:10]
    print(top_ten_items)
    
main_func()
   
```

    [('the', 27546), ('and', 26031), ('i', 19552), ('to', 18698), ('of', 18009), ('a', 14392), ('my', 12455), ('in', 10668), ('you', 10626), ('that', 10486)]


The bottleneck with my implementation of generator was that there was too many nested dictionary from map operation. And this was because the text file has tens of thousand of lines. So I solved the problem by not yielding the line but chunk of 1034*10 bytes of text each time. Which results in less line which means less nested dictionary, this speeds up the time and its now almost the same time compared to solving counting problem without generator but we also save half the memory. Since I am not yielding each line this has slight effect on the number of count. When I yield 1024 bytes it can fetch incomplete words this result in slight undercounting of words.


```python
 def count_words(doc):
  normalised_doc = (word.lower() for word in doc.decode("utf-8").split())
  frequencies = {}
  
  for word in normalised_doc:
    if word != "":
      frequencies[word] = frequencies.get(word,0) + 1
  return frequencies

 file = gen_line("https://ocw.mit.edu/ans7870/6/6.006/s08/lecturenotes/files/t8.shakespeare.txt")
 counts = map(count_words, file)
```
