---
title: "Post: Designing an Exchange"
categories:
  - Post
tags:
  - OS
excerpt: "How do you design a stock exchange."
---

Source: [How to Build an Exchange by Jane Street](https://www.janestreet.com/tech-talks/building-an-exchange/)

## Overview

*Requirements*:
* Functional:
  * Order book:
    * Match buy orders with sell orders for an instrument.
  * Support multiple instruments
* Non-functional:
  * Scale: 10,000 symbols and 3 million messages per second and a few million live orders.
    * Handle by having low latency => higher throughput
  * Fairness among all users - for compliance reasons.
    * Negative example: if the exchange multiplex market data over TCP by iterating through
    all the connections and sending the data one by one, users can exploit this by trying
    to be the first connection in the list.
  * Reliability and durability:
    * Any action by the exchange could set off a chain of reactions.
  * Expect adversarial clients:
    * Cannot allow a single adversarial affect other honest clients.
    * Game theory involve:
      * If client A buy stock at $X and client B sells stock at $X-10, should the stock
      be sold at $X, $X-10 or $X-5? Could result in either party trying every price point
      until a match.
  * Determinism: allow easy explanation to regulators

## Stock Exchange Design

**Components**:
* Matching Engine: stores the order data structure in memory and perform the matching algorithms
* Client Ports: accept connection from individual clients and perform transaction
  (new order, cancel order) with ME.
  * Expose a TCP port for clients to connect
  * Performs sanity checking for incoming orders - helps with flow control.
  * Note: clients interact with exchange through TCP but inside the exchange they use UDP for IPC.
* Drop Port: for the entity who clears or guarantee the trades. Need information an arbitrary combination
of Client Ports.
  * Used for trade reporting
* Market Data:
  * Publish a subset of the transactions for clients to have a view of the market
* Cancel Fairy:
  * Captures client request to cancel an order some time in the future. When that time
  in the future comes, the cancel fairy sends the cancel message to the matching engine.
  * Allow decoupling of the matching engine logic.
* Auction Optimiser:
  * Complex matching algorithms such as auctions will be run outside the matching engine and
    the final result will be sent to the matching engine.

### Multicast

**What is multicast**:
* Switches have logical address (not actual address).
* When a packet reaches the multicast logical address, the packet is **electronically repeated**
to every interested party relatively simultaneously.

**Trade offs**:
* Pros: Allow all components (client port, drop port, market data, etc) to receive a trade execution event at the same time (fairness).
* Cons: Uses UDP
  * Not connection oriented: No seq numbers and ACKs
  * Messages can be dropped.

**Mitigation**: retransmitter
* Each multicast message will have its own sequence number.
* Retransmitter job: record the messages seen.
  * If there are out of sequence messages, retransmitters will ask each other for the missing packets
  * if all of the retransmitters do not have the missing packets, they will ask the sender to resend the
  missing packets.

### State Machine Replications

**General Idea**: all components/process will be designed as a deterministic state machine. The components starts with
an initial state and apply transactions (messages) to transit the state.
  * ie: cannot act on a timer - state change that is invoked by the transaction

**State Replication**:
* If a component goes down, a new component can be spawned on a new machine
* The newly spawned machine will request from the retransmitters all the transactions (messages) since
the start of the day.
* Once all the transactions have been published, the new component will be live and can join the system.
* All messages that are sent when the original component crashed will also be retransmitted to the new component
  * Helps with durability => process crashing can be recovered.

**Secondary ME**:
* To handle ME crashing, there is a secondary ME that will passively listen to the active ME.
* The secondary ME will be in the same state as the active ME.
* If the active ME were to crash, the passive ME will step in perform the necessary actions.
* Passive ME can run the exact same code as the active ME.

### Memory Optimisations

* ME preallocates all the representations for an order.
* When the ME executes an order, the ME will return the pointer to the memory address to the Client Port
  * Allow for quick access to orders in the future. Client port use the pointer as the reference.


### Parallelism

* Parallelize across each symbol - one entire system for one symbol.
  * Assume that each symbol execution are independent from each other.
  * Will not work if you would like to have a fail-safe mechanism across different symbols.
  * Problem with the sequence number that is given to clients.
    * The different ME cannot have same sequence number - this injects an implicit state into ME.
* Having multiple concurrent memory fetches.
  * Preprocess the messages in batch by prefetching the order (pointer) into cache.
  * (klement: Only parallelize the prefetching but the orders still are processed sequentially?)

Locking: handling message that are trying to change the same state
* Each message belongs to a topic - topic represents a component of the state that should
be updated exclusively
* Lock is implemented by having a sequence number for each topic
  * topic sequence number different from global sequence number.
* Each message in the topic are processed sequentially?
* The sequence number for each topic is generated by the publisher. If the sequence number is
used then it will apply the new transaction and propose a new sequence number. Example:
  1. Client A creates buy order with sequence number 3 on topic A
  2. Client B creates a matching sell order with sequence number 8 on topic B
  3. ME creates an execution and send an execution message on topic A with sequence number 4 and topic B with sequence 9
  4. Client A wants to cancel the order with sequence number 4 message.
  5. ME receives the cancel message from client A and drops it.
    * ME does not need to do anything as client A is guaranteed to receive the execution message with sequence number 4
    and will realise that the cancel message did not go through.
  6. Client A will receive execution message with sequence number 4 and realise that order has been executed.
  7. Client A apply the execution message state, update the state and send new meessage (cancel reject).

