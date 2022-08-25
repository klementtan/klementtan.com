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

* What you don't use, you don't pay
* What you do use, you couldn't hand code any better

Non-performance-related C++ language features
(klement: I am aware of these features but this section provides a very good
structured answer as to why C++. Could be useful in interviews.)

* Value Semantics:
  * Allows callers to be guaranteed that their provided arguments will not be
    modified as all arguments are copied.
* Cons correctness:
  * Provide const guarantees in compile time. Available in java (`final`) but
    it is more flexible as member functions can be delclared const
* Object ownership: with RAII and smart pointers we can clearly express the
object ownership. In Java all resource have shared ownership.
* Deterministic destruction:
  * Uses RAII for destruction which is deterministic.
* Avoiding null objects with references:
  * As references in C++ cannot be null and reassigned, functions that takes
  parameters as reference can be guaranteed that the argument is not a null
  object.

## Chapter 2: Essential C++ Techniques

(klement: Skipping the first few items as it is just a brief summary of effective/effective modern C++)

### Contracts

Definitions:
* Precondition: callers responsibility to make sure that the precondition are met.
* Postcondition: function responsibility to make sure that the postcondition are met.
* Invariants: condition that should always hold true. Could apply to loops (at the start and end of loop)
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

* A technique to signal that an error has occurred in the callee
  * It is the only way when there is an error in the constructor
* If a function does not throw an exception use `noexcept`.
  * Note: compiler does not guarantees that no exception will be thrown - up to the author
   to verify that there are no exception.
   * If a `noexcept` function throws an exception, it will call `std::terminate` and the program
   will terminate immediately instead of unwinding the stack.
* Writing exception safe code:
  * Use copy and swap idiom. Mutate a temporary obj, if succeed swap to the main object.
* Performance:
  * Costs:
    * Larger binary program => not zero-heard principle
    * Throwing and catching exception is exception (sad path) => runtime cost is not deterministic
  * In happy path (no throw) => performs quite well
  * Advantage of exception vs return code. For return code branching (`if (rc)`) will need to be performed
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
$$\text{\% improvement} = 100(1- \frac{1}{Speedup})$$

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
* Pitfalls:
  * results are over-generalized (not representative of production) and treated as universal truth.
  * Compiler might optimize microbench code differently from non-microbench code

### Amdahl's law

$$\text{Overall speedup}=\frac{1}{(1-p)+\frac{p}{s}}$$

* $$p$$: represent the fraction of the entire program execution time that the function execute for.
* $$s$$: represent the speed of the new function.

Takeaways: No matter how fast the speedup, the entire improvement of the program is limited by $$p$$

## Chapter 4: Data Structure

### Properties of computer memory

* Covers the basic of how memory works in OS
* Cache:
  * L1 cache: contains instruction cache and data cache
    * 32kb + 32kb
    * Per core
  * L2 cache:
    * ~256kb
    * Per core
  * L3 cache:
    * 8MB
    * Shared among core
  * Note: instruction cache is only on L1 => no cache pollution between different process on different cores.

### Sequence Containers

**Definition**: containers that keep the elements in the order we specify

#### Vectors and arrays

Vector:
* Stores element by value.
* Move or copy construct the element when rearranging the elements
* Use `emplace_back` to create an object inplace, don't need to construct
 the object on the caller side and copy construct/moved into the vector
* C++20: use `std::erase` and `std::erase_if` for erasing elements in an vector

Array:
* fixed size container that stores elements on the stack

#### Deque

* Double end queue that supports inserting to both the front and end of the container.
* Supports **O(1) time random access** (klement: I did not know this. Always thought deque is a linked list)
  * Unlike LL, **O(N)** deletion
* Possibly implemented as a 2D fixed size array - random access is requires an indirection
  * Will keep unused elements positions at the start and end of the allocated memory. Opposed to vector that
  have the first element at the start of the memory.
* Not all the elements are continguous in memory.

#### List and `forward_list`

* `std::list`: doubly linked list
* `std::forward_list`: singly linked list
* If you do not need back pointers (forward iterator) then should use `std::forward_list` as it uses less
memory
* Supports **splicing**: transfer elements between lists without copying.

### Associative Containers

Elements are ordered based on the element itself instead of user code.

#### Ordered sets and map

* Balanced binary tree

#### Unordered sets and maps

* Uses a **chaining** to store the elements
* Requirement:
  * Equality function: function to compare equal
  * Hash function to compare equal: key hashing

### Container adaptors

Containers that uses other container as its underlying data type.

### Using Views

#### `string_view`

* Non-owning view of a contiguous characters
* If a function takes in a const-ref to `std::string` by the call site provide a string literal, using `std::string_view` will be faster
as no heap allocation will need to be created.
* Contiguous character can be owned by `std::string` or string literal (static storage)

## Chapter 5: Algorithms

Note: the `<algorithm>` header in C++20 includes the old iterator based algorithm (pre C++20) and the new algorithms constrained with C++ concepts.

### Solving everyday problems

* Iterating over a sequence: 
  * `std::ranges::for_each` 
    * to execute the function on each element of the container.
    * return value is ignored
  * `std::ranges::transform`
    * Transform each element in the container to a new container
    * Execute a function for each element and the return value of the function is the element in the new container
* Generating elements:
  * `std::ranges::fill`
    * Fill each element with a constant value
  * `std::ranges::generate`
    * Calls a function (generator) that accepts no arguments for each element. The element will be replaced with the return value of the generator
  * `std::iota`:
    * generates a sequence of elements in increasing value
* Sorting:
  * `std::ranges::sort`
* Finding:
  * `std::ranges::find`
  * Using binary search: `std::binary_search`, `std::lower_bound()`, `std::upper_bound()`, `std::equal_range`
    * `std::equal_range` returns a range for instances where there are multiple identical elements
* Testing for certain condition:
  * `std::ranges::all_of()`, `std::ranges::any_of()`, `std::ranges::none_of()`
  * Takes in a container and a unary function (take argument and return true/false)
* Counting elements:
  * `std::ranges::count`
  * Use `std::ranges::equal_range` if we know the container is sorted and has random access
* Minimum, maximum and clamping:
  * `std::min`, `std::max`
  * `std::clamp` (c++ 17): returns a value if within the min/max else return the min or max if exceed one side:
    * `std::camp(some_func(), y_min, y_max) == std::max(std::min(some_func(), y_max), y_min)`
  * `std::ranges::minmax`: returns the min and max value in the range
  * Position of the min/max element: `std::ranges::min_element(v)`, `std::ranges::max_element(v)`

### Ranges and iterators

#### Sentinel values and past-the-end iterators

* Use a past-the-end iterator that indicate that the current iterator has reached the end of the container.
* The past-the-end iterator is a sentinel value that cannot be dereferenced but can be used to compared to other iterator.
  * Need to implement `operator==` and `operator!=` for sentinel values and iterator combinations
* `end()` iterator is referred to as a sentinel value

#### Ranges

* Range concept: any container that exposes the `begin()` and `end()`
  ```cpp
template<class T>
concept range = requires(T& t) {
  ranges::begin(t);
  ranges::end(t);
}
  ```
* In stl containers, the iterator (`begin`) and sentinel (`end`) have the same type. These containers fulfills the concept of
`std::ranges::common_range`.
* For `views` the iterator and sentinel have different type
* This allow the constrained algorithms to operate on ranges instead of iterator pairs 

#### Iterator Categories

Iterator Operations:
* step forward: `std::next(it)` or `++it`
* step backward: `std::prev(it)` or `--it`
* Jump to arbitrary position: `std::advance(it, n)` or `it += n`
* Read: `auto value = *it`
* Write: `*it = value`

Categories:
* `std::input_iterator`:
  * Supports **read only** and **step forward** (once)
  * Base class
* `std::output_iterator`:
  * Supports **write only** and **step forward** (once)
  * Base class
* `std::forward_iterator`:
  * Supports **read**, **write** and **step forward**
  * Inherits from `std::input_iterator` and `std::output_iterator`
* `std::bidirectional_iterator`:
  * Supports **read**, **write**, **step forward** and **step backward**
  * Inherits from `std::forward_iterator`
* `std::random_access_iterator`:
  * Supports **read**, **write**, **step forward**, **step backward** and **jump to arbitrary position**
  * Inherits from `std::bidirectional_iterator`
* `std::contiguous_iterator`:
  * Similar to `std::random_access_iterator` but guarantees that the data is contiguous block of memory like string, vector, array, span

### Features of standard algorithms

* Functions in `<algorithm>` do not change the size of the container.
  * `std::remove` and `std::unique` just swaps the elements to the end of the container and moves back the `end` iterator to exclude the removed element
    * If you would like to delete the element use `erase`. C++20 has `std::erase_if`.
* Algorithms with output require allocated data
  * Algorithms that outputs (ie `std::copy` or `std::transform`) will require the output iterator to point to a valid element to be assigned to.
    Thus it does not create the element
  * To solve this problem, make sure the output container has sufficient elements or use inserter iterator
    * `std::back_inserter` or `std::inserter`
    * Inserter inserter act as an output iterator but will insert the element first before allowing the algorithm to assign it a new value
* Most algorithm uses `operator==()` and `operator<()` by default
  * Implement these member function to be able to use the standard algorithms
  * Constrained algorithms (C++20) allow for projection:
    * Allow executing a projection function on element before calling the comparison function.
    * The projection function can return a fundamental with the comparison value already implemented
      ```cpp
      auto names = std::vector<std::string>{
        "Ralph", "Lisa", "Homer", "Maggie", "Apu", "Bart"
      };
      std::ranges::sort(names, std::less<>{}, &std::string::size);
      ```
    * lambda can be passed to projections
* Algorithms require move operator not to throw:
  * a lot of algorithms uses `std:move_if_noexcept`. Marking them as `noexcept` will allow the algorithms to move the elements instead
  of calling copy constructor.
* Algorithms perform just as well as C library functions:
  * `std::copy`, `std::copy` and `std::fill` performs just as well as `memset`, `memcpy`

### Writing Generic Algorithms

Sorting:
* Use `std::nth_element` to find the median
* Use `std::partial_sort` to sort the first `n` of the range

**Why Standard Algorithms**:
* Deliver performance
* Provide safety (corner cases)
* Future-proof: an algorithm might be replaced by a better one in the future. Using the standard algorithm will abstract that away
* Provide intention: raw loops does not immediately state the intention

## Chapter 6: Ranges and views

### std::views

* Views are non-owning view of a "container". No copying is involved when a view of a container is formed
* Lazily evaluated: any transformation/filtering of a container is only done when accessing the iterator
  of the view. 
* Views are composable:
  * Each view operation (ie transform/filter) takes in a view and returns a view. This will allow views to be transformed
  to simulate a single view but with multiple operation.
  * `std::ranges::transform_view{std::ranges::filter_view{std::ranges::ref_views{s}, by_year}, &Student::score_}`
* Range adapters:
  * Similar to normal range operation but without the `_range` suffix
  * Range adapters are helper stateless object that accepts `viewable_range` instead of actual `std::ranges::view`
  * Remove the need for using `std::rev_views`. Normal containers are `viewable_range`.
  * Allow for easy pipe syntax: `auto socres = s | filter(by_year) | transform(&Student::score_);`
* Views: non-owning
  * O(1) time construction - independent of the size of view
* views do not modify the underlying container. Any modification are done on the proxy object level (iterator).
* Materializing a view:
  * use `std::ranges::copy` to create a container with the view of elements.
* Caveat:
  * Most views are lazily evaluated => random access will not be allowed => will not be able to be sorted
  * ie: performing a `std::ranges::filter` on a vector will remove the random access-ness of it.
  * Only `std::views::take` will preserve the iterator quality

### Types of views

* `std::views::iota`: generating views finitely/infinitely
* Transforming views: `std::views::transform`, `std::views::reverse`, `std::views::split`, `std::views::join`
* Sampling views: `std::views::filter`, `std::views::take`, `std::views::drop`, `std::views::drop_while`
* Utility views: `std::views::ref_views`, `std::views::all_views`, `std::viess::subrange`, `std::views::counted`, `std::vies::istream_view`

## Chapter 7: Memory Management

### Computer Memory

**Virtual address space**:
* Each process has its own virtual address space
* A virtual address is mapped to a physical address by the OS and **memory management unit** (MMU) which is part of the processor (CPU?)
* Mapping or translation happen each time we access a memory address
* Extra indirection allow OS to use physical memory for parts of a process virtual memory and the other parts backed on disk (secondary storage)
  * Secondary storage: swap space, swap file or pagefile
* Virtual address space for each process is always contiguous.

**Memory Pages**:
* Physical address space is divided into multiple fixed size frames.
  * Each frame in the physical address is mapped to a page in the virtual space.
  * Note pages address for each process are contiguous but the corresponding frames in physical memory might not be.
* When a paged is accessed but the frame is not mapped to the physical memory:
  * Hardware interrupt **page fault** is triggered (klement: why is it an interrupt? stop the world?)
  * The page from disk is loaded into physical memory.
* If there are more physical memory to move a page frame from the disk, **pagging** occurs:
  * a page frame on the physical memory is evicted
  * If a page frame is dirty (modified) - page frame will be written to disk and replaced
  * else will be simply evicted from main memory
  * not all OS perform pagging (writing extra pages to disk). iOS and Android does not do so
  as writing to disk uses a lot of battery
* Thrashing:
  * Occurs when system runs low on physical memory can constantly having page fault and pagging.

### Process Memory

**Stack**:
* (klement: normal OS stuff)
* No fragmentation
* Allocating memory is always fast: adding a new stack frame is simply reserving the next K bytes
  * vs heap memory: that will have fragmented memory and will need to get memory from the freelist?

**Heap Memory**:
* `new`: allocate memory first then construct the object at the allocated memory
* `delete`: destruct the object with the destructor then freeing the memory
* Placement new:
  * construct the object at the memory address (new without the allocating memory)
  * need to pair with `std::destory_at`

#### Memory Alignment

* CPU read memory into register one word at a time
  * Word on 64 bit machine is 64 bit and 32 bit machine is 32 bit
  * The virtual address space is segmented into word boundaries (ie 0, 8, 16, 24 ...). As
  CPU read memory one word at a time, it is best to no cross word boundaries.
* Alignment: 
  * Power of 2
  * Formally: the number of bytes between successive address at which a given object can be allocated
    * Successive address does not mean an object is there but an allowed address.
    * ie: if `F` is 8 bytes aligned and address 0 is a valid address then the next successive address is 1
  * Informally: the number of bytes between possible addresses to allocate the object. Address of the object should
  be a multiple of the alignment
  * Alignment is used to make sure that the object do not cross word boundaries.
  * Objects on stack and heap will satisfy the alignment requirements.
* Alignment strictness:
  * The bigger the alignment requirement, the stricter the alignment.
  * Stricter alignment requirement still satisfy weaker alignment requirement. 16 byte aligned object is also a 1 byte aligned, 2 byte aligned, 4 byte aligned. All power of 2.
* Working with alignment:
  * `std::align`: takes a pointer and change it to be aligned according to the alignment requirement
    * ie given `ptr = 0x01` and alignment is `4`, the function will change `ptr = 0x04`
  * `std::max_align_t`: the most strict alignment requirement.
    * Usually 16 bytes.
    * When allocating memory from free store (heap), the OS will make sure that the alignment is the strictest ( 16 bytes )
  * Use `alignas`: to specify the custom alignment requirement. Setting higher alignment requirement can make sure that the objects will never be in the same cache line.
* Struct/User types:
  * The compiler should also try to satisfy the alignment requirement of the individual members:
  * ie: `struct S { char a; short b; int i };`
    * `char a` has alignment requirement of `1`, `short b` has alignment requirement of `2` and `int i` has alignment requirement of `4`.
    * `a` aligned at offset 0.
    * `b` aligned at offset `2` as only `0, 2, 4, 6, ..` are valid addresses.
    * `i` aligned at offset `4` as only `0, 4, 8` are valid addresses.
    
  
#### Padding

**Problem**: compiler will need to ensure that each member of a class/struct satisfy its own alignment requirement.

Example:
```cpp
class Document {
  bool is_cached_{};
  double rank_{};
  int id_{};
}
// Alignment requirement: 8

// without padding
// 0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15
// b  d  d  d  d  d  d  d  d  i  i  i  i  b  d  d 
```
* Without padding, the double data member will be at address `1` even though it needs to be in address that is a multiple of `8`
* The second document starts at address 13 even though the alignment requirement is `8`

**Solution**: compiler will add padding between data members to ensure data members alignment and also padding at the end of the last member to ensure
alignment between continguous identical object.
* Compiler achieve this by making sure the size of the class/struct is multiple of the alignment.

**Reducing the size of a struct**:
* To minimize the number of padding (=> smaller size), we can arrange the data member from largest to smallest alignment, this will make sure that there will not be unnecessary padding.

#### Memory Ownership

* Local variables owned by the current scope.
  * (klement: this is an interesting take that I did not thought about before)
* Static and global variables are owned by the program
* Data members owned by the instances
* Dynamic variables have no owner.

#### Smart Pointers

Unique Pointer:
* Slight overhead as unique pointer has a non-trivial destructor => cannot passed in a register when being passed to a function
  * [Talk by Chandler](https://www.youtube.com/watch?v=rHIkrotSwcc&t=1745s)

Shared Pointer:
* Always remember that `std::make_shared` uses one allocation vs `std::shared_pointer<T>(new T{})`.
  * More info [here](https://klementtan.com/readings/effective-modern-cpp/#item-21-prefer-stdmake_shared-and-stdmake_unique-to-direct-use-of-new)

#### Small object optimization

(klement: this part covers SSO. You can find more info from my take on it [here](https://github.com/klementtan/string_cpp))

#### Custom memory management

(klement: this part briefly covers how do custom allocator work with C++ but did not go in-depth on the allocating strategies. I found the part on pmr very useful as
I did not understand the power of it before this part.)

When will a custom allocator perform better:
* Guaranteed single-thread environment
* Fixed-size allocations: there will not be any fragmentation as chunks can completely fill previously freed chunks
* Limited lifetime: deallocate all at once => do not need to care about fragmentation

**Stack based allocator**:
* With a buffer on the stack or data segment, advance the start of available ptr with more allocation and move back the ptr when deallocating the last allocated segment
* If the buffer is not enough use the default operator new
* Objects can use the custom allocator by overriding the member operator new and delete
  * Allocator will not be used for vectors of the object or shared pointer (only `std::make_shared`)

```cpp
template <size_t N>
class Arena {
 static constexpr size_t alignment = alignof(std::max_align_t);
public:
  Arena() noexcept : ptr_(buffer_) {}
  Arena(const Arena&) = delete;
  Arena& operator=(const Arena) = delete;
  auto reset() noexcept { ptr_ = buffer_; }
  static constexpr auto size() noexcept { return N; }
  auto used() const noexcept {
    return static_cast<size_t>(ptr_ - buffer_);
  }
  auto allocate(size_t n) -> std::byte* {
    // all memory allocated from an allocator should support the
    // strictest alignment (std::max_align_t)
    const auto aligned_n = align_up(n);
    const auto available_bytes = static_cast<decltype(aligned_n)>(buffer_ + N - ptr_);
    if ( available_bytes >= aligned_n ) {
      auto* r = ptr_;
      ptr += aligned_n;
      return r;
    } else {
      return static_cast<std::byte*>(::operator new (n));
    }
  }
  auto dellocate(std::byte* p, size_t n) noexcept -> void {
    if ( pointer_in_buffer(p) ) {
      n = align_up(n);
      if (p + n == ptr_) {
        // only move the ptr backwards if the dellocated was the last allocated memory
        ptr_ = p;
      }
    } else {
      ::operator delete(p);
    }
  }
private:
  static auto align_up(size_t n) noexcept -> size_t {
    // round up n to the nearest alignment
    return (n + alignment - 1) & ~(alignment -1);
  }
  auto pointer_in_buffer(const std::byte* p) const noexcept -> bool {
    // klement: IIRC raw poitner comparison is well defined even if the pointer is out
    // of the buffer (ie ptr_ + N) but it UB if one of the pointer is not derived arithmatically
    // from the other buffer pointer (ie: separate allocation).
    // To resolve tihs use `std::uintptr_t` to convert the pointer to interger which will represent
    // a linear address space
    // more info: https://devblogs.microsoft.com/oldnewthing/20170927-00/?p=97095
    return std::uintptr_t(p) >= std::uintptr_t(buffer_) &&
           std::uintptr_t(p) < std::uintptr_t(buffer_) + N;
  }
  // without the stricter alignment requirement, the alignment requirement will be `1` which will not
  // support the 16 byte alignment requirement for allocators
  alignas(alignment) std::byte buffer_[N];
  std::byte* ptr_{};
}
```

**stl container compliant allocator**:
* Allocator will need to implement `allocate`, `deallocate`, `operator==`, `operator!=`
* Stateless allocator (ie Mallocator) - will always compare equal for the same type
* Need to implement `struct rebind` for `std::set`
  * You might provide an allocator for int (`Allocator<int>`) but for `std::set`, the `int` is a member of a tree node `Allocator<Node<int>>`. This struct will provide
  a `templated typedef` semantics.

**Polymorphic memory allocator**:
* `std::` container uses allocator that will change the type of the container as it is passed as a template param 
  * This will make it hard pass around containers of different allocators (even derived allocators will not work)
  * ie `void fn(std::set<int> s)` will not take in `std::set<int, MyAlloc>`
* Solution: `std::pmr` mirrors `std::` containers but uses `std::pmr::polymorphic_allocator`
  * Extra indirection between the container trying to allocate memory and the memory resource providing the memory
  * `std::pmr::polymorphic_allocator` is a stateful allocator, stores the type of memory resource it uses
   * Control flow:
    1. vector calls `std::pmr::polymorphic_allocator::allocate`
      * Non-standard but `std::pmr` container most probably stores an instance of `std::pmr::polymorphic_allocator`
    2. `std::pmr::polymorphic_allocator` will call the `do_allocate` virtual function of the memory resource
    3. memory resource will allocate memory
  * All memory resources derive from `std::pmr::memory_resource`, this will allow `std::pmr::polymorphic_allocator` to type
  erase the specific `memory_resource` and cast to the base `memory_resource` class => no type issue
* Caveat: pmr container stores a raw pointer to the `memory_resource` => need to ensure that the lifetime of a container does not exceed the memory resource
* Utility resource: the standard library provides quite a few good utility memory resource
  * `std::pmr::new_delete_resource`: resource that uses `new` and `delete`
  * `std::pmr::null_memory_resource`: resource that will always throw `std::bad_alloc`
  * `std::pmr::get_default_resouce`: global default resource. Can be set by the users.
  
(klement: Going to take a break from reading this book as it seems like it does not contain the in-depth insights into high performance I was looking for.)
