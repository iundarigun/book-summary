# 4. Encoding and evolution

Applications inevitably change over time. In most cases, a change to an application's features also requires a change to data that it stores: Perhaps na new field or record type needs to be captured, or perhaps exsiting data needs to be presented in new way.

However, in a large applications, code changes often cannot happen instantaneously:
- With _server-side applications_ you may want to perform a rolling upgrade
- With _client-side applications_ you are at the mercy of the user, who may not install the update for some time.

This means that old and new versions of the code, and old and new data formats, may potentially all coexist in the system at the same time. So, we need to mantain compatibility in both directions:
- **Backward compatibility**: Newer code can read data that was written by older code
- **Forward compatibility**: Older code can read data that was written by newer code

Backward compatibility is normally not hard to achieve: As author of the newer code, you know the format of data written by older code, and so you can explicitly handdle it. Forward compatibility can be tricker, because it requeires older code to ignore additions made by a newer version of the code

## 4.1 Formats for encoding data
Programs usually work with data in (at least) two different representations:

- In memory, data is kept in data structures, and they are optimized for efficient access and manipulation by the CPU (typically using pointers).
- Writing data to a file or send it over the network, you have to encode it as some kind of self-contained sequence of bytes (for example, a JSON document). 

The translation from the in-memory representation to a byte sequence is called encoding , and the reverse is called decoding

### Language-specific format
Many programming languages come with built-in support for enecoding in-memory objects into byte sequences. They are very convinient, but they also have a number of deep problems:
- The encoding is often tied to a particular programming language. You are committing yourself to your current programming language for potentially a very long time
- The decoding process needs to be able to instantiate arbitrary classes. This is frequently a source of security problems.
- Versioning data is often an afterthought in these libraries
- Efficiency sometimes is also an afterthought

For these reasons it’s generally a bad idea to use your language’s built-in encoding for anything other than very transient purposes.

### Json, XML and CSV
Json, XML and CSV are textual formats, and thus somewhat human-readable. They also have some subtle problems:
- There is a lot of ambiguity arround the encoding of numbers. XML and CSV cannot distiguish between numbers and Strings, and JSON have problems with pointing float and precisions.
- JSON and XML have good support for Unicode character strings (i.e., human-readable text), but they don’t support binary strings. Some people get around this limitation by encoding the binary data as text using Base64, and it works, but increases the data size by 33%.
- Applications that don’t use XML/JSON schemas need to potentially hardcode the appropriate encoding/decoding logic instead.

However, JSON, XML, and CSV are good enough for many purposes. It’s likely that they will remain popular, especially as data interchange formats

### Binary encoding
For small dataset, the gains of choose a more compact or faster to parse format are negligible, but once you get into the terabytes, the choice of data format can have a big impact. Json, Xml, and CSV use a lot of space compared to binary format.

Apache Thrift and Protocol Buffers (protobuf) are binary encoding libraries that are based on the same principle. Both require a schema for any data that is encoded. They come with a code generation tool that takes a schema definition, and produces classes that implement the schema in various programming languages.

How do Thrift and Protocol Buffers handle schema changes while keeping backward and forward compatibility? 

Ideally, an encoded record is just the concatenation of its encoded fields. Each field is identified by its tag number. 
- Forward compatibility: You can add new fields to the schema, provided that you give each field a new tag number. If old code tries to read data written by new code, including a new field with a tag number it doesn’t recognize, it can simply ignore that field.
- Backward compatibility: New code can always read old data, because the tag numbers still have the same meaning. The only detail is every field you add after the initial deployment of the schema must be optional or have a default value.

Removing a field is just like adding a field, you can only remove a field that is optional.

#### Avro
Apache Avro is another binary encoding format, and it is quite different form Protobuffer and Thrift. Avro also uses a schema to specify the structure of the data being encoded. 

If you examine the byte sequence, you can see that there is nothing to identify fields or their datatypes. The encoding simply consists of values concatenated together. To parse the binary data, you go through the fields in the order that they appear in the schema and use the schema to tell you the datatype of each field. This means that the binary data can only be decoded correctly if the code reading the data is using the _exact same schema_ as the code that wrote the data. Any mismatch in the schema between the reader and the writer would mean incorrectly decoded data.

- When an application wants to encode some data, it encodes using whatever version of the schema it knows. This is known as the `writer’s schema`.

- When an application wants to decode some data, it is expecting the data to be in some schema, which is known as the `reader's schema`. 

- When data is decoded (read), the Avro library resolves the differences by looking at the writer’s schema and the reader’s schema side by side and translating the data from the writer’s schema into the reader’s schema.

- it’s no problem if the writer’s schema and the reader’s schema have their fields in a different order, because the schema resolution matches up the fields by field name.

- To maintain compatibility, you may only add or remove a field that has a default value (or null). 
    - If you were to add a field that has no default value, new readers wouldn’t be able to read data written by old writers, so you would break backward compatibility. If you were to remove a field that has no default value, old readers wouldn’t be able to read data written by new writers

-  How does the reader know the writer’s schema with which a particular piece of data was encoded?
    - Large file with lots of records: the writer of that file can just include the writer’s schema once at the beginning of the file.
    - Database with individually written records: The simplest solution is to include a version number at the beginning of every encoded record, and to keep a list of schema versions in your database. 
    - Sending records over a network connection: When two processes are communicating over a bidirectional network connection, they can negotiate the schema version on connection setup and then use that schema for the lifetime of the connection.

- A database of schema versions is a useful thing to have in any case, since it acts as documentation and gives you a chance to check schema compatibility

### The Merits of Schemas
The ideas on which these encodings are based are by no means new. Many data systems also implement some kind of proprietary binary encoding for their data. For example, most relational databases have a network protocol over which you can send queries to the database and get back responses

Advantages of binary schema-based:
- They can be much more compact than the various “binary JSON” variants
- The schema is a valuable form of documentation, and because the schema is required for decoding, you can be sure that it is up to date
- Keeping a database of schemas allows you to check forward and backward compatibility of schema changes, before anything is deployed.
- For users of statically typed programming languages, the ability to generate code from the schema is useful, since it enables type checking at compile time.

In summary, schema evolution allows the same kind of flexibility as schemaless/schema-on-read JSON databases provide.

## 4.2 Dataflows
There are many ways data can flow from one process to another:
- Via databases
- Via service calls
- Via asynchronous message passing

### Dataflow via databases
The process that writes to the database encodes the data, and the process that reads from the database decodes it.

Those processes might be several different applications or services, some processes accessing the database will be running newer code and some will be running older code. This means that a value in the database may be written by a _newer_ version of the code, and subsequently read by an _older_ version of the code that is still running.

    An an older version of the code (which doesn’t yet know about the new field) reads the record, updates it, and writes it back. In this situation, the desirable behavior is usually for the old code to keep the new field intact. The encoding formats discussed previously support such preservation of unknown fields, but sometimes you need to take care at an application level

### Dataflow Through Services: REST and RPC

 The most common arrangement is to have two roles: clients and servers. Although HTTP may be used as the transport protocol, the API implemented on top is application-specific, and the client and server need to agree on the details of that API.

This approach is often used to decompose a large application into smaller services by area of functionality, such that one service makes a request to another when it requires some functionality or data from that other service. 

In some ways, services are similar to databases: they typically allow clients to submit and query data. But services expose an application-specific API that only allows inputs and outputs that are predetermined by the business logic. This restriction provides a degree of encapsulation: services can impose fine-grained restrictions on what clients can and cannot do.

#### The problems with remote procedure calls (RPCs)
Some technologies are based on the idea of the _remote procedure call (RPC)_. The RPC model tries to make a request to a remote network service look the same as calling a function or method in your programming language. Although RPC seems convenient at first, the approach is fundamentally flawed:
- A local function call is predictable and either succeeds or fails, depending only on parameters that are under your control. A network request is unpredictable
- A local function call either returns a result, or throws an exception, or never returns. A network request has another possible outcome: it may return without a result, due to a timeout. 
- If you retry a failed network request, it could happen that the requests are actually getting through, and only the responses are getting lost. Retrying will cause the action to be performed multiple times.
- Every time you call a local function, it normally takes about the same time to execute. A network request is much slower than a function call, and its latency is also wildly variable
- When you make a network request, all those parameters need to be encoded into a sequence of bytes that can be sent over the network. That’s okay if the parameters are primitives like numbers or strings, but quickly becomes problematic with larger objects.
- The client and the service may be implemented in different programming languages, so the RPC framework must translate datatypes from one language into another. 

the appeal of REST is that it doesn’t try to hide the fact that it’s a network protocol

Despite all these problems, RPC isn’t going away. gRPC is an RPC implementation using Protocol Buffers. This new generation of RPC frameworks is more explicit about the fact that a remote request is different from a local function call.

Custom RPC protocols with a binary encoding format can achieve better performance than something generic like JSON over REST. However, a RESTful API has other significant advantages: it is good for experimentation and debugging

### Message-Passing Dataflow
The asynchronous message-passing systems are somewhere between RPC and databases. Using a message broker has several advantages compared to direct RPC:
- It can act as a buffer if the recipient is unavailable or overloaded
- It can automatically redeliver messages to a process that has crashed
- It avoids the sender needing to know the IP address and port number of the recipient
- It allows one message to be sent to several recipients
- It logically decouples the sender from the recipient 

A difference compared to RPC is that message-passing communication is usually one-way: a sender normally doesn’t expect to receive a reply to its messages. This communication pattern is asynchronous: the sender doesn’t wait for the message to be delivered, but simply sends it and then forgets about it.

In general, **message brokers** are used as follows: one process sends a message to a named queue or topic, and the broker ensures that the message is delivered to one or more consumers of or subscribers to that queue or topic. There can be many producers and many consumers on the same topic.

    Message brokers typically don’t enforce any particular data model. If the encoding is backward and forward compatible, you have the greatest flexibility to change publishers and consumers independently and deploy them in any order. 
    If a consumer republishes messages to another topic, you may need to be careful to preserve unknown fields, to prevent the issue described previously in the context of databases

---
## Summary
Data encoding formats and their compatibility properties:

- Programming language–specific encodings are restricted to a single programming language and often fail to provide forward and backward compatibility.
- Textual formats like JSON, XML, and CSV are widespread, and their compatibility depends on how you use them. They have optional schema languages, which are sometimes helpful and sometimes a hindrance. These formats are somewhat vague about datatypes, so you have to be careful with things like numbers and binary strings.

- Binary schema–driven formats like Thrift, Protocol Buffers, and Avro allow compact, efficient encoding with clearly defined forward and backward compatibility semantics. The schemas can be useful for documentation and code generation in statically typed languages. However, they have the downside that data needs to be decoded before it is human-readable.

Several modes of dataflow, illustrating different scenarios in which data encodings are important:

- Databases, where the process writing to the database encodes the data and the process reading from the database decodes it

- RPC and REST APIs, where the client encodes a request, the server decodes the request and encodes a response, and the client finally decodes the response

- Asynchronous message passing (using message brokers or actors), where nodes communicate by sending each other messages that are encoded by the sender and decoded by the recipient