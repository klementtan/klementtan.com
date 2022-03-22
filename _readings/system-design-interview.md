---
title: "Readings: System Design Interview"
toc: true
---

Title: System Design Interview

Author: Alex Xue

## General Review

## Chapter 2: Back-Of-The-Envelope Estimation

This chapter provides some useful data and metrics that could be used to make estimation for various
aspect of a system

## Chapter 5: Designing Consistent Hashing

**Rehashing problem**: most hashing function maps a key to an index using `serverIndex = hash(key) % N`. However,
when value of `N` change due to the more servers (scale up) or some servers failing, the value of `severIndex` will
change.

Solution: Ring hashing
1. A hash function is used that provides a fixed range (ie SHA1: `[0, 2^160 -1]`).
2. Form a ring from the start of the hash range to the end of the hash range
3. Each server will be assigned a position in the ring.
4. For each key, find the position on the hash ring and the assigned server will the first server in the counter clockwise direction

**Benefits**:
* Adding or removing a server will not change the assigned server for unaffected keys.

**Drawbacks**:
* Keys might not be evenly distributed which will cause some server to be overloaded
* Removing adjacent servers could result in a large partition for a particular key.

**Mitagtion**: virtual nodes
* Instead of having each node mapped to a single position in the ring, each nodes will
be mapped to multiple nodes in the ring. This will result in smaller partition which would
reduce the additional load on a single node when a node is removed.

## Chapter 6: Design a Key-Value Store

**CAP theorem**: distributed key value must choose two of consistency,
availability and partition tolerance.
* Consistency: all clients see the same data at the same time
    * Strong consistency: any read operation returns a value corresponding to the result of
    the most updated write data item
    * Weak consistency: subsequent read operations may not see the most updated value.
    * Eventual consistency: given enough time, all updates are propagated and all replicas
    are consistent
* Availability: any clients which request data gets a response even if some of the nodes
are down
* Partition Tolerance: a partition indicates a **communication break down** between two nodes.
The node is still operational but only cannot communicate with other nodes.
* Real world: consistency and availability does not exists as partition is a very real problem.

**Master Salve (Naive)**:
* One writer node that have multiple replicated reader node
* Manages to achieve consistency and availability
  * If one reader node goes down, other reader node can take over. If master node goes down, the reader node will be
  promoted to the master.
  * As all data are propagated from the writer to reader, there will not be any inconsistent data.
    * (klement: How do you enforce linearizability? Reading data that has been written but not yet propagated)
* Disadvantages (not partition tolerance):
  * When one of the reader node is partitioned, the it serves might become stale (write operations not propagated)
* Consistency + Partition tolerance modification:
  * If a nodes becomes partitioned, the non-partitioned node must block all write operations
  * Prevents the data from the partitioned reader becoming stale (inconsistent)
  * Disadvantages: unavailable as all write operations are stopped when a node is down
* Availability + Partition tolerance:
  * Continue to allow read and write operations to be excuted -> inconsistent data between non-partitioned and partitioned node
  * Eventually sync after the partitioned node is back

### System Components

**Data Partition**: use consistent ring hash to partition data across multiple nodes
with replication

**Data Replication**:
* Essential for high availability: cannot have a single point of failure that could
bring down the entire application
* Solution: instead of storing the key in the first node in the clockwise direction of
ring hash, store the key in the first **N** nodes in the clockwise direction of the ring hash.
* Each key will have different set of replica nodes.
* Due to virtual nodes, the first **N** nodes should be unique physical nodes.

**Consistency**:
* Synchronizing data across **replicas** (not writer node)
  * (klement: Are all write operations to the nodes the same? Shouldn't they all
  be from the same master writer node?)
* Quorum consensus
  * Definitions:
    * **N** = The number of replicas
    * **W** = write quorum of size **W**. For a write operation to be considered as successful, write operation must
    be acknowledge from `W` replicas
    * **R** = A read quorum of size **R**. For a read operation to be considered as successful, read operation must
    wait for responses from at least **R** replicas
  * Each set of replicas will have a single coordinator that acts as a proxy for all read and write
  operation to the replicas.
  * **N**, **W**, **R**:
    * **R=1** **W=N**, the system is optimised for fast **read**
    * **W=1** **R=N**, the system is optimised for fast **write**
    * **W + R > N**, strong consistency. There must at least be a node that saw both the read and write operation => strong consistency
      * Replicas not accept new reads/write until all of the nodes agreee on the current write
      * Could block new operations => not highly available
    * **W + R <= N**, strong consistency not guaranteed
* Eventual consistency is recommend. Pass the burden of reconciling the eventual value to the client.


**Inconsistency resolution**: versioning
* Inconsistency happens when we choose non-strong consistency (eventual consistency) for data across
**replicas**
* From the client POV, the data are inconsistent
* [Version clock](https://klementtan.com/readings/concurrent-and-distributed-in-java/#vector-clock):
  * Clock is represented by: `D([S1, v1], [S2,v2], ..., [Sn, vn])`. The vector clock
  stores the version of the data from the POV of each of the node
  * Algorithm:
    1. Node `i` increment `[Si, vi]` if exists
    2. Otherwise create a new entry `[Si, 1]`
* Application:
  1. Every time the client reads data, they get the data with the version clock
  2. Every time the client writes data, they will provide the previous clock
    * The server will increment its clock value
  3. If the client gets conflicting data, will compare the vector clock of the two data.
    * A vector clock is a parent (irrelevant) if it is pairwise less than or equal to the other data.
    * If not comparable -> there is a conflict in data and the client can choose either of the sibling

**Handling failures**
* *Failure detection*:
  * Naive: all-to-all multicast
  * Gossip protocol:
    1. Each node periodically increments its heartbeat counter
    2. Each node periodically sends heartbeat to a set of random nodes
    3. Once nodes receive heart beat, membership (table of number of heartbeat)
        list is updated to the latest info
    4. If heart beat is not increased for more than predefined period
        * Sends that offline node heart beat to other nodes to confirm
* *Handling temporary failure*:
  * Sloppy Quorum: instead of choosing W and R random nodes, choose the first W and first
  R nodes. This allows for down nodes to be replaced by the next node in the ring
* *Permanent failure*:
  * Handle replicas being out of synced
  * Create a merkle tree for all the data in node and compare it with identical replicas.
    * (klement: If a sliding window of replicas is chosen, wont each node have unique set of values and thus different merkle hash?)

### KV Architecture

(no master/salve model?)

*Components*:
* Coordinator that is a proxy and carries out the quorum algorithm
* Nodes are assigned to hash ring and data is replicated
* Every node has the same set of responsibility

*Write path*:
1. Write request is persisted on a commit log file
2. Data is saved in the memory cache
3. If cache is full flush data to disk

*Read path*:
1. If the key in memory return the data
2. If data is not in memory, the system checks the bloom filter
3. The bloom filter is used to figure out which SSTable might contain the key
4. return the result to the client

**Coordinator**:
1. All write path and read path will start from the coordinator to all replicas for the key
2. Carry out quorum/sloppy-quorum to determine when to return success/result to the client
3. Utilizes gossip to check if the nodes are down
