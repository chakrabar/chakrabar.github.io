---
layout: post
title: "Messaging Queues, ESB and others"
excerpt: "let's see..."
date: 2018-03-12
tags: [tech, messaging]
categories: articles
comments: true
share: true
published: false
---

#### What is a Messaging Queue

A kind of application that work as queue (first-in-first-out storage), where multiple applications can send to and receive messages from. Message he is some form of data for applicatio to application (or data store) communication.

Main reason for using MQs are - Scalability, distribution, asynchrony etc.

* Scalability - powerful & ditributed MQs can handle huge number of messages concurrently
* Distribution - Many applications can write to, or read from, or both from a (single or distributed) MQ. They make sure all messages are read only once
* Asynchrony - the read-write can work out-of-sync (even when some components are offline), but in order

Examples of MQ are - [Rabbit MQ](http://www.rabbitmq.com/), [MSMQ](https://msdn.microsoft.com/en-us/library/ms711472(VS.85).aspx), [Zero MQ](http://zeromq.org/) etc.

#### What is an ESB

ESB or the Enterprise Service Bus, is a loosely used term to denote a type of application service that are used for coordination of data & events across many applications in a SOA type architecture. ESBs are built on top of Messaging Queues, which more workflow features to manage application communications.

So, ESBs come with all the benfits of MQs and more like routing, adapting, message transformation, protocol bridging, wait-and-retry, platform interoperability, complex event processing, Service orchestration, security etc.

It may come with some disadvantages as well

* Configuration & maintenance complexity
* Slower communictaion for compatible services
* Single point of failure

Examples of current ESBs - [nServiceBus](https://particular.net/nservicebus), [Oracle Service Bus](http://www.oracle.com/technetwork/middleware/service-bus/overview/index.html), [Mule ESB](https://www.mulesoft.com/platform/soa/mule-esb-open-source-esb), [MassTransit](http://masstransit-project.com/) etc.

#### What is an API Gateway

Manily as a proxy server for clients to connect, which in turn connects to multiple back-end services. It's like a facade of APIs and bunch of other functionalities that makes interacting with bunch of APIs much easier for clients. Some of the main features of standard API gateways are - central configuration, service discovery (who's available and where, routing), service aggregation, load balancing, inter process communication, security, centralized (or sticky) session, caching, logging, monitoring, scaling etc.

Examples of API Gateways - [Apigee](https://apigee.com/api-management/), [Layer7](https://www.ca.com/us/company/acquisitions/layer-7-is-now-ca-technologies.html), [3scale](https://www.3scale.net/tag/api-gateway/) etc.


------------------------------------------------

# Scaling Applications with Microservices, MassTransit, and RabbitMQ

MESSAGE BROKER & SERVICE BUS
============================

Target tools:
RabbitMQ is a message broker
MassTransit is a service bus

A distributed system:
---------------------
A system consisting of multiple disjoint applications or services for back-end. One or more front-end built on the back-end.

Why?
More smaller parts are more maintainable individually rather than one big monolith
Separation of concern
Integration with 3rd-party services
Asymmetric scaling
Easy addition or removal of features
Easy & fast deployment
Multiple, distributed, multi-platform teams

Challenges
Now all the disjoint services need a way to communicate between each other.
How?
- RPC or Remote Procedure Call (e.g. .NET Remoting), it is generally tied to one platform
- SOAP (Simple Object Access protocol) services (e.g. WCF), basically a SOAP envelop wrapper around RPC, with WSDL contract etc. Makes it cross-platform
- REST or REpresentational State Transfer (e.g. Web API) - works on HTTP, light-weight and cross-platform. Represents a resource as an URI, that that works based on HTTP verbs like GET, PUT (idempotent, generally update on given uri), POST, DELETE etc.
  REST also supports HATEOS, wherein more details are sent with response that says how to fetch more information about the resource(s).
- Messaging. Message communication between services with direct messages or event based messaging. More reliable due to the inherent queueing nature. If a system goes down, and comes back up, it still works fine. All the other parts of the system can continue meanwhile without any issue.

Message Broker:
--------------

A message broker is a intermediary between services that helps in message based communication. It runs as a background application and services interact with it through APIs, and it acts like a Message Queue. Generally services send messages targeted for another service, and the target system picks them up from the Messaging Queue when it wants. Until that, the message sits with the message broker.

Since a whole distributed system depends on it, and it might become a single point of failure! It should be protected with clustering or other mechanisms.

Typical use cases:
1. Microservices
2. Integrating legacy applications with loss-less data passing
3. Powering dashboards that needs to get data from many sources
4. Simplifying complex monoliths (e.g. break down a big monolith into services, step-by-step), decoupling

Messaging Patterns:
------------------
1. Point-to-point asynchronous: messages act like COMMANDS and the source fires & forgets, does not wait for a response.
2. Point-to-point with response: works more like a RPC. The destination sends back a response message, and source waits for it.
3. Event broadcasting: message sent to bunch of services. The target services have to subscribe to the event, works on a publish-subscribe model

MSMQ vs RabbitMQ:
----------------
MSMQ with NServiceBus: Windows-only, decentralized & scales automatically. Each clients have own queues, and messages are sent directly to target client
RabbitMQ with MassTransit: Cross-platform, centralized and scales with clustering. More standardized (well, MassTransit is for .NET, the latest version v5.1.2 is .NET Standard compliant though)

# Microservices

A distributed system is built as a collection of many small, independent services. They are autonomous, can be written on any language and platform, have their own copy of data or data source.
Generally, all maintains their own dataset with only the part of data that it's concerned with. The data consistency is maintained with a message broker and something like event sourcing (event are queued with relevant data).

(*a .NET Core console application can be easily changed to a windows service using [TopShelf](https://github.com/Topshelf/Topshelf))

## Event sourcing

A pattern used to (generally) communicate between disjoint microservices. On each significant event, the corresponding service sends an event with related data to the message broker. All subscribed services to that event picks up the event with message data from the queue and act on it to update individual states. This brings the whole system to an "eventual consistency".

To remove large data loads, the system might give fully updated states of main entities (e.g. Order, Customer etc.) on a periodic basis, as snapshots. So new systems, or systems coming after a failure, need not replay the whole queue of events. They can take the latest snapshot and the events only after that snapshot timestamp.

Problems: One of the main problems of Event sourcing comes from the fact that event and related messages are immutable. So, it might get difficult to find live issues from the log of continuous events. And if something went wrong, its even more complex to fix the already queued data.

# RabbitMQ

Is a message broker/messaging queue
Written in Erlang
Cross-platform
Open source, with commercial support
Use standardized messaging protocols: AMQP (Advanced Message Queueing Protocol) v 0.9, and others with plugins
Has many technology specific clients, and can be used from any platform using standard protocols (e.g. with JavaScript)
Supports clustering for scalability & fail-over
Needs Erlang runtime to run. Runs as a  service on Windows. Does not have UI, but management plugin can be installed separately for that.

## Architecture

A service application connects to RabbitMQ with a "Connection". A connection can hold a bunch of "Channels". These channels provide concurrency. Ideally, each thread in service application should use a specific channel, whereas a connection can be shared.

The service application that sends data is called a "producer" or "publisher". 

It sends messages over wire, using AMQP to an "Exchange". The exchange does not store data, it routes it to a "Queue" or another exchange, using a "Binding".
A binding consists of, (generally) a queue, an exchange and a routing-key.

A service that is interested in a message is a "Consumer". It watches/monitors the queue(s).
If a consumer successfully consumes a message, it sends back an "Ack" or acknowledgement to the queue.
Once acknowledgement is received, the message is removed from queue.

Messages have routing-key, that is a string with optional dots like "order" or "customer.order.premium". This is used to deliver messages to specific queues with matching key.

Example RabbitMQ connection string "amqp://guest:guest@localhost:5672/" (guest/guest is the default credentials)

A message has body and properties like content type, and other optional properties like headers , expiration etc. It can have a custom correlationId (ideally a GUID) to be used as a PK.
To publish a message, minimum required are - exchange, routing-key, message properties, and the message body as byte array.

To receive messages, it needs to create a channel, specifying a queue & other properties.
Channel has basic QoS (Quality of Service) like if messages are pre-fetched etc.
channel.BasicConsume() method takes parameters like noAck:bool, queue, consumer:DefaultBasicConsumer

### AMQP Exchange Types

* Direct: A message is sent directly to a queue from a exchange. If there are named instances of queues, messages can have that name-key as routing-key. So that the message is only sent to a queue with a matching name as the routing-key. e.g. "order", "customer" etc.
* Topic: The routing key is multiple words with dots like "customer.order.premium". The routing-key in queues can have wild card pattern to match like "customer.order.*" or "customer.#" where * stands for a single word, # for multiple words. It can also have the full key.
* Fan-out: The routing-key is ignored and message is sent to all queues.

### RabbitMQ reliability options

* Acks: Acknowledgement with specific deliveryID (delivery tag). Until consumer sends back acknowledgement, message is not deleted.
* Publisher confirms: Producer gets an acknowledgement back when message is queued. If "Nack" or no-acknowledgement, a strategy needs to be put in place for retry. There is no in-bult timeout.
* Mandatory flag, bool: Unsuccessful messages are sent back ?
* Reply to sender: send an reply back to producer, when consumer gets message. Needs some RPC type strategy
* Connection & topology recover: Automatically reopens channels, binding etc on fail-over

Virtual hosts: RabbitMQ can be configured to host independent systems on the same by on separate virtual hosts, for separation & security. They are named instances and be configured through connection string.

RabbitMQ security is based on users and their permissions.

# SERVICE BUS

MassTransit is a service bus which can run integrated with RabbitMQ (or other type of transports).
It abstracts out the exchange and mainly works on the queue functionalioty.
When using MassTransit, detailed knowledge of RabbitMQ is not required, but many properties of RabbitMQ can still be confihured through MassTransit host configuration.

In MassTransit a receiving end of a message is called an end-point. One service can contain multiple end-point. They are identified with uri like "rabbitmq://hostname/queuename"
await endPoint.Send<IMessageType>(actualMessage); //anonymous classes also work 

## Basic service bus for MassTransit

* Service Bus gives a higher level abstraction around messaging queue. Senders & receivers can send & receive messages. These end-points are identified with unique URIs. 
* They can support multiple transports e.g. MassTransit supports RabbitMQ, Azure Service Bus & in-memory for unit testing.

VS RabbitMQ or general native transport APIs

* More feature rich, like with error handling, logging etc.
* High level abstraction. Doesn't provide all features of transaport, but makes easy to use (e.g. uses specific URI rather than routing-key, with fan-out type exchange)
* Transport is changable, like RabbitMQ, Azure Service Bus & in-memory for unit testing
* More strongly typed (MassTransit specific) than string/byte based systems of RabbitMQ
* Well designed for unit testing

### Scheduling

Uses Quartz.Net for scheduling. In-memory can be used for development ot unit testing.
Uses SQL Server database or in-memory for testing.
  .ScheduleMessage(destination, when, message);
  
### Responding to failures

There are several mechanisms to handle failures:
Connection management: Re-establish connection and queues on fail-over
Skipped queue: for messages that could not be routed
Retries: allows retry policy (immediate with limit number, intervals for wait-n-retry & limit, exponential delay & limit, incremental delay & limit)
Error queue: for messages that could not be processed
Fault<TMessage> response when something ends up in error queue - there should be fault end-point to get the faults thrown back

### MassTransit middleware

MassTransit comes with it's own middleware, which can be used to build a pipeline (with more middlewares) for processing messages.

A middleware in its simplest form is like

publish Task Process(IContext data, IPipe next)
{
  //do something with the data before next middleware
  await next.Process(data);
  //do something with the data after next middleware
}

Circuit breaker: This middleware can be configured to open (as disconnect in a circuit) a connection if the message keeps failing. A percentage can be configured, at which failure rate the connection will break open. After a set time interval, it'll retry.

Rate limiter: This middleware limits the messaging rate to a component, e.g. max of 100 messages per second.

Latest: Keep track of latest value from any given context, e.g. some message value.

```cs
private ILatestFilter<ConsumeContext> latestContext;
cfg.receiveEndpoint(host, queuename, e =>
{
	e.UseLatest(lg => lg.Create = 
		filter => latestContext = filter)
});
```

### Saga in MassTransit

When there is a distributed system and each component can interact with each-other, it can lead to a chaotic communication process. Sagas help manage that.

1. Coordination of message flow
2. Workflow management
3. A central truth in a distributed system

Runs long-running business processes, and is implemented as a state machine. Defines the states and the events to change state. Example: journey of an order like created -> updated -> submitted -> processed -> completed etc.

# RabbitMQ by Example

Open source & cross-platform messaging queue system. Originally written by Ericsson's telecom team using Erlang.

Reliability
Routing
Clustering for high availability
Pluggable web based management interface, apart from CLI
Cross-platform - Windows, macOS, Linux, AWS, Azure
Supports many languages - Erlang, Java, .NET, Ruby, Python, JavaScript, C++, Node etc.
Based on AMQP protocol v 0.9.1

#### Management portal features

* Declare, list & delete entities
* Queue & exchange monitoring
* Send & receive messages
* Monitoring the backend process
* Force close connections & purge queues

#### Components

(Publisher) --> [ Message broker | (Exchange) -routes-> (Queue) ] --> (Consumer)

* Exchanges - exchanges recieve messages and then route them to one or more queues. Exchanges have properties like name, durability, auto-delete & arguments.
* Queues - queues are FIFO message stores inside the messaging broker. They receive messages from the exchange based on routing. Messages have properties like name, durable (bool: if persisted on disk, apart from memory - fail safe), exclusive (one connection, delete queue when not needed), auto-delete (delete queue when consumers unsubscribe)
* Bindings - defines how messages are sent from exchanges to queues. Sometimes has a routing-key that is string used to match queues
* Consumers - applications that read messages from the broker. They subscribe to queues. AMQP provides multiple ways to define when messages are deleted, like when they are received by consumer, or when the consumer sends back acknowledgement etc. A consumer can also reject a message if it is not able to process it. Then the broker can discard or requeue the message.

#### Exchanges

* Direct - delivers message to one (or more) queue matching the routing key
* Fanout - delivers message to all queues that are bound to it. Routing key is ignored. Used for broadcasting
* Topic - sends messages to one or more queues based on pattern matching of the routing key. Commonly used for multicast messaging
* Headers - messages are routed based on headers values rather than routing key. Headers are key-value pairs. Queues can have one or multiple header key-value pairs to match

#### AMQP

* Advanced Message Queueing Protocol. A network protocol that supports clients connecting to compatible messaging queues.
* (Publisher) --> [ Message broker | (Exchange) -routes-> (Queue) ] --> (Consumer)
* Once consumer receives the message, it sends back an acknowledgement. On receiving the acknowledgement, broker removes the message from the queue.

#### RabbitMQ client library

IModel - AMQP data channel, provides AMQP operations
IConnection - an AMQP connection to message broker
ConnectionFactory - constructs IConnection instances
ConnectionParameters - ConnectionFactory configurations
QueueingBasicConsumer - receives message from server

**Note:** RabbitMQ stores and sends messages in binary format as byte array `byte[]`. How to convert the data to and from binary is up to design. If standard binary format is used like base64 encoded JSON string or ProtoBuf, it can work acorss all platforms. In AMQP general standard, "content-type" and "content-encoding" fields/headers are used to communicate this info. For .NET only use cases, the model class/POCOs can have `[Serializable]` attribute and  binary serialized can be used.
{: .notice--info}

```cs
//Install-Package RabbitMQ.Client

ConnectionFactory factory = new ConnectionFactory { HostName = "localhost", UserName = "guest", Password = "guest" };
IConnection connection = factory.CreateConnection();

IModel model = connection.CreateModel(); //channel

//create if does not exist
model.ExchangeDeclare("MyExchangeName", "direct");
model.QueueDeclare(queueName);
model.QueueBind(queueName, exchangeName, ""); //empty routing key

//publish message
model.BasicPublish("", queueName, null, message.Serialize()); //"" uses the default exchange

//receive messages
var consumer = new QueueingBasicConsumer(model);
model.BasicConsume(queueName, true, consumer); //noAck=true
//get message example below

//consume with basic acknowledgement
model.BasicQos(0, 1, false); //Quality of Service, get acknowledgement
model.BasicConsume(queueName, false, consumer); //noAck=false
var messageBytes = consumer.Queue.Dequeue();
var message = messageBytes.Body.Deserealize(); //get back message
model.BasicAck(messageBytes.DeliveryTag, false); //send acknowledgement
```

###### Worker Queue

A queue setup with multiple consumers mostly to benefit from load balancing. They can have direct exchange, and multiple consumers with same routing key, bound to same (single) queue. Consumers send back acknowledgement to say when to remove a message. If not ack-ed, the message is re-queued (put back in the same position, not at the end of the queue).

In direct exchange (not fanout), if multiple consumer bind to same queue, only one of the consumer gets the message. Messages are sent to either of the consumers in round robin way.

###### Publisher Subscriber Queue

Multiple consumers, but everyone receives every message. Separate queues are used for each consumer, and a `fanout` exchange is used to send messages to all queues.

```cs
model.ExchangeDeclare(exchangeName, "fanout", true); //fanout & durable, persisted on disk
model.BasicPublish(exchangeName, "", null, message.Serialize()); //"" to indicate all queues bound to the exchange

model.BasicConsume(queueName, true, consumer);
```

###### Direct Routing

Sends to multiple consumers with direct exchange with specific routing key. Only the ones with correct routing key (queueName) gets the message.

```cs
//producer
model.ExchangeDeclare(exchangeName, "direct");

model.QueueDeclare(CardPaymentQueueName, true, false, false, null); //true:queue is persisted
model.QueueDeclare(PurchaseOrderQueueName, true, false, false, null); //same: durable queue

model.QueueBind(CardPaymentQueueName, exchangeName, "CardPayment");
model.QueueBind(PurchaseOrderQueueName, exchangeName, "PurchaseOrder");

//consumer
model.ExchangeDeclare(exchangeName, "direct");
model.QueueDeclare(CardPaymentQueueName, true, false, false, null);
model.QueueBind(CardPaymentQueueName, exchangeName, "CardPayment");
```

###### Remote procedure Call

The client (producer) and server (consumer) communictaes through queue. Rather than exchange, they talk directly to the queues. There are 2 queues in opposite direction, one request-queue for message from client to the server, and a response-queue for response from server to the client. They request and response are generally matched through a common custom header (like process_id = 1231), and the whole process works synchronously.

#### Benefits of MQs

* Durability (persisted)
* Asynchorny (producer-consumer need not be active at same time)
* Decoupled (producer-consumer need not talk directly or know each other)
* Resilience (systems can come back after failure)
* Delivery guarantee
* Delivery order (FIFO)
* Buffering (messaging rate need not match between components)
* Scalability (scale up, and scale out with clustering)

## To cover

#### Scheduling

Needs [RabbitMQ delayed message plugin](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/). It adds a new exchange type _"x-delayed-type"_, that has capability to _delay_ a message delivery.

To delay a message in the x-delayed-type exchange type, send messages with a special header _"x-delay"_ with an integer value, that represents how many milliseconds the message will be delayed. This delay will apply to the message delivery exchange-to-queue or another exchange.

If you want to schedule a message at a specific time, interval or something more complex, use an external scheduler to push the message to the RabbitMQ. Like a default scheduler on Windows or a cron job on Linux.

##### Clustering

Clusters of RabbitMQ message brokers can be created by having multiple instances of them and connecting them through their address (e.g. hostname & port) in some configuration file. The nodes (message broker instances) must be able to connect to each other in order to form [cluster](https://www.rabbitmq.com/clustering.html).

Generally there's no master/slave*, and all nodes are considered peers. They maintain consistency through replication. Data & state are replicated by default, but not queues. Exchanges, bindings etc. are replicated to all nodes.

To replicate queues, _"queue mirroring"_ needs to be configured. The node on which the queue was declared, works as master* and the mirrored queues works as slaves. Irrespective of which node a client is connected to, messages are sent to master queue then mirrored to others. This enhances availablity but does NOT distribute load. If the master node fails, oldest mirror is promoted as master.

The clients can connect to any of the nnodes. Generally a list of addresses are configured in client connection, so that it can connect to other nodes if one/more are not available. When a node comes back from failure, it joins the cluster back, and sync up from other nodes.

###### Message persistance

Persistent messages are written to disk as soon as they arrive. On the other hand, transient messages can also be written to disk in case of memory pressure, so that it can be recovered afterwards. Persistent messages are also kept in memory for better performance. 

Messages are persisted on RabbitMQ's own database based on index and message storages.

## Installation

* Go to the RabbitMQ [download page](https://www.rabbitmq.com/download.html)
* Select your OS. For Windows, go to [Windows installer](https://www.rabbitmq.com/install-windows.html) and download from GitHub (Do not install yet)
* Check Erlang version compatibility [here](https://www.rabbitmq.com/which-erlang.html)
* Download a compatible Erlang version e.g. [v20.3](http://www.erlang.org/downloads/20.3)
* Download "OTP 20.3 Windows 64-bit Binary File" and install (as Admin)
* Install the RabbitMQ server that was downloaded (as Admin). Allow firewall access if required

Once installed, RabbitMQ service will run automatically. RabbitMQ does not enable the management portal by default. To enable

* Open command prompt
* Go to RabbitMQ installation directory e.g. C:\Program Files\RabbitMQ Server\rabbitmq_server-3.7.6\sbin
* Run command &gt;&gt; rabbitmq-plugins enable rabbitmq_management
* After installation (enabling), restart the RabbitMQ message broker (RabbitMQ service in Windows) to use the management portal
* Once completed, it can be accesses at `http://localhost:15672`. Default credential is guest/guest