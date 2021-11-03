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

## Chapter 3: Resource Management

The contents of this chapter is slightly outdated as it still
refers to `auto_ptr` (deprecated) and not the new `unique_ptr`.

### Item 13: Use objects to manage resource

The main takeaway for this item is to use object to manage
heap based resource instead of raw pointers (just `T*`)

#### Drawbacks of raw pointers

1. Whenever you dynamically allocate memory, you will need to
manually deallocate it.
2. Your instruction to deallocate memory might not happen when
there is an **early return**, **exception**, **break** in a loop.
3. This will result in a memory leak

#### Resource Acquisition Is Initialization (RAII)

Using an object that acquires a resource (dynamically allocated resource)
instead of managing raw pointers.

Key Ideas:
* All object's (on stack) destructor will be called once it goes out of scope
* Tying a resource to an object will allow the destructor of the object to clean
up the resource
* std provides smart pointers that acts as a wrapper to objects
  * `auto_ptr` (deprecated) and `shared_ptr`

#### Smart Pointers

* All resource are assigned to a managing object
* Destructor of objects are called automatically when it is destroyed
* Drawbacks: 
  * Only calls `delete` on the pointer instead of `delete[]`. If the resource is
  a dynamically allocated array, only the first pointer will be deleted but everything else.
  **Compiles but leads to UB**

**auto_ptr** (deprecated)
* wraps around a pointer and calls the destructor of the pointer once it goes out of scope
  ```cpp
  void f() {
    std::auto_ptr<Foo> f(new Foo());
  }
  ```
* Drawbacks:
  * Does not exhibit normal copy constructor and copy assignment behaviour. The objects are "moved"
  instead of being copied
    ```cpp
    std::auto_ptr<Foo> f1(new Foo()); // f1 points to the new Foo
    std::auto_ptr<Foo> f2(f1)         // f2 points to the previous new Foo
                                      // f1 now points to null

    f1 = f2;                          // f1 points to the previous new Foo but
                                      // f2 points to null
    ```
  * Will not work in STL containers as it require elements to exhibit normal **copying behaviour**.

**shared_ptr**

* Uses reference-counting to determine if the resource can be deleted.
  * Every time the object (`shared_ptr`) is copied/assigned it will increase the reference.
  * Call destructor on the pointer once the counter goes to 0.
* **Caveats**: Cannot break cycles. If two objects that are unused has a reference to each other,
  the counter will never go 0.
  * Unlike Java GC, there is no **mark**, **sweep**, **merge** steps to determine if an two objects
  are stuck in a cyclic reference

### Item 14: Think carefully about copying behaviour in resource-managing class

You can implement your own RAII to manage heap resource. However, based on your
requirements, you might want to handle copying separately.
* **Prohibit Copying**
  * If a resource should not be copied (Lock) for a mutex, use [Uncopyable](#item-6-explicitly-disallow-the-use-of-compiler-generated-functions-you-do-not-want)
* **Reference Counting**
  * If the resource should be destroyed once all references to has been deleted.
  * Use a `shared_ptr` data member.
  * If we do not want to delete the underlying resource (unlocking a mutex instead of deleting),
    we can provide `shared_ptr` with a **deleter** funtcion. This will cause the `shared_ptr` to
    call the function instead of the destructor of the resource.
  * Copying the underlying resource (more than one of the resource could exists), we should perform
  a deep copy of the data in the heap instead of just copying the pointer.

### Item 15: Provide access to raw resource in resource-managing classes

There are times you would need to access the underlying heap based resource
from an RAII. You can use the following approach:
* Explicit conversion:
  * Provide a `raii.get()` function to get the underlying heap based resource
  * Overload pointer dereferencing operator (`->`)
  * Drawbacks: might be intrusive to the code
* Implicit conversion:
  * Allow casting from `raii` to the underlying heap based resource pointer.
  * Allow clients to easily use api that require heap based resource pointer without
  having to call explicit conversion
  ```cpp
  class RAII {
  private: 
    Foo f;                              // heap based resource
  public: 
    operator Foo() const{return f;}     // Overload casting operator to return
                                        // the underlying resource when casted to it
  }
  void Bar(Foo f);                      // Require heap based resource
  RAII r;
  Bar(r); // cast RAII to Foo
  ```
  * Drawbacks: allowed accidental casting from RAII to the underlying resource.

### Item 16: Use the same form in corresponding uses of `new` and `delete`

We should always use `delete[]` iff `new T[]` was used and `delete` iff `new T`
was used.

The reason is that when we call `delete[]` or `delete` on address we do not
validate what the address contains and call the destructor on the memory.
When we call `new` or `new T[]` the heap will look like (depends on compiler):

```txt
Single  Object: | Object |
Array         : | n | Object | Object | Object |   
```

* Calling `delete[]` on a single object will result in the first `k` bytes being assumed to be
the size of the array (but is actually part of the Object) and the destructor will be called
on the next contiguous `n'` location in memory.
* Calling `delete` on an array will result in the address for the size of the array to be deleted
and part of the first object to deleted.

### Item 17: Store `newed` objects in smart pointers in standalone statement

To guarantee that there will be no resource leak when using smart pointers, we
will need to make sure that no exceptions are thrown between heap based resource
being allocated and smart pointer being constructed.

The easy solution is to store `new` object in smart pointer in a standalone statement
that guarantees no exceptions thrown

Negative example:
```cpp
processWidget(shared_ptr<Widget>(new Widget), foo());
```
* We will need to perform the following argument evaluations:
  * Execute `new Widget`
  * Construct `shared_ptr`
  * Call `foo`
* In cpp, the compiler is allowed to reorder the evaluations of arguments
* Compiler can reorder to: Execute `new Widget` -> Call `foo` -> Construct `shared_ptr`
  * If there is an exception in `foo`, the `new` Widget would be leaked and will not be
  cleaned up by `shared_ptr`


## Chapter 4: Design and Declarations

### Item 18: Make interfaces easy to use correctly and hard to use incorrectly

Good API design should prevent any client errors by making it easy to use and 
hard for the clients to use incorrectly. We can do so with the following methods:
* Strongly typed parameters:
  * When a function takes in multiple parameters of similar type, client could easily make
  mistakes by using the wrong ordering of parameters
  * If there is a small range of allowed argument for a parameter, use class function
  as a constructor.
  ```
  struct Month {static Month::Jan(){return Month(1)}};
  struct Day {};
  struct Year {};
  Date d(Month::Mar(), Day(30), Year(1995))
  ```
* Return `const` to prevent invalid assignment
* Return smart pointers
  * Do not rely on client calling delete
  * `shared_ptr`: Allow you to add your own `deleter` to perform clean up once the references go to 0
* Interface should be consistent with standard library interfaces

### Item 19: Treat class design as type design

This item mainly state the various questions we should ask ourselves when designing a new type.

### Item 20: Prefer pass-by-reference-to-const to pass-by-value

**Pass-by-value**
* Make a copy of the actual argument. Calls the copy constructor of all the data members and base class
* By default all arguments are passed by value
* **Object Slicing**: Derived object slice off derived class data members when copying.
  * Pass a derived class to a function that takes in the base class
  * Causes the **only the base class** copy constructor to be called on the derived object
* Applies to user defined type with small constructor
  * Compiler treats user-defined type differently from built-in types
  * The constructor overhead of user-defined types might change
* **Exceptions**:
  * Built-in types: same overhead as passing a pointer
  * stl iterator: act as pointers
  * function objects


**Pass-by-reference-to-const**
* When we pass by reference we are essentially passing a pointer as an argument to the function
* Use const so that the caller will know that argument would not be mutated

### Item 21: Don't try to return a reference when you must return an object

When returning an object as reference from the local stack (function body), the object will destructed
once it returns. The reference that the function returns will point to an object that is destructed.

**Notes**
* Returning reference of object on stack:
  * Objects would be destructed at the end of the function
  * Reference to destructed object will still be returned -> leads to UB
* Returning reference of to object on heap:
  * Easily lead to memory leak as we do not know who should delete the object

### Item 22: Declare data members `private`

This items argues why data members should be private and does not cover any c++
specific points.

Arguments:

* Syntactic consistency
* Encapsulation: allow you to easily change the implementation of the data member.
  * ie: changing the data member from a member variable to a combination of member variables

### Item 23: Prefer non-member non-friend functions to member functions

This item mainly applies to member functions that does not actually
require to be a member function (utility functions)

**Member function** drawbacks:
* Reduces encapsulation as it exposes internal data members
* Does not allow separation of utility functions.
  * If a client imports a class, it will have to import all
  utility functions instead of what they just need

**Solution**: Create multiple header files that contains different type of utility functions
on the same class

### Item 24: Declare non-member functions when type conversion should apply to all parameters

How type conversion works:
1. Given a type `T1` and the object can only be constructed from type `T2`
  * type coversion will work if there is conversion from `T2 -> T1`.
2. A temporary `T2` object will be created from `T1` with `T2(T1)`
3. `T2` will be provided as argument to the object's constructor

How operators work:
1. For operators (`*`, `+`, ...), the compiler will try to look for valid operator
declared in the namespace scope to perform the operator on the 2 parameter
2. Type conversion will be used if the operator is not `explicit`

For operators that should perform implicit conversion on lhs/rhs, use non-members
so that both the parameters can be converted.

```cpp
class Rational {

}
const Rational operator*(const Rational& lhs, const Rational& rhs)

Rational oneForth(1,4);
Rational result;
result = oneForth* 2 // convert 2 to Rational
result = 2* operator // convert 2 to Rational
```
### Item 25: Consider support for a non-throwing swap

* Swap operations should not throw exceptions as many operations
rely on swap operations to prevent memory leak

## Chapter 5: Implementations

### Item 26: Postpone variable definition as long as possible

This item argues that we should defer any variable definition
to as long as possible. This includes:
* Defining the variable right before it is needed:
  * Prevent unneccassary calls to the constructor if an exception
  is thrown or returned before it is used.
  ```cpp
  ...
  string encrypted;
  if (...) throw exception; // encrypted is constructed and destructed
  foo(encrypted);
   ```
* Do not define a variable with default constructor then assign it to a new
value afterwards
  * Prevent extra call to constructor then copy assignment when you could just
  have a single call to copy constructor
  ```cpp
  ...
  string encrypted;
  encrypted = password;
  foo(encrypted);
  //vs
  string encrypted(password);
  foo(encrypted);
  ```

### Item 27: Minimize casting

Types of cast
* C-style: `(T) expression`
* Function style: `T(expression)`. Call the type's cast function
* C++ style:
  * `const_cast<T>(expression)`: cast away the const from an object
  * `dynamic_cast<T>(expression)`: cast from a base class to a derived class. Determine polymorphic type in runtime
  * `reinterpret_cast<T>(expression)`: casting a pointer to an int (for low level)
  * `static_cast<T>(expression)`: for implicit conversion (ie int to double)

Problems with casting:
* When casting a derived **object** to base **object** with `static_cast`. C++ will create a temporary copy of the base
object and call the function on it.
  * Solution is to call base class function directly `Base::foo()` in the derived class function
* `dynamic_cast` is slow: under the hood performs string matching to the class name
  * if you find yourself doing multiple `if(auto t = dynamic_cast<T>(ptr))` a lot should use virtual functions instead


### Item 28: Avoid returning "handles" to object internals

We should avoid returning a handler (reference, pointer, iterator)
to an internal data structure in member functions.

Rationale:
* If the return type is not const, it will allow the client side to modify internal
data member using the member function.
* Result in dangling handles, if a temporary object is created and only the handler is
assigned to a variable. The temporary object will destroyed but the handler will be left dangling.

### Item 29: Strive for exception-safe code

**Goals of exception-safe function**
* **Leak no resource**: should not leak any resource
* **Don't allow data structure to become corrupted**: A function should not be in an
invalid state no matter what

**Types of exception-safe guarantees**
* **Basic guarantees**
  * When an exception is thrown a program will be in a valid state. Might be a the same state before exception is thrown
* **Strong guarantees**
  * State changes are atomic. When an exception is thrown a program will revert to the previous state before the exception is thrown
* **Nothrow guarantees**
  * function does not throw any exception

It is ok to only offer **basic guarantee** (depend on **basic guarantee** code) but should
not be non exception-safe.

#### How to achieve exception-safe

The general way to achieve exception-safe is to use the Pimpl idiom with copy and swap
* Use pimpl to allow the underlying data member to be easily copied to a temp copy 
* Perform necessary changes to the temp copy
* Swap the temp copy to replace the current copy

```cpp
struct  PMImpl {
  std::shared_ptr<Image> bgImage;
  int imageChanges;
}

class PrettyMenu {
private:
  Mutex mutex;
  std::shared_ptr<PMImpl> pImpl;
  void changeBackGround(std::istream& imgSrc) {
    using std::swap;
    Lock ml(&mutex);                                  // use RAII to prevent resouce leak
    std::shared_ptr<PMImpl> pNew(new PMImpl(*pImpl)); // create a temporary copy of impl
    pNew->bgImage.reset(new Image(imgSrc));
    ++pNew->imageChanges;

    swap(pImpl, pNew);                                // only change state if no exception
  }
}
```

* Caveats: Pimpl + copy and swap will only work when there are no side effects to outside the local data state.

### Item 30: Understand the ins and out of inlining

Overhead of a function call:
* Saving the current registers on the current function stack
* Pushing the new function argument into the new function stack
* Increment the stack pointer
* Jump to the new instruction of the new function
* When the function is done, need to restore the previous function stack

`inline` functions:
* **Request** the compiler to replace the call to function with the block of code
* For small function body will increase **instruction cache hit rate**.
* Types of inline request
  * implicit: define the function within the class 
    ```cpp
    class Person {
    public:
      ...
      int age() const { return theAge; } // an implicit request
    }
    ```
  * explicit: using `inline` keyword before the function.

**Caveats**:
* Constructors and destructors might seem like good inline functions but under the hood,
they are long functions that have code to construct the data members
* How the function is called matters: Function pointer to inline function would 
cause the inline function to be generated out of line.
  ```cpp
  inline void f() {...}
  void (*pf)() = f;
  ... 
  f();  // inlined
  pf(); // most probably wont be inlined
  ```
* **DLL**: if a library uses inline function, the client must recompile the library instead of
just relinking a non-inline function

### Item 31: Minimize compilation dependencies between files

**Problem**:

* There could be a cascading compilation dependencies when the declaration and definitions are
coupled together.
  * A client only cares about the public functions and not the private data members. 
  * Coupling the declaration and definitions (private data members) would result in the client
  having to recompile when there is a change in private data member's code.
  ```cpp
  #include <string>
  #include "date.h"
  #include "address.h"
  class Person {
  public:
    Person(const std::string& name, const Date& birtday, const Address& addr);
    std::string name() const;
    std::string birthDate() const;
  private:
    std::string theName;
    Date theBirthDate;
    Address theAddress
  }
  ```

**Solution**:
  * **pimpl**
    * Only declares functions and no data members
    * Implement by forwarding the function call to the `impl`
  * Forward declare types
    * Forward declare custom type so that if the client doesn't need the custom type, they do not
    need to depend on it.
    * If the client need the custom type they will just include the custom type header file instead
    of relying on your header file.
    * Separate forward declaration to `*fwd.h` to a separate header file that just forward declares.
  * **interface**
    * Having an abstract base class that does not contain any data members. Having a pointer to abstract
    class with factory function will allow the client not having to include unneccassary dependencies.

## Chapter 6: Inheritance and Object-Oriented Design

### Item 32: Make sure public inheritance models "is-a"

**Public Inheritance**: the derived class will have access to all public members (function and data)
of the base class. The derived class does not have access to private data members (need to access through
public function of base class).

**Take aways**: All derived class should "is-a" base class. This would allow the compiler
to prevent the derived class from having properties it should not have at compile time.

### Item 33: Avoid hiding inherited names

Unlike other PL like Java, overriding a function would hide all functions in the base class with the
same name (does not distinguish between function signature)

```cpp
class Base {
private:
  int x;
public:
  virtual void mf1() = 0;
  virtual void mf1(int);

  virtual void mf2();

  void mf3();
  void mf3(double);
};
class Derived: public Base {
public:
  virtual void mf1();
  void mf3();
  void mf4();
}

Derived d;
int x;

d.mf1();  // calls Derived::mf1
d.mf1(x); // error! Base class mf1(int) got hidden
```

Rationale: C++ chose to hide all function names irregardless of function signature as it

**Solution**: use `using`
```cpp
class Derived: public Base {
public:
  using Base::mf1;
  using Base::mf3;
  virtual void mf1();
  void mf3();
  void mf4();
}
```

* Use this method if you inherit a base class and you want to redeclare or override some but
not all functions

Forward functions

```cpp
class Derived : private Base {
public:
  virtual void mf1() 
  { Base::mf1(); }
}
```


## Chapter 7: Template and Generic Programming

**Introduction** Templates were initially created to allow for type-safe containers (`vector`).
However, it has evolved to be a turing complete stack. The compile will execute and run
the metaprogram in compile time and complete the execution when the program exits.

### Item 41: Understand implicit interfaces and compile time polymorphism.

Traditionally we use explicit interface and runtime polymorphism.
* **Explicit interface**:
  * Interface of a class/struct.
  * Users will be able to look up the type and view the function signature
* **Runtime polymorphism**:
  * Only at runtime we will know what virtual function to call using virtual pointer.


Metaprogramming allows for **implicit interface** and **compile time polymorphism**.

How it works: at compile time, the compiler will take the function template and the template
argument (the actual type to be supplied) and synthesize/instantiate a function
with that type.

#### Implicit interface

Instead of having an explicit interface stating the member functions and variables.
With templates, you will state the functions that you would like use for the generic
type and the compiler will check if the supplied template at compile time support
these functions (technically it should be expressions).

```cpp
template<typename T>
void doProcessing(T& w) {
  if(w.size() > 10 && w != someNastyWidge) {
    T temp(w);
    temp.normalize();
    temp.swap();
  }
}
```
* Compiler will only compile if `w` has `.size(),`, `.normalize()` and `.swap()`

#### Compile time polymorphism

At compile time, the compiler will synthesize the function template for that specific
supplied type. This will let the correct function for each type to be called instead
of relying on virtual functions.

For example:

```
Foo("ssss");
Foo(1234);
```
* The same function template (Foo) will be synthesized for string and int. It will call the
respective int/string member function and thus allowing for polymorphism.

### Item 42: Understand the two meaning of `typename`

I am still a bit confused with this item.

Different `type`s in a template:
* **dependent** name: when a named type is dependent on a template type. ie `T::foo`
* **nested dependent** name: a dependent type that is nested inside a class.
* **non-dependent** name: any type that is not dependent on the template type. ie `int`

Problem with **nested dependent**: when using nested dependent type, we do not know if `T::foo` is another 
type or is a static variable. To solve this, use `typename` at the front.

```cpp
template<typename C>
void Foo(const C& cont) {
  typename C::const_iterator iter;
}
```

## Chapter 8 customizing `new` and `delete`

C++ allows you to define a callback function (`new_handler`) to be executed when it fails to
allocate new memory.

### Item 49: Understand the behaviour of new-handler

**How it works**
1. Declare custom `new_handler`. This would apply to all `new` operations.
  * `set_new_handler` returns the previous `new_handler`.
  ```cpp
  std::set_new_handler(outOfMem);
  ```
2. When `new` is called
  * Sufficient memory: nothing happens and the memory gets allocated
  * Insufficient memory:
    * Keep calling the latest function of `set_new_handler(...)` until there is an exception
    or sufficient memory allocated

As `new` will be recursively called, `new_handler` should have one of the following properties:
* Make more memory available: Next `new` call would have sufficient memory
* Install a different new-handler: Will not call the same function again and stuck in infinite loop
* Deinstall new-handler: set `set_new_handler(nullptr)` so that no function would be called and will
use the default exception
* Throw an exception: of type `bad_alloc`
* Not return: call abort or exit

#### Class specific new handler

Challenges:
* Need to call `set_new_handler(fooHanlder)` and `set_new_handler(oldHandler)` after the new operation
* Will need to handle when there is an exception when calling `new`

Solution:
* Treat `new_handler` as a resource and wrap it in a RAII.
* Use Curiously Recurring Template Pattern (CRTP) to make a mixin base class that could be inheritted by
multiple classes
* (Klement: I do not really understand the solution proposed in the book but you can refer to it if you would 
to view the actual implementation)

### Item 52: Write placement `delete` if you write placement `new`

**Definition placement new and delete**: When `new` is called it usually
calls the `static void* operator new(std::size_t size)` member function.
However, we can declare our own new operator that accepts more than `size_t`
parameter.
* ie `static void* operator new(size_t size, ostream& logstream)`

Placement new with address:
* `static void* operator new(size_t size, void* ptr)`
* Allow you to construct an object on the given address instead of allocating new memory
* Used in vectors

