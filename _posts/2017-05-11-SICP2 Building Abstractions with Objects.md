---
layout: post
title: SICP2 Building Abstractions with Objects
date: 2017-05-11
tag: Python
---   

## 介绍

Chapter 2: Building Abstractions with Objects

### Introduction

* Higher-order functions enhance the power of our language by enabling us to manipulate, and thereby to reason, in terms of general methods of computation. This is much of the essence of programming.

* With structured data, programs can simulate and reason about virtually any domain of human knowledge and experience. 

* Objects are both information and processes, bundled together to represent the properties, interactions, and behaviors of complex things.

* In Python every value is an object.

### Data Abstraction

* The general technique of isolating the parts of a program that deal with how data are represented from the parts of a program that deal with how those data are manipulated is a powerful design methodology called data abstraction.

* Data abstraction makes programs much easier to design, maintain, and modify.

* Data abstraction is a methodology that enables us to isolate how a compound data object is used from the details of how it is constructed.

### Example: Arithmetic on Rational Numbers

```
def add_rat(x, y):
    nx, dx = numer(x), denom(x)
    ny, dy = numer(y), denom(y)
    return make_rat(nx * dy + ny * dx, dx * dy)

def mul_rat(x, y):
    return make_rat(number(x) * numer(y), denom(x) * denom(y))

def eq_rat(x, y):
    return numer(x) * denom(y) == numer(y) * denom(x)
```

### Tuples

* multiple assignment

```
pair = (1, 2)
x, y = pair
```

* getitem

```
from operator import getitem
getitem(pair, 0)
```

* Representing Rational Numbers

```
from fractions import gcd

def make_rat(n, d):
    g = gcd(n, d)
    return (n//g, d//g)

def make_rat(n ,d):
    return (n, d)

def numer(x):
    return getitem(x, 0)

def denom(x):
    return getitem(x, 1)

def str_rat(x):
    """Return a string 'n/d' for numerator n and denominator d."""
    return '{0}/{1}'.format(numer(x), denom(x))
```

### Abstraction Barriers

* One advantage is that they makes programs much easier to maintain and to modify. The fewer functions that depend on a particular representation, the fewer changes are required when one wants to change that representation.

```
——————————————————————————————————————————————
| Rational numbers in the problem domain:     |
|      add_rat  mul_rat  eq_rat               |
——————————————————————————————————————————————
|                                             |
|Rational numbers as numerators & denominators|
|      make_rat  numer  denom                 |
——————————————————————————————————————————————
|                                             |
|  Rational numbers as tuples                 |
|      tuple getitem                          |
——————————————————————————————————————————————
|                                             |
|  However tuples are implemented in Python   |
——————————————————————————————————————————————

只依赖于下一层。
只有上一层依赖它。
```

### The Properties of Data

* In general, we can think of an abstract data type as defined by some collection of selectors and constructors, together with some behavior conditions. As long as the behavior conditions are met (such as the division property above), these functions constitute a valid representation of the data type. If a pair p was constructed from values x and y, then getitem_pair(p, 0) returns x, and getitem_pair(p, 1) returns y.

* In order to implement rational numbers, we needed a form of glue for two integers, which had the following behavior: If a pair p was constructed from values x and y, then getitem_pair(p, 0) returns x, and getitem_pair(p, 1) returns y. We can implement functions make_pair and getitem_pair that fulfill this description just as well as a tuple.

```
def make_pair(x, y):
    """Return a function that behaves like a pair."""
    def dispatch(m):
        if m == 0:
            return x
        elif m == 1:
            return y
    return dispatch

def getitem_pair(p, i):
    """Return the element at index i of pair p."""
    return p(i)

p = make_pair(1, 2)
getitem_pair(p, 0)
getitem_pair(p, 1)
```

* These functions suffice to represent compound data in our programs.

### Recursive Lists

* Representation of a four-element list: 1, 2, 3, 4: (1, (2, (3, (4, None))))

```
empty_rlist = None
def make_rlist(first, rest):
    """Make a recursive list from its first element and the rest."""
    return (first, rest)

def first(s):
    """Return the first element of a recursive list s."""
    return s[0]

def rest(s):
    """Return the rest of the elements of a recursive list s."""
    return s[1]


counts = make_rlist(1, make_rlist(2, make_rlist(3, make_rlist(4, empty_rlist))))
first(counts)
rest(counts)

def len_rlist(s):
    """Return the length of recursive list s."""
    length = 0
    while s != empty_rlist:
        s, length = rest(s), length + 1
    return length

def getitem_rlist(s, i):
    """Return the element at index i of recursive list s."""
    while i > 0:
        s, i = rest(s), i - 1
    return first(s)
```

### Mapping

```
alternates = (-1, 2, -3, 4, -5)
tuple(map(abs, alternates))
```

### A common convention is to use a single underscore character for the name in the for header if the name is unused in the suite:

```
for _ in range(3):
    print('Go Bears!')
```

###  Conventional Interfaces

* Data abstraction permits us to design programs without becoming enmeshed in the details of data representations.

* A conventional interface is a data format that is shared across many modular components, which can be mixed and matched to perform data processing.

```
def fib(k):
    """Compute the kth Fibonacci number."""
    prev, curr = 1, 0
    for _ in range(k - 1):
        prev, curr = curr, prev + curr
    return curr

def iseven(n):
    return n % 2 == 0

def sum_even_fibs(n):
    """Sum the first n even Fibonacci numbers."""
    return sum(filter(iseven ,map(fib, range(1, n+1))))
``` 

```
def first(s):
    return s[0]

def iscap(s):
    return len(s) > 0 and s[0].isupper()

def acronym(name):
    """Return a tuple of the letters that form the acronym for name."""
    return tuple(map(first, filter(iscap, name.split())))
```

* Expressing programs as sequence operations helps us design programs that are modular. That is, our designs are constructed by combining relatively independent pieces, each of which transforms a sequence. 

* In general, we can encourage modular design by providing a library of standard components together with a conventional interface for connecting the components in flexible ways.

### Reduce

``` 
from operator import mul
from functools import reduce

reduce(mul, (1, 2, 3, 4, 5))

def product_even_fibs(n):
    """Return the product of the first n even Fibonacci numbers, except 0."""
    return reduce(mul, filter(iseven, map(fib, range(2, n+1))))
```

### Local State

```
def make_withdraw(balance):
    """Return a withdraw function that draws down balance with each call."""
    def withdraw(amount):
        nonlocal balance:
        if amount > balance:
            return 'Insufficient funds'
        balance = balance - amount
        return balance
    return withdraw

Calling withdraw has the side effect of altering the environment 
that will be extended by future calls to withdraw.
```

* In fact, assignment statements already had a dual role: they either created new bindings or re-bound existing names. The many roles of Python assignment can obscure the effects of executing an assignment statement. It is up to you as a programmer to document your code clearly so that the effects of assignment can be understood by others.

### The Benefits of Non-Local Assignment

*  Non-local assignment has given us the ability to maintain some state that is local to a function, but evolves over successive calls to that function.

### A functional implemention of list