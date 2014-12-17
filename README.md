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

MQTT is currently used mostly in constrained environments were data size must be kept at a minimum. It's more acceptable to put metadata in an external resource directory that can be queried on an as-needed basis than to include metadata in every message as HTTP does.

* *No libraries*. While this spec may be implemented by clients, this spec should be simple enough that anyone could query the resource directory and parse the results without the use of any external libraries.
* *Extensibility*. This spec doesn't presume to understand all goals that other people may have. We only want to standardize methods to communicate metadata.
* *No inflated messages*. Metadata must not be communicated through topic names or through a payload envelope. Use of an external Resource Directory lets the client query metadata only when needed and never more. Its hard to rationalize the inflation messages in hot topics that see high message throughput. 
* *Determinism*. A single GET request should return only a single message. If there are many messages returned, it becomes difficult for the client to know when they have a full listing and to stop listening. This helps achieve the goal of requiring no libraries to query the resource directory.
* *Zero coordination*. MQTT does not have constructs for ACID transactions, so no part of this spec can assume that the client can GET the value of a topic and then PUT to update it. PUT should operate only as "add" or "replace" and never as "PATCH" or "update one or more properties".
 

### 1.2 Web Linking

In lieu of the above stated goals, messages in this Resource Directory MUST use web linking as specified in [RFC 5988][3]. This format is easy to parse and multiple entries can be separated by newlines. This also gives us the flexibility to define custom attributes, which enables this spec to remain flexible.


2. Terminology
--------------

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119].

This specification requires that readers be familiar with all terms and concepts from [MQTT version 3.1.1][2].

These are some additional terms used within this specification:

* **Resource Directory topic** - Any topic starting with the string `$rd/`. There MUST NOT be any characters before this string.
* **PUT Request** - A PUBLISH message with the retained option set to `true` to a Resource Directory topic.
* **GET Request** - A SUBSCRIBE message sent to a Resource Directory topic. Each topic pattern MUST NOT contain any wildcards, but a singe GET request MAY contain multiple topic patterns.
* **DELETE Request** - A PUBLISH message with the retained option set to `true` and an empty (zero byte) payload to a Resource Directory topic.


3. Simple Directory Discovery
-----------------------------





 [1]: https://tools.ietf.org/html/draft-ietf-core-resource-directory-02
 [2]: http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html
 [3]: https://tools.ietf.org/html/rfc5988
