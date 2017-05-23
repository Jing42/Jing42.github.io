---
layout: post
title: SICP5 Sequences and Coroutines
date: 2017-05-22
tag: Python
---   

## 介绍

   In this chapter, we introduce new constructs for working with sequential data that are designed to accommodate collections of unknown or unbounded length, while using limited memory. We also discuss how these tools can be used with a programming construct called a coroutine to create efficient, modular data processing pipelines.

## Implicit Sequences

* elements are computed on the fly.

* Computer science is a discipline that celebrates laziness as an important computational tool.

* Iterators support sequential access.

### Python Iterators

```
class Letters(object):
    def __init__(self):
        self.current = 'a'
    def __next__(self):
        if self.current > 'd':
            raise StopIteration
        result = self.current
        self.current = chr(ord(result)+1)
        return result
    def __iter__(self):
        return self
```

* A Letters instance can only be iterated through once.

* Iterators also allow us to represent infinite series by implementing a __next__ method that never raises a StopIteration exception.

class Positives(object):
    def __init__(self):
        self.current = 0
    def __next__(self):
        result = self.current
        self.current += 1
        return result
    def __iter__(self):
        return self

*  If an object represents sequential data, it can serve as an iterable object in a for statement by returning an iterator object in response to the __iter__ message.

```
counts = [1, 2, 3]
i = counts.__iter__()
try:
    while True:
        item = i.__next__()
        print(item)
    except StopIteration:
        pass
```

### Generators and Yield Statements

* Generators allow us to define more complicated iterations by leveraging the features of the Python interpreter.

* A generator is an iterator returned by a special class of function called a generator function. Generator functions are distinguished from regular functions in that rather than containing return statements in their body, they use yield statement to return elements of a series.

```
def letters_generator():
    current = 'a'
    while current <= 'd':
        yield current
        current = chr(ord(current)+1)

for letter in letters_generator():
    print(letter)
```

* Each call to __next__ continues execution of the generator function from wherever it left off previously until another yield statement is executed.

* The generator does not start executing any of the body statements of its generator function until the first time __next__() is called.

### Iterables

* New iterable classes can be defined by implementing the iterable interface. 

```
class LetterIterable(object):
    def __iter__(self):
        current = 'a'
        while current <= 'd':
            yield current
            current = chr(ord(current)+1)
```