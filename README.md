MQTT Resource Directory
=======================

This specification is an attempt to adapt the concepts from [CoAP Resource Directory][1] to MQTT.


Abstract
--------

MQTT doesn't allow for custom headers, which is fine since it's designed for constrained environments. That also means there's no obvious place for metadata like Content-Type or maximum message size. In other attempts to solve this problem, metadata was stuffed into the topic. In contrast, the resource directory approach keeps message size small and allows for a self-describing data format. 


1. Introduction
---------------

MQTT is a popular publish/subscribe protocol that has seen major adoption within the Internet of Things where constrained devices need to send and receive telemetry data. Since the protocol was designed to be extremely compact, there is very little room left over for transmitting metadata. This specification solves this problem by standardizing a topic space to be used as a resource directory for metadata about certain topic spaces or the broker itself.


### 1.1 Goals

MQTT is currently used mostly in constrained environments were data size must be kept at a minimum so users tend to feel constrained on the amount of data they are sending as well as the simplicity. 

* *Secure*. Or at least the ability to be secure since many MQTT brokers don't implement any kind of topic level security. 
* *No libraries*. While this spec may be implemented by clients, this spec should be simple enough that anyone could query the resource directory and parse the results without the use of any external libraries. More importantly, many of the embedded engineers that we have talked with have been aprehensive about including any external dependencies.
* *Extensibility*. This spec doesn't presume to understand all goals that other people may have. We only want to standardize methods to communicate metadata.
* *No inflated messages*. Metadata must not be communicated through topic names or through a payload envelope. Use of an external Resource Directory lets the client query metadata only when needed and never more. Its hard to rationalize the inflation messages in hot topics that see high message throughput. 
* *Determinism*. A single GET request should return only a single message. If there are many messages returned, it becomes difficult for the client to know when they have a full listing and to stop listening. This helps achieve the goal of requiring no libraries to query the resource directory.
* *Zero coordination*. MQTT does not have constructs for ACID transactions, so no part of this spec can assume that the client can GET the value of a topic and then PUT to update it. PUT should operate only as "add" or "replace" and never as "PATCH" or "update one or more properties".

*Note: I'm not sure that I can have both Determinism and Zero Coordination. If not, than I choose Zero Coordination. As such, I've added INDEX requests, which will be avoided as much as possible.*
 

### 1.2 Web Linking

In lieu of the above stated goals, messages in this Resource Directory MUST use web linking as specified in [RFC 5988][3]. This format is easy to parse and multiple entries can be separated by newlines. This also gives us the flexibility to define custom attributes, which enables this spec to remain flexible.


2. Terminology
--------------

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119][4].

This specification requires that readers be familiar with all terms and concepts from [MQTT version 3.1.1][2].

These are some additional terms used within this specification:

* **Resource Directory topic** - Any topic starting with the string `$rd/`. There MUST NOT be any characters before this string.
* **PUT Request** - A PUBLISH message with the retained option set to `true` to a Resource Directory topic.
* **GET Request** - A SUBSCRIBE message sent to a Resource Directory topic. Each topic pattern MUST NOT contain any wildcards, but a singe GET request MAY contain multiple topic patterns.
* **INDEX Request** - A SUBSCRIBE message sent to a Resource Directory topic that contains exactly one `+` wildcard as the last character of the topic pattern.
* **DELETE Request** - A PUBLISH message with the retained option set to `true` and an empty (zero byte) payload to a Resource Directory topic.
 

### 2.1 Details on GET requests

A GET request is a SUBSCRIBE message sent to a Resource Directory topic. Each topic pattern MUST NOT contain any wildcards, so it will only retrieve a single PUBLISH message. However, a SUBSCRIBE message MAY contain many topic subscriptions, but the client will still know exactly how many PUBLISH messages to listen for.

When the client receives all expected messages, it SHOULD send a UNSUBSCRIBE message to terminate the GET request. If the client has not received one (1) or more expected messages after thirty (30) seconds, it MAY send an UNSUBSCRIBE message to terminate the request.


### 2.2 Details on INDEX requests

An INDEX request is functionally the same as a GET request except that it is non-deterministic. An INDEX request may return zero to many responses that will arrive in no specific order. A client MAY choose to terminate this request by sending an UNSUBSCRIBE message. If the client chooses to terminate the request, it SHOULD wait thirty (30) seconds after the last message has been recieved before sending the UNSUBSCRIBE message. If no messages are received, the client may send the UNSUBSCRIBE message thirty (30) seconds after sending the original UNSUBSCRIBE message.


3. Well-Known Functions
-----------------------

This section describes functions that augment the core MQTT protocol with metadata. Each of these is OPTIONAL. An INDEX request to `$rd/well-known/+` MAY be used to get a full listing of functions. 


### 3.1 Error Messages

The 3.1.1 specification does not allow for a way to communicate errors so this is a stop gap measure. In this feature, the broker SHOULD PUT values to a topic that only that client can subscribe to.

If the broker supports this feature, it MUST publish a retained message to `$rd/well-known/errors` with a payload in CoRE Link Format. In the topic given in the payload, the broker SHOULD include the string `$id`. The client MUST replace all instances of `$id` with it's own client ID.

**Examples**

| Link           | Client ID         | Topic                  |
|----------------|-------------------|------------------------|
| <$errors/$id>  | foobar            | $errors/foobar         |





 [1]: https://tools.ietf.org/html/draft-ietf-core-resource-directory-02
 [2]: http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html
 [3]: https://tools.ietf.org/html/rfc5988
 [4]: https://tools.ietf.org/html/rfc2119
