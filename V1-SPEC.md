# Overview

The coolmsg protocol is a way for clients and servers to communicate over a two way data stream such as TCP. Here we
describe the format for the coolmsg version 1 format.

Some of the goals of the protocol are:

- Simplicity
- Security
- Flexibility

# Protocol

A client sends serialized messages addressed to objects on the server, and those objects send messages as replies. 

Each object has an id, and each message has a well known type.

Each message sent is addressed to an object id, and each response corresponds to a message.

The object ID 0 is reserved for future use, and object ID 1 is always the bootstrap object.

The allocation of objects on the server is an application detail, a message sent to the bootstrap object
may trigger the allocation of a new object. Communicating new id's to the client is an application detail,
but would generally be specified in an application specific message.

A server may process messages concurrently, or serially depending on the application,
but messages always arrive in the order they were sent.

RequestID's are effectively never reused as they are incremented until overflow.

## Wire format

All integers are sent as big endian.

### Request message

RequestID: uint64, ObjectID: uint64, MessageType: uint64, MessageLen: uint64, MessageData : bytes[MessageLen]

### Response message

RequestID: uint64, ResponseType: uint64, ResponseLen: uint64, ResponseData : bytes[ResponseLen]

# Known types

### Type 0xcf3a50d623ee637d (Clunk)

An empty message sent to an object request it to 'clunk' itself.
Clunking means to remove itself from the connection and do any cleanup
necessary.

### Type 0xd782cf4b395eca05 (Object created)

It is common to request the creation of a new coolmsg object, this message can
be used in a reply to signal to the client the id of the new object.

This is a [msgpack](https://msgpack.org/index.html) encoded map with 1 field.

- Id    uint64 # The object id of the new object.

### Type 0xd4924862b91c639d (Ok)

An empty message acknowledging something, usually sent as a response.

### Type 0x81aba3f7522edc6b (Error)

This message is used by the coolmsg system to signal errors, applications are encouraged to 
reuse it with their own error codes to signal errors.

This is a [msgpack](https://msgpack.org/index.html) encoded map with 3 fields.

- Code    uint64 # The error code as described below.
- Display string # An error message to display to a user.
- Debug   string # An error message or data to aid in debugging the error.

#### Known error codes

- 0xab0547366de885bc - A coolmsg has been sent to an object that does not exist.
- 0xd47d4e94917934b2 - A coolmsg has been sent to an object that does not expect this message.

# Generating your own types

Generate a random 64 bit integer, if the type is internal only to your application, this should be sufficient. If this
type is shared across many applications, make a pull request to this repository describing your type.

# Design notes

- Version negotiation is not needed, a different typeid can be assigned for newer messages.
- Method/function names are not needed, an object performs actions based on the messages and types.
- Mixed encoding schemes is fine, each type id also describes the encoding.
- An object id is a [capability](https://en.wikipedia.org/wiki/Capability-based_security). 