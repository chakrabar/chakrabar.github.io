---
layout: post
title: "Interview questions, answers, notes & further reading"
excerpt: "For internal purposes only"
date: 2018-10-02
tags: [database, design, system, code, interview]
categories: articles
comments: true
share: true
published: false
---

**BTW: Use Fira Code fonts**

# Technical discussion experience

## Headlines

Profile with 10 years of experience with application and full stack web development, architecture with .NET, C#, MVC, REST, ASP.NET, jQuery, Entity Framework, SQL Server, JavaScript, AngularJS, Neo4j, DevOps, CI/CD skills based in Bengaluru / Bangalore.

Software developer & architect with 10+ years of experience in large enterprise application and full stack web development. Skills include .NET, C#, MVC, REST, ASP.NET, jQuery, Entity Framework, SQL Server, JavaScript, AngularJS, Neo4j, DevOps, CI/CD, design, performance, security, scalability

## EPC

[1] In a given 2D array with integers, and a specific cell, find the longest path including that cell, where all the cells have same number. And they are continuous, vertically, horizontally or diagonally.

```
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
```

[2] In a T9 keypad, when few keys are pressed, find all possible combination of characters

```
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
```

[3] Clone a linked list with a next and an arbit/random pointer

```
+----+   +----+   +----+   +----+
|    +--->    +--->    +--->    | Next1
| A  |   | B  |   | C  |   | D  |
|    |   |    |   |    +--->    | Next2
+--+-+   +---++   +^---+   +^--++
^  |         |     |        |  |
|  +---------------+        |  |
|            |              +--+
+------------+
```

[4] Design database for product category - can be hierarchical, API to read all products or by sub-category  
[5] Explain the automated dynamic price update process. How version history is mentioned and rollback is enabled  
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
     1. Encapsulation is packing information/data & behavior together and providing selective access to do specific things, e.g. a class encapsulates properties & functions. Gives selective access with access modifiers.
     2. Abstraction is more about generalization, like all animals are mammals. Encapsulation is more about implementation, e.g. it can maintain state, but you don't know how.
5. WAP to implement pub-sub in JS and some weird shit of problems

### OLS (other)

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

### AGD

zbzk

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

Next MxPnskv

[1] Explain experinece. Talk about an interesting recent project  
[2] Why custom SPA? Why not standard Angular? Why not just single HTML with paging or show-hide sections?  
[3] How do you make a code testable? How to get instance of dependency without setter injection? How without using IoC in the class? (hint: factory)

```cs
class DataProvider
{
  [Inject]
  public IDataAPi DataApi { get; set; }
  [Inject]
  public IContainer Container { get; set; }

  public Data GetData(int dataId)
  {
  	//DataApi api = this.DataApi; //new DataApi(); //this one
    //assuming the IoC supports this
    DataApi api = Container.GetInstance<IDataApi>(); //or factory
    string data = api.GetDataFromNetwork(dataId);
    Data res = this.ParseData(data);
    return res;
  }
}
```

[4] How do you actually write a unit test? Test coverage?

```cs
//basic setup
void Test()
{
  // Flow: When GetDataFromNetwork returns "abcd", GetData will return X
  Mock<IDataApi>
  //setup the stubb method to return "abcd"
  var result = DaraProvider.GetData("someKey");
  //assert
}
```

[5] What kind of tests you do? Who is responsible for which one? Where do they run? When? How frequenctly?  
[6] WAP. You have array of numbers from 1 .. N (each number once). We have an additional (single) number 1 <= dup <= N in array and it is not sorted. Write a function that return dup in O(1) space.  
[7] What all test cases you'll write for this function?

```cs
int FindDup(int [] arr)
{
	 var numberOfDistinct = arr.Length - 1;
   var sumOfNonDups = (numberOfDistinct * (numberOfDistinct  + 1))/2;
   var actualSum = arr.Sum(); //Linq
   return actualSum - sumOfNonDups;
}
```

Next

Write a complete application with 2 APIs. A POST API that can accept list of URLs to process. It processes them asynchronously with queues, returns a reference id. A GET API that accepts the reference id and provides the status of the queue.

Tech stack: Java, C#, Kotlin, Scala, Pyhton, Swift, .NET Core (API), SQL, Cassandra, Hadoop, Kafka, Docker, Kubernetes, ML

### UP

Sc

[1] Relational vs NoSQL  
[2] How do you effectively distribute relational database  
[3] Why NoSQL is better for distribution, why writes perform better in NoSQL  
[4] You come across an experimental [new kind of memory](https://adventofcode.com/2017/day/3) stored on an infinite two-dimensional grid.

Each square on the grid is allocated in a spiral pattern starting at a location marked 1 and then counting up while spiraling outward. For example, the first few squares are allocated like this:

```
17  16  15  14  13
18   5   4   3  12
19   6   1   2  11
20   7   8   9  10
21  22  23---> ...
```

Given any number, tell how many steps need to be taken from 1 (only left, right, up, down)...

```
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
```

[5] Get all strictly increasing sub-arrays from an integer array

```
e.g. 0, 1, 3, 1, 5, 2, 2, 1, 4, 6
//0, 1
//1, 3
//0, 1, 3
//1, 5
//1, 4
//4, 6
//1, 4, 6
```

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

Next Ts

1. Current project structure
2. What kind of auth? Is it OAuth?
3. Details on Singleton and code, multithreaded application.
4. Weather service scenario. Basically observer with some enhancements.
5. Caching service, code.

Next Ad

Design a system.

There is a file repository of logs. Each directory is for one day. Each file has record for exactly one hour. There might be data in different files in overlapping time.

Each log record is a csv entry, with first column as timestamp.

Now there is a desktop system to view logs. Whenever you log into the system, you select one day to view logs. And the system shows you the logs starting from the current time (e.g. 2:43pm), and moving forward (2:44pm and so on) for that specified date.

Design an efficient (fast & memory efficient) system to show all the logs as per requirements.

Notes (on probe):

* Each file will have 60 or less log entries
* The desktop system is a monolith for single user

### SHJ

Code assignment: Wikipedia article analyzer problem.

Given a paragraph from a Wikipedia article, there are few questions & answers. But the answers are jumbled up. WAP to match the answers to the questions...!!!

Nxt

Make it work for all scenarios. We don't want to cover all scenarios, we just want the sample case to run! (Then maybe we want to extend this to something that can generate questions and answers from a given piece of text)

### CRSTK

1. How do you evaluate a new technology
2. How did you build custom SPA? How did it help? Why not just Angular?
3. Explain the SPA. Does it allow caching?
4. Why RabbitMQ? What features were used?
5. Why ProtoBuf? Did the APIs interact with ProtoBuf?
6. Explain current architecture. Does it use Microservices?
7. What kind of authentication is used? Is it a custom solution? On Identity Server?

Krtk

1. How does your day look like?
2. How you go about architecting a new application?
3. How do you select a database?
4. How do you distributing a database?
5. How does request flow works in IIS?
6. Explain clean architecture. What is use of SOLID & IoC here?
7. How do you secure your APIs? How do you implement role based authorization in Web API?
8. How can you pass parameters on-demand to a push API? ...lazy parameters!
9. How concurrency works in relational database? => locks & isolation levels
10. How do you make sure data consistency is maintained in a relational database without heavy use of locks? (Hint row number and timestamp)
11. Repository, Unit of work and CQRS?
12. How does garbage collector work? How does it know which objects to collect? Why use IDisposable?
13. Very basic theoretical question on graph
14. Implementation of simple dictionary with small number of backing buckets

Arjn

1. Experience
2. Some technical challenge that you faced recently?
3. How do you use Azure for deployment?
4. Implementation of CI/CD? Tools used?
5. Infrastructure as code?

### MCRCHP

1. Current project, architecture & tech-stack
2. Roles & responsibilities
3. Explain a project & technical choices
4. Why change? Why looking for other options?

### IVNT

Absk

1. Explain experience. What kind of people management you do? Explain your architecture.
2. What type of database? SQL Server vs mongo. When you'll use mongo over SQL server?
3. Design a simple e-commerce system. The whole architecture including database and front end. Hinted at microservices.
4. How will you separate the services? And the database. What kind of database(s)?
5. What about security? How will you design auth layer. How will you maintain session? Secure session on multiple devices? How will you scale a big cluster of auth services/databases? (consistent hashing)
6. What information will be there in auth token?  Where will you store it? 
7. How does HTTPS work? How can you secure a message from man-in-middle attack without TLS?
8. Problems with microservices?
9. Types of attack? XSRF, XSS, CORS, DOS, DDOS? How to safe guard from XSRF or cookie stealing attack? (anti forgery token)
10. How does CORS actually work? The workflow.
11. What front-end frameworks you used? Know about LESS and SASS?
12. How does Git work internally? What happens when a new branch is created?
13. WAP to reverse an array with constant space.
14. WAP to check if two linked lists merge.

Srn

1. Explain experience & role
2. How the requirement flow works? How backlog is created? How they are prioritized?
3. Who are the stake-holders? Who do you report to? WHo reports to you? Who decides on tech?
4. How do you convince them when there is a conflict? How you go about convincing people who just rejects idea based on personal preference? What about people who avoids face-to-face interactions?
5. What you do for estimation? What when work lags behind target? How do you keep track?
6. How do you know if each team member is doing his/her work properly? How do you give feedback to people who are slow/careless? What if he/she is from another group/management? How frequently and in what way you do regular one-to-one discussion with team?
7. Who works as scrum master? Do you follow sprint planning, retrospective etc.? How does demo (to client/product team) and feedback loop work? What if they ask for lot of changes after demo?
8. How do you handle when production issue comes up suddenly? What about the time lost in production issues? And the deviation from estimated timelines?
9. What is you approach when someone comes up with a functional/technical issue?
10. Who does the deployment? How much of the process is automated? Who manages CI/CD & Devops? How comfortable are you with deployment scripting? What you do when you get a deployment failure mail?

VncL, Webex

1. Experience? Tech stack? Current work? Daily work distribution? Percentage distribution of responsibilities?
2. What is on cloud? What is not?
3. What you do if you see a service degradation? What measure you take? How do you find the issue?
4. How do you take decision with insufficient data and less/no time in hand?
5. How do you measure team performance? What you do if it is not up to expectations?
6. What have you done very creative? Why you think it was creative?
7. What have you done that went very wrong? What you could have done to prevent?
8. How do you dig deep to find problems and fix it?
9. What would you do if you deploy a new version and realise it's broken? (functionality or performance)
10. How do you go about debbuging an issue found in production?

### NKA

Ashk, Sram

1. What all tech you have worked on
2. Why Ajax Controls (ASP.NET WebForm) are not good
3. Why use MVC instead of ASP.NET Web Forms
4. What is new in .NET Core? Is .NET Core bound to IIS? How? How does it work with/without IIS?
5. String vs StringBuilder?
6. What was the main intention of .NET Core? How did they design and achieve it?
7. How did they implement the interfaces? (Somehow they wanted to point to OWIN!)
8. What design patterns you use when a Web Application becomes slow when there are many users? What patterns you use if you want to use threading? (..!!!)
9. How does static work?
10. How can you call a method in derived class from constructor of base class (also exception scenarios)
11. Lock based threading? Threading for static class? Instance class?
12. How does async-await work? How do you handle exceptions in async code?

Tech stack: VB.NET, WinForm, ASP.NET WebForm, SQL

### Others

$ problem : EY, ST GNRL, CNT LNK