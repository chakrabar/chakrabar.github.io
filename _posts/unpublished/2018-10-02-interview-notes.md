---
layout: post
title: "Interview questions, answers, notes & further reading"
excerpt: "For internal purposes only"
date: 2018-10-02
tags: [database, couchdb]
categories: articles
comments: true
share: true
published: false
---

**BTW: Use Fira Code fonts**

# Interview experience

## Epcr

1. In a given 2D array with integers, and a specific cell, find the longest path including that cell, where all the cells have same number. And they are continuous, vertically, horizontally or diagonally.

+--------------+
| 2| 1| 2| 3| 0|
+--------------+
| 0| 1| 0| 0| 2|
+--------------+
| 1| 1| 0| 2| 7|
+--------------+
| 3| 2| 1| 0| 1|
+--------------+
| 7| 3| 1| 1| 0|
+--------------+

2. In a T9 keypad, when few keys are pressed, find all possible combination of characters

+----+----+----+
| 1  | 2  | 3  |
|    | abc| def|
+--------------+
| 4  | 5  | 6  |
| ghi| jkl| mno|
+--------------+
| 7  | 8  | 9  |
|pqrs| tuv|wxyz|
+--------------+
     | 0  |
     |    |
     +----+

3. Clone a linked list with a next and an arbit/random pointer
	 

+----+   +----+   +----+   +----+
|    +--->    +--->    +--->    | Next1
| A  |   | B  |   | C  |   | D  |
|    |   |    |   |    +--->    | Next2
+--+-+   +---++   +^---+   +^--++
^  |         |     |        |  |
|  +---------------+        |  |
|            |              +--+
+------------+

## Vsa

1. In a given singly linked list, remove the given node
2. In an array of numbers, find the maximum sum in a continuous sub-array
3. 3 daughters
  1. Multiplication is 72
  2. Summation is known
  3. Eldest daighter loves ice-cream
4. In an array of numbers, find all pairs that sum to a given number

## Dll & others

1. Observer vs Pub-sub: In observer, observer & subject knows each other. In pub-sub, they talk via an intermediary like MQ
2. Post vs Put: Generally post to create new resource, put for update. But actually, put is idempotent. You put on a given URI generally with an identifier: http://myservice.com/api/employees/33802
3. SOA vs Microservices: Microservices are generally smaller, does one small functionality. It's independent. Has own datasource and can be deployed independently.
4. Encapsulation vs Abstraction: Apparently, no one has one concrete convincing answer. Vaguely, abstraction is about what can be done without knowing the actual implementation, like interface or abstract class.
     1. Encapsulation is packing information/data & behavior together and providing selective access to do specific things, e.g. a class encapsulates properties & functions. Gives selective access withaccess modifiers.
     2. Abstraction is more about generalization, like all animals are mammals. Encapsulation is more about implementation, e.g. it can maintain state, but you don't know how.

### Some links

https://en.wikipedia.org/wiki/External_sorting

https://stackoverflow.com/questions/742341/difference-between-abstraction-and-encapsulation
https://stackoverflow.com/questions/12072980/encapsulation-vs-abstraction-real-world-example

---------------------

Profile with 10 years of experience with application and full stack web development, architecture with .NET, C#, MVC, REST, ASP.NET, jQuery, Entity Framework, SQL Server, JavaScript, AngularJS, Neo4j, DevOps, CI/CD skills based in Bengaluru / Bangalore.

Software developer & architect with 10+ years of experience in large enterprise application and full stack web development. Skills include .NET, C#, MVC, REST, ASP.NET, jQuery, Entity Framework, SQL Server, JavaScript, AngularJS, Neo4j, DevOps, CI/CD, design, performance, security, scalability

### Ola

There will be a total of 3 Interview rounds.

you will be rated on:
1) Problem Solving
2) Coding Skills
3) Knowledge on best coding Principles.
4) Learning Capacity
5) Detail orientation

Questions that can be asked in the first round:

Q1: Search in sorted and rotated array.
Q2: Left , right, bottom view of a tree

You might be asked to code as well.

Questions that can be asked in the second round of interviews.

Q1) median of two sorted arrays:
Q2)Deepest node of a tree

You might be asked to code as well.

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
  * Consistent, fast & reliable
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

https://www.linkedin.com/learning/azure-development-essential-training-1-azure-roadmap-and-key-features
https://www.linkedin.com/learning/learning-cloud-computing-serverless-computing
https://www.linkedin.com/learning/learning-cloud-computing-core-concepts
https://www.linkedin.com/learning/cloud-architecture-core-concepts


GAURAV SEN
https://www.youtube.com/channel/UCRPMAqdtSgd0Ipeef7iFsKw

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