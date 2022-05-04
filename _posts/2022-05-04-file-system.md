---
title: "Post: File System"
categories:
  - Post
tags:
  - OS
excerpt: "An Overview on OS File System"
---


## Overview

**Introduction**: this blog post will cover the basics of an operating system' s file systems.

**Hardware**:
* In a spinning hard disk, data are encoded around a circular disk
* To access a certain data, the disk will need to spin such that the data
location on the hard disk coincide with the head.

### File Data

File data: the data that is associated to the file (excludes metadata)

*Structure*:
1. Array of bytes:
    * All the data in a file can be seen as an array of raw bytes
    * Each byte will have a unique offset
2. Records:
    * File data is stored as a collection of *records*.
        * Each file has many records and each records has many bytes
    * Variant 1: fixed length records:
        * Each record has a fixed length
        * Structured as an array of fixed length record
        * Easily access a record by getting the offset in the record array
    * Variant 2: variable length records:
        * Each record can have different length

*Access method*:
1. Sequential Access:
    * Access the bytes in the file data sequentially
    * Cannot skip a byte
2. Random Access:
    * Can access any byte in the file data
    * Using `Read(offset)` or `Seek(offset)`
3. Direct Access:
    * Randomly access any records directly

## File operations

* Open: prepare all the information needed for file. Must be used before any file operation
* Create: new file created
* Read: read data, usually *starting from current position*
* Write: write data usually *starting from current position*
* Repositioning aka seek: move the current position to a new location
* Truncate: removes data between position to end of file

### File Operation as System calls

Information needed for file operation:
* File pointer: current location **inside the file**
* Disk location: actual file location on disk
* Open count: how many process has this file opened

Caveats:
* What if two process open the same file?
* What if process want to open many files?

**Unix Implementation**:
* System-wide Open File Table (OS Level)
  * The OS maintains a global table of open files "context"
    * (klement: nobody uses context but it context seems like an apt description for it)
  * Table is shared among all processes
  * Each entry in the table represents an open file context
    * The permission, file location (offset), the pointer to the data (vnode)
    * If process A and process B open the same file, process A and process B
    will have its own entry.
* File descriptor table (Process Level)
  * Each process maintains a file descriptor table
  * Each entry of the file descriptor table points to an entry in the System Open
  File table.
  * File descriptor: an index in the process' file descriptor table. (ie stdout fd = 0, first entry in the file descriptor table)
  * Independent file descriptor table allow different processes to access files independently.
  * Forking:
    * When a process fork, the child process will inherit the file descriptor table of the parent
    * Each entry in the child and parent file descriptor table will point to the same
    entry in the system open file table.
    * This is the reason why forked children stdout to the same console as the parent.
* vnode:
  * The OS also maintains a table for all open files
  * Each entry is an open file
    * Open file are contextless (no concept of offset in file) vnode
    * (klement: For unique file, there can only be one vnode entry?)
  * Contains pointer to inode (physical file) and information for the file

Reference Counting:
* System wide open file table and vnode table use reference count to decide when to
evict the entry
* Process' file descriptor table has a reference to system wide open file table
* System wide open file table has reference to vnode table
* OS uses reference counting to determine when there are no higher level table's
entries that refers to it.
* Evict that entry once the reference count goes to `0`.


More details: [here](https://www.usna.edu/Users/cs/wcbrown/courses/IC221/classes/L09/Class.html)

## File System Implementation

**Components**:
* Disk structure: a disk can be treated as a 1-D array of **logical block**
* **logical block**: smallest accessible unit in the disk (usually 512b to 4kb)
* **disk sector**: each logical block belongs to a disk sector
* Master Boot Record (MBR): located at the start of *sector 0*
  * Stores the OS boot up information
* Partition: after the MBR
  * Stores information on how the files are located and accessed

**Problem**: similar to memory allocation, file system face the problem of external fragmentation.
  * Should the blocks be contiguous or non-contiguous?
  * (klement: Internal fragmentation is not really a problem as it is restricted by the logical block size)


Naive solutions:
* Linked List:
  * Each file will have a head block and each block will contain a pointer to the next block.
  * Random Access: O(N)
  * Can be optimised by having the block pointers loaded in memory (still O(N)).
* Direct indexing:
  * Each file will have a special index block
  * Index block will contain an array of pointers to all the blocks that belongs to the file.
  * Cons: the file size is limited by the maximum size of the array in a logical block.
  * Can be optimised using multi level table (similar to multi level paging)

### Unix Solution

**Combine Scheme**: uses both direct indexing and multi-level scheme
* Each *inode* has 15 blocks
* 0 - 11 blocks: **direct pointers**
  * direct pointers are actual data on disk
* 12 block: **single pointer**
  * contains an array of *direct pointers* (direct indexing)
* 13 block: **double indirect**
  * points to an array of *single pointers*
* 14 block: **triple indirect**
  * points to an array of *double indirect*

