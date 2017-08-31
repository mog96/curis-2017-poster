# Stanford CURIS 2017 Poster: [*Bringing IP Networking to the Internet of Things*](https://github.com/mog96/curis-2017-poster/blob/master/curis-poster_mateo-garcia.pdf)

Missing from the poster as it was presented is a proposed solution to the issue of unbounded MLE message length in the [Thread](https://www.threadgroup.org/ThreadSpec) protocol. As is stated in the poster under "Findings on MLE Using Type-Length-Value (TLV) Structures":

> According to the authors of the specification, the implied [MLE message length] limit is the maximum transmission unit (MTU) of IPv6 (Layer 3) over IEEE 802.15.4 as outlined in the RFC document where 6LoWPAN is defined. This is confusing, however, because MLE messages are sent using the User Datagram Protocol (Layer 4), which sits on top of IPv6 and technically has its own, larger MTU that is unrelated to that of IPv6.

As is shown in the poster, an MLE message is composed as follows:

![thread-mle-message-unbounded-length](https://github.com/mog96/curis-2017-poster/blob/master/diagrams/thread-mle-message-unbounded-length.jpg)

It seems the only way to ensure that an MLE message does not exceed the IPv6 MTU in length is to fragment messages that do exceed the MTU into multiple MTU-sized messages.

I propose such a solution as follows:
- Introduce a new TLV type called MLE Extension TLV.
```
+--------------+--------------+-----------------------------------------------------------+
|              |              |                           Value                           |
|     Type     |    Length    |                                                           |
|              |              |  this message payload len   |  total message payload len  |
+--- 8 bits ---+--- 8 bits ---+---------- 16 bits ----------+---------- 16 bits ----------+
```
- The value of this TLV comprises two 16-bit values: one representing the total byte length of the payload of this MLE message, the other representing the total byte length of the unfragmented MLE message payload. This TLV must be included in each MLE message composing a fragmented MLE message.
- The receiver will be able to stitch together the complete MLE message from fragment messages based on the type of the MLE message and the source address of the packet containing it.
- An extended MLE message can only be fragmented on the boundary of a TLV, meaning that each composing MLE message must contain full TLVs.
- This implies that the maximum length of a TLV is a single MLE message payload (IPv6 MTU minus one byte for the MLE message command type, shown in the MLE message diagram above). TLVs nested within TLVs (see the TLV diagram in the [poster](https://github.com/mog96/curis-2017-poster/blob/master/curis-poster_mateo-garcia.pdf)) must be composed with the maximum length of the outermost TLV in mind.
