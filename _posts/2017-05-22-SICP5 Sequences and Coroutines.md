---
layout: post
title: SICP5 Sequences and Coroutines
date: 2017-05-22
tag: Python
---   

## Introduction

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

### Streams

* A stream is a lazily computed recursive list. 

* Like the Rlist class from Chapter 3, a Stream instance responds to requests for its first element and the rest of the stream. Like an Rlist, the rest of a Stream is itself a Stream. Unlike an Rlist, the rest of a stream is only computed when it is looked up, rather than being stored in advance. That is, the rest of a stream is computed lazily.

* a stream stores how to compute the rest of the stream, rather than always storing it explicitly.

```
class Stream(object):
    """A lazily computed recuisive list."""
    def __init__(self, first, compute_rest, empty=False):
        self.first = first
        self._compute_rest = compute_rest
        self.empty = empty
        self._rest = None
        self._computed = False
    @property
    def rest(self):
        """Return the rest of the stream, computing it if necessary."""
        assert not self.empty, 'Empty streams have no rest.'
        if not self._computed:
            self._rest = self._cmopute_rest()
            self._computed = True
        return self._rest
    def __repr__(self):
        if self.empty:
            return '<empty stream>'
        return 'Stream({0}, <compute_rest>)'.format(repr(self.first))

Stream.empty = Stream(None, None, True)

s = Stream(1, lambda: Stream(2+3, lambda: Stream.empty))
```

* The essential properties of a compute_rest function are that it takes no arguments, and it returns a Stream.

* Lazy evaluation gives us the ability to represent infinite sequential datasets using streams. For example, we can represent increasing integers, starting at any first value.

```
def make_integer_stream(first=1):
    def compute_rest():
        return make_integer_stream(first+1)
    return Stream(first, compute_rest)


def map_stream(fn, s):
    if s.empty:
        return s
    def compute_rest():
        return map_stream(fn, s.rest)
    returen Stream(fn(s.first), compute_rest)

def filter_stream(fn, s):
    if s.empty:
        return s
    def compute_rest():
        return filter_stream(fn, s.rest)
    if fn(s.first):
        return stream(s.first, compute_rest)
    return compute_rest()

def truncate_stream(s, k):
    if s.empty or k == 0:
        return Stream.empty
    def compute_rest():
        return truncate_stream(s.rest, k-1)
    return Stream(s.first, compute_rest)

def stream_to_list(s):
    r = []
    while not s.empty:
        r.append(s.first)
        s = s.rest
    return r


def primes(pos_stream):
    def not_divible(x):
        return x % pos_stream.first != 0
    def compute_rest():
        return primes(filter_stream(not_divible, pos_stream.rist))
    return Stream(pos_stream.first, compute_rest)
```

* Streams contrast with iterators in that they can be passed to pure functions multiple times and yield the same result each time.

* streams provide a simple, functional, recursive data structure that implements lazy evaluation through the use of higher-order functions.

## Coroutines

* When the logic for a function with complex behavior is divided into several self-contained steps that are themselves functions, these functions are called helper functions or subroutines.

* However, when using coroutines, there is no main function to coordinate results. Instead coroutines themselves link together to form a pipeline. 

* Coroutines are all colleagues, they cooperate to form a pipeline without any supervising function responsible for calling them in a particular order.

### Python Coroutines

* Python generator functions can also consume values using a (yield) statement

```
def match(pattern):
    print('Looking for ' + pattern)
    try:
        while True:
            s = (yield)
            if pattern in s:
                print(s)
    except GeneratorExit:
        print("=== Done ===")
```

* We can chain functions that send() and functions that yield together achieve complex behaviors. For example, the function below splits a string named text into words and sends each word to another coroutine.

```
def read(text, next_coroutine):
    for line in text.split():
        next_coroutine.send(line)
    next_coroutine.close()
```

### Produce, Filter, and Consume

* Coroutines can have different roles depending on how they use yield and send().

```
A Producer creates items in a series and uses send(), but not (yield)
A Filter uses (yield) to consume items and send() to send result to a next step.
A Consumer uses (yield) to consume items, but does not send.


def match_filter(pattern, next_coroutine):
    print('Looking for ' + pattern)
    try:
        while True:
            s = (yield)
            if pattern in s:
                next_coroutine.send(s)
    except GeneratorExit:
        next_coroutine.close()

def print():
    print('Preparing to print')
    try:
        while True:
            line = (yield)
            print(line)
    except GeneratorExit:
        print("=== Done ===")
```

* Even though the name filter implies removing items, filters can transform items as well. The function below is an example of a filter that transforms items.

```
def count_letters(next_coroutine):
    try:
        while True:
            s = (yield)
            counts = {letter:s.count(letter) for letter in set(s)}
            next_coroutine.send(counts)
    except GeneratorExit as e:
        nnext_coroutine.close()

def sum_dictionaries():
    total = {}
    try:
        while True:
            counts = (yield)
            for letter, count in counts.items():
                total[letter] = count + total.get(letter, 0)
    except GeneratorExit:
        max_letter = max(total.items(), key=lambda t: t[1])[0]
        print("Most frequent letter: " + max_letter)
```

### Multitasking

* A producer or filter does not have to be restricted to just one next step. It can have multiple coroutines downstream of it, and send() data to all of them.

```
def read_to_many(text, coroutines):
    for word in text.split():
        for coroutine in coroutines:
            coroutine.send(word)
        for coroutine in coroutines:
            coroutine.close()
```