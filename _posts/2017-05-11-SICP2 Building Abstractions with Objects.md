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

```
def make_mutable_rlist():
    """Return a functional implementation of a mutable recursive lsit."""
    contents = empty_rlist
    def dispatch(message, value=None):
        nonlocal contents
        if message == 'len':
            return len_rlist(contents)
        elif message = 'getitem':
            return getitem_rlist(contents, value)
        elif message == 'push_first':
            contents = make_rlist(value, contents)
        elif message == 'pop_first':
            f = first(contents)
            contents = rest(contents)
            return f
        elif message == 'str':
            return str(contents)
    return dispatch

def to_mutable_rlist(source):
    """Return a functional list with the same contents as source."""
    s = make_mutable_rlist()
    for element in reversed(source):
        s('push_first', element)
    return s
```

### Dictionaries

* The methods keys, values and items all return iterable values.

* A list of key-value pairs can be converted into a dictionary by calling the dict constructor function. dict([(3, 9), (4, 16), (5, 25)])

```
def make_dict():
    """Return a functional implementation of a dictionary."""
    records = []
    def getitem(key):
        for k, v in records:
            if k == key:
                return v
    def setitem(key, value):
        for item in records:
            if item[0] == key:
                item[1] = value
                return
        records.append([key, value])
    def dispatch(message, key=None, value=None):
        if message == 'getitem':
            return getitem(key)
        elif message = 'setitem':
            setitem(key, value)
        elif message = 'keys':
            return tuple(k for k, _ in records)
        elif message == 'values':
            return tuple(v for _, v in records)
    return dispatch
```

### Example: Propagating Constraints (A constraint-based system)

* Expressing programs as constraints is a type of declarative programming, in which a programmer declares the structure of a problem to be solved, but abstracts away the details of exactly how the solution to the problem is computed.

* Computation by such a network proceeds as follows: When a connector is given a value (by the user or by a constraint box to which it is linked), it awakens all of its associated constraints (except for the constraint that just awakened it) to inform them that it has a value. Each awakened constraint box then polls its connectors to see if there is enough information to determine a value for a connector. If so, the box sets that connector, which then awakens all of its associated constraints, and so on.

* The relationship between Fahrenheit and Celsius temperatures: 9 * c = 5 * (f - 32)

```
celsius = make_connector('Celsius')
fahrenheit = make_connector('Fahrenheit')

def make_converter(c, f)
    """Connect c to f with contraints to convert from Celsius to Fahrenheit."""
    u, v, w, x, y = [make_connector() for _ in range(5)]
    multiplier(c, w, u)
    multiplier(v, x, u)
    addre( v, y, f)
    constant(w, 9)
    constant(x, 5)
    constant(y, 32)

make_converter(celsius, fahrenheit)
```

* This non-directionality of computation is the distinguishing feature of constraint-based systems.

```
from operator import add, sub
    def adder(a, b, c):
        """The constraint that a + b = c."""
        return make_ternary_contraint(a, b, c, add, sub, sub)
def make_ternary_contraint(a, b, c, ab, ca, cb):
    """The connstraint that ab(a,b)=c and ca(c,a)=b and cb(c,b)=a."""
    def new_value():
        av, bv, cv = [connector['has_val']() for connector in (a, b, c)]
        if av and bv:
            c['set_val'](contraint, ab(a['val'], b['val']))
        elif av and cv:
            b['set_val'](contraint, ca(c['val'], b['val']))
        elif bv and cv:
            a['set_val'](contraint, cb(c['val'], b['val']))
    def forget_value():
        for connector in (a, b, c):
            connector['forget'](contraint)
    contraint = {'new_val': new_value, 'forget': forget_value}
    for connector in (a, b, c):
        connector['connect'](contraint)
    return constraint
```

```
from operator import mul, truediv
def multiplier(a, b, c):
    """The constraint that a * b = c."""
    return make_ternary_constraint(a, b, c, mul, truediv, truediv)

def constant(connector, value):
    """The constraint that connector = value."""
    constaint = {}
    connector['set_val'](constraint, value)
    return constraint


def make_connector(name=None):
    """A connector between constraints."""
    informant = None
    constraints = []
    def set_value(source, value):
        nonlocal informant
        val = connector['val']
        if val is None:
            informant, connector['val'] = source, value
            if name is not None:
                print(name, '=', value)
            inform_all_except(source, 'new_val', constraints)
        else:
            if val != value":
                print('Contradiction detected:', val, 'vs', value)
    def forget_value(source):
        nonlocal informant
        if informant == source:
            informant, connector['val'] = None, None
            if name is not None:
                print(name, 'is forgotten')
            inform_all_except(source, 'forget', constraints)
    connector = {'val': None,
                'set_val': set_value,
                'forget': forget_value,
                'has_val': lambda: connector['val'] is not None,
                'connect': lambda source: constraints.append(source)}
    return connector


def inform_all_except(source, message, constraints):
    """Inform all constraints of the message, except source."""
    for c in constraints:
        if c != source:
            c[message]()           
```

### OOP

```
class Account(object):
    def __init__(self, account_holder):
        self.balance = 0
        self.holder = account_holder
    def deposit(self, amount):
        self.balance = self.balance + amount
        return self.balance
    def withdraw(self, amount):
        if amount > self.balance:
            return 'Insufficient funds'
        self.balance -= amount
        return self.balance
```

### Message Passing and Dot Expressions

* Dot notation is a syntactic feature of Python that formalizes the message passing metaphor. 

* The built-in function getattr also returns an attribute for an object by name. It is the function equivalent of dot notation. Using getattr, we can look up an attribute using a string, just as we did with a dispatch dictionary. getattr(tom_account, 'balance')

* We can also test whether an object has a named attribute with hasattr. hasattr(tom_account, 'deposit')

* To achieve automatic self binding, Python distinguishes between functions, which we have been creating since the beginning of the course, and bound methods, which couple together a function and the object on which that method will be invoked. A bound method value is already associated with its first argument, the instance on which it was invoked, which will be named self when the method is called.


### Class Attributes

* Class attributes are created by assignment statements in the suite of a class statement, outside of any method definition.

* A single assignment statement to a class attribute changes the value of the attribute for all instances of the class.

* All assignment statements that contain a dot expression on their left-hand side affect attributes for the object of that dot expression. If the object is an instance, then assignment sets an instance attribute. If the object is a class, then assignment sets a class attribute. As a consequence of this rule, assignment to an attribute of an object cannot affect the attributes of its class. 

### Inheritance

```
class CheckingAccount(Account):
        """A bank account that charges for withdrawals."""
        withdraw_charge = 1
        interest = 0.01
        def withdraw(self, amount):
            return Account.withdraw(self, amount + self.withdraw_charge)
```

* The class of an object stays constant throughout. Even though the deposit method was found in the Account class, deposit is called with self bound to an instance of CheckingAccount, not of Account.

* Calling ancestors. Attributes that have been overridden are still accessible via class objects. For instance, we implemented the withdraw method of CheckingAccount by calling the withdraw method of Account with an argument that included the withdraw_charge.

* Notice that we called self.withdraw_charge rather than the equivalent CheckingAccount.withdraw_charge. The benefit of the former over the latter is that a class that inherits from CheckingAccount might override the withdrawal charge. If that is the case, we would like our implementation of withdraw to find that new value instead of the old one.

### The Role of Objects

* On the other hand, classes may not provide the best mechanism for implementing certain abstractions. Functional abstractions provide a more natural metaphor for representing relationships between inputs and outputs. One should not feel compelled to fit every bit of logic in a program within a class, especially when defining independent functions for manipulating data is more natural. Functions can also enforce a separation of concerns.

* Multi-paradigm languages like Python allow programmers to match organizational paradigms to appropriate problems. Learning to identify when to introduce a new class, as opposed to a new function, in order to simplify or modularize a program, is an important design skill in software engineering that deserves careful attention.

### Implementing Classes and Objects

* Using the object metaphor does not require a special programming language. Programs can be object-oriented, even in programming languages that do not have a built-in object system. 

* As we have seen previously in this chapter, dictionaries themselves are abstract data types. We implemented dictionaries with lists, we implemented lists with pairs, and we implemented pairs with functions. As we implement an object system in terms of dictionaries, keep in mind that we could just as well be implementing objects using functions alone.

### Instances

```
def make_instance(cls):
    """Return a new object instance, which is a dispatch dictionary."""
    def get_value(name):
        if name in attributes:
            return attributes[name]
        else:
            value = cls['get'](name)
            return bind_method(value, instance)
    def set_value(name, value):
        attributes[name] = value
    attributes = {}
    instance = {'get': get_value, 'set': set_value}
    return instance


def bind_method(value, instance):
    """Return a bound method if value is callable, or value otherwise."""
    if callable(value):
        def method(*args):
            return value(instance, *args)
        return method
    else:
        return value
```

### Classes

```
def make_class(attributes, base_class=None):
    """Return a new class, which is a dispatch dictionary."""
    def get_value(name):
        if name in attributes:
            return attributes[name]
        elif base_class is not None:
            return base_class['get'](name)
    def set_value(name, value):
        attributes[name] = value
    def new(*args):
        return init_instance(cls, *args)
    cls = {'get': get_value, 'set': set_value, 'new': new}
    return cls


def init_instance(cls, *args):
    """Return a new object with type cls, initialized with args."""
    instance = make_instance(cls)
    init = cls['get']('__init__')
    if init:
        init(instance, *args)
    return instance
```

### Using Implemented Objects

```
def make_account_class():
    """Return the Account class, which has deposit and withdraw methods."""
    def __init__(self, account_holder):
        self['set']('holder', account_holder)
        self['set']('balance', 0)
    def deposit(self, amount):
        """Increase the account balance by amount and return the new balance."""
        new_balance = self['get']('balance') + amount
        self['set']('balance', new_balance)
        return self['get']('balance')
    def withdraw(self, amount):
        """Decrease the account balance by amount and return the new balance."""
        balance = self['get']('balance')
        if amount > balance:
            return 'Insufficient funds'
        self['set']('balance', balance-amount)
        return self['get']('balance')
    return amke_class({'__init__': __init__,
                       'deposit': deposit,
                       'withdraw': withdraw,
                       'interest': 0.02})


Account = make_account_class()
jim_acct = Account['new']('Jim')
jim_acct['get']('holder')
jim_acct['get']('interest')
jim_acct['get']('deposit')(20)
jim_acct['get']('withdraw')(5)


def make_checking_account_class():
    """Return the CheckingAccount class, which imposes a $1 withdrawal fee."""
    def withdraw(self, amount):
        return Account['get']('withdraw')(self, amount + 1)
    return make_class({'withdraw': withdraw, 'interest': 0.01}, Account)
```

### Generic Operations: In this section, we explore alternate methods for combining and manipulating objects of different types.

### String Conversion

* repr(object) -> string Return the canonical string representation of the object. For most object types, eval(repr(object)) == object.

* The result of calling repr on the value of an expression is what Python prints in an interactive session.

* Message passing provides an elegant solution in this case: the repr function invokes a method called __repr__ on its argument. The str constructor is implemented in a similar manner: it invokes a method called __str__ on its argument.

### Multiple Representations

```
def add_complex(z1, z2):
    return ComplexRI(z1.real + z2.real, z1.imag + z2.imag)

def mul_complex(z1, z2):
    return ComplexMA(z1.magnitude * z2.magnitude, z1.angle + z2.angle)
```

* Conceptually, an ADT describes a full representational abstraction of some kind of thing, whereas an interface specifies a set of behaviors that may be shared across many things.

* Python has a simple feature for computing attributes on the fly from zero-argument functions. The @property decorator allows functions to be called without the standard call expression syntax. 

```
from math import atan2
class ComplexRI(object):
    def __init__(self, real, imag):
        self.real = real
        self.imag = imag
    @property
    def magnitude(self):
        return (self.real ** 2 + self.imag ** 2) ** 0.5
    @property
    def angle(self):
        return atan2(self.imag, self.real)
    def __repr__(self):
        return 'ComplexRI({0}, {1}'.format(self.real, self.imag)
```

* A second implementation using magnitude and angle provides the same interface because it responds to the same set of messages.

```
from math import sin, cos
class ComplexMA(object):
    def __init__(slef, magnitude, angle):
        self.magnitude = magnitude
        self.angle = angle
    @property
    def real(self):
        return self.magnitude * cos(self.angle)
    @property
    def imag(self):
        return self.agnitude * sin(self.angle)
    def __repr(self):
        return 'ComplexMA({0}, {1})'.format(self.magnitude, self.angle)
```

* In fact, our implementations of add_complex and mul_complex are now complete; either class of complex number can be used for either argument in either complex arithmetic function. It is worth noting that the object system does not explicitly connect the two complex types in any way (e.g., through inheritance). We have implemented the complex number abstraction by sharing a common set of messages, an interface, across the two classes.

```
from math import pi
add_complex(ComplexRI(1, 2), complexMA(2, pi/2))
mul_complex(ComplexRI(0, 1), complexRI(0, 1))
```

* The class for each representation can be developed separately; they must only agree on the names of the attributes they share.

```
ComplexRI.__add__ = lambda self, other: add_complex(self, other)
ComplexMA.__add__ = lambda self, other: add_complex(self, other)
ComplexRI.__mul__ = lambda self, other: mul_complex(self, other)
ComplexMA.__mul__ = lambda self, other: mul_complex(self, other)
```

### Generic Functions

* Define operations that are generic over different kinds of arguments that do not share a common interface.

```
from fractions import gcd
class Rational(object): 
    def __init__(self, numer, denom):
        g = gcd(numer, denom)
        self.numer = numer // g
        self.denom = denom // g
    def __repr__(self):
        return 'Rational({0}, {1})'.format(self.numer, self.denom)

def add_rational(x, y):
    nx, dx = x.numer, x.denom
    ny, dy = y.numer, y.denom
    return Rational(nx * dy + ny * dx, dx * dy)

def mul_rational(x, y):
    return Rational(x.numer * y.numer, x.denom * y.denom)

def add(z1, z2):
    """Add z1 and z2, which may be complex or rational."""
    if iscomplex(z1) and iscomplex(z2):
        return add_complex(z1, z2)
    elif iscomplex(z1) and isrational(z2):
        return add_complex_and_rational(z1, z2)
    elif isrational(z1) and iscomplex(z2):
        return add_complex_and_rational(z2, z1)
    else:
        return add_rational(z1, z2)


def type_tag(x):
    return type_tag.tags[type(x)]

type_tag.tags = {ComplexRI: 'com', ComplexMA: 'com', Rational: 'rat'}

def add(z1, z2):
    types = (type_tag(z1), type_tag(z2))
    return add.implementations[types](z1, z2)

add.implementations = {}
add.implementations[('com', 'com')] = add_complex
add.implementations[('com', 'rat')] = add_complex_and_rational
add.implementations[('rat', 'com')] = lambda x, y: add_complex_and_rational(y, x)
add.implementations[('rat', 'rat')] = add_rational
```

* A more general version of generic arithmetic would apply arbitrary operators to arbitrary types and use a dictionary to store implementations of various combinations. This fully generic approach to implementing methods is called data-directed programming. In our case, we can implement both generic addition and multiplication without redundant logic.

### Coerce

```
def rational_to_complex(x):
    return ComplexRI(x.numer/x.denom, 0)

coercions = {('rat', 'com'): rational_to_complex}

def coerce_apply(operator_name, x, y):
    tx, ty = type_tag(x), type_tag(y)
    if tx != ty:
        if (tx, ty) in coercions:
            tx, x = ty, coercions[(tx, ty)](x)
        elif (ty, tx) in coercions:
            ty, y = tx, coercions[(ty, tx)](y)
        else:
            return 'No coercion possible.'
    key = (operator_name, tx)
    return coerce_apply.implementations[key](x, y)


coerce_apply.implementations = {('mul', 'com'): mul_complex,
                                ('mul', 'rat'): mul_rational,
                                ('add', 'com'): add_complex,
                                ('add', 'rat'): add_rational}
```