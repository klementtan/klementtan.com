---
title: "Readings: Effective Modern C++"
excerpt: "Notes for Effective Modern C++"
toc: true
---

Title: Effective Modern C++ | 42 specific ways to improve your use of C++11 and C++14
Author: Scott Meyers

## General Review


## Chapter 1: Deducing Types

### Item 1: Understand template type deduction

This item is the basis to understand how type deduction(`auto`) works in C++. `auto` type
deduction is uses the same principals as template type deduction.


#### Template function type deduction

```cpp
template<typename T>
void f(ParamType param);


f(expr)
```

Function templates have two main "variables", `T` and `ParamType`. `T` is usually the type
that of `expr` (except for the cases below) and `ParamType` is the parameter type
declared by the function. `ParamType` usually contain `const` and reference qualifiers.

Trivial example
```
template<typename T>
void f(const T& param);

int x = 0;
f(x);
```
For this example `T` is `int` while `ParamType` is `const int&`

#### Edge Case 1: `ParamType` is a Reference or Pointer, but not a Universal Reference

Occurs when `ParamType` (function parameter declaration) is a **reference** or **pointer**. 

Type deduction
* If `expr` type is a reference, ignore reference part. (do not care if the argument provided is a reference or value)
* Pattern-match `expr`'s type against `ParamType` to determine `T`
    * Will try to match `expr` to the declared `ParamType` to determine what `T` should be

Example
```cpp
template<typename T>
void f(T& param);

int x = 27;
const int cx = x;
const int& rx = x;
```
* `f(x)`
  * `T` is `int`
  * `ParamType` is `int&`
* `f(cx)`
  * `T` is `const int`
  * `ParamType` is `const int&`
  * To match `cx`, will need to set `T` to `const int`
* `f(rx)`
  * `T` is `const int`
  * `ParamType` is `const int&`
  * The type of `T` is different from `rx`, this is due to reference part being ignored in function template

#### Edge Case 2: `ParamType` is a Universal reference

Universal reference looks like *rvalue* reference syntax(`T&&`) but it behaves differently when *lvalue* reference is passed as
an argument

Type deduction for universal reference:
* If `expr` is an *lvalue*, both `T` and `ParamType` are deduced to be *lvalue* reference (not value). 
  * Only time when `T` is deduced to be a reference
  * Even though `ParamType` is declared as *rvalue* reference, it is actually a *lvalue* reference
* If `expr` is an *rvalue*, the normal(parameter is a *rvalue* reference)

Example
```cpp
template<typename T>
void f(T&& param); // param is a uni reference

int x = 27;
const int cx = x;
const int& rx = x;
```
* `f(x)`
  * `T` is `int&`
  * `ParamType` is `int&`
  * Due to `x` being *lvalue*
* `f(cx)`
  * `T` is `const int&`
  * `ParamType` is `cosnt int&`
  * Due to `cx` being *lvalue*
* `f(rx)`
  * `T` is `const int&`
  * `ParamType` is `cosnt int&`
  * Due to `rx` being *lvalue*
* `f(27)`
  * `T` is int
  * `ParamType` is `int&&`
  * Due to `27` being a *rvalue*

Notes:
* Universal references treats lvalue and rvalue differently

#### Edge Case 3: `ParamType` is neither a pointer nor a reference

```cpp
template<typename T>
void f(T param)
```
* The parameter is passed by value

Type deduction
* If `expr` type is a reference, ignore the reference part
* If `expr` is const, ignore the const part
* If `expr` is volatile, ignore the volatile part
* For pass by value, we can ignore all the qualifiers as the parameter is totally independent
from the argument provided.

```cpp
int x = 27;
const int cx = x;
const int& rx = x;

f(x); // T and ParamType are both int
f(cx); // T and ParamType are both int
f(rx); // T and ParamType are both int
```

Special case of const pointer to const object
```cpp
template<typename T>
void f(T param);

const char* const ptr = "Fun with pointers";

f(ptr);
```
* The constness of `ptr` (RHS const) will be ignored but the type of pointer (LHS const) will be preserved
* You can change `ptr` to point to another data but you cannot change the data that `ptr` points to.

#### Array Arguement
```cpp
const char name[] = "J. P. Briggs"; // name type is const char[13]
cosnt char* ptrToName = name;       // array decays to ptr

template<typename T>
void f(T param);

f(name) // T deduced as const char *
```

* When an array is provided as an argument, the type will decay to pointer instead of array as ParamType.
* No such thing as function parameter that is array.

**Caveat**: the parameter is passed by reference
```cpp
template<typename T>
void f(T& param);

f(name); // T deduced to const char[13]
```
* Useful if you want to determine the size of the array
  ```cpp
  template<typename T, std::size_t N>
  constexpr std::size_t arraySize(T (&)[N]) noexcept {return N;}
  ```

#### Function argument

When passing a function as argument, it will decay to function pointers

```cpp
void someFunc(int, double);

template<typename T>
void f1(T param);

template<typename T>
void f2(T& param);

f1(someFunc); // param deduced as ptr-to-func void (*)(int, double);
f2(someFunc); // param deduced to ref-to-func void (&)(int, double);
```

### Item 2: Understand `auto` type deduction

This item covers how `auto` type deduction works for the majority of the time and
the corner cases where it will behave differently.

#### Template type deduction and `auto`

During type deduction for `auto`, `auto` acts like `T` in template type deduction. Similar to
`T`, `auto` can be surrounded by qualifiers (ie `const`, `&` or `&&`). You can see
`auto` type deduction as wrapping a variable in a function template.

```cpp
// auto
const auto& rx = 27;
// same as wrap in function template

template<typename T>
void func_for_rx(const T& rx) {

}
```

For universal reference, auto uses `auto&&` syntax to represent that it could
either be lvalue or rvalue reference depending on the provided expression.

```cpp
auto&& uref1 = x; // x is int and lvalue so uref1's type is int&
auto&& uref2 = cx; // cx is const int and lvalue, so uref2's type is const int&
auto&& uref3 = 27; // 27 is int and rvalue, so uref3's type is int&&
```

**Motivation**: The main reason for using `auto&&`, is when you want to preserve the
`rvalue`ness of a variable. This allows you to move the resource of the variable or forward
it to another function.

#### Edge case (initializer list)

The only difference between auto type deduction and template type deduction is
when braces (`{}`) are used.

**For `auto`**: braces are deduced as `std::initializer_list`. Where each element is of
the same type

```cpp
auto x4 = { 27 }; // type is std::initializer_list<int>, value is {27}
auto x5 = {1,2, 3.0} // error! cant deduce the type
```

**For template type deduction**: cannot deduce the type for expression surrounded by `{}`

```cpp
template<typename T>
void f(T param);

f({1,2,3}); // error!
```

Note: There is no reason why there is a difference in their behaviour but it is what it is.

#### `auto` return type or lambda params

If `auto` is used as return value of functions or parameters of lambda functions, **template type deduction**
is used instead of `auto` type deduction. Thus, it would be compilation error to use `{}` in
function return value or lambda argument.

### Item 3: Understand `decltype`

`decltype` will parrot back the type of a name (variable) or expression. It can be used to 
specify the type of a variable.

```cpp
struct A { double x; };
const A* a;
 
decltype(a->x) y; 
```

**Use Cases**
For function templates, `decltype` can be used to allow the return type depend on the parameters. 
* Use `auto` at the start of the function to state that trailing return type is used
* Use trailing return type `-> decltype(...)` to state that the return type depends on the argument.
  * If return type is used at the start, the parameters would not be available

```cpp
template<typename Container, typename Index>
auto authAndAccess(Container& c, Index i) -> decltype(c[i])
{
  authenticateUser();
  return c[i];
}
```
  * In c++14, the **trailing return** type will not be required as return type will be deduced with just `auto`

*Caveats*:
* For `auto` return type, type template deduction is used and the reference-ness will be ignored.


#### `decltype(auto)`

The combination of `decltype(auto)` means that type deduction is used (`auto`) but with `decltype` type rules (echo the exact type)

```cpp
Widge w;
const Widget& cw = w;
auto myWidget1 = cw; // auto type deduction: myWidget1 type is Widget
decltype(auto) myWidget1 = cw; // auto type deduction: myWidget1 type is const Widget&
```

You can use `decltype(auto)` and `std::forward` to make sure that the return type of the function
preserve the rvalue reference-ness and referenceness

```cpp
template<typename Container, typename Index>
decltype(auto) authAndAccess(Container&& c, Inex i)
{
  authenticateUser();
  return std::forward<Container>(c)[i];
}
```
* This will allow the return type to be reference (with `decltype(auto)`)
* Using universal reference (`&&`) allows both rvalue and lvalue containers to be provided
* Use `std::forward` to perfectly forward the type of each element.

**Caveats**:

Wrapping a name with `()` will change the type from value to reference

```cpp
decltype(auto) f1() 
{
  int x = 0;
  return x; // decltype(x) is int
}

decltype(auto) f2()
{
  int x = 0;
  return (x); // decltype((x)) is int&
}
```

#### Item 4: Know how to view deduced type

* Using compiler:
  * force a compilation error using the type you want to find
* Runtime output
  * Use `std::cout << typeid(x).name() << "\n"`. Only works if the type yields `std::type_info`
  * This does not work as expected for all cases. `typeid` will return the type with the referenceness ignored
* Boost `type_id_with_cvr` (best option)
  * Will keep the constness and referenceness of it

## Chapter 2: auto

### Item 5: Prefer `auto` to explicit type declaration

This item provides multiple reasons why using `auto` is preferred. (klement: personally I am convinced that 
`auto` should be used in a c++ project)

Pros
1. Prevent uninitialized variables.
  * With `auto`, variables will never be able to be declared without any initialization. This will prevent bugs
  such as uninitialized integers or accidentally calling the default constructor on an object
  ```cpp
  int x; // UB
  auto x = 5;

  Foo f; // accidentally calling default constructor
  auto f = Foo(x,y,z);
  ```
2. Prevent writing types that have long name (ie `typename std::iterator_traits<It>value_type`)
3. Declare closures
  * Previously, closures (captures surrounding scope) cannot be defined as it is known only to the compiler.
4. lambda with `auto` instead of `std::function`
  * Using `std::function` involves a template instantiation but `auto` does not
  * `auto` will use the **minimum** memory
  * `std::function` has a **fixed size** on the heap for any given function. If the size is not enough, it would
  allocate memory on the heap instead. This could result in OOM.
  * Due to **implementation details** that results in the that restricts `std::function` from being inlined, `auto`
  closures will definitely be faster
* The return type of a function might be different on different architecture.
  * `std::vector::size()` returns 32 bit on 32-bit windows but 64 bit on 64-bit windows
  * Using `auto` will prevent implicit conversion
* Prevent accidental implicit conversion each entry of map is `pair<const key, val>` instead of `pair<key, val>`.
  Doing `for (pair<std::string, int>& p : m)` will actually create a temporary object and return a reference to it.
  
### Item 6: Use the explicitly typed initializer idiom when `auto` deduces undesired types

This item covers the case where simply replacing explicit type with `auto` will yield different behaviours.
When we use `auto` for types that are **"invisible proxy"**, the type will no longer be invisible.

**proxy type**: Are types that are meant to be intermediate type that represent 
an underlying type. (ie `shared_ptr` or `unique_ptr`)

**invisible proxy type**: Are proxy types that are meant to live in a single statement and be implicitly
converted to the underlying type.

`std::vector<bool>`: returns `std::vector<bool>::reference` instead of the traditional `bool&`.

```cpp
vector<bool> v(5, true);

v[4] // returns std::vector<bool>::reference

bool b1 = v[4] // returns std::vector<bool>::reference but implicityly converts to bool

auto b2 = func_that_return_bool_vec()[4]; // UB! b2 is std::vector<bool>::reference which reference to a temporary object
```
* Using `auto` will prevent the **invisible proxy type** from converting to the desired **bool**
* Why `vector<bool>` return object instead reference of the object: c++ treats `vector<bool>` differently so that the
vector is represented as compactly as possible. 

To solve this, we should use `static_cast` for **invisible proxy type**
```cpp

auto b2 = static_cast<bool>(func_that_return_bool_vec()[4]);
```

## Chapter 3: Moving to Modern C++

### Item 7: Distinguish between `()` and `{}` when creating objects

There are 4 ways to initialize an object:
```cpp
int x(0); // initializer is in parentheses
int y = 0; // initializer follows "="
int z{0}; // initializer is in braces
```
Note that using `=` on an uninitialized variable calls the copy constructor
instead of the copy assignment constructor. (klement: I actually made
this mistake during an interview...)

```cpp
Foo foo; //default constructor
Foo bar = foo; // copy constructor instead of copy assignment
```

**() vs {}**
C++11 introduced uniform initializer(`{}`) which aims to set a standard to initialization
without using the braces syntax.

Trade offs:
* implicit conversion:
  * `{}` does not allow implicit **narrowing** conversion (note that non-narrowing conversion is allowed)
  while `()` allows narrowing conversion. Bugs due to narrowing conversion could be easily
  avoided when we use `{}`
* C++ most vexing parse
  * *most vexing parse*: Anything that can be **parsed** as a declaration can be interpreted as one.
  * This means that calling default constructor with parentheses will result in it declaring a
  function instead of calling the default constructor (klement: this is an interesting behaviour that is
  bug prone)
  * This issue will not occur if you use braces initializer
  ```cpp
  Widget w2(); // declares a function w2 that returns Widget
  Widget w3{}; // calls default constructor
  ```
* `std::initializer_list`
  * braces initializer:
    * If a class has overloaded constructor with one that takes `std::initializer_list`, it will always
    prefer to use the constructor with `std::initializer_list`. 
      * Can occur to any constructor (ie copy constructor, move constructor, ...) as long as there is a
      non-narrowing conversion
    * This will occur when there exist an implicit widening conversion for all elements in the braces
    * Impact: suddenly adding a new `std::initializer_list` constructor would break all of the callers code
    * **Caveat**: if the braces is empty, it would call the default constructor
    instead of the `initializer_list` constructor with no elements. To call the initializer_list constructor
    with no element, you will need to use `Widget w4({});`
  * parentheses initializer: does not suffer from this issue at all
    ```cpp
    class Widge {
      public:
      Widget(int i, bool b);
      Widget(int i, double d);
      Widget(std::initializer_list<long double> il);
    };
    Widget w1(10, true)
    ```

### Item 8: Prefer `nullptr` to `0` and `NULL`

The issue of using `0` and `NULL` to represent a null pointer is that `0` underlying type is an integer
and `NULL` underlying type is an integral type (depending on compiler). This would result in
* Overloaded function with one that accepts integral type and another that accepts a pointer, will always
call the integral type function even though it might be meant to be a pointer
  ```cpp
  void f(int); 
  void f(boo);
  void f(void *);
  f(0); // calls f(int)
  f(NULL); // depending on compiler might call f(int) 
  f(nullptr) // always call f(void *);
  ```
* Improved readability if `auto` is used
* For template functions, `0` and `NULL` will always be deduced as int and integral type instead of `void *`

### Item 9: Prefer alias declarations to typedefs

```cpp
typedef std::unique_ptr<std::unordered_map<std::string, std::string>> UPtrMapSS;
using UPtrMapSS = std::unique_ptr<std::unordered_map<std::string, std::string>>;
```

We should use alias (`using`) instead of `typedef` because of the following reasons:
* It is much readable for types that are function pointers
  ```cpp
  typedef void (*FP)(int, const std::string&);
  using FP = void(*) (int , const std::string&);
  ```
* Template alias are allowed
  ```cpp
  // alias
  template<typename T>
  using MyAllocList = std::list<T, MyAlloc<T>>;

  MyAllocList<Widget> lw;

  // typedef
  template<typename T>
  struct MyAllocList {
    typedef std::list<T, MyAllocT<T>> type
  };
  MyAllocList<Widget>::type lw;
  ```
* Template alias with template class
  * With `typedef` in template class, you will need to use `typename` as
  the templated `typedef` could have full specialisation. This means that
  it could declare `type` as something else (might not be a type). This will result
  in the `typedef` template in template class being a dependent type.
  * For alias template, it will definitely be a type so there is no dependencies
  ```cpp
  template<typename T>
  class Widget {
  private:
    typename MyAllocList<T>::type list;
  }

  template<typename T>
  using MyAllocList = std::list<T, MyAlloc<T>>;

  template<typename T>
  class Widget {
  private:
    MyAllocList<T> list;
  }
  ```

### Item 10: Prefer scoped enums to unscoped enums

Unscoped enums:

```cpp
enum Color {
  black,
  white,
  red
};
```

Scoped enums:
```cpp
enum class Color {
  black,
  white,
  red
};
```
Enumerators: are the entities within each enum. Each enumerator can be assigned a range of values.

Unscoped enums disadvantages:
* Implicit conversion
  * unscoped enums can be implicitly converted to integral types and then to floating point types.
  This will result in code that does not make sense (treating enums as floats)
* Namespace pollution:
  * For unscoped enums, the enumerator (inside the enum) share the same namespace as the enumerator.
  * This does not occur for scoped enums
  ```cpp
  enum Color {
    black,
    white,
    red
  };
  auto white = false; // error! white already declared
  ```
* Forward declaration
  * For unscoped enums, the underlying integral type is determined by the compiler after viewing all of
  the range of values for each enumerator. This means that the compiler will not be able to know how to
  use the enum if it does not know the enumerator. Thus it cannot be forward declared.
  * For scoped enums, the underlying integral type is `int` so the enum can be forward declared
  * Advantage of forward declare: prevent the entire project from recompiling if a new enumerator is added.
  * This can be avoided if you set the default integral type for unscoped enums
    ```cpp
    enum Color : std::uint8_t;
    ```

Advantage of unscoped enums:
* use unscoped enums for getters of tuple 
  ```cpp
  enum UserInfoField = { uiName, uiEmail, uiReputation };
  std::tuple<std::string, std::string, std::size_t> uInfo;
  ...
  auto val = std::get<uiEmail>(uInfo);
  ```

### Item 11: Prefer deleted functions to private undefined

When you want to remove a function that is automatically generated (ie: copy constructor,
implicit conversion, ...)

c++98: If the function is a member function, declare it private and do not define the function.
Only limited to member functions

c++11: 
* use `delete` keyword
  ```cpp
  class Foo {
  public:
    Foo(const Foo&) = delete
  }
  ```
* `delete` functions should be *public* instead of *private*. In some compiler, the error message
would be *private* function not accessible instead of function deleted

Benefits of `delete` over private + undefined
* c++98 method of private + undefined will only be caught in link time while
c++11 method of `delete` will be caught in compile time
* Use overloaded `delete` to prevent implicit conversion. (side note: c++ always prefer to convert float to double instead of int)
  ```cpp
  bool isLucky(int num); // allow the arguments to be int, char, float

  bool isLucky(char) = delete; 
  bool isLucky(bool) = delete;
  bool isLucky(double) = delete; 

  // Lead to compilation error
  isLucky('a');
  isLucky(true);
  isLucky(3.5f);
  ```
* Prevent use of template specialisation for certain template arguments.
  * Using full specialisation to state that the function is deleted
  ```cpp
  template<typename T>
  void processPointer(T* ptr);

  template<>
  void processPointer<void>(void*) = delete;

  template<>
  void processPointer<char>(char*) = delete
  ```
  * If the function template is member function, declare the deleted function in namespace scope(outside of the class)

### Item 12: Declare overriding functions `override`

Conditions for derive class overriding base class:
* Base class function must be virtual
* Base and derived function names must be identical
* Parameter type must be identical (no implicit conversion allowed)
* The constness of the base and derived function must be the same
* The return types and exception specification of the base and derived must be compatible
* The reference qualifiers must be identical
  * Member functions can have trailing `&` or `&&` to overload the function on what type of
  reference the object is.
  ```cpp
  class Widget {
  public:
    using DataType = std::vector<double>;
    ...
    DataType& data() & 
    { return values; }
    DataType&& data() &&
    { 
      // move data instead of copying when it is a rvalue (temp obj)
      return std::move(values); 
    }
  private:
    DataType values;
  } 
  ```

In c++11, we can declare overriden functions `override`.
```cpp
class Base {
public:
  virutal void mf1() const;
};
class Derived : public Base {
public:
  virutal void mf1() const override;
}
```

Benefits of `override` keyword:
* Prevent accidentally not overriding base virtual function if any of the condition not met. Throws compilation error.
* Compiler will show the impact when changing the signature of Base class function and recompiling it.

### Item 13: Prefer `const_iterator` to `iterator`

In C++11, to easily use `const_iterator` you can call the member functions `cbegin()` and `cend()` or
`std::cbegin()` and `std::cend()` to get the `const_iterator`. Use the non-member
`std::cbegin()` and `std::cend()` for generic code.

### Item 14: Declare functions `noexcept` if they won't emit exceptions

In c++11, functions can be declared `noexcept` to guarantee that no exceptions will be thrown.
The **responsibility is on the developer** (not the compiler) to ensure that all the code
in the body will not throw exception.

Benefits of declaring `noexcept`:
* The compiler will not keep the runtime stack in an unwindable state. No need for stack trace
if there is no exception.
* The compiler will not need to ensure that objects are destroyed in inverse 
order of construction (only relevant when exception is thrown).
* Standard library uses `std::move_if_noexcept`. If the move constructor is no except, some functions
would move instead of calling copy constructor for optimisation. No except condition is to ensure
that these functions still have strong safety guarantee.
  * Usually important for the member functions `swap` and `move` constructor

Note: do not false move `noexcept`. It could come at a cost of good code quality.

### Item 15: Use constexpr whenever possible

(klement: I find this item very useful as `constexpr` is a very new concept to me and scott explains it
very well here)

#### `constexpr` objects

Are objects that are `const` and known during compile time (technically translation time: compile time + link time)
* Expressions involving `constexpr` variables can be `constexpr`
* Return values of `constexpr` functions can be `constexpr`

#### `constexpr` functions

* If the arguments provided are `constexpr` objects, the function will be evaluated at compile time and the return value will be constexpr
* else the function would be evaluated like any normal functions
* Arguments can only be literal types
  * Types that can be determined during compilation
  * Objects that have `constexpr` constructors
* function body requirements
  * C++11: 
    * constexpr member functions must be treated as const member function. Cannot change data members.
    * must be single line. However, `?:` is permitted.
  * C++14: no restrictions to function body

**`constexpr` benefits**
* The objects may be placed in read-only memory
* Evaluated at compile time means that it will have shorter run time
* Can be used as template arguments (ie `std::array` size)

### Item 16: Make `const` member functions thread safe

To a caller, `const` member functions should be thread safe as it only involves reading of data. However,
sometimes you would wish to add a layer of caching for `const` member functions. You can do so by setting 
`mutable` (can be modified by `const` member functions) data members. Thus, it is important to make sure
that it is thread safe.

How to add cache:
* `atomic`: use std's `atomic` variables. Should only use if there is only 1
mutable data member.
* `std::mutex`: normal lock and unlock pattern

### Item 17: Understand special member functions

Generated functions
* default constructor
  * Generated if the class does not have user declared function
* copy operations
  * copy constructor
    * Deleted if the class declares a move operation
  * copy assignment
    * Deleted if the class declares a move operation
* move operation
  * move constructor
  * move assignment
  * perform memberwise move on the current object and base class part
  * non-move enabled data members will be copied instead
  * Generated only if the class does not define any of the copy operation or move operation or destructor
* destructor

Conditions for generated functions
* Copy operations are **independent**: declaring one does not prevent compilers from generating the other
* Move operations are **not independent**: declaring one prevents compilers from generating the other


### Item 18: Use `std::unique_ptr` for exclusive ownership resource management

Exclusive ownership:
* Use to represent exclusive ownership
* OnDly supports move semantics (do not allow copy)
* functions can return `std::unique_ptr`. Return in function to represent the ownership transfer from
callee to caller

Extra functionallity
* Most of the time `std::unique_ptr` have the same size as raw pointers
* Support custom deleter
  * Accepts the raw pointer as arguments.
  * Need to manually call the delete.
  * If the deleter captures the surrounding scope, the size will not be the same
  ```cpp
  auto delInvmt = [](Investment* pInv) {
    makeLogEntry(pInv);
    delete pInv;
  }
  std::unique_ptr<Investment, decltype(delInvmt)>pInv(nullptr, decltype)
  ```
* Use `::reset(T*)` to reset the underlying pointer the unique pointer points to
* Can easily change to `std::shared_ptr`
* Allow single object and array to be wrapped
  * `unique_ptr<T>`
    * call `delete`
    * allow `->` syntax
    * do not allow `[]` syntax
  * `unique_ptr<T[]>`
    * call `delete[]`
    * allow `[]` syntax
    * do not allow `->` syntax

### Item 19: Use `std::shared_ptr` for shared ownership resource management

Functionality:
* Represent a shared ownership of a resource
* When the last object stop pointing to the resource, it will destroy the resource
* Supports custom deleter like `unique_ptr`
  * Do not need to state the type custom deleter
  ```cpp
  auto deleter =[](){ ... }
  std::shared_ptr<Widget> pw(new Widget, deleter)
  ```
  * Custom deleter does not change the size of `shared_ptr`
* Unlike `unique_ptr`, shared pointer does not support `T[]`

Under the hood:
* Reference counting:
  * Most constructors(non-move) will increment the reference count
  * Move constructors will not change the reference count.
    * Move the reference count from one to another
    * Faster than non-move (dont need atomic operations)
  * copy assignment will increment the rhs reference count and decrement the lhs reference count
  * when the reference count becomes 0, it will destroy the underlying resource
* Performance limitations
  * **twice the size** of normal raw pointers
  * reference count are dynamically allocated (additional overhead)
  * increment and decrement are atomic

#### `std::shared_ptr` control block

* Each `shared_ptr` should have a control block that stores metadata for it
* metadata
  * reference count
  * weak count
  * custom deleter
  * allocator
* A **new control block** created when:
  * `std::make_shared` is used
  * when `std::unique_ptr` or `std::auto_ptr` is used to construct `shared_ptr`
  * when **raw pointer** pass as argument to the `shared_ptr` constructor
* **Limitation**: Each raw pointer address should only have a unique control block.
  * Virtual function is involved which would lead to more latency

**UB**: multiple control block for a shared pointer

When a single address is used in different `shared_ptr` with different control block, there will be different reference
count of different value and delete will be called twice on the same address

```cpp
auto pw = new Widget;
std::shared_ptr<Widget> spw1(pw, logginDel);
std::shared_ptr<Widget> spw2(pw, logginDel);
```

Usually happen when using `this`. The member function do not know if the current instance is already in a shared pointer

```cpp
class Widget {
private:
  std::vector<std::shared_ptr<Widget>> processWidget;
public:
  void process() {
    processWidget.emplace_back(this);
  }
}
```

Mitigation: `std::enable_share_from_this<T>`
* virtual function `this->shared_from_this()` that will make sure that there
will only be one control block for `this`
* make all constructors private and have a factory function that
returns a `shared_ptr` from `shared_from_this()`. This will make sure that external callers
will not have access to `this`

### Item 20: Use `std::weak_ptr` for `std::shared_ptr` like pointers that can dangle

Motivation: smart pointers that acts like shared pointer but does not take part
in shared ownership

`std::weak_ptr` properties:

* Allowed to dangle: the object on the heap could be destoryed
* Can only be created from `std::shared_ptr`
  ```cpp
  std::shared_ptr<Foo> sptr = make_shared();
  std::weak_ptr<Foo>wptr(sptr);
  ```
  * Will not increase the reference count of `shared_ptr`
* Can check if the underlying object on the heap has been destoryed (not possible with other smart pointers)
  * call `std::weak_ptr::expired`
* Converted back to `shared_ptr`: regain the shared owner ship of the underlying object
  * `std::weak_ptr::lock`: returns the corresponding `shared_ptr<Foo>` if object still on heap or `nullptr`
  if the object has been deleted.
  * `std::shared_ptr<T>(std::weak_ptr<T>)`: use the weak pointer as an argument for
  shared pointer. Throws an exception if the underlying object that the weak pointer points
  to has been deleted.

Uses cases:
* Caching objects on the heap:
  * Stores a containers of weak pointers to the object. Retrieve from cache by converting the weak pointer
  back to shared pointer
* Prevent cyclic reference:
  * Shared pointers will not be able to solve dangling cyclic reference.
  * If the leaves need a reference back to the parent, use weak pointers to prevent cyclic reference 
    * If the child life time will always be shorter than the parent, it is okay for the child to have
    a raw pointer back to the parent. When a child is a live the raw pointer to parent will always be alive
* Observer design pattern:
  * Observer subscribe to subjects by storing a pointer to the observer as a data member in the subject.
  Subject will call the member of functions to notify them
  * Subjects to store a weak pointer to observer:
    * Do not do anything if the weak pointer has expired
    * Convert to shared pointer and notify if the observer is not destroyed
    * Prevent the subject from affecting the life time of the observer

Efficiency: Exactly same as shared pointer

### Item 21: Prefer `std::make_shared` and `std::make_unique` to direct use of new

Resource leak from constructor and `new`
* When:
  1. A function is called with `{unique/shared}_ptr(new T)` and `expcetionFunction()` as argument
  2. Compilers are allowed to reorder the argument instruction if there are no dependencies
  3. Instructions could be reorder to: `new T` -> `exceptionFunction` -> smart pointer exception
  4. This will result in resource leak as `new T` is stored in smart pointer and will not be deleted
* Wrapping the allocating of the resource + constructor of the smart pointer in a function, will prevent
bad interleaving of instructions.


`make_shared` is faster:
* shared pointer will need to allocate memory for both the object and control block
* Using `make_shared` will allocate both the object and control block together
* One allocation is required instead of allocating the object first then the control block
  * Faster and smaller code
  * Control block and object in a contiguous memory location -> spatial locality

`make_*` limitations:
* Cannot specify custom deleter
* Will not be able to perfectly forward braced initializer
  ```cpp
  auto upv = make_unique<vector<int>>(10, 20); 
  // calls vector<int>(10,20) instead of vector<int>{10,20}
  ```
* Custom newed object do not work well `make_shared`
* As control block and the object are in the same chunk of memory, the chunk of memory can
only be freed when the reference count (number of shared pointer) and weak count (number of 
weak pointer) goes to 0.
  * The chunk memory can still be around when the object has been destructed (as long as the number of dangling weak pointers).
  * Using `new T` on shared pointer constructor will have two different chunk of memory
  and the object's chunk memory can be immediately freed when no more shared pointers points to the
  object
  * (klement: I was not aware that the freeing of memory might not occur immediately after the destructor is called)


Mitigation for `new T` and shared pointer constructor
* Construct the shared pointer and `new T` in an independent line -> prevent resource leak
* pass the shared pointer using `move` to preserve rvalueness

### Item 22: When using the Pimpl Idiom, define special member function in the implementation file (.cpp file)

(klement: This items goes in-depth on how to properly utilize the pimpl idiom. This is very crucial as there
are not many resources out there (including effective cpp) that properly show and explain how to do so.)

Motivation: As a library author, if a client include your library header files, they will also include the headers that
your header files include. This will result in them having to recompile all of their code if the indirect headers file change.

Proper pimpl example:

```cpp
// In header (.h) file
class Widget {
private:
  struct Impl;
  std::unique_ptr<Impl> pImpl;
public:
  Widget();
  ~Widget();

  Widget(const Widget& rhs);
  Widget& operator=(const Widget& rhs);

  Widget(Widget&& rhs);
  Widget& operator=(Widget&& rhs);
};

// In implementation (.cpp) file
#include "widget.h"
#include "gadget.h"
#include <string>
#include <vector>

struct Widget::Impl {
  std::string name;
  std::vector<double> data;
  Gadget g1, g2, g3;
}

Widget::Widget() : pImpl(std::make_unique<Impl>()) {}
Widget::~Widget() = default; // force compiler to generate destructor after seeing Widget::Impl

Widget(Widget&& rhs) = default; // moving of underlying impl is desired behaviour
Widget& operator=(Widget&& rhs) = default; // ditto

// perform deep copy of the underlying impl
// do so by using the impl copy operations
Widget(const Widget& rhs) : pImpl(std::make_unique<Impl>(*rhs.impl)) {}
Widget& Widget::operator=(const Widget& rhs)
{
  *pImpl = *rhs.impl;
  return *this;
}
```

Impl:
* Declare Impl in header file but do not define it so that you do not need to 
need to include the dependencies for the definitions
* Any compiler generated functions (destructor, move, copy) that require the 
underlying implementation should also be declared but not defined 
(incomplete type) in the header. This will make sure that when the compiler 
see these function it would have already seen the implementation type.
  * Under the hood, destructor is the only operation that does not work without declaration.
  Without destructor, all other generated function would not be generated

`unique_ptr`: the underlying data has to be allocated on the heap. Use `unique_ptr`
to easily manage the pointer

Destructor and move operations:
* The generated function behaviour for these function are desired but we have to declare them in the header
and define them as `=default` in implementation.
* By default compiler will inline these functions and generate the functions at the header file. By declaring them,
the compiler will go to the implementation file to find the definition and default to default after seeing `= default`

Copy operations:
* Copy operations should be deep copy instead of shallow copy (copying the pointer but still point to the same address)
* Instead of manually copying the data member outisde of `impl`, call the underlying impl copy operations.

Using `shared_ptr` instead of `unique_ptr`
* Destructor for `shared_ptr` are not part of the `shared_ptr` type (stored at runtime control block)

## Chapter 5: Rvalue Reference, Move Semantics and Perfect Forwarding

### Item 23: Understand `std::move` and `std::forward`

Both `std::move` and `std::forward` do not perform any "moving" and do not have any executable code at runtime.

#### `std::move`

Performs *unconditional* cast to rvalue reference

```cpp
template<typename T>
decltype(auto) move(T&& param)
{
  using ReturnType = remove_reference_t<T>&&;
  return static_cast<ReturnType>(param)
}
```
* `T&&` in function template means that it can be of any reference
* move remove all reference and cast it back to the rvalue (unconditional cast)

`std::move` on a `const` variable as overloaded function argument. Call the function in the following order
1. Calls the param with `const T&&`
2. Calls `const T&`: const lvalue reference binds to const rvalue reference 
  * const rvalue reference will never bind to non-const rvalue reference
  * Converse is ok -> binding non-const rvalue reference to const rvalue reference
* Should only move non-const variable otherwise will *usually* bind to const lvalue reference
* `std::move` does not guarantee moving.

#### `std::forward`

Problem: All function parameters are lvalue as they exist as a variable in the parameters (eventhough might be temporary on call-site).
Any function call within a function body with the parameters will be **rvalue**.

Solution: `std::forward` will cast the function template param to **rvalue** if it is **rvalue** on the call-site
* The rvalue-ness is encoded in the function template param `T`
* `std::forward` will conditionally cast to `rvalue` if the argument is `rvalue`.

### Item 24: Distinguish universal references (forwarding reference) from rvalue references

Universal reference: references that can be rvalue or lvalue, const or non-const, volatile or non-volatile

Conditions for universal reference: **type deduction must** occurs specific to the function (ie function template or `auto`)

**Function template**:
```cpp
template<typename T>
void f(T&& param)
```
* When the template is specific to the function. Type is deduced every time the function is called.
  * Function inside template class do not perform type deduction every time it is called 
  ```cpp
  template<class T, class Allocator = allocator<T>>
  class vector {
  public:
    void push_back(T&& x);
  }

  class vector<Widget> {
  public:
    void push_back(Widget&& x); // no type deduction
  }
  ```
* Must be of the form `T&&`. No qualifiers are allowed (ie `const T&&`)

`auto&&`
```cpp
auto&& var2 = var1;

auto fn = [](auto&& func, auto&&... params) {
  ...
}
```
* Using `auto&&` in statement will set the type to whatever the expression is.
* Using `auto&&` in a lambda param uses type deduction and will deduce the type it is.

**Side Note**: universal reference + string literal
* Passing string literal (`"..."`) to a function taking universal reference will be deduced to
array of characters `const char[N]`. This will allow the param to point to the string in
wherever the literal is stored (usually read only memory) instead of creating a temporary string.
* Using `std::forward` on the deduced `const char[N]` will allow the string to only be created at
where it is really needed


### Item 25: Use `std::move` on rvalue references, `std::forward` on universal references

* If the parameter is an rvalue, it can and should be moved. Propagate the rvalueness using
`std::move` for rvalue and `std::forward` for universal reference.
* function returning ref params by value
  * Should propagate rvalueness in the return because there would be no RVO (see below)
  * If the param is rvalue it would call the move constructor for the temp variable instead
  of having to copy the temp object into the return value
  ```cpp
  Matrix operator+(Matrix&& lhs, const Matrix& rhs) {
    lhs += rhs;
    return std::move(lhs);
  }
  template<typename T>
  Fraction reduceAndCopy(T&& frac) {
    frac.reduce();
    return std::forward<T>(frac);
  }
  ```
  * If the object does not have move constructor, `return std::move(lhs)` will default to calling
  copy constructor (non-intrusive)

#### Return Value Optimisation (RVO)

*RVO*: compilers are allowed by constructing the return value in memory allocated (call-site)
for the function's return value. This will prevent the need to **copy** or **move** constructor.

Conditions:
* The local object is the same as returned by the function
* The local object is what's being returned
  * Value parameters are considered local variable and returning them can be RVO'ed
  * DO NOT `return std::move` as it will return a reference to local object and not satisfy
  the condition. The copy will not be elided and will call the move constructor instead.

**Note**: If the compiler does not perform RVO, it will call `return std::move` so that the
return value will be moved constructed.

### Item 26: Avoid overloading on universal references

If a function is overloaded with universal reference and an explicit type,
the universal reference function would be called most of the time.
* Universal reference are greedy: can bind to any type and any qualifier
  * Unless the argument function matches the other function completely (type + qualifier)
* Compilers will still generate the copy move constructor even if a perfectly forwarding constructor is declared
