---
title: "Readings: Effective C++"
excerpt: "Notes for Effective C++ Third Edition"
toc: true
---

Title: Effective C++ Third Edition. 55 Specific Ways to Improve Your Program and
Designs

Author: Scott Meyers

## General Review

## Chapter 1

### Item 1: View C++ as a federation of languages

C++ can be broken down into 4 sub languages:

1. **C**: Working with C part of C++
2. **Object-Oriented C++**: Working with classes and OOP principles in C++
3. **Template C++**: Working with generic programming in C++
4. **STL**: Working with C++ STL library.

Rules for effective C++ will on the sub language.

### Item 2: Prefer `consts`, `enums`, and `inlines` to `#define`

This item mainly discuss effective methods to declare constants and
alternative to macro functions.

When we use `#define`, the compiler will replace the reference to the macro
with the defined value.

#### Declaring constants with `#define`

Drawbacks:

* The macro will not enter the symbol table which will result in confusing error
messages. Where you do not know where is the source of the error.
* Less efficient as the preprocessor will create a copy of the value each time
the macro is referenced

Alternative:

* Bad:

  ```cpp
  #define ASPECT_RATIO 1.653
  ```

* Good:

  ```cpp
  const double AspectRatio = 1.653;
  ```

#### Declaring constants for class

When declaring constant that should only exist within a class scope, use
`static const`.

```cpp
class GamePlayer {
private: 
  static const int NumTurns = 5;
}
```

Rationale:

* Using `static` will ensure that there is at most one copy of the constant

#### Inline function

Use inline functions instead of `#define` macros.

### Item 3: Use `const` whenever possible

#### `const` pointers

* `const char * p`: When `const` is on the **left** of a pointer, it means that the data is `const`.
You cannot change the value that the pointer is pointing to. However, you are allowed to change
pointer.

  ```cpp
  int i = 1;
  int j = 2;
  const int * ptr = &i;
  *ptr = 3; // compilation error
  ptr = &i; // ok
  ```

* `char * const p`: When `const` is on the **right** of a pointer, it means that
the pointer is `const` and you cannot change the variable to a new pointer. However,
you are allowed to change the value the pointer points to.

  ```cpp
  int i = 1;
  int j = 2;
  int * const ptr = &i;
  *ptr = 3; // ok 
  ptr = &j; // compilation error
  ```

#### `const` STL iterator

* Mimicks the behaviour of a pointer
* `const std:xxx<T>iterator iter` same as `T* const`: Can change the value the iterator
points to but cannot change the iterator to a new iterator.
* `std:xxx<T>const_iterator iter` same as `const T*`: Cannot change the value
the iterator points to but can change the iterator to a new iterator.

#### Function return `const`

```cpp
const Rational oeprator* (const Rational& lhs, const& rhs)
```

Generally it is not appropriate to return `const` type as the value
can be easily copied when assigning to a new variable but it prevent
the following human error:

```cpp
(a*b) = c
```

#### `const` Member Functions

Should use `const` member function to state which function
would modify an object and which would not.

Functions can also be overloaded using `const`.

```cpp
class TextBlock {
public:
  const char& operator[](const std::size_t pos) const; // for const objects
  char& operator[](const std::size_t pos); // for non-const objects
}
```

Avoid duplicate code in `const` and non-`const` member functions by casting
non-`const` `*this` to `const *this`. This is ok as we are adding tightening
the non-`const` by adding `const` constraint.

```cpp
const char& operator[](const std::size_t pos) const{...};
char& operator[](const std::size_t pos) {
  return 
    const_cast<char&>(
          static_cast<cosnt TextBlock&>(*this)[position]
        );
}
```

Use `mutable` to allow data member to be modified in `const` member functions.
Usually useful when implementing caching functions.

```cpp
mutable bool lengthIsValid;

std::size_t length() const {
  if(!lengthIsValid) {
    lengthIsValid = true; // modifying data member in const function
  }
  return textLength;
}
```

### Item 4: Make sure that objects are initialized before they're used

#### Initializing member variables

Order of class initialization:

1. Base class initialized before derived class
2. Data member are initialized in the order in which they are declared.

**Bad** example:

```cpp
class ABEntry {
private: 
  std::string theName;
  std::string theAddress;
  std::list<Phonenumber> thePhones;
  int numTimesCosulted;
public: 
  ABEntry(const std::string& name, const std::string& address,
      const std::list<Phonenumber>& phones) {
    theName = name;           // these are all assignments
    theAddress = address;
    thePhones = phones;
    numTimesCosulted = ;
  }
}
```

* Data members are initialized before the body of constructor.
* Inefficient as the default constructors are called on the non-integral
types and then copy assignment constructor called in constructor.

**Good** example (using initialization list):

```cpp

class ABEntry {
private: 
  std::string theName;
  std::string theAddress;
  std::list<Phonenumber> thePhones;
  int numTimesCosulted;
public: 
  ABEntry(const std::string& name, const std::string& address,
      const std::list<Phonenumber>& phones) :
      theName(name),
      theAddress(address),
      thePhones(phones),
      numTimesCosulted(0)
      {}
}
```

* More efficient with a single copy constructor than a default constructor followed
by copy assignment constructor.
* Might be required when initializing for `const` or reference data members.

#### Non-local static objects in different translation unit

* **static object** are objects that exists when it constructed to the end of program.
Does not exist on the stack or heap. They are destoryed when
* **non-local static object** are static objects outside of a function.
* **translation unit**: source code that give rise to single object file.

**Problem** of depending on a **non-local static object in different translation
unit**:

```cpp
// some.cpp
class FileSystem {
public: 
  ...
  std::size_t numDisk() const;
  ...
}
extern FileSystem tfs;

// other.cpp
class Directory {
public:
  Directory(params) {
    std::size_t disks = tfs.numDisks();
  }
}

Directory tmpDir(params);
```

* Relative order initialization of non-local static object in different
translation unit is **undefined**. `tmpDir` could be initialized before
`tfs`.

**Solution** (reference-returning functions):

```cpp
// some.cpp
class FileSystem {
public: 
  ...
  FileSystem& tfs() {
    static FileSystem fs;
    return fs;
  }
  std::size_t numDisk() const;
  ...
}
extern FileSystem tfs;

// other.cpp
class Directory {
public:
  Directory(params) {
    std::size_t disks = tfs().numDisks();
  }
}

Directory tmpDir(params);
```

* This will guarantee that the static object is always initialized.
* If you do not call the method you will never incur the cost of constructing.
* Caveats:
  * Does not work well with multi-threading. Need to call the reference-returning
  functions in master thread before forking child thread so that all child thread
  will have the same reference.

## Chapter 2

### Item 5: Know what functions C++ silently writes and calls

```cpp
class Empty {
  Empty(){} // default constructor
  Empty(const Empty& rhs){} // copy constructor
  Empty& operator=(const Empty& rhs){} // copy assignment constructor
  ~Empty() {} // destructor
}
```

By default, the compile will generate the following constructors inline:

1. **Generated default constructor and destructor**:
  * Default constructor:
    ```cpp
    Emtpy
    Empty e1; // defa
    ```
  * Create a default constructor that calls the base class' constructor/destructor
  * Call the default constructor/destructor of data members
  * **Exception**: When any constructor (default/non-default) is declared, the compiler
    will not create a default constructor but will still create copy constructor and copy
    assignment constructor.
2. **Generated copy constructor**:
  * Copy constructor:
  ```cpp
  NamedObject<int> no1("Smallest Prime Number", 2);
  NamedObject<int> no2(no1); // copy constructor invoked
  ```
  * The generated copy constructor will call the copy constructor on each data member and base class.
3. **Generated copy assignment**:
  * Copy assignment constructor:
  ```cpp
  Empty e1;
  Empty e2;
  e2 = e1; // copy assignment constructor
  ```
  * Call the copy assignment constructor for each data member and base class.
  * Caveats: If the data member is a reference to an object it will not work as c++ does not
  allow changing reference of a reference variable. You will need to define your own copy assignment
  constructor if you have a reference variable.
  ```cpp
  int a = 2;
  int b = 4;
  int& ref = a;
  // cannot change ref to point to b instead
  ```

### Item 6: Explicitly disallow the use of compiler-generated functions you do not want. 

In some cases you would want to disable the compiler from generating copy assignment constructor
and copy constructor (ie: Wanting an object to be unique).

**Method 1**: Declare copy assignment constructor and copy constructor as private and **do not define**
them.

```cpp
class HomeForSale {
private:
  HomeForSale(const HomeForSale&);
  HomeForSale& HomeForSale=(const HomeForSale&);
}
```

* **Making private**: Prevent calling of constructor outside of class
* **Not defining constructor**: Throw link time error when member functions try to call constructor.
* This method is widely use

**Method 2**: Set assignment constructor and copy constructor private in base class.

```cpp
class Uncopyable {
private: 
  Uncopyable(const Uncopyable&);
  Uncopyabl& operator=(const Uncopyable&);
}
class HomeForSale : private Uncopyable {}
```
This result in compilation error when trying to use copy assignment/ copy constructor.
It will call the base counter part which are private.

### Item 7: Declare destructor virtual in polymorphic base classes.

Example:
```cpp
class TimeKeeper {
public: 
  TimeKeeper();
  ~TimeKeeper();
}

class AtomicClock : public TimeKeeper {...}
```

Problems with non-virtual destructor:

* When a pointer to a derived class is cast to a base class, only the base class destructor would be called instead
of the derived class.
* This results in the data members of the base class being destructed but the data members of derived
class being untouched.
* Result in partially destructed objects and memory leaks

**Solution:** Use virtual destructor for all base classes.

Virtual function under the hood:

* Object requires to store information at runtime to determine which virtual method to be called. ie:
  ```cpp
  class B{
    virtual foo();
  }
  class D1 : B {
    virtual foo();
  }
  class D2 : B {
    virtual foo();
  }
  int i;
  cin >> i;
  
  B* b =  i > 5 ? new D1() : new D2();
  // We will only know at run time which virtual method to call.
  b->foo()
  ```
* Each object has a **virtual table pointer**(`vptr`) that points to an array of pointer **virtual table** (`vtbl`)
* Each `vtbl` will point to a function that should be invoked.
* **Drawbacks** Storing virtual table and pointer will increase space
  
Creating an **abstract class**:

* Use pure virtual destructor
* Need to add definition to the destructor of the abstract class as the base class destructor(abstract class) would be called
after the derived class destructor.
  
```cpp
clas AWOV {
public:
  virtual ~AWOV() = 0;
}
AWOV::~AWOV() {} // define virtual destructor
```

### Item 8: Prevent exceptions from leaving destructor

Throwing exceptions in destructor could result in multiple active exceptions (destructing vector of objs)
which would lead to undefined behaviour.

Instead, abstract out the logic to a separate public function to allow clients to call and handle it.
If the client did not call it, execute the function in the destructor but catch all exceptions to
prevent UB.

### Item 9: Never call virtual functions during construction or destruction

Understanding base class construction. When a derived class is constructed.

1. It will first call the base class constructor
2. **Only base class data members** will be initialized. Derived class data members will not be initialized yet.
3. Virtual functions in base class **will not point to derived class' function during base class constructor**
  - Why: derived class data members will not initialized. Calling the derived class function before
  data member initialization will lead to UB.

Calling pure virtual function in base class constructor/destructor

1. Results in link error: base class calls the pure virtual function that is not defined
2. Could bypass link error by calling the function through another function

```cpp
class Transaction {
public:
  Transaction()
  {init)(;)}
  virtual void logTransaction() const = 0;
private:
  void init()
  {
    ...
    logTransaction() // call to virtual function
  }
}
```

Solution: Pass polymorphic data through method args instead of virtual function
```cpp
class Transaction {
public:
  explicit Transaction(const std::string& logInfo);
  void logTransaction(const std::string& logInfo) const;
}

class BuyTransaction : public Transaction {
public:
  BuyTransaction(parameters) 
  : Transaction(createLogString(parameters)) // should not access data members before derived class constructor
  {

  }
}
```

### Item 10: Have assignment operator return a reference to `*this`

By convention, we should return a reference to the current object for all assignment operators

```cpp
Widget& operator(const Widget& rhs){
  ...
  return *this
}
```

This will allow you to chain assignment operators

```cpp
int x,y,z;
x = y = z = 15;
```

* Assignment operations are right associative: `x=(y=(z=15))`

### Item 11: Handle assignment to self in `operator=`

Non-direct ways for potential self-assignment:
* Different pointer to same reference
  ```cpp
  a[i] = a[j]

  *px = *py
  ```
* Base class reference/pointer could point to a derived class
  ```cpp
  Base& rb;
  Drirved* pd;

  rb = *pd; // could be the same
  ```

Without handling self: Deleting self will result to the `rhs` to be deleted as well
```cpp
Widget& 
Widget::operator=(const Widget& rhs)
{
  delete pb;
  pb = new Bitmap(*rhs.pb) // rhs.pb already deleted
  return *this
}
```

Should place delete self as last statement
```cpp
Widget& Widget::operator=(const Widget& rhs)
{
  if(this == &rhs) return *this;
  delete pb;
  pb = new Bitmap(*rhs.pb) // if exception thrown here, pb will point to a deleted addr
  return *this;
}
```
Instead should:
```cpp
Widget& Widget::operator=(const Widget& rhs)
{
  Bitmap* pOrig = pb;
  pb = new Bitmap(*rhs.pb);
  delete pOrig; // only delete after pb points to a valid address
  return *this;
}
```

#### Copy-and-swap solution
Make a copy of the `rhs`  then swap it to the  current file
```cpp
Widget& Widget::operator=(const Widget& rhs) {
  Widget temp(rhs); // make a copy
  swap(temp); // swap to *this
  return *this;
}
```

### Item 12: Copy all parts of an object

1. Only copy-constructor and copy assignment operator should copy an object
2. All local data members should be initialized in the copy constructors
3. Invoke the copying function of **base class** too. Failing to do will result in the base class
member data to not be initialized

  ```cpp
  D(const D& rhs) 
  : B(rhs),
  m_data(rhs.m_data)
  {
    
  }
  D::operator=(const D& rhs)
  {
    B::operator=(rhs);
    m_data = rhs.m_data
    return *this
  }
  ```

**Bad practices**: 
* Do not call the copy assignment from copy constructor
* Do not call the copy constructor from copy assignment
* Use a third `init()` method instead
