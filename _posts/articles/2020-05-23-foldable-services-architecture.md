---
layout: post
title: "Foldable services architecture"
excerpt: "A simple software service architecture to create services that can be scaled out separately, or be run as a single deployment unit"
date: 2020-05-23
tags: [dev, programming, web, api, services, webservice, rest, microservices, soa]
categories: articles
image:
  feature: posts/services/office-wework-8.jpg
comments: true
share: true
published: true
modified: 2020-08-04T23:55:15+05:30
---

# Foldable micro-services

In this short article I'll describe an _approach for building a set of services, which can be deployed as separate services, or a single service with multiple end-points, without any major change in code_.

Though I have said _"micro-services"_ in the heading, it does not have anything to do with the _"standard notion of microservices"_, rather it simply means a bunch of lean services which can exist own their own without necessarily depending on the other services. So, that can be the common type of `microservices`, or `SOA` done the right way (not heavy, multi-fucntional, monolithic services) or just any type of `web services`.

We can say this is a `design pattern`, but I'd rather think of this just as a design idea, that applies to a very specific use case, as described below.

> I need to build a set of independent web services. I want deployment flexibility. I may want to deploy them as independent services on separate servers, or containers. Or in some cases, I'd want to deploy them as a single set of services, deployed as a single deployment unit. And between this move, I do not want a major code change. When deployed as single service, we'd want inter-service calls to be direct function calls rather that API calls, for performance & resources!

Following is a design approach for the problem statement above that worked for us, and I guess it might help others as well.

## Foldable services architecture - Rationale

This is what we want from our services

* In the long term, we want to deploy the web services on cloud
* We’d want to deploy the services as bunch of lean/micro services, behind an API Gateway, with public Auth integrated
* But for some customers, we may also want to deploy everything locally behind a single local web server
* For local deployment, we do not want to get into the complexity of multiple services, containers, api gateway, service orchestration etc.
* In the move from local client to cloud, we want full reuse of code, with least amount of throw-away code
* While in local client, for simplicity & performance, we’d want inter-service communication as plain function calls across DLLs
* In cloud deployment, those inter-service calls will become HTTP based API calls. This should not affect the external APIs exposed for clients to use

## Common standard

* All the services are implemented in the same language/platform/intermediate language (e.g. `JVM` or `.NET Core`)
* Each service is a separate solution/deployable (with one or more projects/modules)
* All the service solutions are part of one bigger solution
    * Easier code sharing with common projects
    * Easy to expose all service APIs together (single deployment)
    * For common models, there should be `shared models` (POJO/POCO)
* All public API definitions are defined in a common `API Contracts` layer/project
    * All services/projects can refer to the API Contracts project
    * API Contracts project refers only to the shared models project
    * If required, some domain agnostic projects like `Utilities`, `Data Access` can also be shared
* The API end-points
    * There is an `API Functions` layer that actually implements the API Contracts in full, i.e. no additional code is required to achieve the required API functionalities beyond this
    * Above that, there’d be a very thin `API Layer` which will simply expose the API Functions as network callable API end-points. In `MVC` world, those can be bunch of simple `Controllers`
    * The Controllers can have security, auth, caching etc. API functionalities plugged in, but there should be 0 (zero) application/domain logic
* In the move from single-client-app to distributed cloud deployment, nothing changes for the front-end. It still calls the same HTTP based APIs, with exact same contracts. Only the base URL will change from local URL to public API Gateway URLs. For public APIs, auth services will be integrated between client & services

### Design 01: Single deployment unit

* All services are installed locally behind a single web server
* All the service APIs are exposed as a single set of APIs
* Inter-service calls are fulfilled through direct function calls
* NOTE: Here, the bigger outer solution is deployed as one unit
* There is a single copy of binaries for all the common modules like Shared Models, API Contracts
* That also means IoC/DI resolves contracts across the bigger solution
* Please note, the generic structure of service core shown on the bottom-left is not absolute. It just means that the service core can have their own internal layers for separation of concern, maintainability, testability etc.

![Image](/images/posts/services/services_01.png)

### Design 02: Distributed deployment

* All services are installed on cloud, clients access them over internet
* All the services are deployed as independent lean/micro services
* Inter-service calls are fulfilled through API calls
* NOTE: Here each service solution is deployed separately
    * There is a separate copy of binaries for the common modules like Shared Models, API Contracts, for each service
    * That also means IoC/DI resolves contracts only within each service project
* Here, the `API Facade` layer is discarded, and each service gets it’s own `API Layer`
* And there is a `Network Layer` in each service that basically implements the API Contracts from other services (ONLY the required parts), through HTTP calls to the actual service APIs. This layer is OPTIONAL i.e. only used if the specific service needs to communicate with other service(s)

![Image](/images/posts/services/services_02.png)

## Limitations

* Possible bunch of `NotImplemented` methods in `Layer 3`
* Works only if the whole system is written in same language or targetting a single platform/intermediate language
* There still is small layer of code that is not re-used in both deployments (e.g. API Facade)