---
title: "Post: Back to Basics: Initialization in C++ "
categories:
  - Post
tags:
  - C++
---

Notes from: https://www.youtube.com/watch?v=_23qmZtDBxg&list=PLqdbPkkHHWL7qB0wA7pePTTX4oI-xOsOu&index=8

## Initialization Rules from C

* Scalers: simple / primitive types.
* Aggregtes: objects made of smaller pieces - an aggregate of smaller types.
* C does not have any initialisation rules.

### Types of C initialization

**scalar initialization**:
```cpp
int x = 3;
```

**aggregate-initialisation**:
* initialize an aggregate with brace-enclosed list
```cpp
struct widget {
    int id;
    double price;
};
widget w1 = {1000, 6.5};
```
**copy-initialization**:
* initialise from another **compatible** type
* only applies to structs/union => cannot copy init an array

```cpp
struct widget {
    int id;
    double price;
};
widget w1 = {1000, 6.5};
widget w2 = w1;
```
**without an explicity initializer**:
* zero-initialized:
    * when static or thread local duration
    * initialized with value `0` for sclar, for aggregate eevery field is initialized with 0, pointer initialised to null
* unitialised (indeterminate value)
    * access uninitialised object will result in undefined behaviour

If brace-enclosed list doesn't contain an initialiser for every element:
* ie aggregate has 10 members but only 5 elements in braced initialiser
* remaining fields are zero initialised

## C++ initialiser

### C vs C+
* C++ an object lifetime begins are its been initialised
* C does not distinguish between storage and lifetime

Problems with aggregate initialisation
* C++ need initialisation rules to maintain **class invariants**
* we cannot allow string to be aggregate initialised 
    * ie: data = nullptr, size = 10
* Class authors can maintains class invariants through constructors
* **rule**: if a class has any (user provided) constructor, class is not an aggregate -> cannot
be aggregate-initialised

### Aggregate

**Definition** (C++20):
* no user-declared or inherited constructos
    * inherited constructors: types with public base class are allowed to be aggregate
* no private or protected **direct** non-static data member
* no virtual, private or protected base classes
* no virtual function
    
### Direct initialisation

* Providing constructor arguments in parenthesis
    ```cpp
    demo_str ds1("Hello", 5);
    demo_str ds2(ds1);
    ```
* can take place in the member initializer list of a constructor
    ```cpp
    C::C() : ds("James") // direct-init
    {
    }
    ```
* C++ allows direct-initialisation for scalar types
```cpp
int x (5);
double y (3.5);
```
* Direct initialisation invoking the copy constructor - as long as using **parenthesis**
```cppp
demo_str ds1("Hello", 5);
demo_str ds2(ds1);
```
### Copy Initialisation

* Using an `=` with the RHS being a compatible type
```cpp
demo_str ds3 = ds1;
```
    * note: does not invoke the copy assignment but copy constructor
* passing an object by value
* returning an object by value
* throwing an exception
* catching an exception by value


Initialisation vs Assignment:
* Initialisation
    * gives an object its initial state: direct-init / copy init
* Assignment
    * overwriting the value of an existing object

Initialisation of members:
* type of initialisation of members depends on how the members are initialised in the constructor and not how the object is initialised
* when brace initialised an aggregate - each members are copy-initialised
```cpp
struct widget {
    int id;
    demo_str ds;
};
widget w1 = {1000, "Spock"};
```

Why do we care copy-initialisation vs direct-initialisation:
* copy-initialisation will not invoke an implicit converting constructor
```cpp
void f(demo_str ds); // pass-by-value - copy init
f("McCoy"); //copy-init
```
* direct-initialisation allow to invoke implicit converting constructor
```cpp
f(demo_str("McCoy"));
```

### Default Initialisation

* Initialise without any argument or `=`
```cpp
T t;
```
* C: scalar objects without initialiser is left uninitialised
```cpp
int x;
```
* C++: 
    * scalar objects without initialiser is vacuous initialisation - initialisation that does nothing
    * default constructor will be called

### Value Initialisation

* initialise with empty parans - c++ most vexing parse
    * empty paran can be used in member initialisation / RHS
* What value initialisation do:
    * If `T` has a user-provided default constructor, call it
        * if default constructor provided, a member is not initialised and is a scaler type - remain uninitialised
    * If `T` is an array, value initialise all of its elements
    * If `T` is a sclar, zero initialised
    * If `T` is a scalar, zero-initialised it
    * Other Value Initialisation is only allow if there no user-provided constructor of T
        * zero-initialise the data member - initial all the members with 0
        * default initialise the data member - the member might be a clas with default constructor
* value initialising a class will recursively value initialise all the members only if no custom class construct is provided (default constructor)
    * if a custom constructor is defined, it will not value initialise all members

C++03: hard to value initialise because of vexing parse
```cpp
demo_str ds1; // default-init
demo_str ds2(); // vexing parse
demo_str ds3 = demo_str(); // value initialise - needs to be copy constructable
```

### Uniform Initialisation Syntax

* All types allow initialisation with braces - works for sclars, aggregates, constructors
* Empty brace (`{}`) - easy way to perform **value-initialisation**
* Empty brach `{}` will always value initialise (don't need to worry about most vexing parse)
```cpp
demo_str ds3{}; // value-init calls demo_str
```

### Direct List Initialisation
* Use braces to perform direct initialisation 

```cpp
int y{x};
```

### Copy List Initialisation
* `=` with braces on the RHS
* will not be permited if the constructor is explicit
```cpp
int y = {5};
```
* allow converting constructor to be invoked
```cpp
C c1{10, "Crusher"}; // ok
c c2 = {10, "Crusher"}; // error, requires implicit conversion
```

### Brace initialisation prevents narrowing conversion

* does not allow type conversion that result in loss of information
    * with the exception of constexpr expression and compiler can check that there isn't a loss of information

### Initialiser List constructor

* allow direct-list init (not `=`) and copy list init (`=`)
* if a default constructor and `std::initialiser_list` constructor, the default constructor will be used
* direct initialisation (`()`) vs direct list initialisation (`{}`) will invoke different constructors if `std::initializer_list` constructor is provided
    * only applies if the elements of `{}` are the same type
* do not add or remove initialiser list constructors

### Initialization in Templates

* need to think of using direct initialisation (`std::initialization_list` constructor will not be called) or direct list initialisation (`std::intialisation_list` constructor can be called)
    * sometime might prefer direct initialisation instead of direct initialisation list because a user can provide `std::initlization_list` explicitly

### Aggregate intialisation

* C++20 allow direct initialisation with `()` for aggregate types
