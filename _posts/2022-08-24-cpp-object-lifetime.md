---
title: "Post: C++ Object Lifetime Deep dive"
categories:
  - Post
tags:
  - C++
excerpt: "A deep dive into C++ Object Lifetime"
---

This post contains my personal notes from Jason Turner's [CppCon 2018: Jason Turner “Surprises in Object Lifetime”](https://youtu.be/uQyT-5iWUow) talk.

## Definitions

**Object**: an object type is a (possibly cv-qualified) type that is not a function type, not a reference type, and not cv void
* Basically anytime that can be const/volatile qualifed.

**Lifetime Begin**:
* Storage with the proper alignment and size for type `T` is obtained
* if the object has non-vacuous (non-trivial) initialization, its initialization is complete
* klement: the lifetime begin when the storage of the object is obtained and the initialization complete for non-trivial type.

**Lifetime Ends**:
* if `T` is a clas type with a non-trivial destructor, the  destructor call starts
  * note: when the destructor call starts instead of end
* the storage which the object occupies is released or reused by an object that is not nested

**Temporary Object**: from Scott Meyers "More Effective C++"
* Temporaries are invisible objects when non-heap objects are created during:
  1. Implicit conversion to make function call succeed.
    ```cpp
    size_t countChar(const string& str, char ch);
    char buffer[MAX_STRING_LEN];
    countChar(buffer);
    ```
    * Function succeed even though the argument is `char *` and param is `const std::string&` => Compiler create a temporary `std::string` from `char *` on the caller and pass it as a const-ref to the callee
    * Note: this will only work from const-ref params
  2. When function return objects. (RVO aside)
    * When a function return an object from the callee to the caller, the object will exist as a temporary on the caller. Then maybe move constructed into a local variable of the caller.
    * ie `S(S&&)` being called on the caller side, the `S&&` is an `rvalue` ref to the temporary (returned value)

## Gotchas

### Reference type is not an object type

Taking a reference to an object does not invoke any constructor as it is not an object => no lifetime.

```cpp
int main() {
  S s;
  {
    [[maybe_unused]] S& s2{s}; // S constructor is not invoked here
  }
}
```

### Returning reference to local (stack memory)

Returning a reference to a local object (ie on stack memory) is UB => compiler should emit a warning

```cpp
const int& get_data() {
  const int i = 5;
  return i;
}

int main() {
  std::cout << get_data();
}
```

**Caveat**: returning a `std::reference_wrapper` to local variable will confuse compiler
* Clang will not emit any warning 
* gcc gives `i` not initialized in function.

```cpp
const std::reference_wrapper<int> get_data() {
  const int i = 5;
  return i;
}

int main() {
  std::cout << get_data();
}
```

### String Literal is global storage

It is well defined to return a string literal(`const char *`)

```cpp
const char* get_data()
{
   return "Hello World";
}
int main() {
  std::cout << get_data();
}
```

It is also well defined for `string_view` to a string literal

**Caveat**: returning a `std::string_view` to a local `std::string` is UB
```cpp
std::string_view get_data() {
  std::string s = "Hello World";
  return s;
}

int main() {
  std::cout << get_data();
}
```
* This is due to the `std::string_view` having a longer life time than the object it is viewing
* note: there is no warning!

**Caveat**: `const char s[]` initialized with a string literal is a local data instead of a global data with static storage.

```cpp
std::string_view get_data() {
  const char s[] = "Hello World"; // local data!! not a pointer to static data
  return s; // decay to a pointer
}

int main() {
  std::cout << get_data();
}
```
* Note: no warning!

### Pushing back an element into vector

```cpp
int main() {
  std::vector<S> vec;
  vec.push_back(S{1});
}
```
* Order of functions:
  1. `S(int)`: `S` is constructed in `main`
  2. `S(S&&)`: `S` is move constructed from `main` into the `vector` underlying array
  3. `~S()`: `S` constructed in `main` has been moved into an "empty" but valid state => will need to be destructed
  4. `~S()`: the `S` in the vector will need to destroyed.

Note: directly replacing `push_back` with `emplace_back` will give the same function executions. It will call the move constructor instead in the underlying array.

### Temporaries

Const-ref to extends the lifetime of a temporary.

```cpp
S get_value() { return {}; }

int main() {
  // taking a reference to a temporary (return value on the calle stack?)
  const auto& val = get_value();
}
```
* Reference extends the lifetime of an object.
* Applies recursively to member initializers 

**Does not apply recursive function call**
```cpp
int main() {
  auto&& range = get_s().get_data();
}
```
* `auto&&` only extends the `S` returned from `get_s()` but not the data return from `get_data()` => `range` is a dangling reference.

### Initializer list

**Hidden Array**

When constructing using initializer list, the compiler will create a **const** array of the elements and then construct a `std::initializer_list` for it which will be passed as a rvalue ref to the constructor

```cpp
std::vector<std::string> vec{"a long string of characters", "b long string of characters"};

// compiler will generate

const std::string __data[] {"a long string of characters", "b long string of characters"};
std::vector<std::string> vec{std::initializer_list<std::string>{__data, __data + 2}};
```
* Array is const => not movable => possible more heap allocations

**Template deduction for string literal**

String literal are deduced as `const char*` instead of `std::string` in C++17 template type deduction => no heap allocation
```cpp
// no heap allocation
std::array a{"a long string of characters", "b long string of characters"};
```

Note: above gotcha of hidden array does not apply to `std::array`. `std::array` has **no constructor** and using initializer list syntax will directly initialize the underlying **c-style array**

### Ranged for loops

Decaying of ranged loops

```cpp
int main() {
  for (const auto& v : get_s().get_data()) {
    std::cout << v;
  }
}

// compiler generates
int main() {
  auto&& __range = get_s().get_data();
  auto __begin = begin(__range);
  auto __end = end(__range);
  for (; __begin != __end; ++__begin) {
    const auto& v = *__begin;
    std::cout << v;
  }
}
```
* no warnings by compiler

Solution: C++20 `for-init`
```cpp
int main() {
  for (const auto s = get_s(); const auto& v : s.get_data()) {
    std::cout << v;
  }
}
```

### if-init

If-init statements are visible for the else blocks as well

```cpp
if (const auto x = get_val(); x > 5) {
  ...
} else {
  ...
}

// same as

const auto x = get_val();
if ( x > 5 ) {
  ...
} else {
  // x is visible here
  ...
}
```

### RVO

Move constructor will be automatically called for rvalue refs (return of functions) 

```cpp
Holder get_Holder() { return {}; } // init Holder => S() called
S get_S() {
  S s = get_Holder().s; // r-value init s => S(S&&), ~S() in Holder called
  return s; // rvo applied
}
int main() {
  S s = get_S(); // nothing
} // ~S()
```

Returning a reference will not allow for RVO

### Structured binding

non-ref structured binding will copy the object and take reference to individual member. 
```cpp
S get_S() {
  auto [s,i] = get_Holder();
  return s;
}

// compiler generates
s get_S() {
  auto e = get_Holder();
  auto &s = e.s;
  auto &i = e.i;
  return s; // no RVO
}
```

### Destructor

If a constructor is not completed (throw in constructor), destructor will not be called. Except for delegating constructor.

```cpp
// no destructor called because constructor did not complete
struct S {
  int i{};
  S(int i_) : i{i_}
  { throw 1; } // constructor did not complete
  ~S() { puts("~S()"); }
};
int main() {
  S s{};
}

// destructor called if throw in delegating constructor
struct S {
  int i{};
  S(int i_) : S{i_}
  { throw 1; } // constructor did not complete
  ~S() { puts("~S()"); }
};
int main() {
  S s{};
}
```
* Can use delegating constructor to always call the destructor 
