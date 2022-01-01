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
