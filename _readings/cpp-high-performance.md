---
title: "C++ High Performance"
toc: true
---

Title: C++ High Performance
Author: Bjorn Andrist and Viktor Sehr

# Chapter 1: A brief Introduction to C++

(klement: This chapter briefly explains C++ which most people should know but
contains some key insights that I have overlooked)

Zero-overhead principle by Bjarne:

- What you don't use, you don't pay
- What you do use, you couldn't hand code any better

Non-performance-related C++ language features
(klement: I am aware of these features but this section provides a very good
structured answer as to why C++. Could be useful in interviews.)

- Value Semantics:
  - Allows callers to be guaranteed that their provided arguments will not be
    modified as all arguments are copied.
- Cons correctness:
  - Provide const guarantees in compile time. Available in java (`final`) but
    it is more flexible as member functions can be delclared const
- Object ownership: with RAII and smart pointers we can clearly express the
object ownership. In Java all resource have shared ownership.
- Deterministic destruction:
  - Uses RAII for destruction which is deterministic.
- Avoiding null objects with references:
  - As references in C++ cannot be null and reassigned, functions that takes
  parameters as reference can be guaranteed that the argument is not a null
  object.

## Chapter 2: Essential C++ Techniques

(klement: Skipping the first few items as it is just a brief summary of effective/effective modern C++)

### Contracts

Definitions:
- Precondition: callers responsibility to make sure that the precondition are met.
- Postcondition: function responsibility to make sure that the postcondition are met.
- Invariants: condition that should always hold true. Could apply to loops (at the start and end of loop)
or class (state of the class)
  * Any time when we can observe the object externally, its invariant should hold.
  * An object can temporarily break the its invariant when transiting to states but once it returns/throw/invoke callback,
  it must have a valid invariant.
  * Implicit form of precondition and postcondition, holding invariant must happen before and after a function.
  * Moving an object will make it in an empty but valid state.
    * Empty state is not very useful as an invariant as an empty state is an invalid state. It will be better
    to make the empty (moved) state to be the default constructed state.

### Programming vs runtime error

* If precondition is not met (ie popping on empty vector), it is a programming error and an assertion should be used
  * Used to assert assumptions
* If an error is due to runtime problem (cannot load from disk) an exception could be thrown.
  * Exceptions should not be used to handle bugs but to recover from an invalid state in runtime.

### Exceptions

- A technique to signal that an error has occurred in the callee
  - It is the only way when there is an error in the constructor
- If a function does not throw an exception use `noexcept`.
  - Note: compiler does not guarantees that no exception will be thrown - up to the author
   to verify that there are no exception.
   - If a `noexcept` function throws an exception, it will call `std::terminate` and the program
   will terminate immediately instead of unwinding the stack.
- Writing exception safe code:
  - Use copy and swap idiom. Mutate a temporary obj, if succeed swap to the main object.
- Performance:
  - Costs:
    - Larger binary program => not zero-heard principle
    - Throwing and catching exception is exception (sad path) => runtime cost is not deterministic
  - In happy path (no throw) => performs quite well
  - Advantage of exception vs return code. For return code branching (`if (rc)`) will need to be performed
  on all paths (more cost on all paths) but not needed for exceptions.

### Lambda

(klement: skipping the first few parts as it is summary of effective modern c++)

Mutating lambda
* By default the call operator (`operator()`) is marked as `const` and cannot mutate the members of
the closure object. To mutate the members we will need to mark the lambda as `mutatable`
* `const`-ness of a function does not apply for reference members => lambda can modify variables captured
by reference without `mutatable`
* Capturing all: using `[=]` will only copy all variables inside of the lambda instead of all variables in the scope (compiler is smart)
* Lambda to regular function pointer - add `+` to the start for the lambda (ie `+[](){...}`) will create a lambda with a regular function pointer.
Note: this means that you cannot have any captures.
* Default constructable: in C++20 lambdas without captures but same signature are the sametime. We can also use `decltype` to make the types the same.
  * Traditionally all lambda expressions are different closure classes => different type.
  * For lambda with captures they are still different closure classes in c++20
  ```cpp
  auto x = []{};
  auto y = x;
  decltype(y) z;
  // x y and z are the same type
  ```

`std::function`
* Uses type erasure to be able to store lambda with the same signature (return and params) - the capture list does not matter
* `std::function` will **store and own** a callable object => implies copying a closure object => copying could have heap allocation.
* Performance cost:
  * Prevents inline optimisation. `std::function` uses type erasure and indirections to be able to execute the function => cannot be inlined
  * Possible heap allocation when assigned to lambda with captured variables/references. Will use heap-allocation to store the variables.
    * Allocation cost + heap memory access => cache miss
    * (klement: this seems very bad. Should not be used in hot path?)
  * Runtime cost: generally slower
* More info in this [blog post by vittorio romeo](https://vittorioromeo.info/index/blog/passing_functions_to_functions.html)

Generic lambda:
* Before c++20, lambdas can only be generic by using `auto` params but in c++20, lambdas can have explicit template arguments `[]<typename T>(){..}`

## Chapter 3

### Time Complexity

(klement: this part covers basic concepts on time complexity)

### Amortized time complexity

**Definition**: the time need to in almost all cases except some.

* Analyzing a *sequence* of operations rather than a single one.
* Still analyzing the worse case but for a sequence of operations.
* Computed by calculating the **total running time of an entire sequence** divided by **length of sequence**.

### Speedup of execution time

$$\text{Speedup of execution time} = \frac{T_old}{T_new}$$
$$\text{% improvement} = 100(1- \frac{1}{Speedup})$$

### Profiling

* Instrumentation profiler:
  * Adds profiling code to your source code. An example will be an RAII style timer.
  * Drawback: changes the binary and the timer itself could be the overhead. Compiler optimisation
  might be removed due to the new source code.
* Sampling profile:
  * periodically takes a sample of the program => can generate a flame graph of the call stack.
  * Disadvantage: will not be able to profile sleeping threads

### Micro Benchmarking

* After knowing where the 20% of the code that is running for 80% of the time, create a microbench mark
for that part of the code.
* With microbench mark you can fine tune the code and try to make it faster

### Amdahl's law

$$\text{Overall speedup}=\frac{1}{(1-p)+\frac{p}{s}}$$

* $$p$$: represent the fraction of the entire program execution time that the function execute for.
* $$s$$: represent the speed of the new function.

Takeaways: No matter how fast the speedup, the entire improvement of the program is limited by $$p$$
