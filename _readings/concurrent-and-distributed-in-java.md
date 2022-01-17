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
