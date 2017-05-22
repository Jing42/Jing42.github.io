---
layout: post
title: SICP4 Distributed and Parallel Computing
date: 2017-05-22
tag: Python
---   

## 介绍

In this chapter, we turn to the problem of coordinating multiple computers and processors.

## Distributed Computing

* A distributed system is a network of autonomous computers that communicate with each other in order to achieve a goal.

* The computers in a distributed system are independent and do not physically share memory or processors. They communicate with each other using messages, pieces of information transferred from one computer to another over a network.

* There are two predominant ways of organizing computers in a distributed system. The first is the client-server architecture, and the second is the peer-to-peer architecture.

### Client/Server Systems

* The concepts of client and server are powerful functional abstractions. A server is simply a unit that provides a service, possibly to multiple clients simultaneously, and a client is a unit that consumes the service. The clients do not need to know the details of how the service is provided, or how the data they are receiving is stored or calculated, and the server does not need to know how the data is going to be used.

* Even systems on a single machine can have client/server architectures.

### Peer-to-peer Systems

* The term peer-to-peer is used to describe distributed systems in which labor is divided among all the components of the system. All the computers send and receive data, and they all contribute some processing power and memory.

* Division of labor among all participants is the identifying characteristic of a peer-to-peer system.

* In some peer-to-peer systems, the job of maintaining the health of the network is taken on by a set of specialized components.

* In some peer-to-peer systems, the job of maintaining the health of the network is taken on by a set of specialized components.

### Modularity

* Modularity is the idea that the components of a system should be black boxes with respect to each other. It should not matter how a component implements its behavior, as long as it upholds an interface: a specification for what outputs will result from inputs.

### Message Passing

* Different components (dispatch functions or computers) exchange them in order to achieve a goal that requires coordinating multiple modular components.

* A message protocol is a set of rules for encoding and decoding messages.

### Messages on the World Wide Web

* All web browsers use the HTTP format to request pages from a web server, and all web servers use the HTTP format to send back their responses.

* A fixed set of response codes is a common feature of a message protocol. Designers of protocols attempt to anticipate common messages that will be sent via the protocol and assign fixed codes to reduce transmission size and establish a common message semantics.

## Parallel Computing

* Distributed Computing

* To circumvent physical and mechanical constraints on individual processor speed, manufacturers are turning to another solution: multiple processors. If two, or three, or more processors are available, then many programs can be executed more quickly. While one processor is doing one aspect of some computation, others can work on another. All of them can share the same data, but the work will proceed in parallel.

* The role of a processor in computation is to carry out the evaluation and execution rules of a programming language. In a shared memory model, different processes may execute different statements, but any statement can affect the shared environment.

### The Problem with Shared State

* parallelizing code is not as easy as dividing up the lines between multiple processors and having them be executed. The order in which variables are read and written matters.

### Correctness in Parallel Computation

* There are two criteria for correctness in parallel computation environments. The first is that the outcome should always be the same. The second is that the outcome should be the same as if the code was executed in serial.

* Problems arise in parallel computation when one process influences another during critical sections of a program. These are sections of code that need to be executed as if they were a single instruction, but are actually made up of smaller statements.

* In order to behave correctly in concurrent situations, the critical sections in a programs code need to be have atomicity -- a guarantee that they will not be interrupted by any other code.

### Protecting Shared State: Locks and Semaphores

* All the methods for synchronization and serialization that we will discuss in this section use the same underlying idea. They use variables in shared state as signals that all the processes understand and respect.

* there is really nothing protecting those shared variables except that the processes are programmed only to access them when a particular signal indicates that it is their turn.

* Locks, also known as mutexes (short for mutual exclusions), are shared objects that are commonly used to signal that shared state is being read or modified.

* For a lock to protect a particular set of variables, all the processes need to be programmed to follow a rule: no process will access any of the shared variables unless it owns that particular lock.

```
from threading import Lock
def make_withdraw(balance):
    balance_lock = Lock()
    def withdraw(amount):
        nonlocal balance
        # try to acquire the lock
        balance_lock.acquire()
        # noce successful, enter the critical section
        if amount > balance:
            print("Insufficient funds")
        else:
            balance -= amount
            print(balance)
        # upon exiting the critical section, release the lock
        balance_lock.release()
```

* Forgetting to release acquired locks is a common error in parallel programming.

* Semaphores are signals used to protect access to limited resources. They are similar to locks, except that they can be acquired multiple times up to a limit. 

* For example, suppose there are many processes that need to read data from a central database server. The server may crash if too many processes access it at once, so it is a good idea to limit the number of connections.

```
from threading import Semaphore
db_semaphore = Semaphore(2) # set up the semaphore
database = []
def insert(data):
    db_semaphore.acquire() # try to acquire the semaphore
    database.append(data)  # if successful, proceed
    db_semaphore.release() # release the semaphore
```

* A semaphore with value 1 behaves like a lock.

### Staying Synchronized: Condition variables

* Condition variables are useful when a parallel computation is composed of a series of steps. A process can use a condition variable to signal it has finished its particular step. Then, the other processes that were waiting for the signal can start their work.

* Condition variables are objects that act as signals that a condition has been satisfied. They are commonly used to coordinate processes that need to wait for something to happen before continuing.

```
step1_finished = 0
start_step2 = Condition()

def do_step_1(index):
    A[index] = B[index] + C[index]
    # access the shared state that determines the condition status
    start_step2.acquire()
    step1_finished += 1
    if(step1_finished == 2): # if the condition is met
        start_step2.notifyAll() # send the signal
    # release access to shared state
    start_step2.release()

def do_step_2(index):
    # wait for the condition
    start_step2.wait()
    V[index] = M[index] * A
```

### Deadlock

* While synchronization methods are effective for protecting shared state, they come with a catch. Because they cause processes to wait on each other, they are vulnerable to deadlock, a situation in which two or more processes are stuck, waiting for each other to finish.

* The source of deadlock is a circular wait.

```
x_lock = Lock()
y_lock = Lock()
x = 1
y = 0
def compute():
    x_lock.acquire()
    y_lock.acquire()
    y = x + y
    x = x * x
    y_lock.release()
    x_lock.release()

def anti_compute():
    y_lock.acquire()
    x_lock.acquire()
    y = y - x
    x = sqrt(x)
    x_lock.release()
    y_lock.release()

P1                                p2
acquire x_lock: ok                acquire y_lock: ok
acquire y_lock: wai               acquire x_lock: wait
wait                              wait    
wait                              wait
wait                              wait
...                               ...
```