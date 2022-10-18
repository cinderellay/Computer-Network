# Computer-Science
# RDT Communication Program Design

Catalog

Experiment 3 - RDT Communication Programming 

​	Experiment Principle

​		Reliable Communication 

​				Stop Wait Protocol (Stop Wait) 

​				Go Back N Protocol (Go Back N) 

​				Selective Retransmit Protocol (Selective Retransmit) 		RDT Packet Format 

​	Experiment Content 

​	Experiment Report 

​	Appendix 

​		poll





In the previous experiment, we learned the use of socket and wrote TCP and UDP communication programs. In order to deepen the understanding of checksum, sequence number, acknowledgement mechanism, timeout retransmission and sliding window in reliable data retransmission, this experiment requires implementing a simplified version of **Reliable data transfer protocols (RDT, RDP)** on the basis of UDP, and using RDT to transfer a file.



## Experimental principle

### Reliable Communication

In practice, we can use many methods to achieve reliable communication. For example, the simplest approach is for the sender to send one packet at a time and then block and wait until it receives an `ACK` or timeout retransmission from the receiver. This approach is known to us as the **stop-and-wait protocol**. However, we found that this blocking and waiting approach is too inefficient. The sender can send many packets at a time and then wait for the receiver to send `ACK` back. So, will the receiver receive each packet in order? Do we need to return `ACK` to each packet? Will this take up too much network bandwidth resources? Based on the thought of communication efficiency and quality, we designed the **Backoff N protocol** and **Selective retransmission protocol**. In these two protocols, the sender can send a set of packets at once and wait for the receiver to operate. In the fallback N protocol, the receiver can perform a cumulative acknowledgement (i.e., reply with the maximum sequence number of the received complete packet), and at the same time, if there is a timeout or a `NACK` is received at the sender, then the sender must retransmit that packet and all subsequent packets, because the sender cannot know if the packets after that were received correctly (because the receiver performs a cumulative acknowledgement). However, we believe that this approach is less efficient because if an intermediate packet is lost and all subsequent packets arrive correctly, the sender will waste a lot of resources on retransmissions. On the other hand, in the selective retransmission protocol, the receiver picks the non-received packets to reply `NACK` and the sender sends only the packets that timeout or receive `NACK`, which greatly reduces the retransmission overhead.

Specifically, the three different protocols are described below as shown below:

#### Stop Wait Protocol (Stop Wait)

In this protocol, the sender sends one packet at a time, sets a timeout timer for this packet, and then goes into blocking waiting for an acknowledgement message for that packet. If the sender receives an `ACK` message from the receiver, it starts sending the next packet; if it times out or receives a `NACK` message, it resends the packet. After receiving a packet, the receiver checks the packet sequence number and the sequence number of the currently expected packet `expectedseqnum`; if a complete packet is received and the packet sequence number is equal to `expectedseqnum`, an acknowledgement `ACK` message is sent; otherwise, a `NACK` message is sent.

#### Go Back N Protocol (Go Back N)

In the GBN protocol, the sender sends a sliding window of `[base, base+N-1]` that sets a timer for each packet being sent. If packet `k` times out, all packets `k` and beyond are resent. The receiver only needs to remember the sequence number `expectedseqnum` of the currently expected packet; if a complete packet is received and the packet sequence number is equal to `expectedseqnum`, an acknowledgement message is sent; otherwise the packet is discarded and `NACK` is sent. and sends `ACK` packets using cumulative acknowledgement.  

In the abstract, the state of the GBN sender side can be represented in the figure above. In Figure 1, since the receiver successfully received four packets, it only needs to reply with `ACK,4` for cumulative acknowledgement. In Figure 2, since the third packet was lost, the receiver can only reply `ACK,2` to the sender, even though the packet with sequence number 4 was received. The sender will retransmit the two packets with sequence numbers 3 and 4.



#### Selective Retransmit Protocol (SRP)

The GBN protocol is less efficient because if the sender finds an error in a packet, all its subsequent packets need to be retransmitted, regardless of whether the packet has been received correctly or not. The selective retransmission protocol allows the sender to retransmit only the erroneous packets. In this protocol, the receiver maintains a cache window `[base, base+N-1]`. If the sequence number of the received packet is within this window, it is cached; if the sequence number of the received packet is equal to the leftmost value of `base` in the window, the packet and the following consecutive packets that have received acknowledgements are delivered to the upper layer, and the window is shifted to the right. For the sender, a timer needs to be maintained for each packet, and if the packet times out, the packet is resent.



### RDT packet format

The header fields of our defined reliable RDT protocol are composed in the following way:

~~~
|----------------|-------------------|--------------|----------------------|
|   UDP Header   |  RDT_Packet_type  | RDT_Seq_num  |   Application Data   |
|----------------|-------------------|--------------|----------------------|
~~~

First, the underlying communication of the RDT packet we defined relies on the UDP protocol. In the upper layer of the UDP protocol, the RDT data header we defined contains only the control field (RDT_Packet_type) and the sequence number field (RDT_Seq_num). Finally, there is the application layer content of the packet.

- Control field: used to identify the packet type of the RDT, the possible values are as follows：

  ~~~ c
  #define RDT_CTRL_BEGN 0 // Starter Packages
  #define RDT_CTRL_DATA 1 // Data Packages
  #define RDT_CTRL_ACK 2  // ACK Packages
  #define RDT_CTRL_END 3  // End Package
  ~~~

- Sequence number field: The sequence number used to identify this RDT package, starting from 1 and never looping in the experiment:

  ~~~ c
  #define RDT_BEGIN_SEQ 1  // RDT packet initial sequence number, assuming the packet sequence number is not cyclic
  ~~~

In practice, both RDT_Packet_type and RDT_Seq_num are int variables. Therefore, the length of the header field of an RDT packet is `RDT_HEADER_LEN = (4 + 4)` .

## Experimental content

This experiment requires students to use the RDT protocol to achieve reliable transmission of files. There are sample codes in the  **code reference**attached to . The part of the TODO annotation mark is written by the students themselves. Please understand the code and complete it according to the prompt of the annotation.

1. Write the RDT receiver side program for the stop protocol. A copy of the stop protocol sender executable is included with the code and can be used to aid in debugging.
2. Write the RDT sender side of the stop protocol. Debug with the receiver side program written above. 3.
3. write the RDT sender side of Go-back-N protocol and understand the role of semaphore. the receiver side of Go-back-N protocol is the same as the receiver side of stop protocol and debug it with the receiver side program written above.

Finally, demonstrate the program to the teaching assistant and complete the acceptance.

## Lab Report

1. the underlying layer of RDT is UDP, why can the program use `recv`/`send` instead of `recvfrom`/`sendto` to send and receive data?
2. How is timeout retransmission implemented in the stop and wait sender program?
3. How to ensure that both sides end the communication correctly when there are send and receive failures (refer to the two armies against each other problem)?
4. Why must the window size be less than or equal to half the size of the sequence number space in the select retransmission protocol?

## Appendix

### poll

For this experiment we will use a more advanced socket function, *poll()*. On Linux, it has been replaced by *ppoll()* and *epoll()*. For the sake of simplicity, we will still use *poll()* for this experiment.

~~~c
#include <poll.h>
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
~~~

The word poll means polling, and the function *poll()* can be used to wait until one of a set of file descriptors is ready to perform an I/O operation. *poll()* takes three arguments.

- *fds*: the set of file descriptors to be monitored, structured as follows.

~~~c
struct pollfd {
    int fd; // file descriptor
    short events; // events of interest: is the descriptor readable, writable, or abnormal?
    short revents; // events returned: what really happened
}
~~~

The value of *events* needs to be set to define the events we are concerned about. For example, `pollfd.events = POLLIN | POLLOUT` means we are concerned about "data readable" or "data writeable".

- *nfds*: field of type *nfds_t* specifying the length of the *fds* array.
- *timeout*: specifies the maximum time, in milliseconds, that the *poll()* function will wait. -1 means wait forever blocking. 0 means return immediately, no blocking.

If executed successfully, *poll()* returns a non-negative value for the number of elements in *fds* for which the *revents* field is set to a non-zero value (indicating an event or error). A return value of 0 indicates that no events occurred within the timeout period. If an error occurs, -1 is returned, and `errno` is set to indicate the error.

For this experiment, we can use *poll()* on the sender side of the stop-and-wait protocol to check if the socket has readable `ACK` data.

