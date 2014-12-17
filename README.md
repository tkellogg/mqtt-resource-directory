MQTT Resource Directory
=======================

This specification is an attempt to adapt the concepts from [CoAP Resource Directory][1] to MQTT.


Abstract
--------

MQTT doesn't allow for custom headers, which is fine since it's designed for constrained environments. That also means there's no obvious place for metadata like Content-Type or maximum message size. In other attempts to solve this problem, metadata was stuffed into the topic. In contrast, the resource directory approach keeps message size small and allows for a self-describing data format. More importantly, there are smaller conventions that gaining popularity but no easy way to know if they are employed on a broker. This resource directory provides a starting point for other standards.


1. Introduction
---------------


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
