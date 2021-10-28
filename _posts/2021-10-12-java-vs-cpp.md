---
title: "Post: C++ vs Java"
categories:
  - Post
tags:
  - Programming Language
excerpt: "A comparison of C++ and Java"
---

In this post I will try to holistically compare the differences between Java and C++.
I will comparison of the programming languages will be split concrete differences and
subjective differences.

## Introduction

Before we can dive into the differences of the two programming languages, we need to
first understand the different motivations of them. In my opinion, Java is a programming
language that designed for reliable production system. However, C++ is designed for highly
optimized systems that could be deployed to production. That is why you will often hear
the saying "don't pay for what you don't use". We can better understand these design
trade-offs in the following sections.

## Architecture

* Java runs on a virtual machine (JVM) that will execute code in a protected environment.
* C++
  * C++ code is compiled into machine code and executed on the actual machine.

### Life-cycle

The life cycle of a C++ program is split into **compile time**, **link time** and **run time**
while a Java program will only have **compile time** and **run time**. The presence of **link time**
in a C++ program allows developers to choose if they would like to link external dependencies **dynamically**
or **statically**.

Link Time pros and cons
* Pro
  * Allow developer to choose to share libraries executable across process or have them compiled 
  for each application
* Con
  * DLL Hell

### Semantics

**Java Semantics**:

Technically, Java uses value semantics as all objects are passed by value. However as all objects are
allocated on the heap and object are stored and passed by references in the code, it implicitly becomes
**reference semantics**.


**C++ Semantics**: C++ supports the following semantics

* **Value semantics**: we can pass the object by value (copying of object)
* **Reference semantics**: we can pass the object by reference using pointer or pass-by-reference
* **Move semantics**: we can pass the object by moving it to the function.
  * C++ has the idea of rvalue reference that allows us to separate the reference of an object that
  is permanent or temporary. rvalue reference can occur when creating a temporary object
  `foo("foo" + "bar")`) or explicitly moved with `std::move(...)`.


### Typing

Both Java and C++ are statically typed languages but they differ significantly in the way
they handle types.

**Java generics**:

* Java handle generics using polymorphism.
* By default all generic types are assumed to be an object.
* Developer are able to limit the types to be derived from a certain base class
to allow for the base class methods to be used by the generic type
* Under the hood, Java will erase the derived type into the base class type at runtime.
This method is called **erasure**.

**C++ templates**:

* Handles generic types by having a template that can be called.
* Unlike Java, function template are not actual function.
* The compiler will synthesize a specific function for that specific type at runtime with the function template.
* Analogous to the function template being a **cookie cutter** and specialising of the function template
by the client as the **cookie dough**.
* The compiler will use the template to check the required signature for the type at compile
time.
* Compiler will expand the template at compile time and perform type checking. Similar to macro expansion.
* Pros:
  * Non-intrusive generics: do not need to change the inheritance structure of the type
  * Compile time: Allow for compile time data structures. Immutable and faster at runtime



## Memory Management

### C++ Memory Management

C++ allows objects to be instantiated on the
* Stack: normal instantiation
  * Benefits: allow for RAII
    * Resource management, add bench marks and etc
* Heap: using `new/malloc`
  * Users have to call `delete` or `delete[]` (array operator)
  * Pros:
    * Instantly free up the memory
  * Cons:
    * Very easy to get memory leak
      * Return or exception before delete statement
    * Access a pointer after being deleted

#### Smart Pointer

* `unique_ptr`: Allow for exclusive ownership of a resource. Resource is freed once the object goes out of scope
* `shared_ptr`:
  * Allow for multiple ownership of a resource. Resource is freed once all the reference has been removed
  * Uses a reference counter to keep track of the number of reference. Additional overhead.
  * Does not break cyclic reference (Unlike Java GC).

### Java Memory Management

In Java, memory is managed through a garbage collector. Java garbage collector (GC) is actually a daemon
thread that runs in the background to the actual application. The main goal of the GC
is to ensure that the heap is fully utilised to prevent the application from running out
of memory.

**How GC works**
* **Heap**: The heap is dived into 4 main components, Eden, Survivor 1, Survivor 2 and Tenure.
  * At any point in time, Survivor 1 or Survivor 2 needs to be empty.
* **Object instantiation**: When all objects are instantiated, they will be allocated into the
Eden region of the heap.
* **When the heaps gets full**, GC will perform three main steps.
  * **Mark**:
    * GC will traverse all objects mark the visited objects
  * **Sweep**:
    * After marking all the objects, the GC will destroy any object that has not been marked (solve cyclic reference problem)
  * **Compact**:
    * GC will compact the object by moving them to the next region (Eden -> Survivor) and place them 
    contigously to prevent any gaps.
  * **Why segment region**: Allow GC to perform mark sweep and compact on only
  new/semi-old objects. ie if only eden is full then mark-sweep-compact

**Trades Offs**:
* Pros:
  * **Lesser memory leaks**: Only happens when an application holds a reference to an object
  longer than what is needed.
* Cons:
  * **High overhead**: will require additional computation resource to run the GC thread 
  * **No guarantee object destruction**: GC will only perform destruction when a region
  in the heap runs out of memory.
    * Techniques such as using RAII to implement a timer cannot be achieved

## Programming Paradigms

Both Java and C++ supports the major programming paradigms such as procedural,
objected oriented programming (big difference in approach) and functional
programming. However, C++ supports an additional paradigm called
**meta programming**.

### Object Oriented Programming

#### Polymorphism

* Presence/Lack of **Interface**: In Java polymorphism can be achieved by either
extending a base class or implementing an **interface**. However, in C++ developer
can only extend a base class and developers achieve **interface** like function by
declaring an **absolute virtual function**.
* **Virtual function**: In Java all functions are virtual. This means that all child
instances would execute the overridden function. However, in C++ virtual functions
have to be declared on the parent class.
  * Under the hood, declaring all functions as virtual would mean that every instance
  will need to have a virtual pointer (`vptr`) that points to a virtual table (`vtbl`)
  that actual object belongs to (base class/derived class). `vtbl` will store the actual
  implementation for the overridden function.
  * Pros of C++:
    * Simple structs will not have the overhead of storing `vptr` and `vtbl`. A point struct
    with `int x` and `int y` could be able to fit into a 64 bit memory allocation (easily pass to C) 
  * Cons of C++:
    * Error prone: developers might forget to declare some important function as virtual (ie: destructor)
    which would result in undesired behaviour.
* **Location of object**: In C++ developers could either choose to instantiate the object on the
  **heap or the stack** while Java developers can only instantiate the object on the **heap**.
