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

### Identifying threads
* Use `std::this_thread::get_id()`
* Useful if the main thread needs to perform similar but slightly separate work.
* Defaults to `std::thread::id`

## Chapter 3: Sharing data between threads

### Problems with sharing threads

**Invariants** definition: statement that are always true about a paritcular data structure.
* Doubly linked list: If the next pointer of node `A` is `B`, then the previous pointer of `B` must be `A`
* Broken invariants usually occur mid way through updating a data structure.

#### Race Conditions

**Generic Definition**: outcome depends on the relative ordering of execution of operations on two or more threads.

**Benign race condition**: when the invariant of the data structure is not broken

**Problematic race condition**: when the invariant of the data structure is broken
* Data race: concurrent modification of a single object (UB)
* Usually occurs when an operation requires modifying two or more distinct pieces of data.

### Protecting shared data with mutex

**Using mutex**
* Create a mutex with `std::mutex`
* RAII-fy mutex with `std::lock_guard` which locks on construction and unlocks on destruction.
* Best practice is to group the mutex and data in a class instead of as global variables.
* Locking a `std::mutex` already held is UB

**Structuring code for protecting data**
* Shoud not pass a pointer or reference to the protected data **out** of the function (return value)
* Should not pass a pointer or reference to the protected data **into** other function (arguments to other functions)
  * Other functions can save a pointer to the data.

**Spotting race conditions inherent in interfaces**: (thread safe-stack)
* Problem: calling `s.empty()`, getting the top value `s.top()` and lastly `s.pop()`. As, `s.empty()` and `s.top()` are separate operations, the stack could be poped after the empty check but before `top()`.
* Solutions:
  * Combing `top()` and  `pop()` (similar to python)
    * When the value of the top element is returned to the caller, it will copy constructed into the call site code. If the copy constructor throws an exception, the top value will be lost forever.
  * Pass in a reference (out-param)
    * Instead of returning the top value, you can pass an out-param for the function to assign the top value to.
    * This will allow the stack to make the `top` and `pop` operation to be atomic.
    * Cavaets:
      * Need to default construct the object first and pass the reference to `pop`
    ```cpp
    std::vector<int> resutlt;
    some_stack.pop(result);
    ```
  * Require a no-throw copy constructor or move constructor
  * Return a pointer to the popped item.
    * Allocate the top object on the heap and returning a pointer to it.
  * Example:
    ```cpp
    template<typename T>
    class threadsafe_stack
    {
    private:
      std::stack<T> data;
      mutable std::mutex m;
    public:
      threadsafe_stack() {}
      threadsafe_stack(const threadsafe_stack& other)
      {
        std::lock_guard<std::mutex> lock(other.m);
        data = other.data;
      }
      threadsafe_stack& operator=(const threadsafe_stack&) = delete;
      void push(T new_value)
      {
        std::lock_guard<std::mutex> lock(m);
        data.push(std::move(new_value));
      }
      std::shard_ptr<T> pop()
      {
        std::lock_guard<std::mutex> lock(m);
        if (data.empty()) throw empty_stack();
        std::shared_ptr<T> const res(std::make_shared<T>(data.top()));
        data.pop();
        return res;
      }
      void pop(T& value)
      {
        std::lock_guard<std::mutex> lock(m);
        if (data.empty()) throw empty_stack();
        value = data.top();
        data.pop();
      }

      bool empty() const
      {
        std:::lock_guard<std::mutex> lock(m);
        return data.empty()
      }
    }
    ```

### Deadlock: the problem and solution

**Deadlock**: occurs when having to lock two or more mutexes in order to perform an operation.

**Solution**:
* Lock the mutexes in the same order
  * Not straightforward when the mutex belongs to two instances of the same class. (Cannot order the mutexes)
* `std::lock`: can lock two or more mutexes at once (all or nothing semantic)
  ```cpp
  friend void seap(X& lhs, X& rhs) {
    if (&lhs != &rhs) return;
    std::lock(lhs.m, rhs.m);
    std::lock_guard<std::mutex> lock_a(lhs.m, std::adopt_lock);
    std::lock_guard<std::mutex> lock_b(lhs.m, std::adopt_lock);
    swap(lhs.some_details, rhs.some_details);
  }
  ```
  * Check that the address are not the same to prevent double locking of the same mutex
  * `std::adopt_lock` used to indicate the lock guard should adpot ownership of the lock.
  * `std::scoped_lock guar(lhs.m, rhs.m)` that does the `lock` with RAII-fying it.
  * Does not help if the lock should be acquired separately.
* Avoid nested locks:
  * Avoid acquring a lock if you already have one. This will ensure that each thread has a most one lock and will not lead to deadlock.
* Avoid calling user supplied code while holding a lock
* Acquiring locks in fixed order (Bad thread-safe linked list)
  * Uses hand-over-hand locking style. To read the `ith` node you will need to lock `[0, i-1]` node.
  * If the ordering of locking the nodes are not guaranteed, a forward iterator and backward iterator could progress and have a deadlock halfway.
* Use lock hierachy:
  * A special kind of lock that ensures a total ordering of locks you can acquire.
  * After locking a mutex with hierachy `k`, you can only lock mutexes with hierachy `<k`
  * This will ensure the ordering of nested lock and the same for all threads.
  * Hierarchical mutex
    ```cpp
    class hierarchical_mutex
    {
      std::mutex internal_mutex;
      unsigned long const hierarchy_value;
      unsigned long previous_hierarchy_value;
      // thread_local -> the static variable can be different for different threads 
      // but same throughtout different instances on the same thread
      static thread_local unsigned long this_thread_hierarchy_value;

      void check_for_hierarchy_violation()
      {
        // checks against the static variable
        if (this_thread_hierarchy_value <= hierarchy_violated) {
          throw std::logic_error("mutex hierarchy violated")
        }
      }

      void update_hierarchy_value()
      {
        previous_hierachy_value = this_thread_hierarchy_value;
        this_thread_hierachy_value = hierarchy_value;
      }

    public:
      explicit hierchical_mutex(unsigned long value) :
        hierarch_value(value), previous_hierarchy_value(0) {}
      
      void lock()
      {
        check_for_hierarchy_violated();
        internal_mutex.lock();
        update_hierarchy_value;
      }

      void unlock()
      {
        if (this_thread_hierarchy_value != hierarchy_value)
          throw std::logic_error("mutex hierarchy violated")
        this_thread_hierarchy_value = previous_hierarchy_value;
        internal_mutex.unlock();
      }
      bool try_lock() {
        check_for_hierarchy_violation();
        if (!internal_mutex.try_lock())
          return false;
        update_hierarchy_vlaue();
        return true;
      }
    }
    thread_local unsigned long hierarchical_mutex::this_thread_hierarchy_value(UINT_MAX);
    ```
  * Flexible locking with `std::unique_lock`:
    * Unlike `lock_guard`, `unique_lock` does not nee dto always lock the mutex
    * Can pass `std::adopt_lock` to lock the object on construction or `std::defer_lock` to make the lock `unlocked` upon construction. The lock can be acquired later by calling `lock` or `std::lock`
    * `unique_lock` is move constructable 
    * Cost:
      * Stores a flag to indicate if the mutex is locked or unlocked. 
      * Need to know during destruction, if mutex lock then should unlocked if mutex is unlocked then do nothing

**Locking at an appropriate grannularity**
* Lock a mutex only when accessing shared data
* Any processing  of data should be done outside of the lock

### Alterantives facilities to protecting shared data

**Protecting shared data during initialization**
* Static variable initialization:
  * Pre C++11: static variable are initialized the first time control (imperative code order) flow passes through its declaration. Multiple thread may believe that they are the first causing multiple concurrent initialization of the same variable.
* Lazy initialization of a singleton class (DB connection)
  * Requires checking if the shared data is initialized, then initializing it if it is not present
* Solutions:
  * Locking before the initiliazation check, unlocking after the initializaiton 
    * Expensive as it requires lock acquisation even after initialization 
  * Double locking:
    * pointer is first read without acquiring the lock. If pinter is present, return the pointer
    * Else acquire the lock and check the pointer again. Return the pointer if present (the pointer just initialized)
    * Else initialize the data
    * UB: The read outside the lock is not synchronized with the write outside the lock. We do not know that the read and write of pointer is single instruction.
  * Call once:
    * Using `std::call_once` that makes sure that a function is called once. Uses an additional `std::once_flag` to remember the state of the once
    ```cpp
    class X {
    private:
      connection_info connection_details;
      connection_handle connection;
      std::once_flag connection_init_flag;
      void open_connection () {
        // expensive operation
        connection = connection_manager.open(connection_details);
      }
    public:
      X(connection_info const& connection_details_) :
        connection_details{connection_details_} {}
      
      void send_data(data_packet const& data)
      {
        std::call_once(connection_init_flag, &X::open_connection, this);
        connection.send_data(data);
      }
      data_packet receive_data()
      {
        std::call_once(connection_init_flag, &X::open_connection, this);
        return connection.receive_data();
      }
    }
    ```
    * `std::call_once` makes sure that the expensive initialization is only called once.

#### Protecting rarely updated data structure (Read-Writer)

Problem: There are data structure that are rarely updated by writers (exclusive access) and access a lot by readers (no writers)

`shared_mutex`:
* Allow for easy reader and writer synchronization
* Readers: Use `std::shared_lock<std::shared_mutex>` to obtain shared access.
* Writer: Use normal `std::lock_guard` or `std::unique_lock`
* If a thread has a shared lock then any thread trying to get exclusive access (`std::lock_guard`) will be blocked and vice versa

#### Recursive Lock

* Allow for multiple locking of the same mutex.
* Must release all locks of the same mutex before other threads can lock it.
* Is a code smell but could be useful when all public methods need to acquire lock and some public method need to call another public method.

## Chapter 4: Synchronizing concurrent operations

**Futures** summary:
* `std::future`:
  * Represents a future result that might not be available now.
  * Threads can block on the future until the result is available.
  * Must have an opposite end that provides the future result
* Opposite ends of a future:
  * `std::async`: a function that is invoked in a separate thread (depending on policy)
  and the return result is the future result
  * `std::package_task`: a packaged function object can be invoked in the future explicitly
  . The return result is also the future result
  * `std::promise`: an object that can set the future's result with `set_value()`

### Waiting for an event or condition

Problem: thread A needs to wait for some condition to be true where that condition can be set to
true by thread B

Bad Solutions:
1. Busy waiting:
  * thread A continuously checks if the condition changes to true
  * As the condition is shared condition between thread A and B, will most probably need
  a mutex to make sure there is no data race between read and writing to the condition
  * Problems:
    * Thread A could hold on to the lock indefinitely. Thread A holds the mutex -> continuously checks
    that the condition is false -> threads B wants to set condition to true -> lock the already locked
    mutex -> dead lock
    * Resource intensive: thread A will use take CPU resource from thread B when continuously checking the lock
2. Busy waiting with wait:
  * Thread A locks the mutex, if the condition fail, unlock the lock then sleep for X amount time. Repeat.
    ```cpp
    bool flag;
    std::mutex m;
    void wait_for_flag()
    {
      std::unique_lock<std::mutex> lk(m);
      while(!flag)
      {
        lk.unlock();
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
        lk.lock();
      }
    }
    ```
  * This solution will remove the problem of deadlock (lock released when sleeping)
  * Problem: hard to determine a good sleeping period.

#### Waiting for a condition with a condition variable

Types of condition variable
* `std::condition_variable`: condition variable that only uses `std::mutex`
* `std::condition_variable_any`: condition variable that uses any `mutex`-like
  * Additional overhead of size, performance and OS resource (prefer `std::condition_variable` for lower overhead)


Condition variable thread synchronization:
Problem: a thread will produce items into a queue and another thread will consume items from the queue

```cpp
std::mutex mut;
std::queue<data_chunk> data_queue;
std::condition_variable data_cond;

// Producer thread
void data_preparation_thread()
{
  while(more_data_to_prepare()) {
    data_chunk const data = prepare_data();
    {
      std::lock_guard<std::mutex> lk(mut);
      data_queue.push(data);
    }
    data_cond.notify_one();
  }
}

// Consumer thread
void data_processing_thread()
{
  while(true)
  {
    std::unique_lock<std::mutex> lk(mut);
    data_cond.wait(lk, []{return !data_queue.empty();});
    data_chunk data = data_queue.front();
    data_queue.pop();
    lk.unlock();
    process(data);
    if (is_last_chunk(data))
      break;
  }
}
```

* Shared data:
  * `data_queue`: object to produce and consume on
  * `mut`: producer and consumer share a mutex to protect access to the queue
  * `data_cond`: condition variable that is shared with producer and consumer to synchronize
  code
* Producer:
  * Tries to lock the mutex to gain exclusive access to queue
  * Append a new item to the queue
  * Release lock (unblock any other producer waiting on the lock)
  * `notify()` condition variable: signal to consumer that there is an item to consume from the queue.
* Consumer:
  * Create a `std::unique_lock<std:mutex>`: unique lock does not lock the mutex immediately
  * `wait(...)` on the condition variable:
    * If another thread `notify` the condition variable, the consumer will unblock with the mutex being **locked**.
    * Check if the lambda (predicate) returns true
      * If predicate false:
        * Release the mutex and blocks until another producer calls `notify`.
      * If predicate true:
        * Continue holding the mutex and return from `wait(...)`
  * Need `unique_lock` as it must be able to lock and unlock the mutex without destroying the lock

Cavaets (Spurious Wake):
* Even though the thread calling the `notify` might view as the condition as true, when the `wait`-ed thread 
unblocks the condition might not be true anymore.
* Due to condition variable being [non-hoare style monitor](https://klementtan.com/readings/concurrent-and-distributed-in-java/#hoare-monitor):
other threads might modify condition between `notify` and the `wait`-ed thread acquiring the lock
* Might result in multiple (indeterminate) number of calls to the predicate
* Mitigated with the predicate arguments.

Condition Variable: `wait(...)` will return immediately with the mutex locked if
and only if the condition return true.

Sample condition variable implementation

```cpp
template<typename Predicate>
void minimal_wait(std::unique_lock<std::mutex>& lk, Predicate pred) {
  while(!pred()) {
    lk.unlock();
    lk.lock();
  }
}
```

**Building a thread-safe queue with condition variable**

Queue works well with condition variable as the condition for popping from the queue
is when the queue is not empty.

```cpp
#include <memory>
#include <queue>
#include <mutex>
#include <condition_variable>

template<typename T>
class threadsafe_queu
{
  private:
    mutable std::mutex mutex;
    std::queue<T> data_queue;
    std::condition_variable data_cond;
  public:
    threadsafe_queue() {}
    threadsafe_queue(threadsafe_queue const& other)
    {
      std::lock_guard<std::mutex>lk (other.mut);
      data_queue = other.data_queue;
    }

    void push(T new_value)
    {
      std::lock_guard<std::mutex> lk(mut);
      data_queue.push(new_value);
      data_cond.notify_one();
    }

    void wait_and_pop(T& value)
    {
      std::unique_lock<std::mutex> lk(mut);
      data_cond.wait(lk, [this]{return !data_queue.empty();});
      value = data_cond.front();
      data_queue.pop();
    }
    std::shared_ptr<T> wait_and_pop()
    {
      std::unique_lock<std::mutex> lk(mut);
      data_cond.wait(lk, [this]{return !data_queue.empty()});
      std::shared_ptr<T> res(std::make_shared<T>(data_queue.front()));
      data_queue.pop();
      return res;
    }
    std::shared_ptr<T> try_pop()
    {
      std::lock_guard<std::mutex> lk(mut);
      if (!data_queue.empty())
        return std::shared_ptr<T>();
      std::shared_ptr<T> res(std::make_shared<T>(data_queue.front()));
      data_queue.pop();
      return res;
    }

    std::shared_ptr<T> try_pop(T& value)
    {
      std::lock_guard<std::mutex> lk(mut);
      if (!data_queue.empty())
        return false;
      value = data_queue.front();
      data_queue.pop();
      return true;
    }
    bool empty() const
    {
      std::lock_guard,std::mutex> lk(mut);
      return data_queue.empty();
    }
}
```
* Use condition variable that represents if the queue is not empty.
  * If the `notify()` is called when no thread consuming, the consuming thread will
  lock the mutex later and check that the queue is not empty and exit the condition variable immediately
  - no deadlock.
* Lock mutex when copy constructing or checking empty queue
  * Other threads can modify during these read operation 
* `mutex` is `mutable` - all read operation (`const` context) require a locking the mutex

### Waiting for one-off events with futures

**Future**: a thread can obtain the result of a one-off event with `std::future`.
* Thread can periodically wait for the future
* Perform another task and when the result of the future is needed, wait until it is ready
* Similar to threads `std::future` will copy lvalue and move rvalue arguments.
  * For member functions, first arguments will be the function pointer (ie: `&X:;foo`) and the second argument
  is the instance. If a reference to the instance is used(`&x` or `std::ref(x)`) then the 
  member function of that instance will be called else it will be copied into a temp variable and called on the
  temp

Types of futures:
1. Unique future (`std::future<>`): one and only instance of the associated event and result
2. Shared future (`std::shared_future<>`): multiple instances of the future may refer to the same event. All the instances will become
ready at the same time

Execution Policy:
1. `std::launch::deferred`: deferred until `wait()` is called.
2. `std::launch::async`: launch and execute on a new thread.
3. `std::launch::async | std::launch::deferred`: let the implementation decide to
be deferred or new thread (default)

#### Associating a task with future

Problem: `std::async` does not allow for the task to be executed later programmitically 

`std::package_task<>`:
* Links a future (future result) with a callable object.
* Allows the callable object to be invoked independently (different thread / code) 
and the result of the callable object be returned at a different thread or code.
* Useful for packaging tasks for thread pool to invoke. The thread pool does not need
to care about the return value
* Getting future: `std::package_task<>::get_future` returns the future object for the callable
object. If the callable object is invoked the future will be ready and the results can be returned
* Takes a callable function template argument:
  * return value of the template argument determines the type of future

Example: passing tasks between background task and gui thread

```cpp
void guid_thread()
{
  while(!gui_shutdown_mesage_received()) {
    get_and_process_guid_message();
    std::package_task<void()> task;
    {
      std::lock_guard<std::mutex> lk(m); // lock the queue that passes package_task between threads
      if(tasks.empty())
        conitnue
      task = std::move(tasks.front()); // get package task from independent context
      tasks.pop_front();
    }
    task(); // call package task
  }
}

template<typename Func>
std::future<void> post_task_for_gui_thread(Func f)
{
  std::packaged_task<void()> task(f); // create a package task from the func
  std::future,void> res = task.get_future(); // get the future for the result
  std::lock_guard<std::mutex> lk(m); // lock the queue to pass packaged_task
  tasks.push_back(std::mvoe(task)); // add the task to the queue
  return res;
}
```

#### Making `(std::)promise`

Problem: `std::async` and `std::packaged_task` sets the result for a value through the return value of a callable function.
This makes it hard for a thread to provide results for multiple different futures at the same time.

Solution: `std::promise` allows a thread to set the result of a future with a single operation.
Allow for multiple future's result to be easily set.

**Multiple network connection problem**:
* An application have multiple network connection that needs to receive and send packets.
* Having a `std::async` for each connection, where the task will return data packets to the future
only when the it received or send data to or from the connection.
  * This is not scalable as the number of threads > number of processing units will have adverse impact on the latency
* Example implementation:
  ```cpp
  #include <future>
  void process_connections(connection_set& connection)
  {
    while(!done(connections)) {
      for (connection_iterator connection = connections.begin(); connection_iterator != connections.end(); ++connection_iterator) {
        if (connection->has_incoming_data())
        {
          data_packet data = connection->incoming();
          std::promise<payload_type>& p = connection->get_promise(data.id);
          p.set_value(data.payload);
        }
        if (connection->has_outgoing_data())
        {
          outgoing_packet data = connection->top_of_outgoing_queue();
          connection->send(data.payload())
          data.promise.set_value(true);
        }

      }
    }
  }
  ```
  * Each incoming data must have an associated promise for the connection manager to set the result
  for the future of the incoming data.
  * Each outgoing data must have an associated promise for the connection manager to set the success result
  for the future of the outgoing data.

#### Saving an exception for the future

* For `std::async` and `std::packaged_task`, if the called function throws an exception, the thread owning the
corresponding `std::future` will re-throw the exception when `.get()` (not immediately when the exception is thrown from the called function)
is called.
* For `std::packaged_task` and `std::promise`, if the packaged task is not called or the promise value is not set
upon destruction, `.get()` will throw `std::futrue_error`. Creating a promise or package task guarantees that the future
will be set eventually and destructing it means it cannot no longer be set.

#### Waiting from multiple threads

**Note**: `std::future` helps to synchronize data transfer from a thread to another but accessing the future object
is not thread safe and more than one thread should not have the same reference to the same future. However, `std::future`
is a move only type and cannot be copied across multiple threads.

Solution: use `std::shared_future` instead to allow multiple threads to have access to a single future result.
* Can be constructed with rvalue `std::future`  or `std::future::get()` that allows for `auto`

Cavaets:
* `std::shared_future` should be copied across threads instead of pass by reference. This will prevent concurrent calls to the
member functions of the same instance.
* Make sure that each `std::shared_future` is a local object


```cpp
std::shared_future<int> sf;

void run() {
  auto local_sf = sf; // copy the shared_future 
  local_sf.wait();
}
```

### Using synchronization of operations to simplify code

#### Functional programming with futures

Reasons:
* Having pure function that doesn't depend or change external state is very useful
in making sure that concurrent code does not have dependency or race condition.

Parallel Quick Sort:
```cpp
template<typename T>
std::list<T> parallel_quick_sort(std::list<T> input)
{
  if(input.empty())
    return;

  std::list<T> result;
  result.splice(result.begin(), input, input.begin()); // the first value from input into result (pivot)
  T const& pivot = *result.begin();

  auto divide_piont = std::partition(input.begin(), input.end(), [&](T const& t){ return t < pivot; });

  std::list<T> lower_part;
  lower_part.splice(lower_part.end(), input, intput.begin(), divide_point); // move the values less than input into it

  std::future<std::list<T>> new_lower(std::async(&parallel_quick_sort<T>, std::move(lower_part))); // execute sort lower in new thread

  auto new_higher = parallel_quick_sort(std::move(input)); // input only left with higer

  result.splice(result.end(), new_higher); // move higer after the pivot element
  result.splice(result.begin(), new_lower.get()); // move lower to before the pivot element
  return result;
}
```
* One of the two recursive call will be make to a new thread (up to implementation to decide if new thread or defrred)
* Pure functional approach as each function takes the list by value and does not modify any global state

Spawning a new generic task in a thread (substitute for `std::async`)
```cpp
template<typename F, typename A>
std::future<std::result_of<F(A&&)>::type> spawn_task(F&& f, A&& a)
{
  typedef std::result_of<F(A&&)>::type result_type;

  std::packaged_task<result_type(A&&)> task(std::move(f)); // package function in task

  std::future<result_type> res (res.get_future()); // get the future that represent the future return value of the task

  std::thread t(std::move(task), std::move(a)); // Create a thread that executes the task

  t.detach(); // Allow the thread to run free with the task

  return res;
}
```

#### Synchronization operations with message passing

Utilises **Actor** model where each thread are discrete actors that have independent state (no race condition)
and updates the state based on the message sent and received.

Implementing a state machine

```cpp
struct card_inserted
{
  std::string account;
}
class atm
{
  messaging::receiver incoming;
  messaging::sender bank;
  messaging::sender interface_hardware;
  void (atm::*state)(); // state is represented by a function pointer that will be updated once a state is completed
  std::string account;
  std::string pin;
  void waiting_for_card()
  {
    interface_hardware.send(display_enter_card());
    incoming.wait()
      .handle<card_inserted>(
        [&] (card_inserted const& msg)
        {
          account = msg.account;
          pin = "";
          interface_hardware.send(display_enter_card());
          state = &atm::getting_pin; // updates 
        }
          )
      .handle<...> // allow for chaining of different handlers
  }

public:
  void run()
  {
    state = &atm::waiting_for_card; // set the initial state
    try
    {
      for (;;)
      {
        (this->*state)(); // keep calling the states
      }
    }
  }
}
```

#### Continuation-style concurrency with the Concurrency TS

Under the namespace `std::experimental::future`, you can chain futures. This will result in only the last future
being valid.
```
std::experimental::future<int> find_the_answer(){...};
find_the_answer().then(find_the_question);
```
* The `.then` takes a callable objects that takes `std::experimental::future<T>` as an argument.
Where `T` is the return type of the initial future.

`std::async` currently does not return `std::experimental::future` but you can wrap it in experimental future to utilise 
continuation.

Chaining (not in C++23):
* As `.then` returns `std::experimental::future<T>`, it can be continuously chained
* If the continuation function in `.then(f)` returns a `std::experimental::future`,
the compiler will automatically unwrap the future and pass `std::experimental::future`
to the next function instead of `std::experimental::future<std::experimental::future>`
* `std::experimental::shared_future`
  * `std::experimental::shared_future<T>` can be chained as well. The continuation will
  be invoked for all reference to the same `shared_future`
  * The return for `.then` is a `std::experimental::future` unless it is called with `.then`

When all (not in C++23):
* Use `std::experimental::when_all` to wait for a container of futures to all be finished.

When any (not in C++23):
* Use `std::experimental::when_any`

**Latches and Barriers** (in C++20):
* `std::latch`
  * Initialize with a positive integer. Threads can either `count_down` the value of the latch
  or `wait` on the value to become `0`
* `std::barrier`
  * Initialize with the number of threads in a group
  * Each thread will call `std::barrier::arrive_and_wait` to be blocked at the barrier until all threads in the group
  has reached the barrier.
  * Can be reused multiple times.
* `std::flex_barrier`
  * Similar to `std::barrier` but accepts a lambda function that will be called on one thread once all the
  threads will reached the barrier.
  * Allow for execution of one sequential code
  * The return value of the lambda will update the number of threads in the group for the next cycle.

## Chapter 5: The C++ memory model and operations on atomic types

### Memory Model

#### Structural

*Structural memory model definition*: how things are laid out in memory

**Objects and Memory location**:
* All data in C++ are **objects**
  * Different from java or ruby where all types have member functions.
  * Objects are "a region of storage" that
* Each object is stored in one or more **memory location**
* Each memory location is either:
  * an object (or sub-object) of a scalar type
  * a sequence of bit field. Multiple adjacent bit fields are multiple objects
  that are located in the same memory location
    * Zero length bit field do not exists in a bit field but can separate adjacent
    bit fields into separate location.
* Struct
  * is an object that contains multiple sub-objects (data members)
* Take aways
    1. Every variable is an object (including members)
    2. Every object occupies at least one memory location.
    3. Variable of fundamental type (ie `int`, `char`) occupy
    exactly **one** memory location even if they are adjacent or
    part of an array
        * (klement: what about `uint64_t`?)
    4. Adjacent bit fields are part of the same memory location.

#### Concurrency

Concurrency Memory model:
1. If two threads access **separate** memory location then **no problem**
2. If two thread access the **same** memory location:
    * And neither threads update the memory location then **no problem**
    * And both threads update the memory location then **UB**

UB from concurrent modification:
* Occurs when there is not enforced ordering between two access.
* Using *atomic operation* has enforced ordering between access but
it does not enforce which operation happens first.

#### Modification orders

*Modification order* definition: the ordering of all modification to an object
from all threads.

Defined Behaviour
* To achieve defined behaviour, all threads must agree on the same modification
order in an execution.
  * Allowed to have different modification across different execution but within each
  execution, the modification order must be the same.
* If the object isn't atomic, you have to make sure that all threads agree on the same
order else the responsibility is on the compiler.

Results of agreed modification order:
* Once a thread seen a particular entry in the modification order, all subsequent reads
must return later values
* Additionally, subsequent writes from that thread must occur later in the modification order (cannot go back in time)
* Read of an object after a write on the same thread (WR order enforced?) must either return
the value written or another value later in the modification order.
    * Cannot read data before the write.

Note: the relative order between separate objects does not need to be enforced.
  
### Atomic Operations and types in C++

*Atomic operation definition*: you (any other threads) cannot observe such an
operation half-done from any thread in the system. It is either done or not
done.
* Non atomic operation could be seen as half-done, only a portion of the data
members are updated.

**Standard Atomic Types**:
* All operations are atomic
  * Could be implemented using machine instructions or locks
* Provides `is_lock_free()` member function that determines if it is lock free
  * Only `std::atomic_flag` does not provide this function as it **required** to be always
  lock free.
* In C++17 provides `static constexpr` `is_always_lock_free` to check if it is always
lock free for all architectures
* Provides macros `ATOMIC_{...}_LOCK_FREE` that returns `0` if it is **never** lock free
* `std::atomic<>` specialisation may not be lock-free for builtin types (expected but not required)
* On old compilers `std::atomic<>` and corresponding atomic type (`atomic_llong`) might represent
different data - in C++17 onwards atomic type are alias for the specialisation.
* Copy behaviour
  * Do no support copying or assigning operation (ie `atomic -> atomic`) but provides assignment or
  copying from implicit type to atomic
  * Copying and assigning involves two objects and there is not way to make the operations on two
  objects atomic.
* All `std::atomic` (including custom user defined) types provides `load()`, `store()`, `compare_exchange_strong()`,
`compare_exchange_weak()`, `fetch_add()`, `fetch_or()`, ...
* All operations on atomics can be classified into the following types:
  * Store operations: store data into the object
  * Load operations: load data from the object
  * Read-modify-write operations: read and then modify the data of an object

`std::atomic_flag`:
* Simple object that provides:
  * `set`: set the flag to `1`
  * `clear`: set the flag to `0`
  * `test_and_set`: set the flag to `1` and return the previous value
    * Note: the testing and setting are atomic operations, no two threads can call `test_and_set`
    and both return `0`
* Building block for other atomic types
* Spin Lock:
  ```cpp
  class spinlock_mutex
  {
    std::atomic_flag flag;
  public:
    spinlock_mutex():
      flag{ATOMIC_FLAG_INIT}
      {}
    void lock()
    {
      while(flag.test_and_set(std::memory_order_acquire)) // only one thread can return 0
    }
    void unlock
    {
      flag.clear(std::memory_order_release);
    }
  }
  ```

`std::atomic<bool>`
* Can be assigned to the non-atomic variant (ie `bool`)
* Assignment operations return value instead of the usual `ref`
* Has `exchange(T, order)` member function that sets the underlying atomic object to the argument
and return the previous value of the atomic before it was set.

`std::atomic<T*>`
* Additionally allow for pointer arithmetic
  * `++ | fetch_add(1), += | fetch_add(n)`
  * `-- | fetch_sub(1), -= | fetch_sub(n)`
* Need to use `fetch_add,sub` if you would like to use memory order semantics

`std::atomic<integral type>`
* allows for all normal integral type operation except for multiplication division and shift operation missing.

`std::atomic<>` primary class template
* Requirements:
  * Trivial copy assignment operator
    * Must not have any virtual functions or virtual base class
    * Compiler generated copy-assignment operator
    * Every base class and non-static data members needs to be trivially copy-assignment
    * *Implication*: compiler can just perform `memcopy`
  * Bitwise comparison
    * Equal instances should be bitwise same
      * Cannot be `std::vector` as equal vector contains different pointers.
      * But can use class with counters, flags or array
    * Cannot have padding bits
    * Cannot have custom comparison operator
    * *Implication*: compiler can use `memcmp`
* Reasons:
  * If the compiler need to use locks, the pointer or reference to protected
  should not be passed to any protected data
    * Having a custom copy-assignment or comparator could violate this rule
  * Allowing user to execute arbitrary code in the copy assignment or comparison
  could lead to deadlock.
  * The restriction increase the change for the compiler to use atomic instructions
  instead of lock
* If the user type is same size as `int` or `void*`, most probably can be lock free.
  * Some platforms can perform atomic instructions for sizes twice of `int` or `void*`
  (Double-word-compare-and-swap)
* Best Practice: should only use `std::atomic<>` for simple user defined type

Free functions for atomic operations:
* Provides corresponding non-member functions for atomic types
  * Takes `T*` instead of `T&` as argument to be C-compatible
* `std::shared_ptr`
  * Provides `load`, `store`, `exchange` and `compare-exchange` overloads for `std::shared_ptr`
  eventhough it is not a specialisation of `std::atomic`

#### Storing a new value depending on the current value

(klement: IMO this segment is very confusing to me but it seems important for later chapters
on lock-free programming)

*Compare and Exchange functional requirements*:
1. Compares the expected value argument with the atomic value
2. If the expected value is **equal** to the atomic value:
  1. Signifies that atomic value **did not change** from the POV of the setting thread,
  it is safe for the thread to update the atomic value with the desired value.
  2. Returns `true`
  3. Note: Store performed
3. If the expected value is **not equal** to the atomic value:
  1. Signifies that the atomic value **changed** from when the setting thread decide on
  the desired value to when the thread actually sets it
  2. Updates the expected value argument to the change atomic value
  3. Returns `false`
  4. Note: No store performed
  
`compare_exchange_strong(T& expected, T desired, ...)`
* Carries out the full functionallity of compare and exchange.

`compare_exchange_weak(T& expected, T desired, ...)`
* If the expected value equals to atomic value, might not successfully store the desired value
  * Functionally should always have successful store when atomic value == expected
  * Returns `false` (eventhough atomic value == expected)
  * False negative - spurious failure
    * Should try again
  * Spurious failure happens when the machine cannot atomically compare atomic value == expected
  and set the atomic value to desired.
* Solution keep looping when spurious failure. Occurs when expected is not
changed and `compare_exchange_weak` returns false.
  ```cpp
  bool expected = false;
  extern atomic<bool> b; // initialize to true
  while(!b.compare_exchange_weak(expected, true) && !expected) 
  ```

Using compare and exchange to perform dependent updates (klement: my own example - not 100% is correct)
```cpp
extern atomic<Foo> g_f;
void fn() {
  Foo prev_f = g_f.load();
  Foo new_f;
  do {
    // upated g_f will be loaded into prev_f
    new_f = some_dependent_update(prev_f);
  } while(!g_f.compare_exchange_strong(prev_f, new_f));
}
```
* Each failed `compare_exchange_strong` (another thread updated `g_f`), reload `prev_f` with the upated
`g_f`
* Re-compute desired (`new_f`) that is dependent on the current `g_f`
* Try to update `g_f` and hope no other thread will update `g_f`
* If it is cheap to perform `some_dependent_update` the value,
should use `compare_exchange_weak` as it more lightweight as we need to
perform the loop anyways?
* (klement: this does not seem to prevent starvation?)

*Memory Order*:
* the third argument is the memory order in the case of success and forth is in the case of failure
* Success performs read-modify-write operations while failed only perform read
* Failure memory order cannot be stricter than success.

### Synchronizing operations and enforcing ordering

#### Relationships

*synchronizes-with*:
* Only occurs between atomic types or types that atomic data members
* Rule:
  * A read operation synchronizes with a write operation, `W`, if it:
    1. reads the value written by `W`
    2. reads the value written by write operations after `W` on the **same thread** as `W`
    3. reads the value written by read-modify-write operation that depends on the `W` value on **any thread**
* Occurs for suitably tagged read and write operations
* Saying operation `A` on a thread synchronizes with operation `B` on another thread is equivalent to `A` leads transitively to `B`

*sequence-before*:
* layman: the program order within the same function/thread
* Formal definition ([more details](https://en.cppreference.com/w/cpp/language/eval_order))
  * Full Expression: each statement (anything terminated with `;`) is considered to a full expression
  * According to the standard:
      > Each value computation and side effect of a full expression is sequenced before each value computation and side effect of the next full expression
  * Any full expression (ie statement) will be evaluated before the next expression.
    * This means that the program order within the same function must be enforced
  * Does not apply if there are "sub" expression within a full expression (evaluating the args)
* (klement: The book did not touch on sequence-before relationship but these articles were
useful in my understanding [SO](https://stackoverflow.com/a/4183735/11635160))

*happens-before*
* On a single thread: if an operation is sequence-before another thread then there is a *happen-before* relationship
  * (klement: no *happen-before* relationship between different atomic variable on the same thread?)
* On multi thread: If an operation on thread A *synchronizes-with* operation on thread B, then A inter thread happens before B
  * **Inter thread happens before** occurs transitively across other happens before and other sequence before 
* Requirements:
  * A sequenced before B
  * A inter-thread happens before B (transitive)
    * A and B can operate on different objects

*strong-happens-before*
* Requirements:
  * A sequenced before B
  * A synchronizes-with B (not transitive)
    * A and B must operate on the same object
  * Transitive through *strong-happens-before*
* Operation with `memory_order_consume` will participate in *happens-before* but
not *strong-happens-before*

#### Memory ordering for atomic operations

##### Sequentially Consistent Ordering
* Operations as if performed on a single thread
* A sequentially consistent store synchronizes with a sequentially consistent load of the same variable. 
  * If thread A stores 1 and threads B loads 1 then they synchronize
* Condition carry forward: Sequential consistency also applies to all sequentially consistent after the load
  * Continuing from the previous example, if X occur after thread B loads 1 then X must also occur after thread A stores 1
* Condition does not carry forward to threads that use relaxed memory model
* Maintains ordering between different variable through sequenced before
  * thread A writes `x=1`, thread A writes `y=1`, thread B reads `y=1`, thread B must read `x=1`
* Cost:
  * Can incur huge overhead as the OS needs to synchronize all atomic operations

#### Non-Sequential Ordering
* Different threads have different view of all the operations on all the atomic variables
  * Due to the CPU cache having inconsistent data
* Only guarantee: all threads will view the **same modification order** for within the **same variable**
  * Ie `x=1` -> `x=2`, if `x.load() == 2` then `x.load()` will never equals to `1` afterwards

**Relaxed Ordering**:
* No synchronize with relationship between different threads and different variable
  ```cpp
  void write_x_then_y()
  {
    x.store(true, std::memory_order_relaxed);
    y.store(true, std::memory_order_relaxed);
  }
  void read_y_then_x()
  {
    while(!y.load(std::memory_order_relaxed));
    if(x.load(std::memory_order_relaxed));
      ++z
  }
  int main()
  {
    std::thread a(write_x_then_y)
    std::thread b(read_y_then_x)
    a.join();
    b.join();
    assert(z.load() != 0);
  }
  ```
  * Could read `z=1` as `x=1` happen before `y=1` does not need to hold
* Within the same thread and same atomic variable still have **happen-before** relationship
* Threads agree on the **same modification order of each individual variable**
  * If a thread sees a certain value, it cannot read any earlier value.
* Intuition:
  * Each atomic variable will have its own change list of values starting with the first value
  * Each thread will have its own pointer to where it is at on the change list
  * At $$T_i$$ if you load $$v_i$$, at $$T_{i+1}$$ you can get $$v_{>=i}$$ either the same value
  as before or a new value but never a old value
    * Another thread might load another value from $$v_i$$ at $$T_i$$
  * When trying to store the value, the value will be added to the end of the change list but
  the pointer might still remain at the same position.

#### Acquire-Release Ordering

Requirements for acquire-release:
* Establishes a synchronize-with relationship between an acquire (read operation)
and release (load operation ).

Benefits:
* Problem: In a relax model, if operation A is sequence before B in thread 1 and thread 2 sees operation B,
it is does not mean thread 2 can see operation A as well.
* With acquire release, thread 2 seeing operation B means if thread 2 try to load operation A with relaxed, 
it will guarantee to see it. A - sequence before -> B and B - synchronize with -> load B so loading A will
transitive synchronize with A.
* Can be used as a synchronization point for any previous relaxed operation.
  * Operation 0,..., N-1 are relaxed but N is a release operation
  * If another thread acquire the Nth operation then it will also see all 0,..., 
  N-1 even though it is relaxed

Example:

```cpp
void write_x_then_y()
{
  x.store(true, std::memory_order_relaxed); // (1)
  y.store(true, std::memory_order_release); // (2) - release
}

void read_y_then_x()
{
  while(!y.load(std::memory_order_acquire)); // (3)
  if (x.load(std::memory_order_relaxed)) // (4)
    z++
}

int main()
{
  x = false;
  y = false;
  z = 0;
  std::thread a(write_x_then_y);
  std::thread b(read_y_then_x);
  a.join();
  b.join();
  assert(z.load() != 0); // never false
}
```

* (1) and (5) are relaxed
* Why `z != 0`:
  * (1) happens before (2) because they are on the same thread
  * (2) synchronize with (3) because of acquire-release
    * Acquire a value that was released => establish a synchronize with operation
    * (klement: what if two thread release the same value, which one will the acquire synchronize with?)
    * (klement: how does this work under the hood? The books talks about batch id but there is no idea of ids here)
  * By transitive properties, (3) happens before (1) => (4) happens before (1)
  * (4) will definitely see (1) and x will be true.

**Transitive Synchronization**

```cpp
void thread_1()
{
  data[0].store(42, std::memory_order_relaxed);
  data[1].store(97, std::memory_order_relaxed);
  data[2].store(17, std::memory_order_relaxed);
  data[3].store(-141, std::memory_order_relaxed);
  data[4].store(2003, std::memory_order_relaxed);
  sync1.store(true, std::memory_order_release); // (1)
}

void thread_2()
{
  while(!sync1.load(std::memory_order_acquire)); // (2)
  sync2.store(true, std::memory_order_release); // (3)
}
void thread_3()
{
  while(!sync2.load(std::memory_order_acquire)); // (4)
  assert(data[0].load(std::memory_order_relaxed) == 42);
  assert(data[1].load(std::memory_order_relaxed) == 97);
  assert(data[2].load(std::memory_order_relaxed) == 17);
  assert(data[3].load(std::memory_order_relaxed) == -141);
  assert(data[4].load(std::memory_order_relaxed) == 2003);
}
```
* The assertion pass for all `data` even thought all the operations are relaxed.
* Operation in the same thread have sequence before relationship.
* Why assertion pass all the time:
  1. All the relaxed store operations sequence before (1) in thread1
  2. The store-release synchronize (1) in thread 1 with the load-acquire (2) in thread 2 as they read the same value (`x=true`)
    * By transitive property, the relaxed store happen before (2)
  3. The load-acquire (2) of sync1 sequence before store-release of sync2 (3) because in the same thread
    * By transitive property, the relaxed store happen before (3)
  4. The load-acquire (4) of sync2 synchronize with the store-release of sync2
    * By transitive property, the relaxed store happen before (4)
    * All other relax loads of data will happen after the relax store.

#### When to use the models

* Bad usage of `memory_order_acq_rel`
  * When a read modify operation does not check for a particular value
  * The absence of the check means there is no "acquiring"
  * Example:
    * `fetch_sub` (--x): what the original (acquire) value of `x` is not used.
    Using `memory_order_acq_rel` will not have synchronize with relationship with any
    of the store-release operation by other thread
    * `fetch_or` (x |= ...): same reason as above
* sequential consistency:
  * act as a default acquire-release
  * Can be replaced with acquire-release if a set operations should be done atomically
    * As above, the first N-1 operation are relaxed except for the Nth operation which will
    synchronize with another acquire-release pair
* acquire-release:
  * always prefer as it can provide sequential consistency like behaviour without the global ordering
  overhead

#### Memory Order Consume

* WARNING: do not use this. Standard says that it always prefer acquire-release
instead of consume-release
* Instead of all relaxed operation before a store-release happening before
all relaxed operation in a synchronized load-consume, only the data dependent operations
on the load-consume are happened before.

#### Release Sequence and synchronize-with

* When a single store operation with release or sequential order followed by a
chain of read-modify with acquire order.
* Then: The first store-release synchronize with the last load-acquire even though there might be
a sequence of non-store-release operation in the middle.
* Benefit: do not need the whole chain to be synchronized if we have all operation to only
synchronize with the first store-release.

Example:
```cpp
// Thread 1:
A;
x.store(2, memory_order_release);

// Thread 2:
B;
int n = x.fetch_add(1, memory_order_relaxed);
C;

// Thread 3:
int m = x.load(memory_order_acquire);
D;
```
* When the sequence is thread 1 -> 2 -> 3.
* The load in thread 3 will be the value stored by thread 2 but since thread 2 did not use
release, then technically there is no synchronize with.
* With release sequence, it does not matter and will establish the synchronize with relationship
even though thread 2 is in the middle.

#### Memory Fence

* `std::atomic_thread_fence(std::memory_order_release)`: all operations after the fence are release operation
* `std::atomic_thread_fence(std::memory_order_acquire)`: all operations before the fence are acquire operation
* To synchronize the fences, a load operation before the acquire fence need to
synchronize with the store operation after the release fence.
* **Note**: Works with non-atomics as long as the synchronizing function is atomic.

#### Ordering non-atomic operations

* If a non-atomic operation is sequence before an atomic operation in the same thread. When the atomic
operation happens before another operation in another thread, the non-atomic operation also happens before it.

## Chapter 6: Designing lock-based concurrent data structures

*Serialization definition*: threads takes turns access the data protected. The access to the data structure can
be serialized into a linear chain.

**Requirements**:
* Multiple threads access the data structures concurrently
* All threads have a consistent view of the data structures
* Invariants of the data structure will not be violated
* Providing for as much concurrency as possible while maintaining the above requirements
  * Aim for smaller protected region => fewer operation serialized => more concurrency


**Guidelines**:
* Safety:
  * Ensure no threads can see a broken invariant of a state
  * Interface should be provide complete function to prevent the need on synchronization
  on the user side
  * Be mindful of how exceptions happen and how they could break the invariants
  * Restrict the scope of the locks and avoid nested locks to prevent deadlocks
* Enable genuine concurrency (minimize Serialization):
  * Restrict scope to allow operations to be performed outside the lock
  * Can different part of the DS be protected with different mutex (dont need to compete for the same lock on independent parts)
  * Do all operations need same level of protection

### Thread-safe stack


```cpp
#include <exception>
struct empty_stack: std::exception
{
  const char* what() const throw();
}

template<typename T>
class threadsafe_stack
{
private:
  std::stack<T> data;
  mutable std::mutex m;
public:
  threadsafe_stack() {}
  threadsafe_stack(const threadsafe_stack& other)
  {
    std::lock_guard<std::mutex>(other.m);
    data=other.data;
  }
  threadsafe_stack& operator=(const threadsafe_stack&) = delete;
  void push(T new_value)
  {
    std::lock_guard<std::mutex> lock(m);
    data.push(std::move(new_value));
  }
  std::shared_ptr<T> pop()
  {
    std::lock_guard<std::mutex> lock(m);
    if (data.empty()) throw empty_stack();
    std::shared_ptr<T> const res(std::make_shared<T>(std::move(data.top())));
    data.pop();
    return res;
  }
  void pop(T& value)
  {
    std::lock_guard,std::mutex> lock(m);
    if (data.empty()) throw empty_stack();
    value = std::move(data.top());
    data.pop();
  }
  bool empty() const
  {
    std::lock_guard<std::mutex> lock(m);
    return data.empty();
  }
}
```

Thread Safety:
* All member functions accessing and modifying the stack are protected by the mutex
* Possible race condition if user call `empty()` then `pop`.
  * Mitigated by always checking if the stack is empty within the `pop` and throwing
  an exception if it is.
* Possible Exceptions:
  * Locking a mutex can throw exception (very rare)
    * mitigated by locking the mutex at the top of the function, if throw an exception, invariant
    not broken
    * unlocking a mutex will never throw
  * Empty stack exception: only thrown before modifying the underlying stack => invariant not broken
  * `std:make_shared`: could throw if OOM but will not affect as the underlying stack will not be modified.
  * Copy constructor/move constructor could throw but does not affect as the underlying stack will not be modified.
* Possible deadlock:
  * Calling user provided copy constructor and move constructor
  * If the user's copy/move constructor call the thread safe stack functions, there will be deadlock
* constructor and destructor: okay to assume that the user will not call the member function on a partially constructed/deleted stack

Cons:
* Large serialization in the access to the stack.
* Cannot wait for an item.

### Thread Safe Queue

#### Thread Safe Queue using a single lock

```cpp
template<typename T>
clas threadsafe_queue
{
private:
  mutable std::mutex mut;
  std::queue<std::shared_ptr<T>> data_queue;
  std::condition_variable data_cond;
public:
  threadsafe_queue()
  {  }
  void wait_and_pop(T& value)
  {
    std::unique_lock<std::mutex>lk(mut);
    data_cond.wait(lk, [this]{return !data_queue.empty()});
    value = std::move(*data_queue.front());
    data_queue.pop();
  }
  bool try_pop(T& value)
  {
    std::lock_guard<std::mutex> lk(mut);
    if (data.mepty())
      return false;
    value = std::move(*data_queue.front());
    data_queue.pop();
    data_queue.pop();
  }
  std::shared_ptr<T> wait_and_pop()
  {
    std::unique_lock<std::mutex> lk(mut);
    data_cond.wait(lk, [this]{return !data_queue.empty(); });
    std::shared_ptr<T> res = data_queue.front();
    data_queue.pop();
    return res;
  }
  std::shared_ptr<T> try_pop()
  {
    std::lock_guard<std::mutex> lk(mut);
    if (data_queue.empty())
      return std::shared_ptr<T>();
    std::shared_ptr<T> res = data_queue.front();
    data_queue.pop();
    return res;
  }
  void push(T new_value)
  {
    std::shared_ptr<T> data(std::make_shared<T>(std::move(new_value)));
    std::lock_gaurd<std::mutex> lk(mut);
    data_queue.push(data);
    data_cond.notify_one();
  }
  bool empty()
  {
    std::lock_guard<std::mutex> lk(mut);
    return data_queue.emtpy();
  }
}
```
* Implementation same as thread safe stack but have additional try_pop that uses condition variable
* Storing of shared pointer in the underlying queue will prevent any OOM exception
when trying to convert the queue value to shared pointer in pop
  * Copying of shared pointer will not throw exception
* Con: not much concurrency as all access to the underlying queue need to be serialized

#### Thread Safe queue using fine-grained locks and conditional variable

General Idea:
* Instead of a using `std::queue` that requires a lock every time we access or modify the it, use a self implemented
node based queue.
* DS: a `unique_ptr` to the head of the queue and a pointer (observer) to the tail of the queue
  * Use a sentinal node for the tail to prevent having to lock both the head and tail when performing push/pop
    * With sentinal:
      * If the head is not equal to tail, any other operation (only pop because push will be locked) will never
      make the head equal to tail. Tail can only advance "backward".
      * when popping, as long as the head does not equal to the tail, it is guaranteed that advancing the head will not affect anything. This is due to pushing only modifying the tail (not equals to head).
      * allow for minimum locking (only when checking if head equals to tail)
* Algorithm:
  * pop:
    * Check if the tail is equal to the head. If not equal => not empty.
    * Advance the head to the next node and return the previous head.
  * push:
    * de-sentinalize the tail node by moving the value into the node
    * create a new sentinal tail node and append it to the end


```cpp
template<typename T>
class threadsafe_queue
{
private:
  struct node
  {
    std::shared_ptr<T> data;
    std::unique_ptr<node> next;
  };
  std::mutex head_mutex;
  std::unique_ptr<node> head;
  std::mutex tail mutex;
  node* tail;
  std;:condition_variable data_cond;

  node* get_tail()
  {
    // get the tail in the minimum time
    std::lock_gaurd<std::mutex> tail_lock(tail_mutex);
    return tail;
  }
  std::unique_ptr<node> pop_head()
  {
    std::unique_ptr<node> old_head = std::move(head);
    head = std::move(old_head->next);
    return old_head;
  }
  std::unique_ptr<std::mutex> wait_for_data()
  {
    std::unique_ptr<std::mutex> head_lock(head_mutex);
    data_cond.wait(head_lock, [&]{return head.get() != get_tail();});
    return std::move(head_lock);
  }
  std::unique_ptr<node> wait_pop_head()
  {
    std::unique_lock<std::mutex> head_lock(wait_for_data());
    return pop_head();
  }
  std::unique_ptr<node> wait_pop_head(T& value)
  {
    std::unique_lock<std::mutex> head_lock(wait_for_data());
    value = std::move(*head->data);
    return pop_head();
  }

  std::unique_ptr<node> try_pop_head()
  {
    std::lock_gaurd<std::mutex> head_lock(head_mutex);
    if (head.get() == get_tail()) {
      return std::unique_ptr<node>();
    }
    return pop_head();
  }
  std;:unique_ptr<node> try_pop_head(T& value)
  {
    std::lock_gaurd<std::mutex> head_lock(head_mutex);
    if (head.get() == get_tail())
    {
      return std::unique_ptr<node>();
    }
    value = std::move(*head->data)
    return pop_head();
  }
public:
  threadsafe_queue():
    head{new node}, tail(head.get()) 
  {}
  threadsafe_queue(const threadsafe_queue& other) = delete;
  threadsafe_queue& operation=(const threadsafe_queue& other) = delete;
  std::shared_ptr<T> try_pop(T& value) {
    std::unique_ptr<node> old_head=try_pop_head();
    return old_head ? old_head->data : std::shared_ptr<T>();
  }
  bool try_pop(T& value) {
    std::unique_ptr<node> const old_head = try_pop_head(value);
    return old_head;
  }
  std::shared_ptr<T> wait_and_pop() {
    std::unique_ptr<node> const old_head = wait_pop_head();
    return old_head->data;
  }
  void wait_and_pop(T& value) {
    std::unique_ptr<node> const old_head = wait_and_pop(value);
  }
  void push(T new_value);
  bool empty() {
    std::lock_guard<std::mutex> head_lock(head_mutex);
    return (head.get() == get_tail());
  }
};

template<typename T>
void threadsafe_queue<T>::push(T new_value)
{
  std::shared_ptr<T> new_data(std::make_shared<T>(std::move(new_value)));
  std::unique_ptr<node> p(new node);
  {
    std::lock_guard<std::mutex> tail_lock(tail_mutex);
    tail->data = new_data;
    node* const new_tail = p.get();
    tail->next = std::move(p);
  }
  data_cond.notify_one();
}
```
* `get_tail` is refactored for reusable code and ensure that minimum lock time is used for checking the tail
