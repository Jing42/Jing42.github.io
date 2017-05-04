---
layout: post
title: SICP1 Building Abstractions with Functions
date: 2017-05-04
tag: Python
---   

## 介绍

我会在这系列文章里记录学习SICP(Python版)的体会~

### The Elements of Programming

* A programming language is more than just a means for instructing a computer to perform tasks. The language also serves as a framework within which we organize our ideas about processes. Programs serve to communicate those ideas among the members of a programming community. Thus, **programs must be written for people to read, and only incidentally for machines to execute**.

* Choosing Names: Never use "l" (lowercase ell), "O" (capital oh), or "I" (capital i) to avoid confusion with numerals.

* A user should not need to know how the function is implemented in order to use it.

### Docstrings

When you call help with the name of a function as an argument, you see its docstring. The first line describes the job of the function in one line. The following lines can describe arguments and clarify the behavior of the function

```
def pressure(v, t, n):
    """Compute the pressure in pascals of an ideal gas.

    Applies the ideal gas law: http://en.wikipedia.org/wiki/Ideal_gas_law

    v -- volume of gas, in cubic meters
    t -- absolute temperature in degrees kelvin
    n -- particles of gas
    """
    k = 1.38e-23  # Boltzmann's constant
    return n * k * t / v
```

### Default Argument Values

As a guideline, most data values used in a function's body should be expressed as default values to named arguments, so that they are easy to inspect and can be changed by the function caller. Some values that never change, like the fundamental constant k_b, can be defined in the global frame.
    

### Higher-Order Functions

One of the things we should demand from a powerful programming language is the ability to build abstractions by assigning names to common patterns and then to work in terms of the abstractions directly.

Functions that manipulate functions are called higher-order functions.

```
def summation(n, term, next):
    total, k = 0, 1
    while k <= n:
        total, k = total + term(k), next(k)
    return total

def identity(k):
    return k

def successor(k):
    return k + 1

def sum_naturals(n):
    return summation(n, identity, successor)

def pi_term(k):
    denominator = k * (k + 2)
    return 8 / denominator

def pi_next(k):
    return k + 4

def pi_sum(n):
    return summation(n, pi_term, pi_next)
```

Now, we can do all kinds of summations using one common function.

### Functions as General Methods

Some functions express general methods of computation, independent of the particular functions they call.

* Naming and functions allow us to abstract away a vast amount of complexity. 
* It is only by virtue of the fact that we have an extremely general evaluation procedure that small components can be composed into complex processes. 

```
def iter_improve(update, test, guess=1):
    while not test(guess):
        guiss = update(guess)

def near(x, f, g):
    return approx_eq(f(x), g(x))

def approx_eq(x, y, tolerance=1e-5):
    return abs(x - y) < tolerance

def golden_update(guess):
    return 1/guess + 1

def golden_test(guess):
    return near(guess, square, successor)

iter_improve(golden_update, golden_test)
```

### Nested Definitions

```
def square_root(x):
    def update(guess):
        return average(guess, x/guess)
    def test(guess):
        return approx_eq(square(guess), x)
    return iter_improve(update, test)
```

The update function carries with it some data: the values referenced in the environment in which it was defined. Because they enclose information in this way, locally defined functions are often called ##closures## .

### Functions as Returned Values

An important feature of lexically scoped programming languages is that locally defined functions keep their associated environment when they are returned. 

### Lambda Expressions

```
def compose1(f, g):
    return lambda x: f(g(x))
```

### First-Class Element

>* 1. They may be bound to names.
>* 2. They may be passed as arguments to functions.
>* 3. They may be returned as the results of functions.
>* 4. They may be included in data structures.