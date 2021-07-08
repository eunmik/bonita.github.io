---
layout: post
title: "[Network] Packet Switching & Circuit Switching?"
date: 2021-06-24
excerpt: "Computer Networking A Top Down Notch 7th (2)"
tags: [Network, Packet Switching, Circuit Switching]
comments: true
---
# Chapter 1 Computer Networks and the Internet

## 1.3 The Network Core 

### 1.3.1 Packet Switching 
<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0625/슬라이드1.JPG" />
In a network application, end systems exchange messages with each other.
Now, we know end systems indicate desktop, laptop, smartphone, and so on. 

To send a message from a source end system to a destination end system, the source breaks long messages into smaller chunks of data known as packets. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0625/슬라이드2.JPG" />
Between source and destination, each packet travels through communication links and packet switches. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0625/슬라이드3.JPG" />
The packet transmission time in seconds can be obtained from the packet size in bit and the bit rate int bit/s.

### store-and-Forward Transmission 
<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0625/슬라이드4.JPG" />
Most packet switches use store-and-forward transmission at the inputs to the links.
Store-and-forward transmission means that the packet switch must receive the entire packet before it can begin to transmit the first bit of the packet onto the outbound link. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0625/슬라이드5.JPG" />
In this example, the source has three packets. The source has transmitted some of packet 1, and the front of packet 1 has already arrived at the router.
Because the router employs store-and-forwarding, at this instant of time, the router cannot transmit the bits it has received.
Instead it must first "store" the packet's bits. Only after the router has received all of the pacekts' bits can it begin to "forward" the packet on the outbound link.
Routers need to receive, store, and process the entire packet before forearding. 

### Queuing Delays and Packet Loss
<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0625/슬라이드6.JPG" />
Each packet switch has multiple links attached to it. 
For each attached link, the packet switch has an output buffer (also called an output queue), which stores packets that the router is about to send into that link.
If an arriving packet needs to be transmitted onto a link but finds the link busy with the transmission of another packet, the arriving packet must wait in the output buffer.
Thus, in addition to the store-and-forward delays, packets suffer output buffer queueing delays. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0625/슬라이드7.JPG" />
Since the abount of buffer space is finite, packet loss will occur - either the arriving packet or one of the already-queued packets will be dropped. 

### Forwarding Tables and Routing Protocols
<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0625/슬라이드8.JPG" />
In the Internet, every end system has an address called an IP address. 
When a packet arrives at a router, the router examines the address and searches its forwarding table, using this destination address, to find the appropriate outbound link.
and the Internet has a number of special routing protocols that are used to automatically set the forwarding tables. 

### 1.3.2Circuit Switching
<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0625/슬라이드9.JPG" />
There are two fundamental approaches to moving data through a network of links and switches: circuit switching and packet switching. 
In a circuit-switched networks,m the resources needed along a path (buffers, link transmission rate) to provide for communication between the end systems are reserved for the duration of the communication session between the end systems. 
In a packet-switched networks, these resources are not reserved. 
As a simple analogy, one restaurant requires reservations. When we arrive we can immediately be seated and order our meal. For the restaraunt that does not require reservations, we don't need to bother to reserve a table. But when we arrive at the restaurant, we may have to wait for a table. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0625/슬라이드10.JPG" />
Traditional telephone networks are examples of circuit-switched networks. 
Before the sender can send the information, the network must establish a connection between the sender and the receiver.
The switches on the path between the sender and receiver maintain connection state for that connection. This connection is called circuit.
The network reserves a constant transmission rate in the network's links so the sender can transfer the data to the recevier at the guranteed constant rate. 

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2021/0625/슬라이드11.JPG" />
In this example,. the four circuit switches are interconnected by four links. Each of these linke has four circuits, so that each link can support four simultaneous connections. 
When two hosts want to communicate, the network establishes a dedicated end-to-end connection between the two hosts. 
The dedicated end-to-endconnection uses the second circuit in the first link and the fourth circuit in the second link. the connection gets one fourth of the link's total transmission capacity for the duration of the connection.




> 📘 모르는 단어들 
>
> predominant : present as the strongest or main element. 
>
> incident : an event or occurrence. 
>
> slab : a large, thick, flat piece of stone, concrete, or wood, typically rectangular
>
> hence : as a consequence; for this reason. 이런 까닭에 
>
> hierarchical /ˌhī(ə)ˈrärkək(ə)l/ : of the nature of a hierarchy; arranged in order of rank 
>
> adjacent /əˈjās(ə)nt/ : next to or adjoining something else.
>
> to whet your appetite : to sharpen one's desire for food
>
> facsimile /fækˈsɪm.əl.i/ : an exact copy, especially of a document. 
> 
> 



