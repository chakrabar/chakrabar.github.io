---
layout: post
title: "Study notes & further reading"
excerpt: "For internal purposes only"
date: 2018-10-03
tags: [database, couchdb]
categories: articles
comments: true
share: true
published: false
---

# Study notes

## Microservices

* **Heartbeat**: A mechanism in a distributed system such that it keeps sending a signal (generally at 2Hz) as a mean of notifying it's alive. If no heartbeat is received for few intervals, it's assumed to be down. Then the cluster adjusts accordingly, e.g. by selecting a new master. Also, when a new node enters the cluster, it sends heartbeat to notify it's presence.
* **Gossip protocol**: On a high level, nodes in a cluster keeps talking (sending messages) to each other, generally one's neighbours, as a mean of spreading messages and also to keep a tab on who's available and who's not.
* **Zookeeper**: Apache project, initially started by Yahoo, as part of Hadoop. At it's core, it's a distributed configuration service for a distributed system. it maintains a distributed configuration in terms of a hierarchical key-value configuration, and maintains status of nodes. It does stuffs like
  * Naming: Identify nodes by name. Somewhat like DNS but for a cluster. Can tell who's where, and how to connect to them
  * Configuration management :
  * Cluster management: Nodes joining/leaving, live status
  * Leader election: Identify and update master
  * Locking & synchronization: Lock & update, rollback for fail over
  * Highly reliable data registry. Can scale by adding more servers
  * Ordered, consistent, atomic, fast & reliable
* **Chaos Monkey**: Developed by Netflix. An applications that runs on a distributed cluster and randomly shuts down systems. The idea is to test & verify that the systems can keep running even when multiple nodes are failing, on a live system!

## Angular

angular web app for stock ticker
angular basics

service vs factory vs provider vs const vs value ~~
how 2 way binding works
directive structure ~~
watch ~~
digest cycle & apply ~~
issues with watch ~~ causes repeated digest loops & fails if runs more than 10

ng-if vs ng-show/ng-hide :

* ng-if removes the DOM element from HTML if evaluates to false. If it turns true again, the element is inserted in DOM and scope is re-evaluated
* ng-show applies display:none css to hide the element when condition is false. If it turns true, it simply changes the css
* ng-if runs with higher priority, generally before other directives are kicked in
* ng-switch works in the same way as ng-if (with multiple conditions)

## ASP.NET

### ASP.NET MVC request lifecycle

* UrlRoutingModule gets registered with MVC applications which has all routes registered in RegisterRoutes method
* It checks incoming requests against the routes
  * If a match a found, it invokes MVCRouteHandler which in turn invokes MVCHttpHandler (or simply MVCHandler) as IHttpHandler
   * It uses ControllerFactory to create and instance of the specific controller and invokes the action method (also invokes DI if used)
   * Before actually invoking the method, the model binding happens
   * Also, if there are filters with OnActionExecuting, that is invoked before action method (after authorization filter etc.)
   * Once action method is executed, OnActionExecuted and other things are called
   * Then the OnResultExecuting is invoked followed by result execution, which calls razor (or aspx) engine to find and render a view 
   * If the result is other type of IActionResult that view, results are returned as-is to http response
   * OnResultExecuted, if any, is executed after this
  
  * The series of events are as follows
    * UrlRoutingModule
    * MVCRouteHandler
    * MVCHttpHandler
    * Controller creation
    * Action method identification
    * IAuthenticationFilter
    * IAuthorizationFilter
    * ModelBinding
    * IActionFilter 
    * Action invocation
    * IActionFilter
    * IResultFilter
    * View selection & execution
    * IResultFilter

### MVC filters

  * ActionFilterAttribute abstract base class has OnActionExecuting, OnActionExecuted, OnResultExecuting, OnResultExecuted in order
  * Other specific filters are implemented via interfaces
    * e.g. IAuthenticationFilter (MVC 5+), IAuthorizationFilter, IActionFilter, IResultFilter, IExceptionFilter in same order
	  * In-built authorization filter example - RequireHttpsAttribute, AuthorizeAttribute(string policy)
	  * In-built result filter attribute example - OutputCacheAttribute
	  * In-built exception filter attribute example - HandleErrorAttribute
	* IAuthenticationFilter has 2 methods in order, 
	   * OnAuthentication
	   * OnAuthenhenticationChallenge

## DS/Algo

cursor
temp table, temp variable in SQL server
paging

CAS ~~
OData --

SSL --
Token based authentication --
OAuth --

SQL tuning
CTE
profiler

mobile app development & security

stress test ~ with VS web performance & load test, with VSTS, and JMeter scripts 

older____

OData --
OAuth --
MVVM
RabbitMQ debugging/logs
Spark
API gateway
Microservice patterns

Azure patterns
Azure automatic fault detection/recovery

static & singleton

C# topics

MongoDB IQues

Goldman2018#

### Random links

https://www.youtube.com/watch?v=7pitmJNYUz8 Belly fat
https://www.youtube.com/watch?v=Y6Ev8GIlbxc Distributed system in 1 hour

https://www.linkedin.com/learning/azure-development-essential-training-1-azure-roadmap-and-key-features
https://www.linkedin.com/learning/learning-cloud-computing-serverless-computing
https://www.linkedin.com/learning/learning-cloud-computing-core-concepts
https://www.linkedin.com/learning/cloud-architecture-core-concepts

GAURAV SEN
https://www.youtube.com/channel/UCRPMAqdtSgd0Ipeef7iFsKw
https://en.wikipedia.org/wiki/External_sorting

https://stackoverflow.com/questions/742341/difference-between-abstraction-and-encapsulation
https://stackoverflow.com/questions/12072980/encapsulation-vs-abstraction-real-world-example


http://microservices.io/patterns/microservices.html
  http://microservices.io/patterns/apigateway.html
  http://microservices.io/patterns/client-side-discovery.html
  http://microservices.io/patterns/server-side-discovery.html
    http://microservices.io/patterns/microservice-chassis.html
	http://microservices.io/patterns/observability/distributed-tracing.html
	http://microservices.io/patterns/observability/health-check-api.html
https://smartbear.com/learn/api-design/what-are-microservices/
https://dzone.com/articles/microservices-architecture-what-when-how
https://www.nginx.com/blog/microservices-at-netflix-architectural-best-practices/
https://docs.microsoft.com/en-us/azure/architecture/guide/architecture-styles/microservices