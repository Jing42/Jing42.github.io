---
layout: post
title: SICP1 Building Abstractions with Objects
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

