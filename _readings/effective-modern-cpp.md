---
title: "Readings: Effective Modern C++"
excerpt: "Notes for Effective Modern C++"
toc: true
---

Title: Effective Modern C++ | 42 specific ways to improve your use of C++11 and C++14
Author: Scott Meyers

## General Review


## Chapter 1

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
