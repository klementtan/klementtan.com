# C++ 20 Complete Guide

## 6 Ranges


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
