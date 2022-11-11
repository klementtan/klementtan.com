---
title: "Computer Networks - A Top Down Approach"
toc: true
---

## Chapter 3 Transport Layer

### Introduction and Transport Layer Service

* Overview: provides a **logical communication** between application process running on different hosts
    * **Logical communication**: from the pov of the application, the host are connected by a single wire,
    they do not need to care about the switches and routers in-between
    * Provide a way for two application on different host to communicate
* Implemented in the end hosts but not network routers inbetween
* The unit of packet in transport layer is **SEGMENT**
* Flow:
    1. Converts messages received from the **application process** into transport layer **segments**
        * The transport layer might break the application message into smaller chunk and add transport layer header.
    2. Pass the segment to network layer which will wrap the segment in network layer headers as **datagram**.
    3. Receiving side will extract the transport layer segment from the **datagram**
