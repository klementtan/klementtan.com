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
  * The number will be later the request, the higher the number
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
      number[i]++;
      choosing[i] = false;
    }

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
    3. $$P_1$$: `number[1]++ `
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
    
    


