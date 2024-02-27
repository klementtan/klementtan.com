# C++ 20 Complete Guide

## 6 Ranges

#### 6.1.2 Contraints

Range parameters are template args (no common type) that are constrained by concepts.

`std::range` concept: you must be able to iterate through a elementes with `std::ranges::begin` and `std::ranges::end`

concepts:
* `range`: iterated from begin to end
    * base concept for all ranges - all other ranges subsume this
* `output_range`: range of elements to write values to
* `input_range`: range to read values from
* `forward_range`: iterate over the element multiple times
* `bidirectional_range`: iterate forward and back
* `random_acess`:
* `continguous_range`: contigous memory
* `sized_range`: constant time `size()`

* directional range (output/input) follows the hierachy of the normal iterators

* `view`: ranges that arecheap to copy or move and assign
* `viewable_range`: range that can be converted to a view (with `std::ranges::all`)
* `borrowed_range`: iterators not tied to the lifetime of the range (what this means?)
* `common_range`: begin and end have the same type

### 6.1.3 Views

**views**: lightweight range that are cheap to create and copy/move

**range adaptors**: create  view and could perform some operation on the view

Views needs to support:
* `ranges::begin`, `ranges::end`, `operator++`, `operator*`

**Generating views**: iota

**Lifetime**:
* Uses reference semantics
* Ensure the views' referred range lifetime still exists

Writing views:
* Views use reference semantics so it can modified the underlying range

**Lazy Evaluation**: uses lazy evaluation, processing of the next element only done in `begin()` or `++`

**Caching**: optimisation could happen if we cache the return value of the operation (ie expensive transform)
* Concurrent iteration could be a problem

### 6.1.4 Sentinels

**Sentinels**: represent the end of range
* traditionally, the end of a stl container is `.end()` iterator
    * drawback: this means we traditionally need to have begin and end iterator to have the same type
    . this might be expensive or not possible. ie cstring will be expensive as it needs to find the null char
    * not possible when it is an input stream of file content
        * actually possible but need to create a an end of file iterator

having different end iterator type:
* don't need to reach the end before process
* prevent UB of dereferencing `end()`

Defining a sentinel:
* define a `operator==` that takes the `begin()` operator but don't need to be the same type
* just need `operator==` as `operator!=` will be automatically generated

### 6.1.5 Defining range

**subrange**: defined by passing the begin and end iterator to `std::ranges::subrange`
* subrange is actually a view of the provided range

### 6.1.6 Projections

most ranges algorithm takes an additional `Proj` template that will transform the element before executing the algorithm

### 6.2 Borrowed Iterators and Ranges

Passing range as a single argument means there could be lifetime issue when the range is a temporary
* for example if you return an iterator to a temporary passed to a function arg, the returned iterator will point to an already destroyed temporary
* passing a temporary range to `std::find` -> return an iterator to the temporary range
* 

`std::ranges::borrowed_iterator_t<Rg>` borrowed iterator: 
* ensures that the lifetime of the iterator is not dependent on a temporary range (prvalue)
    * deferencing the `borrowed_iterator` is always valid
* algorithm can return a `borrowed_iterator`: depending on the provided range (temporary or non-temporary) the returned iterator can be de-referenced or not (compile error)
* algorithm can always return an iterator that is safe to use
    * if it would dangle then a special return value will be used
* using borrowed iterator:
    * if the arg is a temporary (`prvalue`) the return type will be `std::ranges::dangling`

#### 6.2.2 Borrowed Ranges

concept: `std::ranges::borrowed_range`

Borrowed Range:
* Definition:
    * when the iterator of the range does not depend on the lifetime of the range
    * Or when the range is and lvalue
    * is a concept
* (klement): what is the actual implementation of the borrowed range concept
* What this means:
    * iterators of a range can be used after the range is destroyed
* borrowed range is usually a subrange/view of another range
* getting the iterator of the borrowed range outlives the actual borrowed range itself
    * underlying range/container => borrowed range that is a view of the underlying range/container => borrowed range's iterator that is still valid if the borrowed range is destroyed
* all lvalues are borrowed range: the iterator on the lvalue ref is still valid eventhough the lvalue ref might be out of scope
    * what actually is a borrowed range?
* main idea of a borrow range is if the iterator of the borrow range does not care about the life time of the borrowed range - hence the name borrowed because it is not actually a range

for custom user type, add `std::ranges::enable_borrowed_range`

### 6.3 Views

views: light weight ranges

**range adaptor**:
* create a view from passing a range in a param
* functors

**range factory**:
* creates a view without passing an existing range
* functors (aka lambda)

standard library provides some general range adaptors or range factories that create these views
* all in `std::ranges::views` or `std::views` standard library

source views:
* functors that create a view from an underling resource (ie vector) or generate from nothing

copy and move: all views should provide constant time copy and move

#### 6.3.1 Views on ranges

* Containers and strings are not views because it is expensive to copy and move
* Converting a range (ie container to a view):
    * by passing it to `std::ranges::views::all`
    * by passing `begin`/`end`/`size` to `subrange` or `counted`
    * by passing it as an argument to one of the range adapters
        * adapters will implicity convert the container or range to `views`
        * passing a lvalue container to an adapter will create `ref_view` wrapper -> it is a non-owning
        * passing a rvalue temporary container to an adapter will create `owning_view` 
            * ie `std::vector<int>{1,2,3} | std::views::take(5)`

**Views use lazy evaluation**

**Caching Views**:
* for some views (adapter) we will cache the `begin` iterator to allow a second iterator to be faster
* example: `filter`, `drop`, `drop-while`, `reverse` will cache the iterator 
* a consequene is that modifying a range after calling `begin` (ie through iteration) will result in unexpected results
* it will require a range to be non-const (to cache the range)

**Performance impact of filter**
* iterating through a range 2 step process
    1. Calcuating the position/iterator (++/begin)
    2. Use `*` to deference the iterator
* Filter impact:
    * For calculating the iterator to be used, it will need to derefence the passed range
        * this could be expensive if the previous adapter is transform
    * a second deference of the previous iterator when we try to iterate through the final range



## 14 Coroutines

### 14.1 What are Coroutines

Coroutine
* A function that can pause/resume its execution
* Coroutine **does not** mean that the both the caller and coroutine runs in parallel
    * but it is possible for both to run on different thread

Coroutine syntax:
* function that contains `co_await`, `co_yield`, `co_return`
* If a function does not have any of those keyword, use `co_return`

Return of Coroutine: most coroutine returns a object as *coroutine interface* for the caller. The interface can represent
* A running task that suspends or switches context
* Generator that yields values from time to time
* Factory that retunrs one or more values lazily

### 14.2 Couroutine Example

```cpp
#include <iostream>
#include "corotask.hpp" // for CoroTask
 CoroTask coro(int max)
 {
    std::cout << "         CORO " << max << " start\n";
    for (int val = 1; val <= max; ++val) {
        // print next value:
        std::cout<<" CORO"<<val<< / <<max<< \n;
        co_await std::suspend_always{}; // SUSPEND 
    }
    std::cout << " CORO " << max << " end\n";
}
int main() {
    // start coroutine:
    auto coroTask = coro(3);
    std::cout << "coro() started\n";
    // loop to resume the coroutine until it is done:
    while (coroTask.resume()) {
        std::cout << "coro() suspended\n";  
    }
    std::cout << "coro() done\n"; 
}
```
* `co_await` expression: suspends the coroutine ("blocks") and pass the control flow back to caller
    * called a **suspend point**
    * the behaviour of **suspend point** is dependent on the next expression (ie `std::alway_suspend`)
* return type: `CoroTask`: coroutines do not `return` statements but still need to have `return` type to define the coroutine interface
* Calling coroutine: do not wait for function to end
    * depending on the interface type (`CoroType`), calling a coro might have an implicit starting suspend point (immediately pass the control back to the caller)
* `resume`: pass the control flow back to the coroutine until the next suspend point
 * multiple coroutine: the same coroutine can be called twice will lead to two separet coroutine (execution flow)
* lifetime: coroutine live longer than the first call to the coroutine to create the coroutine:
    * the lifetime of args provided might not always be valid
    * in general don't use references to declare coroutine parameters
    * ie:
        ```cpp
        #include <iostream>
        #include "cororef.hpp"
        int main() {
            auto coroTask = coro(3);
            std::cout << "coro(3) started\n";
            coro(375);
            std::cout << "coro(375) started\n";
            // loop to resume the coroutine until it is done:
            while (coroTask.resume()) {
                std::cout << "coro() suspended\n";
            }
            std::cout << "coro() done\n"; 
        }
        ```
* coroutine can call other coroutine:
    * calling `resume` on a coroutine only resume the callee and does not transitively call resume on all nested coros
* Delegating `resume()`:
    * Calling `co_await coro()`: means that current coroutine delegate the control flow the sub coroutine.
    * Calling `resume` on the outer coroutine will call resume on the inner coroutine and inner coroutine suspending will pass the control flow back to main caller
    * The coroutine becomes "invisible" until the inner coroutine ends
    * Caveat: the inner coroutine needs to be `awaitable`

### 14.2.5 Impelmenting Coroutine Interface

* **promise type**: customization points for dealing with a coroutine
* **coroutine handle**: 
    * object created when coroutine is called
    * allow to manage the state of the coroutine by providing low-level interface to resume a corotuine

```cpp
#include <coroutine>
// coroutine interface to deal with a simple task
// - providing resume() to resume the coroutine

class [[nodiscard]] CoroTask {
public:
    // initialize members for state and customization:
    struct promise_type; // definition later in corotaskpromise.hpp
    using CoroHdl = std::coroutine_handle<promise_type>;
private:
    CoroHdl hdl; // native coroutine handle
public:
    // constructor and destructor:
     CoroTask(auto h)
      : hdl{h} {
     }
    ~CoroTask() {
        if (hdl) {
            hdl.destroy();
        }
    }
    // don’t copy or move:
    // native coroutine handle
    // store coroutine handle in interface
    // destroy coroutine handle
    CoroTask(const CoroTask&) = delete;
    CoroTask& operator=(const CoroTask&) = delete;
    // API to resume the coroutine
    // - returns whether there is still something to process
    bool resume() const {
      if (!hdl || hdl.done()) {
          return false;
        }
        hdl.resume();
        return !hdl.done();
    } 
};
// nothing (more) to process
// RESUME (blocks until suspended again or the end)
#include "corotaskpromise.hpp" // definition of promise_type ⌃
```

Key Components of an interface:
* needs to define the type `promise_type`: the compiler needs to see this typedef
* The handle of the coroutine will be specialised on the `promise_type`
    * `std::coroutine_handle<>`
    * any data stored in the promise type is part of the handle
    * The handle is not required to be the member or have a certain typedef
    * the glue with the compiler is that the compiler will construct the coroutine interface with the `std::coroutine_handle`? No, depends on how the promise type is implemented - need to somehow store the handle in the interface
```cpp
class [[nodiscard]] CoroTask {
  public:
    // initialize members for state and customization:
    struct promise_type; // definition later in corotaskpromise.hpp
    using CoroHdl = std::coroutine_handle<promise_type>;
  private:
    CoroHdl hdl; // native coroutine handle
    ...
};
```

Constructor and destructor of the coroutine:
* initialized with the handle and incharge of cleaning up the handle
```cpp
class CoroTask { ...
public:
    CoroTask(auto h)
    : hdl{h} { } // store coroutine handle internally
    ~CoroTask() {
        if (hdl) {
            hdl.destroy();
            // destroy coroutine handle (if there is one)
        } 
    }
    ...
};
```

Resume: the key part in determining if the coroutine should resume
```cpp
class CoroTask { ...
    bool resume() const {
        if (!hdl || hdl.done()) {
            return false;
        }
        hdl.resume();
        return !hdl.done();
    } 
};
```
* This resume propagate the resume to the native handle and check if it makes sense to resume
* Checks if the handle is present or ended
    * need to check handle if the coroutine supports move semantics
* Calling resume on `std::coroutine_handle`:
    * precodition: coroutine is suspended and has not reached the end (`!hdl.done()`)
    * Can be called using `operator()` ie `hdl()` instead of `hdl.resume()`;

#### `promise_type`:
* Purpose:
    1. Define how to create or get the underlying return value
    2. Decide if the coroutine should suspend at its begining (implicit suspend) or end.
    3. Deal with value exchanged between caller and coroutine
    4. Handle exceptions

```cpp
struct CoroTask::promise_type { 
    auto get_return_object() { // initial and return the coroutine interface
        return CoroTask{CoroHdl::from_promise(*this);};
    }
    auto initial_suspend() { // initial suspend point
        return std::suspend_always{}; // suspend immediately
    }
    void unhandle_exception() { // deal with exceptions
        std::terminate;         // terminate the program
    }
    void return_void() { // deal with the end or co_return;

    }
    auto final_suspend() noexcept {     // final suspend point
        return std::suspend_always{};   // - suspend immediately
    }
}
```
* `get_return_object`: used to initialize the coroutine interface
    * The promise itself is automatically created (default constructed?)
    * Steps:
        1. Create a native coroutine handle from the `promise_type`
            * `coroHld = CoroHdl::from_promise(*this);`
        2. Using the handle, it will create the coroutine interface
            * `coroIf = CoroTask{coroHdl};`
        3. Return the interface object `return coroIf;`
    * Could return the native coroutine handle - the coroutine interace automatically be constructed by the handle
* `initial_suspend()`: defines if a coroutine should start lazy / eager
    * `std::suspend_never{}` the coroutine starts eagerly
        * immediately execute the statements until the first co_await
    * `std::suspend_always{}` the coroutine starts lazily
* `return_void()`: defines teh reaction when reaching the end (or `co_return`)
    * if function is define, the coroutine cannot return / `co_yield`
* `unhandle_exception()`: how to deal with exceptions not handled locally in the coroutine, exception that is not caught
* `final_suspend()`: states if the coroutine should be finally suspended.

####  14.2.7 Memory Management

* Coroutine handles are cheap and do not manage memory.
* The coroutine interface will get the handle and will need to delete the coroutine.

### 14.3 Coroutine That Yield or Return Values

`co_yield`: yield intermedaite result when it is suspended (`co_await` just suspends)
```cpp
CoroGen coro(int max)
{
    for (int val = 1; val <= max; ++val) {
        co_yield val;
    }
}
```
* When `co_yield` is called, the coroutine will be suspended with a `val`
* Underthehood, the coroutine frame maps this to a val to `yield_value()` for the promise type
    * the coroutine interface should then store this value as a member in the promise type
    * `yield_value()` is a function that is called by compiler
* The users will need to implement a way to save the args passed to `yield_value()` and return it in the corotuine
interface later (`getValue()`)
    * luckily the native `std::coroutine_handle` provide a walt to get the promise (`promise()`)

Example:
```cpp
struct promise_type {
    int coroValue = 0;
    auto yield_value(int val) {
        coroValue = val; -- store value locally
        return std::suspend_always{}; -- suspend
    }
}
```

Caller:
```cpp
#include "coyield.hpp"
#include <iostream>
int main() {
    // start coroutine:
    auto coroGen = coro(3);
    std::cout << "coro() started\n";
    // loop to resume the coroutine until it is done:
    while (coroGen.resume()) {
        auto val = coroGen.getValue();
        std::cout << "coro() suspended with " << val <<  \n ;
    }
    std::cout << "coro() done\n";
}

struct CoroGen {
    ...
    int getValue() const {
        // return the value that was saved in the promise
        return hdl.promise().coroValue;
    }
}
```
* 
