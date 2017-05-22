---
layout: post
title: SICP3 The Structure and Interpretation of Computer Programs
date: 2017-05-20
tag: Python
---   

## Introduction

* It is no exaggeration to regard this as the most fundamental idea in programming, that an interpreter, which determines the meaning of expressions in a programming language, is just another program.

* In this chapter, we study the design of interpreters and the computational processes that they create when executing programs.

* Typical interpreters have an elegant common structure: two mutually recursive functions. The first evaluates expressions in environments; the second applies functions to arguments.

### Recursive Functions

* The state of the computation is entirely contained within the structure of the expression tree.

* Recursive functions can rely more heavily on the interpreter itself, by storing the state of the computation as part of the expression tree and environment, rather than explicitly using names in the local frame.

```
def pig_latin(w):
    """Return the Pig Latin equivalent of English word w."""
    if starts_with_a_vowel(w):
        return w + 'ay'
    return pig_latin(w[1:] + w[0]) 
```

### Recursive leap of faith

* While we can unwind the recursion using our model of computation, it is often clearer to think about recursive calls as functional abstractions. That is, we should not care about how fact(n-1) is implemented in the body of fact; we should simply trust that it computes the factorial of n-1.

* Treating a recursive call as a functional abstraction has been called a recursive leap of faith. 

### Memoization

```
def fib(n):
    if n == 1:
        return 0
    if n == 2:
        return 1
    return fib(n-2) + fib(n-1)

def memo(f):
    """Return a memoized version of single-argument function f."""
    cache = {}
    def memoized(n):
        if n not in cache:
            cache[n] = f(n)
        return cache[n]
    return memoized


fib = memo(fib)
fib(40)
```

* Memoization exemplifies a common pattern in programming that computation time can often be decreased at the expense of increased use of space, or vis versa.

### Example: Counting Change

```
def count_change(a, kinds=(50, 25, 10, 5, 1)):
    """Return the number of ways to change amount a using coin kinds."""
    if a == 0:
        return 1
    if a < 0 or len(kinds) == 0:
        return 0
    d = kinds[0]
    return count_change(a, kinds[1:]) + count_change(a - d, kinds)
```

### Processing Recursive Lists

```
class Rlist(object):
    """A recursive list consisting of a first element and the rest."""
    class EmptyList(object):
        def __len__(self):
            return 0
    empty = EmptyList()
    def __init__(self, first, rest=empty):
        self.first = first
        self.rest = rest
    def __repr__(self):
        args = repr(self.first)
        if self.rest is not Rlist.empty:
            args += ', {0}'.format(repr(self.rest))
        return 'Rlist({0})'.format(args)
    def __len__(self):
        return 1 + len(self.rest)
    def __getitem__(self.rest)
        if i == 0:
            return self.first
        return self.rest[i-1]

def extend_rlist(s1, s2):
    if s1 is Rlist.empty:
        return s2
    return Rlist(s1.first, extend_rlist(s1.rest), s2)

def map_rlist(s, fn):
    if s is Rlist.empty:
        return s
    return Rlist(fn(s.first), map_rlist(s.rest, fn))

def filter_rlist(s, fn):
    if s is Rlist.empty:
        return s
    rest = filter_rlist(s.rest, fn)
    if fn(s.first):
        return Rlist(s.first, rest)
    return rest
```

* Recursive lists are taken apart and constructed incrementally as a consequence of function application. As a result, they have linear orders of growth in both the number of steps and space required.

* This connection between nested expressions and tree-structured data type plays a central role in our discussion of designing interpreters later in this chapter.

### Sets

* As with any data abstraction, we are free to implement any functions over any representation of sets that provides this collection of behaviors.

* Set as unordered sequences.

```
def empty(s):
    return s is Rlist.empty

# θ(n)
def set_contains(s, v):
    """Return True if and only if set s contains v."""
    if empty(s):
        return False
    elif s.first == v:
        return True
    return set_contains(s.rest, v)

# θ(n)
def adjoin_set(s, v):
    """Return a set containing all elements of s and element v."""
    if set_contains(s, v):
        return s
    return Rlist(v, s)

# θ(n^2)
def intersect_set(set1, set2):
    """Return a set containing all elements common to set1 and set2."""
    return filter_rlist(set1, lambda v: set_contains(set2, v))

# θ(n^2)
def union_set(set1, set2):
    """Return a set containing all elements either in set1 or set2."""
    set1_not_set2  filter_rlist(set1, lambda v: not set_contains(set2, v))
    return extend_rlist(set1_not_set2, set2)
```

* Sets as ordered tuples: We will represent a set of numbers by listing its elements in increasing order.

```
# θ(n)
def set_contains(s, v):
    if empty(s) or s.first > v:
        return False
    elif s.first == v:
        return True
    return set_contains(s.rest, v)

# θ(n)
def intersect_set(set1, set2):
    if empty(set1) or empty(set2):
        return Rlist.empty
    e1, e2 = set1.first, set2.first
    elif e1 == e2:
        return Rlist(e1, intersect_set(set1.rest, set2.rest))
    elif e1 < e2:
        return intersect_set(set1.rest, set2)
    elif e2 < e1:
        return intersect_set(set1, set2.rest)
```

* Sets as binary trees: The entry of the root of the tree holds one element of the set. The entries within the left branch include all elements smaller than the one at the root. Entries in the right branch include all elements greater than the one at the root.

* Python also includes a built-in immutable frozenset class that shares methods with the set class but excludes mutation methods and operators.

### Exceptions 

* In any case, programmers must make conscious choices about how their programs should react to exceptional conditions.

* Exception objects themselves carry attributes。

### Interpreters for Languages with Combination

* The software running on any modern computer is written in a variety of programming languages.

* There are physical languages, such as the machine languages for particular computers. These languages are concerned with the representation of data and control in terms of individual bits of storage and primitive machine instructions.

* An interpreter for a programming language is a function that, when applied to an expression of the language, performs the actions required to evaluate that expression.

### Calculator 

* We will implement an interpreter for Calculator in Python. That is, we will write a Python program that takes a string as input and either returns the result of evaluating that string if it is a well-formed Calculator expression or raises an appropriate exception if it is not. The core of the interpreter for the Calculator language is a recursive function called calc_eval that evaluates a tree-structured expression object.

### Expression trees

```
class Exp(object):
    """A call expression in Calculator."""
    def __init__(self, operator, operands):
        self.operator = operator
        self.operands = operands
    def __reps(self):
        return 'Exp({0}, {1})'.format(repr(self.operator), repr(self.operands))
    def __str__(self):
        operands_strs = ', '.join(map(str, self.operands))
        return '{0}({1})'.format(self.operator, operand_strs)
```

### Evaluation

```
def calc_eval(exp):
    """Evaluate a Calculator expression."""
    if type(exp) in (int, float):
        return exp
    elif type(exp) == Exp:
        arguments = list(map(calc_eval, exp.operands))
        return calc_apply(exp.operator, arguments)
```

```
from operator import mul
from functools import reduce
def calc_apply(operator, args):
    """Apply the named operator to a list of args."""
    if operator in ('add', '+'):
        return sum(args)
    if operator in ('sub', '-'):
        if len(args) == 0:
            raise TypeError(operator + ' requires at least 1 argument')
        if len(args) == 1:
            return -args[0]
        return sum(args[:1] + [-arg for arg in args[1:]])
    if operator in ('mul', '*'):
        return reduce(mul, args, 1)
    if operator in ('div', '/'):
        if len(args) != 2:
            raise TypeError(operator + ' requires exactly 2 arguments')
        numer, denom = args
        return numer/denom
```

* Primitive expressions that do not require an additional evaluation step are called self-evaluating.

### Read-eval-print loops

* A typical approach to interacting with an interpreter.(Python interactive session)

```
def read_eval_print_loop():
    """Run a read-eval-print loop for calculator."""
    while True:
        try:
            expression_tree = calc_parse(input('calc> '))
            print(calc_eval(expression_tree))
        except (SyntaxError, TypeError, ZeroDivisionError) as err:
            print(type(err).__name__ + ':', err)
        except (KeyboardInterrupt, EOFError): # <Control>-D, etc.
            print('Calculation completed')
            return
```

### Parsing

* Parsing is the process of generating expression trees from raw text input. It is the job of the evaluation function to interpret those expression trees, but the parser must supply well-formed expression trees to the evaluator.

* A parser is in fact a composition of two components: a lexical analyzer and a syntactic analyzer. 

```
def calc_parse(line):
    """Parse a line of calculator input and return an expression tree."""
    tokens = tokenize(line)
    expression_tree = analyze(tokens)
    if len(tokens) > 0:
        raise SyntaxError('Extra token(s): ' + join(tokens))
    return expression_tree
```

### Lexical analysis

* The component that interprets a string as a token sequence is called a tokenizer or lexical analyzer. In our implementation, the tokenizer is a function called tokenize.

```
def tokenize(line):
    """Convert a string into a list of tokens."""
    spaced = line.replace('(', ' ( ').replace(')', ' ) ').replace(',', ' , ')
    return spaced.split()
```

* Languages with a more complicated syntax may require a more sophisticated tokenizer. In particular, many tokenizers resolve the syntactic type of each token returned. This classification can simplify the process of parsing the token sequence.

### Syntactic analysis

* The component that interprets a token sequence as an expression tree

```
def analyze(tokens):
    """Create a tree of nested lists from a sequence of tokens."""
    token = analyze_token(tokens.pop(0))
    if type(token) in (int, float):
        return token
    else:
        tokens.pop(0)   # Remove (
        return Exp(token, analyze_operands(tokens))

def analyze_operands(tokens):
    """Read a list of comma-separated operands."""
    operands = []
    while tokens[0] != ')':
        if operands:
            tokens.pop(0)   # Remove ,
        operands.append(analyze(tokens))
    tokens.pop(0) # Remove )
    return operands

def analyze_token(token):
    """Return the value of token if it can be analyzed as a number, or token."""
    try:
        return int(token)
    except (TypeError, ValueError):
        try:
            return float(token)
        except (TypeError, ValueError):
            return token
```

* Add error detection. The following revisions ensure that each step of the syntactic analysis finds the token it expects.

```
known_operators = ['add', 'sub', 'mul', 'div', '+', '-', '*', '/']

def analyze(tokens):
    """Create a tree of nested lists from a sequence of tokens."""
    assert_non_empty(tokens)
    token = analyze_token(tokens.pop(0))
    if type(token) in (int, float):
        return token
    if token in known_operators:
        if len(tokens) == 0 or tokens.pop(0) != '(':
            raise SyntaxError('expected ( after ' + token)
        return Exp(token, analyze_operands(tokens))
    else:
        raise SyntaxError('unexpected ' + token)

def analyze_operands(tokens):
    """Analyze a sequence of comma-separated operands."""
    assert_non_empty(tokens)
    operands = []
    while tokens[0] != ')':
        if operands and tokens.pop(0) != ',':
            raise SyntaxError('expected ,')
        operands.append(analyze(tokens))
        assert_non_empty(tokens)
    tokens.pop(0)   # Remove )
    return elements

def assert_non_empty(tokens):
    """Raise an exception if tokens is empty."""
    if len(tokens) == 0:
        raise SyntaxError('unexpected end of line')
```

* Informative syntax errors improve the usability of an interpreter substantially.

* These error strings also serve to document the definitions of these analysis functions.

### The Scheme Language

* Scheme demonstrates that a very small number of rules for forming expressions, with no restrictions on how they are composed, suffice to form a practical and efficient programming language that is flexible enough to support most of the major programming paradigms in use today.

### The Logo Language (code-as-data property)

* The central idea in Logo that accounts for its expressivity is that its built-in container type, the Logo sentence (also called a list), can easily store Logo source code! Logo programs can write and interpret Logo expressions as part of their evaluation process.

* The ability to represent code as data and later interpret it as part of the program is a defining feature of Lisp-style languages. The idea that a program can rewrite itself as it executes is a powerful one, and served as the foundation for early research in artificial intelligence (AI).

* Logo is a dynamically scoped language. A dynamically scoped language has the advantage that its procedures may not need to take as many arguments.

### Structure

* The parser actually does very little syntactic analysis. The parser performs so little analysis because the dynamic character of Logo requires that the evaluator resolve the structure of nested expressions.