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
