---
layout: post
title: "Foldable services architecture"
excerpt: "A simple software service architecture to create services that can be scaled out separately, or be run a single set of service end-points"
date: 2020-05-23
tags: [dev, programming, web, api, services, webservice, rest, microservices, soa]
categories: articles
image:
  feature: posts/services/services_02.png
comments: true
share: true
published: true
---

## Foldable services architecture - Rationale

* In the long term, we want to deploy the application (web front-end & back-end services) on cloud
* For initial release, we may want to deploy everything locally behind a local web server
* For local client, we do not want to get into the complexity of multiple services, service orchestration etc.
* In future, we’d want to deploy the services as bunch of lean/micro services (the DB might be shared though) behind an API Gateway, with public Auth integrated
* In the move from local client to cloud, we want full reuse of code, with least amount of throw-away code
* While in local client, for simplicity & performance, we’d want inter-service communication as plain function calls across DLLs (the web front-end calls the back-end through HTTP based APIs)
* In cloud deployment, those inter-service calls will become HTTP based API calls (no change for FE)

## Common standard

* All the services are implemented in .NET Standard/Core
* Each service is a separate solution (with one or more projects)
* All the service solutions are part of a bigger solution
    * Easier code sharing with common projects
    * Easy to expose all service APIs together (initial release)
    * For common models, there is a shared models projects
* All public API definitions are defined in a common API Contracts layer/project
    * All services/projects can refer to the API Contracts project
    * API Contracts project refers only to the shared models project
    * If required, some domain agnostic projects like Utilities, Data Access can also be shared
* The API end-points
    * There is an API Functions layer that actually implements the API Contracts in full, i.e. no additional code is required to achieve the required API functionalities beyond this
    * Above that, there’d be a very thin API Layer which will simply expose the API Functions as network callable API end-points. In .NET world, those can be bunch of Controllers
    * The Controllers can have security, auth, caching etc. API functionalities plugged in, but there should be 0 (zero) application/domain logic
* In the move from client app to cloud, nothing changes for the front-end. It still calls the same HTTP based APIs, with exact same contracts. Only the base URL will change from local URL to public API Gateway URLs. For public APIs, auth services will be integrated between client & services

### Design: V1 Local client

* All services are installed locally behind a local web server
* All the service APIs are exposed as a single set of APIs
* Inter-service calls are fulfilled through direct function calls
* NOTE: Here, the bigger outer solution is deployed as one unit
* There is a single copy of binaries for all the common modules like Shared Models, API Contracts
* That also means IoC/DI resolves contracts across the bigger solution
* Also, the generic structure of service core shown on the bottom-left is not absolute. It just means that the service cores can have their own internal layers for separation of concern

![Image](/images/posts/services/services_01.png)

### Design: Centralized web / Cloud version

* All services are installed on cloud, clients access them over internet
* All the services are deployed as independent lean/micro services
* Inter-service calls are fulfilled through API calls
* NOTE: Here each service solution is deployed separately
    * There is a separate copy of binaries for the common modules like Shared Models, API Contracts, for each service
    * That also means IoC/DI resolves contracts within each service project
* Here, the API Facade layer is discarded, and each service gets it’s own API Layer
* And there is a Network Layer in each service that basically implements the API Contracts from other services (ONLY the required ones), through HTTP calls to the actual service APIs. This layer is OPTIONAL i.e. only used if the specific service needs to communicate with other service(s)

![Image](/images/posts/services/services_02.png)

### Cons:

* Bunch of NotImplemented methods in `Layer 3`