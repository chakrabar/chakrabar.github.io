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

Manily as a proxy server for clients to connect, which in turn connects to multiple back-end services.