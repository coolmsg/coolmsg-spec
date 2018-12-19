# Overview

The coolmsg protocol is a way for clients and servers to communicate over a two way data stream such as TCP. Here we
describe the format for the coolmsg version 1 format.

Some of the goals of the protocol are:

- Simplicity.
- Security, (an object id is a capability - https://en.wikipedia.org/wiki/Capability-based_security).
- Flexibility.

# Protocol

A client sends serialized messages to objects on the server, and those objects send replies. The allocation of objects on the server
is an application detail, but the main idea is that each object has an id, and each message has a well known type.

The object ID 0 is reserved for future use, and object ID 1 is always the bootstrap object.

A server may process messages concurrently, or serially depending on the application,
but messages always arrive in the order they were sent.

RequestID's are effectively never reused as they are incremented until overflow .

## Wire format

All integers are sent as big endian.

### Request message

RequestID: uint64, ObjectID: uint64, MessageType: uint64, MessageLen: uint64, MessageData : bytes[MessageLen]

### Response message

RequestID: uint64, ResponseType: uint64, ResponseLen: uint64, ResponseData : bytes[ResponseLen]

# Known types


## Generating your own types

Generate a random 64 bit integer, if the type is internal only to your application, this should be sufficient. If this
type is shared across many applications, make a pull request to this repository describing your type.

### 0x81aba3f7522edc6b (coolmsg error)

This is an https://msgpack.org/index.html encoded map with 3 fields.

- Code    uint64 # The error code as described below.
- Display string # An error message to display to a user.
- Debug   string # An error message or data to aid in debugging the error.

#### Known error codes

- 0xab0547366de885bc - A coolmsg has been sent to an object that does not exist.
- 0xd47d4e94917934b2 - A coolmsg has been sent to an object that does not expect this message.

