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

## Headlines

Profile with 10 years of experience with application and full stack web development, architecture with .NET, C#, MVC, REST, ASP.NET, jQuery, Entity Framework, SQL Server, JavaScript, AngularJS, Neo4j, DevOps, CI/CD skills based in Bengaluru / Bangalore.

Software developer & architect with 10+ years of experience in large enterprise application and full stack web development. Skills include .NET, C#, MVC, REST, ASP.NET, jQuery, Entity Framework, SQL Server, JavaScript, AngularJS, Neo4j, DevOps, CI/CD, design, performance, security, scalability

## EPC

[1] In a given 2D array with integers, and a specific cell, find the longest path including that cell, where all the cells have same number. And they are continuous, vertically, horizontally or diagonally.

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

[2] In a T9 keypad, when few keys are pressed, find all possible combination of characters

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

[3] Clone a linked list with a next and an arbit/random pointer

+----+   +----+   +----+   +----+
|    +--->    +--->    +--->    | Next1
| A  |   | B  |   | C  |   | D  |
|    |   |    |   |    +--->    | Next2
+--+-+   +---++   +^---+   +^--++
^  |         |     |        |  |
|  +---------------+        |  |
|            |              +--+
+------------+

[4] Design database for product category - can be hierarchical, API to read all products or by sub-category

[5] Explain the automated dynamic price u[pdate process. How version history is mentioned and rollback is enabled

[6] How do you handle multiple conflicting critical tasks with fixed time? How you manage overall deliverable timelines?

Tech stack: C#, Python, Angular 4, AWS, PostgreSQL

## VSA

Questions on controller, dictionary, authentication

1. In a given singly linked list, remove the given node (head is not given) => O(1)
2. In an array of numbers, find the maximum sum in a continuous sub-array => O(n)
3. In an array of numbers, find all pairs that sum to a given number => O(n)
4. 3 daughters
   1. Multiplication is 72
   2. Summation is known
   3. Eldest daighter loves ice-cream
5. How does SSL work end-to-end? How is client identified?
6. Mutex vs Semaphore
7. OData
8. Design a high-avilable, low-latency distributed caching system. Cache read should be as fast as possible. The distributed system should be dynamically scalable with no/minimal downtime => A cluster of key-value data stores with no master. Everyone is given a key-range based on a function. That function is available with all nodes. A request can come to any node, and that node should know if it has the key or not. If it does not have, it should redirect client to the correct node. How to scale dynamically => consistent hashing?

Tech stack: C#, MVC, .NET Core, Authorize.net, own big network, high scalability

## DLL

1. Observer vs Pub-sub: In observer, observer & subject knows each other. In pub-sub, they talk via an intermediary like MQ
2. Post vs Put: Generally post to create new resource, put for update. But actually, PUT is idempotent. You put on a given URI generally with an identifier: http://myservice.com/api/employees/33802
3. SOA vs Microservices: Microservices are generally smaller, does one small functionality. It's independent. Has own datasource and can be deployed independently.
4. Encapsulation vs Abstraction: Apparently, no one has one concrete convincing answer. Vaguely, abstraction is about what can be done without knowing the actual implementation, like interface or abstract class.
     1. Encapsulation is packing information/data & behavior together and providing selective access to do specific things, e.g. a class encapsulates properties & functions. Gives selective access withaccess modifiers.
     2. Abstraction is more about generalization, like all animals are mammals. Encapsulation is more about implementation, e.g. it can maintain state, but you don't know how.
5. WAP to implement pub-sub in JS and some weird shit of problems

### OLS (collected)

There will be a total of 3 Interview rounds. Questions that can be asked in the first round:

[1] Search in sorted and rotated array
[2] Left , right, bottom view of a tree

You might be asked to code as well. Questions that can be asked in the second round of interviews.

[3] Median of two sorted arrays
[4] Deepest node of a tree

### CPLR

1. How does async-await work. How to share context in a chain of asynchronous calls (async-local)

Next

1. WAP to find triplets of numbers summing up to 0 from an array (actual complexity O(n^2))
2. What is the role of IIS worker process
3. What is the role of Queue size in IIS? What happens when the queue is full (502 bad gateway(I doubt!))
4. How Framework level, machine level and application level config behave
5. Database design for a fully functional car parking system (different vehicles, car details, parking charge, sequential parking etc.)

Next

1. Database index, SQL Server clustered index
2. Some other questions

Next

1. How does WCF work
2. How WCF clients work
3. Method overloading in WCF
4. How two different teams can work simultaneously building server and client, while actual service and models are not ready - hinted at reflection, but doesn't really work (lol)
5. Why use XML
6. Why .NET Core, what exact benefit or performance gain (hmm...)
7. Singleton - thread safety, why double checking
8. What patterns you use in an order management system (hmm??)

Next

1. How https traffic works? How physical machines are located using IP?
2. How clients/server know it's talking to right system?
3. How certificates work?
4. How certificates work when multiple apps are running on same server (hint SNI)
5. How .Net core app run on IIS (diff. with .NET Framework app)
6. What does app pool reset do
7. WAP to find supernumber of a number (649 => 19 => 10 => 1)
8. Write css classes for a given layout (you dont remember all the css properties? that's pretty basic!)
9. Design a web building framework, where widgets can be put in a page any given layout
10. How will you retain this data and render the page - like layout, rows, spacing etc.
11. How does a SPA work
12. How to create fully client side SPA without hash in url. How to stop browser from navigation

Next

1. How your day looks like
2. How you manage teams
3. How you manage work if big ad-hoc issue comes up

Next

How you work? What you work on? Do you know about XYZ?

Tech stack: .NET, .NET Core, Java, JavaScript, SQL Server, Redis, AWS

### MDKND

1. SQL vs NoSQL. Scalability, schema-less design, when to use SQL vs NoSQL. Schema-less pros and cons.
2. How to synchronize threads. How deadlock occurs. Right way to make sure deadlock does not occur (aquire locks sequentially rather than in nested manner)
3. SVN vs Git

Next

1. IDisposable
2. How to you handle exceptions, How you use AOP, throw vs throw ex
3. How topic based messaging works in RabbitMQ, are there topic-specific stores
4. Why protobuf, what are the advantages
5. Are you moving all applications to .NET Core? If not, why
6. What all design patterns used

Next

1. How current your system is built
2. There is a critical bug that needs to be fixed. Need another person's help from same team), but he is stuck in another critical bug. How do you approach, how to use his time, do you stretch

Next

1. Design REST APIs so that users can purchase videos from any devices
2. WAP to create a simple caching class. What happens if you make the whole class static, and only the methods static? Why did you choose a Dictionary?
3. Create a sorted dictionary that also supports indexing? Which data structure to use? => I started with Linked List and BST, ruled out BST for complxity of indexing. Linear is fine as they are sorted on entry - like insertion sort. So binary sorting search can be used with O(log n). Then ruled out LL again for difficulty in indexing. Chose array. After hints realized memory problems, so went to List<T>
4. How dictionary & hashing works? Discussion around GetHashCode() that returns int. Messed up with range and division.
5. How does List<T> work? Is there a LL or array? How the array manages size?

Tech stack: C#, .NET Core, containers, microservices, Azure, Google cloud, Cassandra, MongoDB, ElasticSearch, Kafka, high scalability

^ sucked up

### WINVST

3-4 hour long stupid coding problem.

A 2-D array of numbers given to indicate seating blocks in an airplane. Seats can be Window, aisle & middle. Seating rule is - all passengers must be seated left to right, front to rear and should fill seats in the order of - aisle, window then middle. Given seating block sizes and number of passangers, display the whole seating arrangement.

Write code, unit tests, display final seating arrangement in console! Generate executable, write brief documentation on how to run.

[GitHub link](https://github.com/chakrabar/FlightSeatingManager)

### AGD

1. WAP to get unique numbers from a array of numbers, without library function
2. Extend that to any type of object. How do you make sure uniqueness is evaluated correctly
3. Design a logging system where many clients can post messages at the same time. It should have API to log message, maintain an internal queue and allow subscriptions to read from queue. It should also allow different output types like screen, file system etc.
   1. Desin the interfaces
   2. How will you maintain the queue. In-memory or dedicated queue system
   3. Who and how should decide on output type. Compare the options
4. In the above system, how would you make sure many clients work seamlessly (threads). What are the options - simple locking, producer-consumer queue
5. Strengths, areas of improvement, knowledge with scripting (Python, Perl etc.)

```cs
static T[] GetUnique2<T>(T[] data) where T : IEqualityComparer<T>
{
    var unique = new HashSet<T>();
    foreach (var item in data)
    {
        unique.Add(item);
    }
    var arrUnique = new T[unique.Count];
    unique.CopyTo(arrUnique);
    return arrUnique;
}
```

Tech stack: High scalable, available, redundant. C#, .NET, .NET Core, Scala, Python, Automation, CI

### UIP

[1] Relational vs NoSQL

[2] How do you effectively distribute relational database

[3] Why NoSQL is better for distribution, why writes perform better in NoSQL

[4] https://adventofcode.com/2017/day/3
You come across an experimental new kind of memory stored on an infinite two-dimensional grid.

Each square on the grid is allocated in a spiral pattern starting at a location marked 1 and then counting up while spiraling outward. For example, the first few squares are allocated like this:

17  16  15  14  13
18   5   4   3  12
19   6   1   2  11
20   7   8   9  10
21  22  23---> ...

Given any number, tell how many steps need to be taken from 1 (only left, right, up, down)...

App #1 
(i*2 + 1) right
(i*2 + 1) up
((i+1) *2) left
((i+1) *2) down

4i + 2 + 4i + 4 + 1
8i + 6 + 1

after n iteration
6n + 1 + 8 * (n * (n+1))/2

App #2 (hint)
Diagonal right bottom increases as 1, 9, 25...

[5] Get all strictly increasing sub-arrays from an integer array

e.g. 0, 1, 3, 1, 5, 2, 2, 1, 4, 6
//0, 1
//1, 3
//0, 1, 3
//1, 5
//1, 4
//4, 6
//1, 4, 6

```cs
int GetNumberOfIncreasingSubArrays(int[] data)
{
    var result = 0;
    var last = data[0];
    var i = 1;
    while (i < data.Length)
    {
        seqLen = 0;
        var current = data[i];
        while (current > last)
        {
            seqLength++;
            i++;
            current = data[i];
        }
        result += GetSequenceCount(seqLength); //TODO:
        i = i + seqlength;
    }
    return result;
}
```

### SHJ

Code assignment: Wikipedia article analyzer problem.

Given a paragraph from a Wikipedia article, there are few questions & answers. But the answers are jumbled up. WAP to match the answers to the questions...!!!

### CRSTK

1. How do you evaluate a new technology
2. How did you build custom SPA? How did it help? Why not just Angular?
3. Explain the SPA. Does it allow caching?
4. Why RabbitMQ? What features were used?
5. Why ProtoBuf? Did the APIs interact with ProtoBuf?
6. Explain current architecture. Does it use Microservices?
7. What kind of authentication is used? Is it a custom solution? On Identity Server?

### Others

Do not pay : EY, ST GNRL, CNT LNK
Struggling to pay: IVNT