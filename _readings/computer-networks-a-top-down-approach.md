---
title: "Computer Networks - A Top Down Approach"
toc: true
---

## Chapter 3 Transport Layer

### 3.1 Introduction and Transport Layer Service

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

#### 3.1.1 Relationship Between Transport and Network Layers

Differences: Transport layer provide logical communication **between process** but network layer provide logical communication **between host**
* Transport Layer only takes data from the network layer and "dispatch" it to the process - does not care about how the segments
are transferred between hosts.
* Services can be provided on the transport layer:
    * Reliable data even though the underlying is not reliable

#### 3.1.2 Overview of the Transport Layer

Terminology:
* Transport-layer packet for TCP: is called **SEGMENT**
* Transport-layer packet for UDP: is called **DATAGRAM** (internet literature) or **SEGMENT** (book)
    * **DATAGRAM** is also being used on the network-layer packet

IP service model: **best-effort delivery service**
* Does not provide and guarantee of:
    * segment delivery
    * order of delivery
    * integrity of data 
* **unreliable service**
* every host have at least one IP address

UDP and TCP similarities:
* Responsibility: extend IP's delivery service between **two end systems** to between **two process running**
    * Called **transport layer multiplexing** and **transport layer demultiplexing**
* Provide integrity checking by including error detection fields

UDP:
* Provide two minimal services: process to process data delivery and error checking

TCP: Provides multiple servies
* **reliable data transer**:
    * ensures data is deliverred from sender process to receiver process correctly and in order
    * Converts unreliable IP service to reliable TCP service
    * Using flow control, sequence number, acknoledgements and timers to ensure data is transferred 
* **congestion control**:
    * service provided to the internet to ensure that each TCP 

### 3.2 Multiplexing and Demultiplexing

Demultiplexing: taking a single stream of transport layer segment and dispatching to the correct application
* Each process can have one or more **socket**
    * How the process send/receive data to/from the network layer
* When a host receive network layer packets, it will be sent to an **intermediate** socket
    * at any point can have more than 1 socket
* Transport layer has fields that uniquely identifies the socket
* Uses the **source** and **desitination** port to uniquely identify the process to dispatch to.

Multiplexing: getting the transport layer segments from all the processes and wrapping them in the transport layer headers to create segments.

**Connectionless Multiplexing and Demultiplexing**:
* When creating a socket we can choose the assign the port number or let the kernel decide
* Client-Server:
    * Server should have a fixed port number for its socket so that clients will know how to send a segment to the server process.
    * Client can use the port the kernel assign as there will be no process will initiate a new message with the client.
* UDP sockets can be identified by two tuple (destionation IP, destination port)
    * If a UDP segments with different source IP/port but same destination IP/port, the segments will be directed to the **same** destination process
* why source IP/port:
    * To allow two way communication, source IP/port acts as a "return address" for the receiver to send data to sender


**Connection-Oriented Multiplexing and Demultiplexing**:
* TCP socket are identified by four-tuple (source IP, source Port, destination IP, destination Port)
    * Host uses all four fields to demultiplex the segments
    * 2 TCP segments with same destination IP/Port but different source IP/Port will demultiplexed to the **same process** but **different socket**
* Steps:
    * TCP server will bind to a port and have a "welcome socket" and a hardcode port
    * TCP client will generate create a socket and generate connection-establishment segment
    * The connection-establishment segment will be directed to the welcome socket and the host will accept the connection
    * The host will create a new socket with the four-tuple (source IP, source Port, destination IP, destination Port)
    * Any subsequent segment from the same sender will be directed to that new socket - other senders with the same destination IP/Port will not be routed to that socket.
* Servers can have multiple socket which are uniquely identified by the four-tuple

**Web Servers and TCP**

* Servers can spawn a new thread/process for each new connection
* Persistent HTTP: use the same socket for the entire duration
* Non-persistent HTTP: create a new socket for every request/response

### 3.3 Connectionless Transport: UDP

UDP:
* Responsibility:
    * Meet the minimal responsibilty of the transport layer by multiplexing/demultiplexing packets from and to applications
    * Provide light error checking
* Flow:
    1. UDP takes messages from the application layer and add source and desitination port number and small fields
    2. UDP pass the application layer message with UDP header and pass it to the network layer
    3. Network layer will encapsualte the message with transport layer segment into IP datagram and make best effort to send to receiving host
    4. When the UDP segment reach the UDP destination, the destination port number is used to deliver the segment data to the correct application
* Non-responsibility:
    * Does not perform handshake
    * Connectionless

**Benefits of UDP**:
* Finer application level control over what data is sent:
    * Deterministically add UDP headers to form UDP segment and send to the network
    * vs TCP:
        * Perform congestion-control - might cause the transport layer segment to be throttled and sent later
        * TCP has retransmission of lost packets: could in deterministically see duplicate packet.
    * TCP additional features will add overhead which will not be good for real-time application
    * UDP application can add choose what TCP feature it wants ("pay for what you use")
* No connection establishment:
    * Does not introduce any delay when wanting to send out a packet
* No connection state:
    * UDP protocol is stateless and does not require storeing the states
    * TCP states: send/recevie buffer (byte stream), congestion control parameter, sequence and acknowledgment number parameters
    * Servers that uses UDP can easily support many receivers/senders
* Small packet overhead:
    * UDP packet header size is `8` but TCP is `20`

TCP congestion control:
* UDP lack of congestion control can result in highly congested networks and high loss rate
    * This can affect other TCP connection in the network

UDP hybrid: on the application we can implement the TCP features we want (pay for what you use)

#### 3.3.1 UDP Segment Structure

* Data field: application layer data - variable size
* Header:
    * 2 bytes for source port
    * 2 bytes for destination port
    * 2 bytes for check sum
    * 2 bytes for length

#### 3.3.2 UDP Checksum

 * Sender perform 1s complement of the sum of all 16 bit words in the segment (header + payload - checksum?)
    * any overflow will be wrapped around - the 17th bit produced will be added to the lsb
* The checksum is the complement of the final sum
* Receving side - perform the same 16 bit word adding and the final sum should be `111111111111111`
* Motivation:
    * bit error could be introduced when being buffered in the router (after L3)

### 3.4 Principles of Reliable Data Transfer

#### Optimisation: Pipe-lining

Sending multiple packets before receiving the ACK

**Go-Back-N**:
* can only send a maximum of N sequential packets without getting back the ACK
* sequence number will be a ring and wrap back to 0 once it reach the max

#### 3.4.2 Pipelined Reliable Data 


[TODO: mutlicast](https://netlab.ulusofona.pt/rc/book/4-network/4_08/index.htm)
