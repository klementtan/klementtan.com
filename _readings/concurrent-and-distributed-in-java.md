---
title: "Concurrent and Distributed in Java"
excerpt: "Personal notes"
toc: true
---

Title: Concurrent and Distributed Computing in Java
Author: Vijay Garg

## General Review

## Chapter 2: Mutual Exclusion Problem

### Introduction

#### Problem

If two processors $$P_0$$ and $$P_1$$ increment(`x = x + 1`) a shared variable that is initialized to
`0`, the final value of `x` might not be `2`.

Machine instructions for `x = x + 1`:

* `LD R, x`: load register `R` from `x`
* `INC R`: increment register `R`
* `ST R, x`: store register `R` to `x`

Certain interleaving of instructions could result in the value of `x` being `1`:

* $$P_0$$ - `LD R0, x`: load register `R0` from `x` (`R0 = 0`)
* $$P_0$$ - `INC R0`: increment register `R0` (`R0 = 1`)
* $$P_0$$ - `LD R1, x`: load register `R1` from `x` (`R1 = 0`)
* $$P_0$$ - `INC R1`: increment register `R1` (`R1 = 1`)
* $$P_0$$ - `ST R0, x`: store register `R0` to `x` (`x = 1`)
* $$P_1$$ - `ST R1, x`: store register `R1` to `x` (`x = 1`)

The underlying issue is that `x = x + 1` needs to be executed atomically

* Code that needs to be executed automatically are called **critical section**
* Problem of ensure critical section is executed atomically is called the **mutual exclusion problem**

### Mutual Exclusion

#### Properties

1. **Mutual exclusion**: two process cannot be in the critical section at the same time
2. **Progress**: If one more process are *trying to enter* the critical section and *no process inside* the critical section,
then *at least one of the process succeeds* in entering the critical section.
3. **Starvation-freedom**: if a process is trying to enter the critical section, then it eventually succeeds in doing so.

#### Peterson's Alogrithm

##### Single flag

Use a single flag (`openDoor`) to state if the critical section is empty

```java
class Attempt1 implements Lock {
  boolean openDoor = true;
  public void requestCS(int i) {
    while(!openDoor);
    openDoor = false;
  }
  public void releaseCS(int i) {
    openDoor = true;
  }
}
```

Issues: violates mutual exclusion

* Checking and setting of `openDoor` are not done atomically
* two process can read `openDoor = true`, exits the spinlock and enter CS

#### Double flag

Use double flag to state if a process wants the lock. Only enter CS if the other process does not want it

```java
class Attempt2 implements Lock {
  boolean wantCS[] = {false, false};
  public void requestCS(int i) {
    wantCS[i] = true;
    while(wantCS[1-i]);
  }
  public void releaseCS(int i) {
    wantCS[i] = false;
  }
}
```

Issues: starvation

* Both process could set `wantCS` together, this will result both process to forever wait on each other

#### Explicit turns

Explicitly set the turns in which process will enter CS. Pass the turn over once it exits the CS

```java
class Attempt3 implements Lock {
  int turn = 0;
  public void requestCS(int i) {
    while (turn == i - 1); // wait when it is no longer the other turn
  }
  public void releaseCS(int i) {
    turn = 1 - i;
  }
}
```

Issues: starvation

* When a process enters and exits a CS, only the other process is allowed to enter the critical section
* If the other process is not waiting to enter the CS, the process will wait indefinitely to **re-enter** the CS.

#### Peterson's Algorithm

Combine the idea of double flag (to state intent to enter CS) and explicit turn.

* Spin lock if the opposite party wants the CS and its the opposite party turn
* Exit spin lock if the opposite does not want CS or it is not the opposite party turn
* Exit CS by stating that it no longer wants CS

Disadvantage: Only works with two processors

```java
class PetersonAlgorithm implements Lock {
  boolean wantCS[] = {false, false};
  int turn = 1;

  public void requestCS(int i) {
    int j = 1 - i;
    wantCS[i] = true;
    int turn = j;
    while(wantCS[j] && (turn == j));
  }

  public void releaseCS(int i) {
    wantCS[i] = false;
  }
}
```

##### Mutual Exclusion Proof

**Prove by contradiction**:

* Assume both $$P_0$$ and $$P_1$$ are in the critical section
* (klement: seems like proving by contradiction is the easiest way to prove mutual exclusion)

**Case 1**: $$P_0$$ read `wantCS[1] = false` (break from the spinlock)

* If $$P_0$$ reads `wantCS[1] = false` and $$P_1$$ in critical section, then $$P_1$$ can only set `wantCS[1] = true` after $$P_0$$ reads it
* This means $$P_1$$ sets `turn = 0` after $$P_0$$ enters CS and before $$P_1$$ waits on `turn == 1`
* Implies the following sequence
  * $$P_0$$ sets `turn = 1`
  * $$P_0$$ reads `wantCS[1] = false` $$\Rightarrow$$ $$P_0$$ enters CS
  * $$P_1$$ sets `wantCS[1] = true`
  * $$P_1$$ sets `turn = 0`
  * $$P_1$$ reads `turn = 0` (stuck at spinlock)
* $$P_0$$ reads `wantCS[1] = false` $$\Rightarrow$$ $$P_0$$ already sets `wantCS[0] = true` and $$P_1$$ reads `wantsCS[0]` after it
* Since $$P_1$$ `wantCS[1] = true` and `turn = 0` **therefore** $$P_1$$ cannot be in CS

**Case 2**: $$P_0$$ reads `turn = 0`

* $$P_0$$ reads `turn` $$\Rightarrow$$ $$P_0$$ sets `wantCS[0] = true` and `turn = 1` before
* $$P_0$$ sets `turn = 1` but still reads `turn = 0` $$\Rightarrow$$ $$P_1$$ sets `turn = 0` $$P_0$$ sets `turn = 1`
* $$P_0$$ sets `turn = 1` $$\Rightarrow$$ $$P_0$$ sets `wantCS[0] = true`
* $$P_1$$ must read `turn = 0` and `wantCS[0] = true` **therefore** $$P_1$$ will be in spinlock and not critical section

For both process to be in CS:
$$wantCS[1] \bigwedge (turn = 1) \bigwedge wantsCS[0] \bigwedge (turn = 1)$$

##### Progress Proof

Case 1: When both $$P_0$$ and $$P_1$$ are in spinlock, the one whose `turn = i` it is will go in
Case 2: When only $$P_0$$ or $$P_1$$ are in spinlock, the opposite `wantCS[i] = false` and break spinlock

##### No starvation proof

* Will unstarve the other process by setting `wantCS` to false
* If re-try CS after returning will set the `turn` to the opposite party

#### Lamport's Bakery

Aims to overcome the disadvantage of Peterson's Algorithm of only being able to work with
two processors

*Intuition*:

1. Doorway Step: Each process that requests critical section will receive a number

* The later the request, the higher the number

2. All process will check that all other process has completed the doorway step
3. The process with the smallest number will be allowed into the critical section

```java
class Bakery implements Lock {
  int N;
  boolean[] choosing;
  int[] number;

  public Bakery(int numProc) {
    N = numProc;
    choosing = new boolean[N];
    number = new int[N];
    for (int j = 0; j < N; j++) {
      choosing[j] = false;
      number[j] = 0;
    }
  }

  public void requestCS(int i) {
    // step 1 doorway
    choosing[i] = true;
    for (int j = 0; j < N; j++) {
      if (number[j] > number[i]) {
        number[i] = number[j];
      }
    }
    number[i]++;
    choosing[i] = false;

    for (int j = 0; j < N; j++) {
      while( choosing[j] );
      while ((number[j] != 0) &&
              ((number[j] < number[i]) ||
                ((number[j] == number[i]) && j < i));
    }
  }
  public void releaseCS(int i)  {
    number[i] = 0
  }
}
```

##### Mutual Exclusion Proof

**Text book proof** (klement: IMO this proof is much cleaner as compared to the one in the lecture notes)

*Assertion 1*: If $$P_i$$ is in critical section and some other process $$P_k$$ has
already chosen its number, then $$(number[i], i) < (number[k], k)$$

* If $$P_i$$ is in critical section, then it has exited the kth iteration(`int j = k`)
of busy waiting. It implies that:
  * `number[k] = 0`:
    * *Case 1*: $$P_k$$ has not entered the doorway $$\Rightarrow$$ will read the latest value
    of `number[i]` $$\Rightarrow$$ `number[i] < number[j]`
    * *Case 2*: $$P_k$$ has entered the doorway $$\Rightarrow$$ $$P_k$$ enter after $$P_i$$ checks `while(choosing[k])`
    $$\Rightarrow$$ $$P_i$$ has already set `number[i]` $$\Rightarrow$$ $$P_k$$ will set `number[k] > number[i]`
  * `(number[i], i) < (number[j], j)`: satisfy assertion

*Assertion 2*: If $$P_i$$ is in critical section, then (`number[i] > 0`)

* From the code the `number[i]` is at least `0` initially and will be incremented by `1`

*Mutual exclusion proof*:

* If $$P_i$$ and $$P_k$$ are in critical section then their `number[i/j] > 0` (**assertion 2**)
* From **assertion 1**, only either `(number[i], i) < (number[k], k)` or `(number[k], k) < (number[i], i)` can satisfy
but for both to be in the critical section both must be true which is a contradiction

(TODO: add lecture notes proof)

##### Progress Proof

* Let $$P_i$$ be the process with the smallest number
* All other process $$P_j$$ will eventually set `choosing[j] = false`
  * $$P_i$$ breaks spinlock 1
* $$P_i$$ will break spinlock 2 as it has the smallest number
* **therefore** there will always be progress

##### Starvation Proof

* Let $$P_i$$ be the process that is starved
* Any new request from $$P_j$$ will have number more than $$P_i$$
* Eventually, all process will have higher number than $$P_i$$
* $$P_i$$ will be have the smallest number $$\Rightarrow$$ $$P_i$$ will enter CS (contradiction)

### Hardware Solution

#### Disabling interrupts

Disable interrupts when entering CS and enable interrupts when leaving CS

Disable all interrupts $$\Rightarrow$$ **disable context switching** $$\Rightarrow$$ no interleaving of instructions
$$\Rightarrow$$ no race condition

Disadvantage:

* not feasible if threads are mapped across different CPU
* Interrupts are required for some operations

#### Instruction with higher atomicity

`TestAndSet`

```java
// whole function excuted automatically
boolean TestAndSet(Boolean openDoor, boolean newValue) {
  boolean tmp = openDoor.getValue();
  openDoor.setValue(newValue);
  return tmp;
}

shared Boolean variable openDoor initialized to true;
RequestCS(progress_id) {
  while (TestAndSet(openDoor, false) == false);
}

ReleaseCS(process_id) {openDoor.setValue(true);}
```

* Only the first process will get `TestAndSet(openDoor, true) == true` and break the spinlock
* All other process will get the value set by that process and stuck in spin lock

### Chapter 2 homework

* **2.1**: Show that any of the follow modification to Peterson's algorithm makes it incorrect
  * **2.1.a**: A process in Peterson's algorithm sets the turn variable to itself instead of setting it to the other process
    * klement: This could lead to starvation as a process could be stuck in the spinlock forever. If $$P_0$$ is stuck in the spin lock and
    $$P_1$$ leaves the CS and immediately request the critical section. $$P_1$$ could set `wantCS[1] = true` immediately after setting `wantCS[1] = true` when exiting the CS and since the
    processes self-assign the turn, `turn` remains at `1`. This will result in $$P_0$$ getting starved
    * answer:
      1. $$P_0$$: `wantCS[0] = true`
      2. $$P_0$$: `turn = 0`
      3. $$P_0$$: `wantCS[1] && (turn == 1)`
      * enter CS as `wantCS[1] = false`
      4. $$P_1$$: `wantCS[1] = true`
      5. $$P_1$$: `turn = 1`
      6. $$P_1$$: `wantCS[0] && (turn == 0)`
      * enter CS as `turn = 1`
  * **2.1.b** A process sets the turn variable before setting the `wantCS` variable
    * klement: idk
    * answer:
      1. $$P_1$$: `turn = 1`
      2. $$P_0$$: `turn = 0`
      3. $$P_0$$: `wantCS[0] = true`
      4. $$P_0$$: `while(wantCS[1] == true && turn == 1)`
      * $$P_0$$ enters CS as `wantCS[1] = false`
      5. $$P_1$$: `wantCS[1] = true`
      6. $$P_1$$: `while(wantCS[0] == true && turn == 0)`
      * $$P_1$$ enters CS as `turn = 1`
* **2.2**: Show that Peterson's algorithm also guarantee freedom from starvation
  * klement:
    * Assume that $$P_0$$ has been starved. This means $$P_0$$ is stuck on the spinlock for all iteration and $$P_1$$ gains the CS for all iteration 0 to N
    * If $$P_1$$ gains CS for iteration `i` and `i+1`. This means that `turn = 1` at iteration `i` and `i + 1`.
    * However, $$P_0$$ is stuck at spinlock while $$P_1$$ sets `turn = 0` before checking the spin lock at `i + 1`
    * Therefore, at the `i + 1` iteration the value of `turn = 0` and contradicts that `turn = 1` at iteration `i` and `i+1`
* **2.3**: Show that the bakery algorithm does not work in absence of `choosing` variable
  * klement: idk
  * Ansewr:
    1. $$P_1$$: `number[1] = number[0]`
    2. $$P_0$$: `number[0] = number[1]`
    * Both `number[0] = number[1] = 0`
    3. $$P_1$$: `number[1]++`
    4. $$P_1$$: `while(number[0] != 0 && Smaller(number[0], 0, number[1], 1))`
    * Enter CS as `number[0] = 0`
    5. $$P_0$$: `number[0]++`
    6. $$P_0$$: `while(number[0] != 0 && Smaller(number[0], 0, number[1], 1))`
    * Enter CS as `number[0] == number[1]` but pid of $$P_0$$ is smaller
* **2.4**:
  * Question does the Dekker alogrithm fullfill all properties

    ```java
    class Dekker implements Lock {
      boolean wantCS[] = {false, false};
      int turn = 1;
      public void requestCS(int i) {
        int j = 1 -1;
        wantCS[1] = true;
        while(wantCS[j]) {
          if (turn == j) {
            wantCS[i] = false;
            while(turn == j);
            wantCS[i] = true;
          }
        }
      }
      public void releaseCS(int i) {
        turn = 1 - i;
        wantCS[i] = false;
      }
    }
    ```

  * Answer: True for all 3 properties
  * Mutual Exclusion Proof:
    * Case 1: `turn = 0`
      * $$P_1$$ must have seen `wantCS[0] = false` (if not it will stuck at `while(turn == 0)`)
      * $$\Rightarrow P_1$$ see that `wantCS[0] = false` in `while(wantCS[0])`.
      * $$\Rightarrow$$ $$P_0$$ havent execute `wantCS[0] = true`
      * $$\Rightarrow$$ when $$P_0$$ reach `while(wantCS[1])`, `wantCS[1] = true` and stuck in spinlock
    * Case 2: symmetry
  * Progress Proof:
    * Case 1: `turn = 0`
      * `P1`: `wantCS[1] = false` eventually (does not matter which part)
      * `P0` will enter the critical section after `wantCS[1] = false`
    * Case 2: `turn = 1`
  * Starvation Proof: TODO understand this proof

## Chapter 3: Synchronization Primitives

The OS provides support for alternatives to busy waiting using spin lock

### Semaphores

* All statements within the semaphores are mutually exclusive (`synchronized` qualifier in method)
* Binary semaphores: keep waiting until `value = true` when trying to gain lock. Set `value = true` when releasing the lock
* Counting semaphores:
  * `P()`: decrease the value by `1` and wait if `value < 0` (reached maximum number of allowed in CS)
  * `V()`: increase the value by `1` and notify if `value <= 0` (replace another waiting process with it)

#### Semaphore - Producer and Consumer

Problem:

* Producer and consumer share a single buffer of fixed size
* Maintain a `inBuf` and `outBuf` (`outBuf` <= `inBuf` cyclically)
* Requirement:
  * Consumer should not fetch any time from an empty buffer
  * Producer should not deposit any item in the buffer if it is full
  * Considered conditional synchronization as the process need to synchronize on custom logic
* Any reading to shared variable (`inBuf`, `outBuf`, `size`, `buffer`) must be done atomically

```cpp
class BoundedBuffer {
  final int size = 10;
  double[] buffer = new double[size];
  int inBuf = 0, outBuf = 0;
  BinarySemaphore mutex = new BinarySemaphore(true);
  CountingSemaphore allowedProducer = new CountingSemaphore(size);
  CountingSemaphore allowedConsumder = new CountingSemaphore(0);

  public void deposit (double value) {
    allowedProducer.P();
    mutex.P();
    buffer[inBuf] = value;
    inBuf = (inBuf+1) % size;
    mutex.V();
    allowedConsumder.V();
  }

  public void fetch() {
    double value;
    allowedConsumder.P();
    mutex.P();
    value = buffer[outBuf];
    outBuf = (outBuf + 1) % size;
    mutex.V();
    allowedProducer.V();
  }
}
```

* General Idea:
  * Use semaphore to keep track the number of allowed deposit and fetch at any time
    * Initially size *deposit and 0* fetch operations allowed
  * When a deposit action is used up, a corresponding fetch operation will be allowed. Consumer can consume the produced item
  * The converse is true when a fetch operation is used up
* deposit:
  * Decrement the number of allowed deposit. Block if all the allowed deposit has been used up (semaphore < 0)
  * Lock the mutex (gain access to shared variable)
  * deposit the item
  * Unlock the mutex (release the shared variable)
  * Increment the number of allowed fetch
* fetch:
  * Decrement the number of allowed fetch. Block if all the allowed fetch has been used up (semaphore < 0)
  * Lock the mutex (gain access to shared variable)
  * fetch the item
  * unlock the mutex (relase the shared variable)
  * Increment the number of allowed deposit

#### Semaphore - Reader and Writer

Problem:

* No read-write conflict: reader and a writer do not access the database concurrently
* No write-write conflict: two writer do not access the database concurrently

With starvation

```cpp
class ReaderWriter {
  int numReaders = 0;
  BinarySemaphore mutex = new BinarySemaphore(true);
  BinarySemaphore wlock = new BinarySemaphore(true);

  public void startRead() {
    mutex.P();
    numReaders++;
    if (numReaders == 1) wlock.P();
    mutex.V();
  }

  public void endRead() {
     mutex.P();
     numReaders--;
     if (numReaders == 0) wlock.V();
     mutex.V();
  }
  public void startWrite() {
    wlock.P();
  }
  public void endWrite() {
    wlock.V();
  }
}
```

* startRead:
  * Lock shared variable `numReaders`
  * If the current reader is the first reader, try to get the write lock
    * Other readers will not be able to enter this body as the shared variable is locked
  * Once acquire the write lock, release the share variable lock to allow other readers
* endRead:
  * Lock shared variable `numReaders`
  * If the current reader is the last reader release the write lock
  * Once release the write lock, release the share variable lock to allow other readers

TODO: Add starvation free Reader and Writer

### Monitors

Overview:

* Monitor is a class that has data members and methods
* Monitor Lock
  * Have synchronized methods or synchronized code block then ensure mutual exclusion with all
  other synchronized methods or code block.
  * Only a single thread can execute code across all synchronized method or block at a time.
    * If thread 1 is in synchronized method 1 thread 2 cannot enter synchronized method 2
  * If a thread acquires the monitor lock, it is "in the monitor"
* Conditional lock
  * Each monitor has a single conditional lock that starts off as **locked**
  * When a thread enters the monitor and checks that a condition is not valid,
  it can call `wait()` and get blocked until another threads changes the condition
  and call `notify()` on it
  * When a thread calls `wait()` and block, it will exit the monitor and be added into the
  condition queue
  * When another thread calls `notify()`, a thread in the condition queue will be blocked.
    * What happens next depends on the type of monitor lock (Hoare monitor or Java monitor)

#### Hoare Monitor

1. `t1` calls `wait()` and get added to condition queue
2. `t2` calls `notify()` and wakes up `t1`
3. `t2` **immediately exits monitor** and enter the monitor lock queue (lose execution)
4. `t1` **immediately enters the monitor** and gain access to monitor lock

**Advantage**: There is no change in state when a thread (`t2`) calls `notify` when a condition
is valid to when the corresponding thread (`t1`) unblocks from `wait`. This means that the condition
that `t2` checks will be the same when `t1` unblocks

#### Java Monitor

1. `t1` call `wait()` and get added to condition queue
2. `t2` calls `notify()` and wakes up `t1`
3. `t1` gets pushed over to the **monitor lock queue**
4. `t2` continues the execution in the monitor
5. `t1` enters the monitor and continue from `wait()` when `t2` exits the monitor

**Disadvantage**:

* There could be a change in state when `t2` calls `notify()` and `t1` unblocks from `wait()`.
* `t2` could change the state/condition after calling `notify()`
* To resolve this:

  ```cpp
   while (!B) x.wait();
  ```

**Producer and Consumer**

```java
class BoundedBufferMonitor {
  final int sizeBuf = 10;
  double [] buffer = new double[sizeBuf];
  int inBuf = 0, outBuf = 0, count = 0;

  public synchronized void deposit (double value) {
    while (count == sizeBuf)
      wait();
    buffer[inBuf] = value;
    inBuf = (inBuf +1) % sizeBuf;
    count++;
    if (count == 1)
      notify();
  }

  public synchronized double fetch() {
    double value;
    while (count == 0)
      wait();
    value = buffer[outBuf];
    outBuf = (outBuf + 1) % sizeBuf;
    if (count == sizeBuf - 1)
      notify();
    return value;
  }
}
```

* When any producer or consume start executing, it will always gain the monitor
lock
* Producer will block until there are space (`count < sizeBuf`) and consumer will wait until there are items (`count > 0`)
* When either the producer or consumer finish execution, there will be change in
state and it will call `notify` to wake up any threads that are blocked
* Blocked thread will wake up and check the state mutually execlusively

### Homework

Lock free reader writer

```java
Vector queue;
int numReader; int numWriter;

// Writer entry
Writer w = new Writer(myname);
synchronized(queue) {
  // need to be blocked
  if (numReader > 0 || numWriter > 0) {
    w.OkToGo = false;
    queue.add(w);
    // if do w.wait() here will result in nested monitor
  } else {
    w.okToGo = true;
    numWriter++;
  }
}

// w only blocked to itself
synchronized (w) { if (!w.okToGo) w.wait(); }

// code for writer to exit
synchronized (queue) {
  numWriter--;
  if (queue is not empty) {
    // rmeove a single write or a btach of readers from head of queue
    for each wrier/reader removed do {
      numWriter++ or numReader++;
      synchronized(queue.front()) {
        queue.front().okToGo = true;
        request.notify();
      }
    }
  }
}

// code for reader entry
Reader r = new Reader(myname);
synchronized (queue) {

  if ((numWriter > 0) || !queue.isEmpty()) {
    r.OkToGo = false;
    queue.add(r);
  } else {
    r.OkToGo = true;
    numRead++;
  }
}
syschronized (r) {if (!r.okToGo) r.wait();}

synchronized (queue) {
  numReader--;
  if (numReader > 0) exit();
  if (!queue.isEmpty()) {
    // remove single writer or a batch of readers from queue
    for each request removed do {
      numWriter++ or numReader++;
      synchronized (request) {
        request.okToGo = true;
        request.notify();
      }
    }
  }
}
```

## Chapter 4: Consistency Conditions

### System Model

#### General Definitions

**Concurrent System**: consist of a set of *sequential process* that communicate
through *concurrent objects*

**Concurrent Object**:

* Has a *name*
* Has a *type*:
  * defines the set of possible values for the object
  * defines the set of primitive *operations* to manipulate the object

**Execution of operation**:

* It takes time to execute an operation
* Modelled by *invocation event* and *response event*
* Notation:
  * $$op(arg)$$ is an oepration on object $$x$$ issued at $$P$$. $$arg$$ denotes the operation input and output
  * Invocation and Response events: $$inv(op(arg))$$ and $$resp(op(arg))$$ are abbreviated as $$inv(op)$$ and $$resp(op)$$

$$proc(e)$$: denotes the process for the operation

$$object(e)$$: denotes the *objects* associated with the operation

* For this chapter, all operations are applied by a single process on a single object

#### History

**History** ($$H$$, $$<_ H$$):

* Is a directed acyclic graph
* $$H$$ is the set of operations
* $$<_ H$$ is an irreflexible (no edge to itself) transitive (through other nodes) relation
  * There is no transitive path to itself
* Captures the occurred before relationship between operations

**$$<_ H$$** definition:

$$e <_ H f$$

* $$resp(e)$$ occurred before $$inv(f)$$
* If $$e$$ and $$f$$ overlap there is no $$<_ H$$ relation

**Process order** definition

$$(proc(e) = proc(f)) \wedge (resp(e) \text{ occurred before } inv(e))$$

* For any two operations $$e$$ and $$f$$, the response of the first operation must occur before the invocation of the next (no overlapping operation) for
the same process

**Object order** definition

$$(object(e) \cap object(f) \neq \emptyset) \wedge (resp(e) \text{ occurred before }inv(f))$$

* For any two operation on the same object, the response of the first operation occur before the second
* TODO: check if this definition

**Process subhistory** ( $$H\mid P$$ ): is sequence of the events in $$e$$ in $$H$$ such that $$proc(e) = P$$ (all events involves $$P$$)

**Object subhistory** ( $$H\mid P$$ ): is sequence of the events in $$e$$ in $$H$$ such that $$object(e) = x$$ (all events involves $$x$$)

**Equivalent history**: two history are composed of exactly the same set of invocation and events.

* Ordering of operations does not matter

**Sequential History**: if $$<_ H$$ is a total order (DAG is a linear line).

* History would occur if there was only one sequential process in the system (linear line of DAG)

**Legal Sequential History**: it meets the sequential specification of all the objects (the specification for the object if only a single sequential process)

* ie read-write register $$x$$ is legal if:
  * every read operation that returns $$v$$, the latest write operation before the read operation has to write $$v$$
* ie sequential queue:
  * If the queue is not empty, it should return the item that is enqueued the earliest.
  * If the queue is empty, then it should return null

### Sequential Consistency

**Sequential Consistency** definition: A history $$(H, <_ H)$$ is sequentially consistent if

* there exists a sequential history $$S$$ equivalent to $$H$$ such that
  * $$S$$ is legal sequential execution
  * and $$S$$ satisfies process order
    * Each process behaviour is the same in concurrent and sequential execution (ordering within the process is the same)

Sequential Consistency Examples

1. $$H_1 = P \ write(x,1), Q\ read(x), Q\ ok(0), P \ ok()$$

* $$H_1$$ is sequentially consistent as it is equivalent to the following legal sequential history
    $$S = Q\ read(x), Q\ ok(0), P\ write(x,1), P\ ok()$$

2. $$H_2 = P \ write(x,1), P \ ok(), Q\ read(x), Q\ ok(0)$$

* $$H_2$$ is a sequential history (operations are linear) but it is not legal (write 1 then read 0).
* It is sequentially consistent as it is equivalent to
    $$S = Q\ read(x), Q\ ok(0), P\ write(x,1), P\ ok()$$

3. $$H_3 = P \ write(x,1), Q\ read(x),P \ ok(), Q\ ok(), P\ read(x), P\ ok(0)$$

* Is not sequentially consistent as the process order must be maintained
  * P writes 1 before P reads 0 which is not possible as there are no other write operation

4. $$H_4 = P\ write(x,1), Q\ read(x), P\ ok(), Q\ ok(2)$$

* Not sequentially consistent as no equivalent history will contain a write(x,2) for Q to read 2

### Linearizability

**Intuition**: If we assume that each operation occurs instantaneously at any point between the invocation and response,
the execution is legal.

**Linearizability Definitions**:

* Lecture Notes:
  * execution is equivalent to some legal execution such that each operation happens instantaneously at some point between the invocation and response
* Text Book:
  * A history ($$H, <_ H$$) is linearizable if
    * There exists a sequential history ($$S, < $$) equivalent to $$H$$
    * And it preserves $$<_ H$$
      * Non-overlapping operations need to maintain ordering

**Examples**:

* $$H_1 = P\ write(x,1), Q\ read(x), Q\ ok(x), P \ ok()$$
  * Is linearizable as both operations are overlapping and do not have $$<_ H$$ relationship
  * Equivalent to:
    $$Q\ read(x), Q\ ok(0), P\ write(x,1), P\ ok()$$
* $$H_2 = P\ write(x,1), P\ ok(), Q\ read(x), Q\ ok()$$
  * There is a $$P\ write <_ H Q\ read$$ relationship and we need to maintain it.
  * This means that we have to write x = 1 before reading x = 0 which is not legal

### Local Property

**Local Property** for linearizablity definition: for all objects $$x$$, $$H\mid x$$ is linearizable $$\Rightarrow$$ $$H$$ is linearizable

**Local Property** for sequential consistent:

* Does not hold for sequential consistency

#### Local Property Proof

1. Given linearization of all objects $$x$$ ($$H\mid x$$), we will have a linear DAG ($$S_x, <_ x$$)
2. Construct an acyclic graph that orders all operations on any object

* Combining all the DAGs across all objects ($$<_ x$$)
* Includes all the ordering in history across objects ($$<_ H$$)
  * This graph preserves $$<_ H$$ which checks the second condition of linearizablity

3. Show that the new graph is acyclic (sequential history):

* $$<_ x$$ are acyclic (linearizable) so if a cycle exists it must involve two objects
* Can combine all adjacent similar edge:
  * Two adjacent $$<_{x_1}$$  and $$<_ {x_2}$$ into a single edge
  * Two adjacent $$<_{H_1}$$  and $$<_ {H_2}$$ into a single edge
* End up with $$<_H , <_ x , <_H$$ or  $$<_ x , <_H , <_ x$$
* If $$<_H , <_ x , <_H$$ leads to a cycle means there is a cycle in $$<_ H$$ which contradicts as all $$<_ H$$ is transitively irreflex
  
### Casual Consistency (out of CS4231 scope)

**Intuition**: Minimum possible time to read and write is less than the communication latency

* It takes a long time for a process to know of write events by other process (ie distributed memory)
* There is a communication delay for events

**Definitions**: A history ($$H, <_H$$) is casusally consisten if for each process $$P_i$$, there is a legal sequential history ($$S_ i, <_ {S_i}$$)

* Where $$S_i$$ is the set of all operations of $$P_i$$ (only $$P_i$$)
* All the write operations in $$H$$
* $$<_ {S_i}$$ respects the following order:
  * Process order: If $$P_i$$ performs $$e$$ before $$f$$, then $$e$$ is ordered before $$f$$ in $$S_i$$
  * Object order: all operations must be legal (read-write legal)

Notes:

* Related writes be seen by all process in the same order
* Concurrent writes may be seen in different order
* Sequential consistency -> Casual Consistency
* Casual Consistency implies that legal sequential history for every process and not for the entire system

### Consistency Definitions for Registers

* **atomic**:
  * Linearizable history
* No name for sequential consistent history
* Regular:
  * When read does not overlap with write
    * read returns the most recent write
  * When read overlap with write
    * read returns one of the most recent write
    * or the value written by one of the overlapping writes
* Safe:
  * When read does not overlap with any write:
    * returns the value written by one of the most recent writes
  * When read overlaps with writes, can return anything

## Chapter 7: Models and Clocks

**Motivation**: to allow for ordering of events in a distributed system. Types of ordering:

* Physical time model: assume that there is a shared physical clock where you
can get the time of all events. This will allow you order of all the events where
events of different processes will be interleaved.
* Happened before model: where you only order events if one **happen-before** another. This will result in partial ordering as not all
events happened-before.

### Model of a Distributed Systems

**Characteristics**:

* **Absence of a shared clock**: it is impossible to have a completely in-sync
shared clock.
* **Absence of shared memory**: no process can have a global view of all process
* **Absence of accurate failure detection**: not possible to determine if a process is slow or has
failure.

**Application**: based on message passing between processes.

* Messages are passed asynchronously between processes
* Consist of a set of $$N$$ processes.
* *Channel*: Consists of a set of unidirectional communication channel between two processes.
  * Channel has infinite buffer and error free.
  * State of a channel is the sequence of messages sent but not yet received.
* *Process*:
  * Has a defined set of states and initial condition (initial state)
  * Each event:
    * May change the state of the process
    * And state of at most one channel (cannot send to multiple channel at the sametime)

### Model of Distributed Computation

#### Interleaving Model

* Defines a *run* as a global sequence of events (across multiple processes)
* Events can be totally ordered

#### Happened-Before Model

* Definition: The **happened-before** relationship ($$\rightarrow$$) is the **smallest** relation that satisfy
  * If $$e$$ before $$f$$ in the **same process** then $$e \rightarrow f$$ (Same process)
  * If $$e$$ is the **send event** of a message and $$f$$ is **receive event** then $$e \rightarrow f$$ (Send-receive)
  * If there exists an event $$g$$ such that $$e \rightarrow g$$ and $$g \rightarrow f$$ then $$e\rightarrow f$$ (transitive)
* A *run* is defined by a tuple $$(E, \rightarrow)$$ where $$E$$ is the set of all events and $$\rightarrow$$ is the partial ordering
* Transitive edges are not drawn (smallest relation)
* In distributed computing only has **happened-before**

## Logical Clocks

Goal: Gives a *total order* that *could have* happened.

Logical Clock constraint: map from set of events to a set of natural number (each event has a number) that satisfy the following constraint

$$\forall e,f \in E: e \rightarrow f \Rightarrow C(e) < C(f)$$

* $$\rightarrow$$ represent happen before
* Contrapositive not true: the clock of $$e$$ can be smaller than $$f$$ but that does not mean that the two events have a happen before relationship

```java
public class LamportClock {

  int c;
  public LamportClock() {
    c = 1;
  }
  public int getValue() {
    return c;
  }

  public void tick() { // internal event
    c = c + 1;
  }

  public void sendAction() {
    // send message to receiver with c
    c = c + 1;
  }

  public void receiveAction() {
    c = Util.max(c, sentValue) + 1;
  }
}
```

* Initialization: all process will initialize their clock to `1`
* tick: represent an internal event occurred -> increment the
* sendAction: send the message with the current time and then increment the current time at the end.
* receiveAction: get the max of current time and time sent but sender then increment the time.

Total Ordering $$<$$:

$$ (s.c, s.p) < (t.c, t.p) = (s.c < t.c) \vee ((s.c = t.c) \wedge (s.p < t.p))$$

>

## Vector Clock

Definition: a map from state to a vector of dimension (n) with the following constraint

$$\forall s,t : s \rightarrow t \Leftrightarrow s.v < t.v$$

* As $$\rightarrow$$ (happen-before) is a partial order, $$<$$(>) is also a partial order

Comparing vectors:

$$x < y = (\forall k : 1 \leq k \leq N : x[k] \leq y[k]) \wedge (\exists j : 1 \leq j \leq N: x[j] < y[j])$$

* Elementwise $$\leq$$ but one element must be strictly less than

$$x \leq y = (x < y) \vee (x = y)$$

```java
public class VectorClock {
  public int[] v;
  int myId;
  int N;
  public VectorClock(int numProc, int id) {
    myId = id;
    N = numProc;
    v = new int [numProc];
    for (int i = 0; i < N; i++) v[i] = 0;
    v[myId] = 1
  }

  public void tick() {
    v[myId]++;
  }

  public void sendAction() {
    // send message and include the vector in the message
    v[myId]++;
  }

  public void receiveAction(int[] sentValue) {
    for (int i = 0; i < N; i++)
      v[i] = Util.max(v[i], sentValue[i]);
    v[myId]++;
  }

  public int getValue(int i) {
    return v[i];
  }
}
```

* Each process will have an interpretation of what progress (time) of all other process.
  * The interpretation of the current process time can only be updated by the current process
  * The interpretation of other process time can only be updated by receiving other process interpretation
* Construction:
  * Initialize all other process except the current process to `0`, current process is set to `1`
* tick:
  * Only increment the current process time by `1`
* sendAction:
  * Send the current process interpretation of all processes' time (send entire vector clock)
* receiveAction:
  * Perform element wise max with the sender interpretation of all process time (sender's vector clock)
    * Updates the current process interpretation if the sender interpretation is more updated
  * Progress the current process clock

### Direct Dependency Clock

* Same as vector clock but instead of updating the current process interpretation of all process time (vector clock) will
only update the current process interpretation of the sender time.

### Matrix Clock

A process will store all process interpretation of other processes time.

* This will allow a process know if all other process saw that process's event.

Components:

* Each process with store a $$N\times N$$ matrix where the $$j$$th row of $$P_i$$
represents $$P_i$$ interpretation of what $$P_j$$ interpret other processes time to be.
* $$i$$th row of $$P_i$$ represents the vector clock of $$P_i$$

```java
public class MatrixClock {
  int[][] M;
  int N;
  int myId;
  public MatrixClock(int numProc, int id) {
    myId = id;
    N = numProc;
    M = new int[N][n];
    for (int i = 0; i < N; i++)
      for (int j = 0; j < N; j++)
        M[i][j] = 0
    M[myId][myId] = 1;
  }

  public void tick() {
    M[myId][myId]++;
  }

  public void sendAction() {
    // send the matrix with the message first
    M[myId][myId]++;
  }

  public void receiveAction() {
    for (int i = 0; i < N; i++)
      if (i != myId)
        for (int j = 0; j < N; j++)
          M[i][j] = Util.max(M[i][j], W[i][j]);

    for (int j = 0; j < N; j++)
      M[myId][j] = Util.max(M[myId][j], W[srcId][j])
    M[myId][myId]++;
  }
}
```

* Construction:
  * Initialize all to `0` except for $$P_i$$ interpretation of its own time `M[i][i] = 0`
* tick:
  * Progress its own interpretation (`M[i][i]`) by 1
* sendAction:
  * Send its own interpretation (matrix clock) and progress its own interpretation by 1
* receiveAction:
  * for all other process's interpretation (`M[j]` where `j!=i`) perform elementwise max
  * for the current process interpretation (`M[i]`) perform elementwise max with the sender's interpretation (`W[j]`)
  * lastly progress the current time

Application:

* If $$\forall j : M[j][i] \geq k$$, all process are aware of event `k` in `M`.

## Chapter 9: Global Snapshot

### Overview

Problem: take a snapshot of a distributed system that could allow a system to be replayed.

*Global State* definition:

* *Time-based model*: a snapshot taken at a certain time
* *Happened-before model*: a set of local states that are concurrent with each other.
  * *Concurrent*: no two state have happened-before relationship

Global state *relationship*:

* A global state in *time-based* model **is** a global state in *happened-before*.
  * If you take a snapshot with a synchronized clock it will also be a happened-before
* A global state in *happened-before* **is not** a *time-based* model (converse not true)
  * A *happened-before* state might not have actually happened in an instance of time previously.

Choosing *happened-before* over *time-based* model:

* Impossible to get *time-based* model without a synchronized clock.
* *happened-before* allows for easy reasoning of mutual exclusion.

**Cut** (Global State) definition: of a computation model $$(E, \rightarrow)$$ (events and happened before relationship)
with total order ($$\succ$$) [$$e \succ f$$: $$e$$ can be ordered before $$f$$ based on time].

A *cut* is any subset $$F \subseteq E$$ such that

$$ f \in F \wedge e \succ f \Rightarrow e \in F$$

* If $$f$$ is in the cut and $$e$$ is ordered before $$f$$ then $$e$$ is also in the cut

**Consistent Cut** definition:

$$ f \in F \wedge e \rightarrow f \Rightarrow e \in F$$

* If $$f$$ is in the cut and $$e$$ **happened-before** $$f$$ then $$e$$ is also in the cut.

**Snapshot of messages**:

* Cut should include messages that are in transit at that instance
* Messages can change the local state of a process.
* If the snaphot is taken on $$P_i$$ after message is sent from $$P_i$$ to $$P_j$$
  * but $$P_j$$ has already taken a local snapshot (maybe $$P_i$$ to $$P_j$$ slower than $$P_i$$ to $$P_k$$ to $$P_j$$)
  * the messages sent by $$P_i$$ to $$P_j$$ initially should be part of the global state.

### Chandy and Lamport's Global Snapshot Algorithm
  
#### Algorithm Overview

**Requirement**:

* Communication channel are unidirectional (easily extend to bidirectional by adding two channel)
* Communication channel are FIFO

**Colouring**:

* Each process is assigned a colour of **white** or **red**:
  * **White**: represents before the snapshot was taken
  * **Red**: represents after the snapshot was taken
* A global state snapshot is right before all the process turn **red**
  * A local state snapshot is right before a process turn **red**

**Messages**:

* When a process turns **red** it will send a **marker** to all of its neighbours
* A process will turn **red** once it receives a **marker** if it has not already done so.
* Types of state of channels:
  * **ww** - white to white messages: sent and received before the snapshot. Process will update local
  state based on the message before snapshot thus ww messages can be ignored.
  * **rr** - red to red messages: sent and received after the snapshot. This message does not affect
  the local snapshot and can be ignored.
  * **rw** - red to white messages: not possible as all red process will send a marker to all neighbours. All neighbours
  will turn red before receiving any red messages.
  * **wr** - white to red messages: messages sent by white process.
    * Intuively, this could be a ww message if it was sent instantaneously. Thus the receiving process could
      be in a different state before the snapshot if the message was processed instantaneously.
    * Happens when the receiving process already received a "marker" from another process and already recorded
      local snapshot before receiving the message.

#### Algorithm

**State**:

* `myColour`: Process will track their own colour
* `closed[k]`: Each process will track the colour of neighbouring process.
  * `closed[k] = true` when receive a marker from $$P_k$$
* `chan[]`: for each neighbour, store a buffer that contains all **wr** messages. Print from buffer once the sender (previously white)
turns **red** (send marker).

```java
public class RecvCamera extends Process implements Camera {
  static final int white = 0; red = 1;
  int myColor = white;
  boolean closed[];
  CamUser app;
  LinkedList chan[] = null;
  public RecvCamera (Linker initComm, CamUser app) {
    super(init(Comm));
    closed = new boolean[N];
    chan = new LinkedList[N];
    for (int i = 0; i < N; i++) {
      if (isNeighbour(i)) {
        closed[i] = false;
        chan[i] = new LinkedList();
      }
    }
    this.app = app;
  }

  public synchronized void globalState() {
    myColor = red;
    // record local state
    app.localState(); 
    // send marker to neighbours
    sendToNeighbours("marker", myId);
  }

  public synchronized void handleMsg(Msg, m, int src, String tag) {
    if (tag.equals("marker")) {
      // signifies that src is red
      if (myColor == white) globalState(); // record global state once
      // will no longer receive white messages from src
      closed[src] = true;

      // Check if all channels are closed
      if (isDone()) {
        System.out.println("Channel State: Transit Messages");
        for (int i = 0; i < N; i++) {
          if (isNeighbour(i)) {
            // print any wr messages
            while (!chan[i].isEmpty()) {
              System.out.println(
                char[i].removeFirst().toString()
                  );
            }
          }
        }
      } else {
        if (myColor == red && !closed[src]) {
          // signifies wr message. Src is white and this process is red
          chan[src].add(m);
        } 

        // all other messages can be handled normally.
        app.handleMsg(m,src, tag);
      }
    }
  }

  bool isDone() {
    if (myColor == white) return false;
    for (int i = 0; i < N; i++) {
      if (!closed[i]) return false;
    }
    return true;
  }
}
```

Steps

1. When globalState is initiated at a particular process, it will send a marker to all neighbouring process.
2. When the other process receive the marker it will turn red and relay to its neighbours
1. If in other white process send a message to this red process, that message matters and will be added
  to the channel state
3. End when all process has turned red.

### Global Snapshot for non-FIFO channel

* Each message will contain the colour of the process.
* The receiver will keep looking for white messages from the sender eventhough it receives a read message already (ordering no guaranteed)

(not in scope of CS4231)

### Channel Recording by the Sender

* To prevent any lost message not being in the state.
* A white sender will keep a buffer of all outing messages before the marker is sent.
* A white receiver will ACK all white messages. ww messages can be removed from the state.

## Chapter 12: Message Ordering

$$s_i \rightsquigarrow r_i$$: The send event ($$s_i$$) that corresponds with the receive
event ($$r_i$$)

**FIFO Order**:
* The message that is first sent by the sender is first received
by the receiver.
* Limited to a sender and receiver process (2 process)
* Definition: any two message from a process $$P_i$$ to $$P_j$$ are received in the same order as they
were sent.
* Does not have transitive property across different pairs of processes.
* Mathematical definition: if a send event $$s_1$$ occurred before $$s_2$$ then the receive
event $$r_1$$ of $$s_1$$ must occur before the receive event of $$r_2$$ of $$s_2$$

$$s_1 \succ s_2 \Rightarrow \neg (r_2 \succ r_1)$$


### Casual Ordering

**Casual Order**:
* A sequence of messages (through different process) cannot overtake any message.
* ie: If a message send from A to C first then two messages sent from A to B then B to C. The later
two message cannot overtake the first message.
* Has transitive property across different paris of process
* Definition: If a $$s_1$$ happens before $$s_2$$ ($$s_1 \rightarrow s_2$$) then the receive event $$r_2$$
of $$s_2$$  cannot occur before the receive event $$r_1$$ of $$s_1$$

$$s_1 \rightarrow s_2 \Rightarrow \neg (r_2 \succ r_1)$$

**Casual Ordering Algorithm** for process $$P_i$$:
* Data Members:
  * `M`: 2D array `[1...N, 1...N]` of initialised to `0`
* To send message to $$P_j$$
  * $$M[i,j]= M[i,j] + 1$$ (increment) the `[i][j]` entry by one
  * piggy back $$M$$ as part of the message (increment first then send)
* To receive a message from $$P_j$$ with $$W$$ piggy-backed in the message
  * Enable if
    * $$W$$'s `[i][j]` entry is more than the $$M$$'s `[i][j]` entry by $$1$$
    * All other entry of $$M$$ is pair wise greater than or equal to $$W$$
    * Update $$M$$ to be the pairwise max of $$W$$ and original $$M$$
    * else buffer then message until the condition is true
    * Formally:

    $$\text{enabled if } (W[j,i] = M[j,i] + 1) \wedge (\forall k \neq j: M[k,i] \geq W[k,i])$$

Intuition:
* Increment the count when sending message: to make it known that a message is sent from $$P_i$$ to $$P_j$$
* Piggy back the matrix: to allow $$P_j$$ to know all the messages that hash been sent to $$P_i$$. This will
enforce happen-before's transitive property.
* If $$W[k,i] > M[k,i]$$ there is a message sent from $$P_k$$ to $$P_i$$ that happens before the current message
from $$P_j$$ to $$P_i$$ that has not been received.

### Synchronous Ordering

**Synchronous Ordering**:
* Messages are sent instantaneously, the time of send and receive are exactly the same
* Formally: $$s \rightsquigarrow r \Rightarrow T(s) = T(r)$$
  * If $$s$$ is the send event and $$r$$ is the receive event of the same message, then their time is the same
* Lemmas
  * $$e \succ f \Rightarrow T(e) < T(f)$$
* Corollary:
  * $$(e \rightarrow f) \wedge \neg (e \rightsquigarrow f) \Rightarrow T(e) < T(f)$$
  * If $$e$$ happen before $$f$$ and $$e$$ is not the send event of $$f$$ then $$e$$ time is less than $$f$$


Message Ordering hierarchy: Synchronous $$\subseteq$$ casusally ordered $$\subseteq$$ FIFO $$\subseteq$$ asynchronous
* Casually ordered $$\subseteq$$ FIFO $$\equiv$$ casusally $$\Rightarrow$$ FIFO
  * In the same process, if $$s_1$$ *happen before* $$s_2$$, then $$s_1$$ *occur before* $$s_2$$ and the receive event
  of $$s_2$$ cannot occur before $$s_1$$

  $$s_1 \succ s_2 \Rightarrow s_1 \rightarrow s_2 \Rightarrow \neg (r_2 \succ r_1)$$
* Synchronous order $$\subseteq$$ casusally ordered
  * Assume synchronously ordered. Let $$s_1 \rightsquigarrow r_1$$ and $$s_2 \rightsquigarrow r_2$$ and $$s_1 \rightarrow $$s_2$$
  * $$T(s_1) = T(r_1)$$ and $$T(s_2) = T(r_2)$$ and $$T(s_1) < T(s_2)$$ (corollary)
  * Then $$T(r_1) < T(r_2)$$ and $$\neg(r_2 \rightarrow r_1)$$

**Synchronous Ordering Algorithm**:
* Utilised non-synchronous control message to make all other send and receive message synchronous
* For each pair of process, assign one to be a big process and one to be small process
* Each process will have two states. Active: can send or receive message, passive: cannot send or receive message
* Big process to small process:
  1. Big process can only send to small process when **big process is active**
  2. Big process will send a message a message and change state to **passive**
  3. Small process (receiver) will send an **ack** message then the big process will become **active** again
* Small process to big process:
  1. Small process will request permission from big process. Block until receive permission.
  2. Big process will only **give permission** when it is **active**
  3. Big process turn **passive** after giving permission and until it receive the message from the small process
  4. Note: small process dont need to change the state.


### Total order for multicast Message

**Total Order in multicast**: For all message $$x$$ and $$y$$ and process $$P$$ and $$Q$$, if $$x$$ is received
before $$y$$ in $$P$$ then $$y$$ is not received before $$x$$ in $$Q$$. (plain english: the receive partial ordering
of messages must between the same across all messages)
  * total order does not imply FIFO, only enforce the ordering received across all process. Allow for non-FIFO as long
  as all process also receive non-FIFO

**Causal total order**: if satisfy causal order and total order

**Centralized Algorithm**:
* A coordinator has FIFO channels with all processes.
* If a process wants to multicast:
  1. it will send a request to the coordinator.
  2. Coordinator will add the request to the request queue.
  3. Coordinator will process the request from the queue synchronously send the message through the
  FIFO channel to the receivers

**Skeen's Algorithm**:
* All process have access to lamport's logical clock
* Steps to send a message:
  * A process (initiator) sends a timestamped message (single value) to all destination process
  * When a process receive a message, it will mark it as *undeliverable* and send it to the sender
  its logical clock as the proposed timestamp.
  * Once the initiator receive all the proposed timestamp, it will take the maximum of the timestamp
  and assign it as the final timestamp to that message and sent to all destination
  * When a process receive the final timestamp, it willl update the local message timestamp and mark as deliverable
  * When the message is marked as delivered, it is added to a queue and the application layer will process the message
  that has the smallest timestamp.

## Chapter 13: Leader Election

### Ring-Based Algorithms

**Anonymous Ring**:
* Properties: process in the ring do not unique identifier and initialised with identical initial state.
* No deterministic leader election algorithm:
  1. There is complete symmetry among all process
  2. Any deterministic algorithm that starts symmetrical will end symmetrical
  3. The end symmetrical does not have a leader

**Chang-Roberts Algorithm**:

* Requirement: each process have a unique identifier
* Algorithm:
  * Every process sends election message to the left process with its identifier
  * Also forwards any election message to the left process, if the message's leader identifier is larger
  than the its own
  * If a process receives its own message then it is the leader.
* Note: send to clockwise and receive from anti-clockwise
* When more than one process starts election:
  * When a process wakes up by leader election with a smaller leader then it will start its own leader election
  else it will relay the current leader election
  
**Chang-Roberts** analysis:
* Worse case:
  * When: process are arrange in clockwise decreasing order
  * The message sent by process $$j$$ will travel for $$j$$ processes before being swalloed
  * total number of election message: $$\sum_{j=1}^{j=N} j = O(N^2)$$
* Best case:
  * When the process are arrange in clockwise increasing order
  * Each election message will be consumed by the left neighbour immediately. Travel only one process
  * total number number of election message $$N$$

### General Graph

**Complete Graph**:
* Every node is connected to each node
* Each node will just need to send all other node their ID. The highest ID is the leader

**Construct Spanning Tree**:
* Any process will initiate spanning tree construct. The initiator will send its id with the initiate message.
* Steps:
  1. Initiator will send *invite* message to all of its neighbour
  2. When a process receives an invite message
      1. If the process does not have previous a parent, it will treat the sender of the invite as the parent. Sends
      an accept message to the parent
      2. else the process will send a reject message to the sender
  3. Forward the invite message to all of its neighbour except the sender of the original invite.
  4. Wait for all neighbour to send accept (child) or reject (not child)
* Multiple instance of spanning tree construct:
  * If a process receive an instance of spanning tree construct from a process with higher
  id than the current instance id, it will purge all current state and restart construct for the new
  instance

**Computing global function**:
* Use the same idea as spanning tree, instead of just constructing the spanning tree, will ask the children
node to execute computation and send the answer to the parent

## Chapter 15: Agreement

*Failure model* definition: how the environment can fail

* ie: process crash, adversarial process or link (communication) failure

*Timing model* definition: how long will messages take (bounded or unbounded)

### Fischer, Lynch and Paterson (FLP)

*Agreement*: a set of process need to agree on a single bit (set or unset)

*Failure model*: Only **process crash** but **no link failure**

*Timing model*: finite but **unbounded** time to send messages

*Problem*:
* `N` processes and `N` is known to all processes.
* Each process starts with initial value of `{0,1}`
* Some process will need to make a decision of $$0,1,\perp$$ (not decided)
* Once a process has decided, it cannot change the decided value

*Assumptions of environment*:
* *Initial Independence*: process choose their initial value independently (all
permutation of initial value allowed)
* *Commute property of disjoint events* (cummutative):
  * The order in which events on different process are executed does not matter. $$e(f(G)) == f(e(G))$$
* *Asynchrony of events*: any receive events might be arbitrarily delayed


*Faulty process model*:
* There are infinite runs of the algorithm and a fault process only has a finite number of steps
in the run (process crashed)
* *admissible* run: at most one process is faulty
* Reliable message system => all messages sent to nonfaulty

*Protocol Requirements*:
* **Agreement**: two nonfaulty process cannot commit on different values (must
be in agreement)
* **Nontriviality**: both values `0` and `1` should be possible outcomes. Protocol cannot hard code
the decision
* **Termination**: a nonfaulty process will decide in finite time.

*Indecision*:

* Let $$G.V$$ be a set of decision values the global state can reach form $$G$$ eventually.
* Bivalent if $$\mid G.V\mid = 2$$ - can reach `0,1`
    * Bivalent state signifies indecision - cannot decide between `0` or `1`
* Univalent if $$\mid G.V \mid = 1$$ - can reach only `0` or only `1`
    * $$G.V = \{ 0 \}$$ - G 0-valent
    * $$G.V = \{ 1 \}$$ - G 1-valent

*Step*:
* The system will move from one global state to another
* When a process `p` receives a message (can be null)
* `p` based on local state and message send other messages
* `p` based on local state and m, change p's local state

#### Impossibility with 1 faulty process

**Lemma 1**: every protocol must have a bivalent initial state (cannot decide on 0 or 1). Proof:
* Proof by contradiction: assume that there is a protocol with univalent initial state
* There exist an initial 0-valent (`S0`) state and 1-valent (`S1`) state that differs only in the initial state of process `p`.
* If `p` were to be fault (take not steps), `S0` and `S1` will be identical and should decide on the
same value.
* Contradicts the protocol as it allows for one process to be faulty and `S0` should not equal
to `S1`

**Lemma 2**: we can keep a system in an indecisive state
* $$G$$ starts with a bivalent state, let event $$e$$ on $$p$$ be allowed
* Let $$\mathcal{G}$$ be states reachable from $$G$$ **without applying $$e$$**
* Let $$\mathcal{H}$$ be states after applying $$e$$ to $$\mathcal{G}$$. $$\mathcal{H} = e (\mathcal{G})$$
* Proof of claim 1: $$\mathcal{H}$$ is bivalent
  * Let $$E_i$$ be an $$i$$-valent global stae reachable from $$G$$.
  * Let $$F_i$$ represent the result of applying $$e$$
  * If $$E_i \in \mathcal{G}$$ means $$E_i$$ reachable without applying $$e$$
  and $$F_i = e(E_i) \in \mathcal{H}$$
  * else $$E_i$$ is reached by applying $$e$$ and some events to $$\mathcal{G}$$ (no $$e$$)
  so $$F_i \in \mathcal{H}$$ (reached after applying $$e$$ to $$\mathcal{G}$$)
  * $$\mathcal{H}$$ will contain both $$E_0$$ and $$E_1$$
* Proof of claim 2: there exists neighbours $$G_0,G_1$$ such that
  $$H_0 = e(G_0)$$ is 0-valent and $$H_1 = e(G_1)$$ is 1-valent
  * Layman: there exist neighbouring state that are 0-valent and 1-valent 
  respectively such that applying an event will not change the valency
  * Let $$t$$ be smallest sequence to $$G$$ such that valency of $$et(G)$$ different from $$e(G)$$
    * This exists as claim 1 states that $$\mathcal{H}$$ is bivalent => there
    exists a sequence of events such that $$et(G)$$ different from $$e(G)$$
  * As $$G_0$$ and $$G_1$$ are neighbours, there exists $$f$$ such that
  $$G_1 = f(G_0)$$
    * If the process of event of $$f$$ and $$e$$ are different, this means

      $$e(G_1) = ef(G_0)$$

      $$\Rightarrow e(G_1) = fe(G_0) \ cummutative$$

      $$\Rightarrow e(G_1) = f(H_0)$$

      $$\Rightarrow \ 1-valency = \ 0-valency \ (contradiction)$$
    * If the process of events of $$f$$ and $$e$$ are the same
      * If the process becomes fault, then the final state is still bivalent

Intuition of proof:
* Any process that goes from bivalent to univalent state will have a critical step ($f$)
* Critical step cannot be based on the order of process independent execution
* Critical step is done by single process.
* As other process cannot distinguish between if the critical step is slow or faulty => forever stuck


### Consensus in Synchronous Systems

*Synchronous*: there is an **upper bound** on the **message delay** and 
**duration of actions**

Types of faults:
* *Crash*: a process halt, does not perform anything and stay halted forever
(detectable in synchrnous)
* *Crash + Link*: processor can crash or a link may go down. If a link goes down,
it stays down forever.
  * Link go down could cause partition (some pair of nodes cannot communicate)
* Omission:
  * Send omission: only send a proper subset of messages it supposed to send
  * Receive omission: only receive a proper subset of messages it supposed to receive
* Byzantine failure: fails by exhibiting arbitrary behavior
  * Most extreme form of failure

**Synchronous system consensus**:
* Set of values is any set of values that can be **totally ordered**.
* Each process $$P_i$$ starts with any value $$v_i$$ from the set.
* The goal is for all process to set a value $$y$$. The value can only be set once and is called the decided value
* Requirements:
  * *Agreement*: two non-faulty process cannot decide on different value
  * *Validity*: if all process propose the same value,
    the decided value should be the same
  * *Termination*: non-faulty process decides in finite time.

#### Crash failure consensus

Algorithm:
* Definitions:
  * $$f$$ is the maximum number of process that can fail.
  * Each process maintains $$V$$ which is the set of values that it knows that have been proposed.
* Steps:
  1. Process $$P_i$$ proses a value and adds it to $$V$$
  2. For $$f +1$$ rounds:
    * Each process sends to all other process new values added to $$V$$
    * Each process will also receives the values sent by other process $$P_j$$
      * As the system is synchrnous, $$P_i$$ will only wait for $$P_j$$ until the
      upper bound is up otherwise $$P_j$$ is deemed as crashed.
* Intuition:
  * Make all process to agree on the set $$V$$ and decide on the lowest number
* Proof:
  * termination: each process will only execute $$f+1$$ rounds
  * validity: decided value is chose from $$V$$
  * agreement: all process same $$V$$, agree on the same lowest value
  * All non-faulty process will end up with the $$V$$
    * If $$x$$ in $$V_i$$ but not in $$V_j$$ at round $$ < f+1$$ then $$P_i$$ will successfully transmit
    $$x$$ to $$P_j$$ in round $$f+1$$
    * If $$x$$ in $$V_i$$ in round $$ = f+1$$, then $$x$$ has been transmitted through $$f+1$$
    process. As there are only $$f$$ faulty process, another process would have added $$x$$ before $$f+1$$
    and at $$f+1$$ that process would have sent it to $$V_j$$


#### Byzantine General Agreement

Problem: $$f$$ process are adversarial and can execute any behaviour

Algorithm:
* State:
  * $$ N > 4f$$
  * Each process has a preference
  * Each process has $$V$$ as well (view of all process values)
* Steps:
  1. For $$f+1$$ rounds:
    1. Each process exchanges its value with all other process
      * It determines an estimate for the value (`myvalue`) by getting a value that is the majority (>N/2)
    2. Processor receives the value from the **coordinator** 
      * If no value received => coordinator failed and assumes a default value for the king
      * Set the value as `kingvalue`
    3. Decide if the process should use the king value.
      * If $$V$$ has more than $$N/2 + f$$ copies of `myvalue` then my value is chosen for the process
      * otherwise use the `kingvalue`
* Proof:
  * Validity:
    * There are $$>= N-f$$ valid process => valid process receive $$N-f$$
    ($$> N/2 + f$$) valid $$v$$. If all process agree on $$v$$ initially,
    then the valid $$v$$ will always trump the king value.
  * Agreement:
    * There are $$f$$ adversarial process but $$f+1$$ rounds => 1 round will have a non-faulty process
    * The 1 honest king will have $$N/2 + 1$$ at least correct $$w$$ and will broadcast it to all other
    honest node. 
    * Each processor decides on the same value at the end of a round with an honest king

## Chapter 18: Self-Stabilisation

### Self-Stabilisation Spanning Tree Construction

*Problem*: building a spanning tree of node. The seen topology of the tree of nodes
could be corrupted at any point.

*Constraints*:
* Each node have a set of neighbouring nodes
* Each node's state/view of tree can be corrupted

*Algorithm State*:
* `parent`: the parent node
* `dist` the distance from the root node

**Algorithm**:
* Non root node:
  * reads the values (dist) of neighbouring nodes by sending a query message
  * For all neighbouring node, the node with the shortest distance is the parent of the node
