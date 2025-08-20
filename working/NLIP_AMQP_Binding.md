# AMQP Binding for NLIP

## Introduction

## Normative Content

### Connection Establishment

### Sessions and Links for Flow Control and Multiplexing

### Addressing

### To, Reply-To, and Dynamic Addresses

### Message Content Mapping

NLIP messages are mapped onto AMQP message structures one-to-one.  One NLIP message, regardless of the number of sub-messages it contains is carried in one AMQP message. 

An AMQP message consists of seven sections, three of which are considered the “bare message.”  The bare message can be cryptographically signed to ensure that it has not been altered in flight from the sender to the receiver.  The other four sections are the header, delivery-annotations, message-annotations, and footer.  This binding has nothing to say about the content of these four sections.  They may be used in any way that the situation calls for (i.e. to provide reliable delivery, to set priority, to sign the bare message, etc.).

The bare message consists of three sections: properties, application-properties, and data.  Some of these sections contain fields that are relevant to NLIP.  These are described in the following subsections.

Any field that is not enumerated below may be used as the situation and implementation requires.  The NLIP binding has nothing to say about their use.

#### properties.user-id

This field is optional.  But if it is present, it MUST contain the authentic identity of the user that originated the message.

If the AMQP connection is direct between the client and server, the server can enforce this requirement because the server has authenticated the client peer.

If the message passes through intermediaries, the connection to the server is not from the client, but from the last intermediary hop.  Therefore, the authenticated identity of the connection client is _not_ the same as the originator of the message.

The origin intermediary must ensure that the identity in the user-id field matches the authenticated identity for the connection from the originating client.

The NLIP server and any of the intermediary components MAY use the authentic identity in user-id as an index into an access policy subsystem to determine whether access to the addresses resource is permitted.

#### properties.to

This field contains the AMQP terminus address of the recipient of the message.  This may be a service hosted by an NLIP server.  Or it may be the sender of an NLIP request for the purpose of routing a response back to its requestor.

#### properties.reply-to

#### properties.correlation-id

#### properties.content-type

#### application-properties

#### data

### Multi-Format Replies

### Streaming Content

## Questions and Issues
