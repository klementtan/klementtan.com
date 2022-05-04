---
title: "Post: Performance Tuning"
categories:
  - Post
tags:
  - C++
excerpt: "An overview of how we can tune the performance of our system."
---

## Memory Access

**Preventing TLB miss**:

* Table Lookaside Buffer (TLB) is a hardware cache for page tables (address mapping).
  * Maps virtual page **address** to physical frame **address**.
  * The actual page is not stored on the TLB but on the physical memory
* Most TLB are limited in size (ie at most `K` entries in the table).
* Solution: to reduce the number of TLB miss, use a bigger page size => reduce the number
of page address for the whole memory => fit more page address to physical frame mapping in TLB
* Huge page:
  * Can be manually requested by user (*hugetlbfs*) or automatically created by kernel (*TransparentHugePage*)
  * TransparentHugePage: uses a synchrnous memory compaction algorithm to group smaller pages to big page. Incur cost
  of the algorithm.
  * Additional benefit of having more contiguous memory in physical memory.
    * Contigous virtual address might span across different frames but each frame
    might not be adjacent
    * Huge page => bigger frame => more contiguous physical memory


## Non-Uniform Memory Access (NUMA)

Sources:
* [INTRODUCTION 2016 NUMA DEEP DIVE SERIES](https://frankdenneman.nl/2016/07/06/introduction-2016-numa-deep-dive-series/)
* [NUMA Deep Dive Part 3: Cache Coherency](http://www.staroceans.org/cache_coherency.htm)
* [NUMA DEEP DIVE PART 1: FROM UMA TO NUMA](https://frankdenneman.nl/2016/07/07/numa-deep-dive-part-1-uma-numa/)

**What is NUMA**:
* Memory access time depends on the position of the processor and position of the memory.
  * Local memory (same numa node) can be access faster than remote memory (different numa node)

**Why NUMA**:

* Allow for segregation of memory (each NUMA node has their segregated memory)
  * UMA problem:
    * On the wire level, the memory one processor can access the memory at one time
    * Single bus to access the shared memory has **limited bandwidth**.
    * Scalability problem: More processor => longer bus length => higher latency.
  * NUMA: Allow for simultaneous memory access when a process on different nodes
  access.
  different local memory.
  * Prevent a single process from starving memory access of other process
* Easy to scale with more processors => scale horizontally by adding more nodes

### Cache Coherence NUMA

* NUMA cache architecture:
  * Each core has its own private L1 and L2 cache
    * Private cache that cannot be accessed and written by other core
  * Last Level Cache (LLC) shared amongst core
  

### Fine Tuning NUMA

**NUMA balancing**:
* Automatically move memory pages to the numa node that is used the most
* Kernel determine the NUMA node to move to by regularly removing entries in
page table to force page fault => allow kernel to maintain the page fault statistic
=> determine if the page should be moved to a new node
* Performance impact: the forced page fault will result in higher latency
* Source: [Automatic NUMA balancing](https://www.linux-kvm.org/images/7/75/01x07b-NumaAutobalancing.pdf)
