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
* Header: 8 bytes
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

### 3.5 Connection-Oriented Transport: TCP

**Connection-oriented**: two process must "handshake" to initial many state variables

**Full-duplexed**: data can be transferred between two process at the same time

**Point to Pointer**: data is only transferred between the send and receiver (no multicast)

Sending a stream of data:
1. Application layer will send data to the socket.
2. Socket will read the data and write to the send buffer
    * Note: there is not fixed interval on when to send the data in send buffer to ip layer
    but it is limited by the MSS

MSS: Maximum Segment Size
* The maximum application layer data (excluding TCP headers) that can be sent in a single TCP segment
* Limited by the link layer **maximum transmission unit** (MTU)
    * Routers will drop packets if they exceed the MTU (includes Link Layer headers)
    * Common MTU: 1460 bytes
* A application layer data and TCP header pair form a TCP Segment

#### 3.5.2 TCP Segment Structure

TCP Header: 20 bytes
* Similar to UDP:
    * 2 bytes source port
    * 2 bytes destination port
* 2 bytes receive window (flow control)
* 2 bytes Sequence number
* 2 bytes Ack number
* 4 bit header length: TCP header is variable length
* 6 bit flag field:
    * ACK bit: the ACK number is valid and this message is an ack - first packet in a handshake
    will not have this bit set
    * RST, SYN, FIN: connection setup and teardown
    * PSH: receiver should pass the data up immediately
    * URG: urgent bit, the urgent pointer is valid and points to the last byte of the urgent data
* 2 bytes receive window
* 2 bytes check sum
* 2 bytes urgent data pointer
* options fields:
    * sender and receiver to negotiate the MSS
    * timestamping

TCP data:
* Needs to be within the MSS (size of data field excluding header)
* If the application layer provides data larger than MSS, the TCP layer will usually fragmentize it into chunks of MSS (not rely on IP layer to do the fragmenting)

**Sequence Number**:
* TCP views data as a stream of bytes instead of multiple TCP segments
* Sequence number is the position of the first byte of the TCP segment in the entire 
stream of bytes
* A random initial sequence number is used:
    * to prevent a TCP segment from a previously terminated TCP connection with the same ports being overlap
    * This might cause the first segment in the receiver, receiving the wrong segment

**Acknowledgment Number**:
* Full duplex: data can be sent between two host simulateneously 
* The next byte that the receiver is expecting the sender's byte stream
* cumulative ack: sending the next byte that the receiver is expecting is impliciting that all bytes before
that

#### 3.5.3 Round-Trip Time Estimation and Timeout

* TCP time manager takes a continous `SampleRTT`: the RTT for a segment to be acked
* maintains an RTT average `EstimatedRTT` = 0.875 * `EstimatedRTT` + 0.125 * `SampleRTT`
* mains a deviation `DevRTT` = (1 - B) * `DevRTT` + B * | SampleRTT - EstimatedRTT |

Timeouts
* A timeout will always be running for any segment.
    * If the timer is currently inactive set it to the current segment
* Doubling timeout: if a timeout is reach for the ACK, the sender will double timeout the timeout
for the next retransmission to prevent congesting the network
* Duplicate ACK: solve the problem of fast retransmission (don't need to wait for timeout)
    * Receiver usually send the same ACK if it receive out of order sequence number.
    * When the sender receives three duplicate ACK it will then retransmit the segment with the ACK seq number
    * Why three: sender usuallys sends mulitple segments K -> K + N if K is lost but [K+1, K+N] is received,
    the reciver would have sent N duplicate ack for K
    * Will only retransmit **ONE** of segment with sequence number = ACK seq number
    * Assume that the receiver will buffer the packet

#### 3.5.6 TCP Connection Management

Establishing Connection - Three way Handshake:

* Step 1: Client sends **SYN segment** to server
    * Client sends a special TCP segment
    * No application layer data (payload length 0)
    * `SYN` bit set to 1
    * Randomly choose the initial sequence number (`client_isn`)
* Step 2: Server sends **SYNACK segment**
    * Server allocates TCP buffer and variables
    * Server sends connection granted segment with no application data
    * `SYN` bit set to 1
    * `ACK`: set to `client_isn+1` (tell client to send the next packet after the SYN segment)
    * Randomly choose `Seq=server_isn`
* Step 3: client ACK server's `SYN`
    * When client receives `SYNACK` it will allocate receive buffer and sets the variables
    * ACK's server's `SYN` by sending `ACK=server_isn+1`
    * `SYN` bit set to 0 - no longer needed
    * Client can send application layer data

Other Possible outcomes after SYN segment by host:
* TCP RST: the packet the host but the host does not have any application listening to that port
* Receives nothing: pack might be blocked by a fire wall.

Closing TCP Connection:
1. Client sends a special TCP segment with FIN bit set to 1
2. Server acknowledges the shutdown by ACKing that sequence number
3. Server sends any remaining buffered data
4. Server closes the connection by sending a special TCP segment with FIN bit set to 1
5. Client acknowledges the server's FIN segment.

## Chapter 4: network layer

Definitions:
* **Forwarding**: transfer of a packet from incoming link to outgoing link within a **single router**
* **Routing**: involves all of a network routers
* **Network layer packets**: Datagram
* **packet switch**: general packet switching device transfer packet from input link interface to output link interface (forwarding) based on information in the packet header
    * **link-layer switch**: forwarding based on link layer fields
    * **routers**: base theri forwarding decision on the network layer fields

Internet service model: Best effort
* Bandwidth Guarantee: None
* No-loss guarantee: none
* Ordering: any order
* Timing: not maintained
* Congestion indication: None

Components:
* IP Protocol
* Routing component
* Reporting errors in datagram

### 4.2 Virtual Circuit and Datagram Networks

* Virtual Circuit: connection based network
* Datagram Network: connectionless based network (ie internet)

(skipping virtual circuit as it is irrelevant)

#### Datagram Network overview

* Packets will have the destination address added into the header
* Packet pass through multiple routers before finally reaching the destination
* Each router have a forwarding table of destination address to link interfaces
    * Router uses the destination address to lookup in the table and find the entry with
    the longest matching prefix
    * Forwarding table is built by routing algorithm

### 4.3 What is inside a Router 

Input ports:
* Line Termination: taking a IP datagram form an interface and finding (not forwarding) which output interface to forward to.
    * Terminates the input physical link and deserializes the data
* Data link processing (protocol decapsulation): ?
* Lookup: a shadow of the lookup table is stored in each input port (parallelise the work)
    * Within the lookup uses a Trie to find the corresponding output interface
    * Sets the output port
* Sends the packet with the output to the switch frabric - might be queued.

Switching Fabric: switch the datagram from one input to another output
* Switching via memory: switching is done by the CPU
* Switching via bus:
    * A single bus that connects all input ports to output ports
    * Only one datagram can be send through the bus at a time -> queueing
* Switching via interconnection:
    * A connection of 2n buses - n x n grid
    * Only when 2 input ports wanting to switch to the same bus will there be queueing
        * When the verticle bus is full
    * Variable length IP datagrams are broken down into multiple fixed size cells and ressablemed at the output port
* Output port:
    * Data link processing - protocol encapsulation
    * line termination - start of the physical line out of the router
    * queueing: when the switch fabric deliver faster than the link rate

### 4.4 Internet Protocol (IP): Forwarding and Addressing in the internet

#### 4.4.1 Datagram Format

* Version Number: 4 btis to specify the version number of the protocol
* Header length: IPv4 datagram has variable option -> variable length
    * Most IPv4 datagram has no options and length is 20 bytes
* Type of service: allow differnet type of IP Datagram to be identified (real time vs non-realtime)
* Datagram Length: actual length of the IP Datagram
    * theoritically the maximum size is 65563
* Identifier, flags and fragmentation offset: fields to deal with ip fragmenation
* Time to Live: each Datagram will start with a TTL of N and at every hop the TTL number will
decrease by 1. if the TTL number reaches 0 drop the packet
* Protocol: field only used when reached the destination. Contains UDP/TCP identifier so that
the destination knows how to parse the payload of the datagram
* Header checksum:
    * Note: only checksum for the header
    * 2 bytes summing using 1s complement arithmetic
    * Routers drop the packet if fail the checksum
* Source and destination IP address:
    * added by the source host
* Options: rarely used - optional fields
* Data (payload): usually contains transport layer segment

#### IP Datagram Fragmentation

* Why: No all link layer can carry packets of the same size
    * Ethernet can carry a maximum of 1500 bytes of data per frame
    * MTU: Link Layer maximum amount of data
* Solution: fragment IP datagram into smaller IP datagram that fits in the MTU. Each smaller IP datagram will be encapsulated in a single link layer frame.
    * Each smaller IP datagram is a **fragment**
* Details:
    * Source: When a host a transport layer segment > MTU or router receive a IP datagram > MTU of outgoing link it will fragmentize the orignal IP datagram
    * Fragmenting:
        * Each original IP Datagram will be given an indentifier which will be copiped into the smaller fragmented IP Datagram
        * IP fragmentation offset:
            * The number of 8 bytes the fragmented IPdatagram should start at
        * flag: set to 0 for the last fragment
    * Resassembly: done that the end host - routers will just forward the fragmented IP datagram
        * what is need to perform multiple fragamentation (ie 1500 -> 480 MTU)

#### IPv4 Addressing

Host Setup:
* Each host needs to be connected to the network through a single link
* The part that connects the host to the link is called an **interface** (ie ethernet port)
* Each host needs at least one interface to be connected to the network
* Each router has **more than one** interface - connects to more than one link

Interface:
* Each interface has an IP address - 1-1 relationship
    * IP address is globally unique if not behind a NAT
* IP address is determined by the **subnet it is connected to**.

Subnet:
* Is an IP network, every host is able to send an IP datagram to every host in the network without going through a host
* The IP address of the interfaces in the host have the same IP **subnet mask**
* Routers that have multiple interfaces are able to be part of multiple subnets
* Detach each interface from the host -> each isolated network is a subnet

**Class Interdomain Routing** (CIDR):
* where each organisation is asigned a.b.c.d/x
* Alternative: **classful addressing** 
    * Each x is a multiple of 8,16,24
* within the organsation, the remaining least significant bits can be used as the IP subnet mask

Broadcast IP: 255.255.255.255
* datagram is sent to every interface in the subnet
* routers can optionally forward to neighbouring subnets

**Dynamic Host Configuration Protocol** (DHCP):
* The admin will assign the ip address into the router/subnet
* DHCP provides new interfaces a way to be dynamically assigned IP address
* DHCP also proves a way to tell the interface what is the first hop router (default gateway)
* Steps:
    1. new host will send a broadcast message to the subnet with a `DHCP server discovery` message
        * Destination IP `255.255.255.255` and source IP `0.0.0.0`
    2. Any DHCP server in the subnet will send a `DHCP server offer` message with the ip address allocated to it
    3. new host will send a `DHCP request` with the ip address it choose (there could be multiple DHCP server and the host will choose one)
    4. DHCP server will respond with `DHCP ACK`

**NAT**:
* Allow solve IP address shortage issue by having an entire subnet look like a single device (single IP instead of a range of IP for the subnet)
* NAT router mapping of WAN PORT <=> LAN IP, PORT
* Draw back: devices outside of the LAN cannot make connection to devices within the WAN

#### Internet Control Message Protocol (ICMP)

* messaging protocol that uses IP layer to communicate between host (parallel to TCP/UDP)
* ie sending ping message between hosts or trace route

#### IPv6 (skipping)    

#### Routing Algorithms

* **Default router**: first hop router that all host is attached directly to

(skipping routing algorigthms)

#### 4.6 Routing Internet

**Autonomous Systems** (AS): group of routers that are controlled by the same organisation

Goal of routing: find a route between routers - by transivity find a route between subnets

##### 4.6.3 BGP - Inter-AS Routing

Goals:
* Obtain subnet reachability information from neighboring ASs
* Propagte the reachability information to all routers internal to the AS.
* Determine good routes to subnets

Basics:
* semi permanent TCP connection is established between routers of different AS
    * TCP connection is called BGP session
    * Internal BGP (iBGP) => BGP session within ASs
    * External BGP (eBGP) => BGP session across ASs
* BGP allow AS to learn which destination (CIRized prefix) is reachable by its neighbouring AS
    * eBGP sends the neighbouring AS the list of prefixes that it is reachable
* Each AS have a globally unique ASN
* BGP session advertise **BGP attribute** + **CDRized prefix** => **BGP route**
    * AS-PATH, the path of ASs to reach the CDRIzed prefix
    * NEXT-HOP, the router interface that begins the AS-PATH
        * AS-PATH only contains AS but not the router interface

(skipping the rest?)

#### 4.7 Broadcast and Multicast Routing

* multicast routing: single source node send a copy of a packet to a subset of other network nodes

##### 4.7.2 Multicast

**Multicast Service**: packet is delivered to only a subset of network nodes

Problem:
1. how to address a packet sent (what destination ip to use?) 
    * does not scale to add all receivers ip address in the packet
2. how to identify the receivers?

**Multicast Address**:
* address indirection: a single identifier is used for the group of receivers
    * any packet sent to that identifier is sent to all in the group
* Single identifier that represents all the group is called the class D multicast 

**Internet Group Management Protocol** (IGMP):
* IGMP Protocol operates between a host <=> a router
    * multicast routing is between host and algorithm router <=> router
* Host informs router that it wants to join a multicast group
* IGMP specification:
    * message type: encapsulated in IP datagram
    * IP protocol number: 2
    * `membership_query`: 
        * is sent by router to all host in LAN to determine the set of all multicast group that have been joined
        * host can send this message to join a multicast group
    * `membership_report`:
        * is a response to the query
    * `memberleave_group`:
        * for a host to leave a group
        * optional - a router determine if a host left it stop responding to `membership_query` (timeout)

## Chapter 5: Link Layer and Local Area Networks

Terminology:
* nodes: Host and routers
* link: communication channel that connects the nodes
* link layer frame: nodes encapsulate network layer datagram

#### 5.1.1 Services Provided by the Link Layer

* Move network layer datagrams from node to node over the same link
* link layer frame can be carried across different protocols

#### 5.1.2 Where is Link Layer Implemented

* Implemented in hardware (NIC) for receiving and transmitting Network Datagrams
* NIC attach to the host through a bus (PCIe)
* host does high level link layer functionality (setting the address)

### 5.2

[TODO: mutlicast](https://netlab.ulusofona.pt/rc/book/4-network/4_08/index.htm)
