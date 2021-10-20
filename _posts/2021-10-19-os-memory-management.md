---
title: "Post: OS memory management"
categories:
  - Post
tags:
  - OS
---

In this post I will briefly describe how does an operating system handle memory management.


## Overview

On a high level, an operating system has a main memory that will store data. Depending on the
hardware, the main memory will have multiple 32/64 bit addresses that will allow the OS/process
to write value into these addresses. These addresses are known as physical address.

### Problem

When assigning an address space to a process, we would ideally like to keep these address
in a contiguous array to allow for cache by locality. However, the problem of **Fragmentation**
would arise if we naively assign contiguous address for each process.

#### Fragmentation

**External Fragmentation**: Occurs when we allocate a small address space to a small process that
gets killed eventually. Other processes require a larger address space and cannot be assigned to
that small address space. This result in small address being under utilized.

**Internal Fragmentation**: Occurs when the OS allocate a process an address space that is slightly bigger
than what the process require. The extra address space is too small to be assigned to any process and the
OS will just assign that extra address space to the process.


## Solution

The general idea to solving this problem is to allow the process handle memory using a logical/virtual address.
The logical address does not coincide with an address in main memory but instead the OS will use
some mechanism to map the virtual address to a physical address in memory.
* **Solve external fragmentation**: As each address space is split into multiple
identical frames, there will no longer be a chunk of address space that is too
small to be used by other process.

### Paging

Paging is the idea of splitting the main address into multiple **frames** of a certain size (usually a multiple of a word).
On the process side, it will see each frame as a **page**. Thus for a purely paging mechanism
the virtual address is made up of `page_id|offset` the `page_id` is for getting the correct
**frame** in physical memory and the `offset` (d) is to get the address of the correct word in the frame.

**Translate page to frame**

To map the `page_id` to `frame`, the OS has a **paging table** that maps the page number to the frame address in
the physical memory. To further speed this up, the OS has a **translation look-aside buffer** (hardware optimization)
that will cache these results to allow for fast querying of the frame id.

**Properties**:
  * When context switch nee to flush TLB

### Swapping

For idle processes, the OS will swap the frames that belongs to the process into the disk
until the process is back to running state. This will free up the number of frames.

### Segmentation

**Problem**: To a process it would like to be able to change the address space of the different components
of memory (heap/stack/data).

**Solution**: To solve this each process will have `k` number of disjoint segment for heap/stack/...
. Address with have a `seg_id` that maps to the base address of the segment in the main memory.
The mapping will be stored in a **segment table**.
  * A problem to this is that it could cause external fragmentation.

### Paging + Segmentation

The best solution is to use both paging and segmentation techniques. Where each **segment** points to
a **page table** that points to a **frame** in physical memory and would be accessed using the offset.

A virtual address will look like `seg_id|page_id|offset`. This will allow each memory segment
to grow/shrink dynamically without having to suffer from external fragmentation as the 
memory for each segment is disjoint into frames

### Virtual Memory

**Problem**: An issue arise when the number of process that pages wants exceeds the number of frames.

**Solution**: 
* To solve this OS will utilize the secondary storage (disk) to store frames that are
LFU/LRU.
* Each entry on the page table will have a **is memory resident** bit. If `1` the corresponding
frame is in main memory (RAM) else it is in secondary memory (disk).

*Steps*
1. Check **is memory resident** bit.
  * If true just fetch the frame from memory
2. Else Trap to OS (**Page fault**)
3. OS will locate the frame in disk
4. Load the frame in disk into main memory
5. Update the page table (set bit to 1)
6. Continue step 1
