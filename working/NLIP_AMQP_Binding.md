# AMQP Binding for NLIP

## Introduction

Advanced Message Queueing Protocol (AMQP) is an ISO-standard, enterprise-caliber message transfer protocol with a rich set of capabilities that make it well suited for use in complex and large-scale distributed software systems.

Aside from the handshakes used to establish a TLS security layer, and the optional SASL authentication exchange, AMQP is symmetric and asynchronous.

### Protocol Symmetry

Once a transport connection is established between two peer endpoints, there is no distinction in role between those endpoints.  Whereas HTTP has client and server roles and other messaging protocols have client and broker roles, AMQP is simply peer and peer.  This means that both sides of an AMQP connection are equally able to send and receive messages when and as they see fit.

### Asynchrony

AMQP communication is not characterized by temporal relationships between sent messages.  There is no requirement that any sent message be followed by a response.  Asynchrony gives flexibility to application architects and allows for high performance data transfer in a wide range of applications.

A sent message may not require a response.  Or a sent message may result in multiple responses.  Many messages can be sent and their responses collected over time as they are completed.  There is no requirement that responses be returned in the same order in which the requests were sent.  AMQP has the ability to explicitly route responses and to correlate responses to requests.

### Addressing

AMQP provides the ability to assign an address to a terminus.  A terminus is the endpoint of a link and is either a producer or a consumer of messages.  An AMQP connection in a running agent may have many termini associated with it.

Unlike an IP address, which designates a single host, the AMQP address designates a fine-grained source or destination inside a running process.

Because the AMQP address exists at the application layer, and is not associated with any host address, it allows communication to occur at a higher level of abstraction than that offered by TCP/IP.  In an intermediated topology, where agents are not directly connected to each other but communicate via intermediate processes, agents on hosts that cannot communicate via TCP/IP can interchange messages via AMQP.  This is particularly powerful in hybrid/multi-cloud environments where communication is traditionally difficult.  For example, an agent hosted in the private data center of one enterprise cannot communicate with another agent hosted in the private data center of another enterprise.  An AMQP network involving a public, or mutually reachable, intermediary can facilitate the needed communication with fine granularity.  There is no need to open networks or hosts with a VPN.

The semantics of AMQP addressing assume that there will be multiple termini with the same addresses.  This opens the possibility of anycast and multicast routing.  Anycast is used to balance work load across multiple servers.  Multicast is used to efficiently distribute data to multiple destinations.

### Direct vs. Intermediated Communication

AMQP is ultimately a message-oriented exchange protocol for point-to-point use.  It runs over a reliable network transport like TLS/TCP.  As such, it can be used to directly connect NLIP agents with one acting as the TCP client and the other as the TCP server.

AMQP, however, was designed to facilitate intermediated message transfer.  The traditional form of this is to use a message broker to store and forward messages in queues and topics.  But because the AMQP protocol is symmetric and does not require a message server (broker), it can also be used with itermediary routers.  An AMQP router provides a way to create a network over which AMQP messages can be transferred end-to-end, possibly through multiple hops, with the same reliability and delivery guarantees that can be had with direct communication.

Brokers are useful when temporal disconnect is desired (i.e. the producer and consumer are not present at the same time).  Routers are useful when store-and-forward is not needed and a more flexible, faster, and lighter weight medium is desired.

### Sessions and Links for Flow Control and Multiplexing

### To, Reply-To, and Dynamic Addresses

## Normative Content

### Message Content Mapping

NLIP messages are mapped onto AMQP message structures one-to-one.  One NLIP message, regardless of the number of sub-messages it contains is carried in one AMQP message. 

An AMQP message consists of seven sections, three of which are considered the “bare message.”  The bare message can be cryptographically signed to ensure that it has not been altered in flight from the sender to the receiver.  The other four sections are the header, delivery-annotations, message-annotations, and footer.  This binding has nothing to say about the content of these four sections.  They may be used in any way that the situation calls for (i.e. to provide reliable delivery, to set priority, to sign the bare message, etc.).

The bare message consists of three sections: properties, application-properties, and data.  Some of these sections contain fields that are relevant to NLIP.  These are described in the following subsections.

Any field that is not enumerated below may be used as the situation and implementation requires.  The NLIP binding has nothing to say about their use.

Fields in the protocol binding:

| **Section** | **Field** |
|---|---|
|properties|user-id|
|properties|to|
|properties|reply-to|
|properties|correlation-id|
|properties|content-type|
|application-properties|nlip-reply-\<format\>|
|application-data|data|


#### properties.user-id

This field is optional, but if it is present, it MUST contain the authentic identity of the user that originated the message.

If the AMQP connection is direct between the client and server, the server can enforce this requirement because the server has authenticated the client's identity.

If the message passes through intermediaries, the connection to the server is not from the client, but from the last intermediary hop.  Therefore, the authenticated identity of the connection client is _not_ the same as the originator of the message.

The origin intermediary must ensure that the identity in the user-id field matches the authenticated identity for the connection from the originating client.

The NLIP server and any of the intermediary components MAY use the authentic identity in user-id as an index into an access policy subsystem to determine whether access to the addressed resource is permitted.

#### properties.to

This field contains the AMQP terminus address of the recipient of the message.

For a request message, this is the address of the server agent.  For a response message, this is the address of the original requestor or a designated destination for the response.

#### properties.reply-to

The reply-to field is used in a message for which there will be an eventual response.  The field contains the terminus address of the endpoint where the response is to be sent.  Typically, this is a dynamic address allocated to the original requestor.

A server agent that receives a request copies the reply-to from the request into the 'to' field of the response.

#### properties.correlation-id

The correlation-id field MAY be used by a requesting client to positively correlate received responses to their requests.  A server that receives a request with a correlation-id MUST copy the contents of the correlation-id field in the request to the correlation-id field of the response.

#### properties.content-type

The content-type field MUST correctly describe the format of the NLIP message data.

If the message data is in binary CBOR format, the content-type MUST be "application/cbor".

If the message data is in JSON format, the content-type MUST be "application/json".

#### application-properties

Application-properties is an optional message section that carries a map of application-specific key/value pairs.  This binding enumerates NLIP application properties used for multi-format replies (see the section below).

#### application-data

The application data section is where the NLIP payload is encoded.  NLIP messages are encoded as "Data" sections which contain pure binary payload.  This section contains the entire NLIP message in its raw binary form or its JSON-encoded form (as indicated by the content-type field).

### Multi-Format Replies

There is a request-reply pattern that is unique to NLIP whereby a requestor may wish to receive the response in more than one format.  An example of this is a response that is generated both in machine-readable JSON format and in natural-language text for human consumption or for audit logging.  Furthermore, these different responses may need to be delivered to different consumers in the network.

This is achieved by adding items to the application-properties map using a key of the form:

`nlip-reply-<content-type>`

The value referenced by the key is an address to be used for replies of the indicated content-type.

NLIP-reply fields serve two purposes:  They indicate to the receiver which formats are desired for responses; and they provide overrides for the `reply-to` field in properties.  If an agent acting as an NLIP server receives a request with one or more NLIP-reply fields in the application-properties, it SHOULD generate responses in each of the indicated content-types and use the NLIP-reply addresses as destinations for their respective response messages.

## Questions and Issues

 - Is streaming content relevant and desired for NLIP?  As an example, might NLIP be used to transfer a live audio or video stream to an agent for processing?
 - Can there be multiple responses to a request?  This might be useful in the streamed video case:  Request contains a streaming video, responses come in real-time when certain patterns are identified in the video stream and may contain frame captures of the identified artifacts.