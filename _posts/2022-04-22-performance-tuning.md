---
title: "Post: Performance Tuning"
categories:
  - Post
tags:
  - C++
excerpt: "An overview of NUMA and the various aspects for it"
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


## Non-Uniform Memory Access (NUMA)

**Why NUMA**:
*

**NUMA Trade offs**

### Fine Tuning NUMA

**NUMA balancing**:
* Automatically move memory pages to the numa node that is used the most
* Kernel determine the NUMA node to move to by regularly removing entries in
page table to force page fault => allow kernel to maintain the page fault statistic
=> determine if the page should be moved to a new node
* Performance impact: the forced page fault will result in higher latency
* Source: [Automatic NUMA balancing](https://www.linux-kvm.org/images/7/75/01x07b-NumaAutobalancing.pdf)
