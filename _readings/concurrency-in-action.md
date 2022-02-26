---
title: "Readings: Concurrency in Action"
excerpt: "Notes for Concurrency in Action"
toc: true
---

Title: Concurrency in Action Second Edition

Author: Anthony Williams

## General Review

## Chapter 1: Hello, world of concurrency in C++!

klement: This chapter gives a brief overview of what is concurrency on the OS level and what
it means in C++.

### What is concurrency?

**Definition**: two or more separate activities happening at the same time.

**Concurrency in computer system**:
* A single system performing multiple independent activities in parallel.
* Computers with a single processing unit or core
  * Can only perform one task at a time.
  * They can achieve concurrency by performing *task switching*: continuously switch between task but at any moment
  only executing one task.
  * Has to perform context switching at every task switch which will add to the overhead
* Hardware Concurrency:
  * The have multiple processing unit (core) that allows them to execute multiple task across different core
  * At any moment can be executing more than one task.
  * AKA: genuine hardware concurrency
* Number of Hardware Threads: corresponds to the number of processing unit available for genuine concurrency.
* Concurrency is achieved by task switching or genuine concurrency.

**Concurrency with multiple process**:
* Divide the application into multiple, separate single-threaded process that run at the sametime
* Process  will need to communicate through IPC channel (signals, sockets, files, pipes, ...)
* Disadvantages:
  * OS provide a lot of protection between process -> slow
  * Start up time for process is slow
  * C++ standard does not provide any support for IPC
* Advantages:
  * Added protection -> easier to write safe and concurrent code
  * Separate process can run on different machine connected over a network
    * Longer communicate cost 
    * Allow for horizontal scalling

**Concurrency with multiple thread**:
* Running multiple thread in a single process
* Each thread will run independently and different sequence of instructions.
* Shares global variable, pointers
  * More complicated between processes as memory address for the same data might be different
* 
Shared address space will counter the disadvantage of process

**Concurrency vs Parallelism**:
* Concurrency: concerned about **separation of concerns** and **responsiveness**
* Parallelism: Taking advantage of available hardware

### Why use concurrency

**Separation of concern**:
* Separate distinct areas of functionallity
* Allow separate functionallity to be executed concurrently which will give the illusion of responsiveness.
* Number of threads being used is related to the number of distinct areas and not the number of processing unit.
* Not concern about the throughput

**Performance: task and data parallelism**
* Task parallelism:
  * Divide the a task into multiple tasks that can be executed simultaneously
* Data parallelism:
  * Each thread will perform the same tasks on different set of data
* Embarrassing parallelism: algorithms that are susceptible to task and data parallelism.
  * Good scalability probabilities: more hardware threads -> parallelism can be increased.
* Increase throughput:
  * Use parallelism to execute on more data. Will not decease the latency but will increase the throughput.

**When not to use concurrency**:
* Concurrency code is harder to understand
* Only use when performance gain is large enough or separation of concerns is clear enough
* Using too many threads can exhaust the available memory
  * Each thread will need to have its own address space for the stack -> 32 bit system can only have
  4GB of memory and 4,096 threads.
* Only worth it for performance critical part of the code

### Concurrency and multithreading in C++

* C++11:
  * A thread aware memory model
  * Classes for managing thread, protecting shared data, synchronization operation between them and low level atomic
  operation
* C++14 and C++17:
  * Parallel algorithms

**Efficiency in C++ thread library**:
* Abstraction penalty:
  * The cost associated with having a high-level facilities
* Thread library design goals: should be little or no benefit to be gained from using the lower-level APIs
directly
* Only obscure cases will incur abstraction penalty.
* Uses "you don't pay for what you don't use"

### Hello Concurrent World

```cpp
#include<iostream>
#include<thread>

void hello(){
  std::cout << "Hello Concurrent World";
}
int main() {
  std::thread t(hello);
  t.join();
}
```
* All thread needs an initial function that starts the control flow
  * Main thread uses `main` and `t` uses `hello`.

## Chapter 2: Managing threads

### Basic thread management

* Launching a thread:
  * Starting a thread boils down to creating a thread object with a callable
  object and the arguments
  * Callable object:
    * Functor (`operator()` member function): 
      * Might be susceptible to "C++ most vexing parse": cannot instantitate a functor using
      `Functor()`
    * Lambda expression.
  * Must decide to finish the thread `join` or leave it to run on its own (`detach`) else the
  program will terminate.
  * Detach drawbacks: dangling reference
    * The thread will live beyond the function of the caller
    * The thread cannot have reference to the caller stack memory which will result in dangling reference
      if the function returns before the thread completes.
    * Mitigation: make thread self-contained and copy data into the thread.
* Waiting for thread to complete:
  * Call `join`: waits on thread and cleans up any storage associated with it.
  * Use **RAII** to call `join` on the thread on all paths.
    ```cpp
        class thread_guard{
          std::thread& t;
          public:
            explicit thread_guard(std::thread& t_): t(t_) {}
            ~thread_guard(){
              if (t.joinable())
              {
                t.join();
              }
            }
        }
    ```
      * Should test if joinable before joining as thread_guard holds a reference and the underlying thread
      could be joined before destruction.
* Running threads in background:
  * Use `detatch`.
  * Also useful for running multiple independent inner application.

### Passing arguments to a thread function

Life cycle:
1. Args are *copied into internal storage* of the thread
2. Passed to the callable object by *rvalue* in the newly created thread.

Note:
* This is done even if the parameter is expecting a reference. Will still invoke a copy operation.
* The arguments are copied according to the supplied arguments and not the parameter
  * Affects automatic variable.
  * ie: Pass a string literal (`const char*`) to a `std::string` will result in string literal (`const char *`)
  being copied into internal storage then constructed to `std::string` when the callable object is called.
  * Problem: automatic conversion of arguments could result in dangling reference if the thread
  detaches and converts the arguments only after the arguments has been destructed.
    ```cpp
    void f(int i, std::string& s);
    void ooops() {
        char buffer[1024];
        std::thread t{f, 3, buffer};
        t.detach;
    }
    ```
    * Lifecycle: `buffer` gets copied into internal thread storage as `const char *` then converted to 
    `std::string` when `f` is called.
    * A dangling reference could occur when the `oops` return before the `const char *` to `std::string`
    conversion occurs. This will result in converting an invalid `const char *` to `std::string`.
    * Solution: pass the arguments by non automatic variable (cast `buffer` to `std::string`). 

Parameter reference:
* For functions that are passed by reference, you should wrap the arguments with `std::ref` so that
the arguments are references and not copied.

Similarity to bind:
* You can make a thread call a member function by using the member function pointer and object pointer
```cpp
class X{
public:
  void do_work();
}
X x;
std::thread t(&X::do_work, &x);
```

Move only parameter:
* Use `std::move` when providing the arguments.

### Transferring ownership of a thread

* Use `std::move` when transferring ownership of a thread
* Move assignment with `std::move` will terminate the program if `std::thread`
is associated with a current thread.

Joining Thread
```cpp
class joining_thread {
  std::thread t;
public:
  joining_thread() noexcept = default;
  // Copy std::thread constructor
  template<typename Callable, typename... Args>
  explicit joining_thread(Callable&& func, Args&&... args): 
  t(std::forward<Callable>(func), std::forward<Args>(args)...)
  {
  }

  // Construct from std::thread
  explicit joining_thread(std::thread t_)  noexcept :
  t(std::move(t_)) {}

  explicit joining_thread(std::thread&& t_)  noexcept :
  t(std::move(t_)) {}

  joining_thread& operator=(joining_thread&& other) {
    if (joinable()) {
      join();
    }
    t = std::move(other.t);
    return *this;
  }

  joining_thread& operator=(joining_thread& other) {
    if (joinable()) {
      join();
    }
    t = std::move(other.t);
    return *this;
  }
  ~joining_thread() {
    if (joinable()) join();
  }
  // other helper methods...
}
```

### Choosing the number of threads at runtime

Use `std::thread::hardware_concurrency()` for the number of threads that can truly run concurrently.

Parallel `std::accumulate` (using `operator+` as binary ope) that choose the number of threads at runtime

```cpp
template<typename Iterator, typename T>
struct accumulate_block{
  void operator()(Iterator first, Iterator last, T& result) {
    result = std::accumulate(first, last, result);
  }
}

template<typename Iterator, typename T>
T parallel_accumulate(Iterator first, Iterator last, T init) {
  size_t length = std::distance(first, last);
  if (!length) {
    // if no item to accumulate then return initial
    return init;
  }

  // minimum number of items per thread
  size_t min_per_thread = 25;

  // maximum number of threads if all threads process the minimum number of items
  size_t maximum_threads = (length + min_per_thread  -1) / min_per_thread;

  // number of hardware threads
  size_t hardware_threads = std::thread::hardware_concurrency();

  // number of threads that should be spawned (inclusive of main thread)
  size_t num_threads = std::min(
    hardware_threads == 0 ? 2 // defautl to 2 if cannot determine the number of hardware thread
      : hardware_threads
    , maximum_threads
  ); // choose the minimum of the hardware threads or the maximum number of threads based on problem size
  
  std::vector<T> result (num_threads); // T needs to be default constructed
  std::vector<std::thread> thread(num_threads - 1); // container for new threads spawned
  size_t block_size = length/num_threads; // the number of items each new thread should process

  Iterator block_start = first;
  for (size_t i = 0; i < (num_threads - 1); i++) {
    // each thread will process items between [block_start, block_start + block_size] 

    // set the bounds
    Iterator block_end = block_start;
    std::advance(block_end, block_size); 

    thread[i] = std::thread(
        accumulate_block<Iterator, T>(),
        block_start, block_end, std::ref(results[i])
    ); // give work to new thread
    block_start = block_end; // for the next thread
  }

  // main thread will accumulate the last block
  accumulate_block<Iterator, T>()(block_start, last, result[num_threads - 1]);

  for (auto& entry : threads) entry.join();

  // accumulate on the blocks
  return std::accumulate(result.begin(), result.end(), init);
}
```

### Identifying threadso
* Use `std::this_thread::get_id()`
* Useful if the main thread needs to perform similar but slightly separate work.
* Defaults to `std::thread::id` 

