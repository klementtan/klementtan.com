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

## Sharing data between threads

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

Problem: There are data strucutes that are rarely updated by writers (exclusive access) and access a lot by readers (no writers)

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
