---
layout: post
title: "Thread synchronization basics in .NET - part II"
excerpt: "High level overview of some less common cases of thread synchronization"
date: 2018-05-28
tags: [thread, synchronization, signaling, monitor, semaphore, deadlock]
categories: articles
image:
  feature: posts/threads/pxhere.jpg
  credit: pxhere
  creditlink: https://pxhere.com/en/photo/1032879
comments: true
share: true
published: false
---

**Note:** This article is currently incomplete & in-progress. It'll be updated soon.
{: .notice--danger}

#### References

* [Asnc Await - MSDN](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/)
* [Control flow in Async Programs](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/control-flow-in-async-programs)
* [TAP - Task-based Asynchronous Pattern](https://docs.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap)
* StackOverflow QA - how does it work?
  * [CLR implementation](https://stackoverflow.com/questions/4047427/c-sharp-async-how-does-it-work/4047607#4047607)
  * [int to Task&lt;int&gt;](https://stackoverflow.com/questions/13159080/how-does-taskint-become-a-int)
  * [The wrong implementation](https://stackoverflow.com/questions/14455293/how-and-when-to-use-async-and-await)
* Async-Await & threads
  * [SO - how applications are responsive if there are no threads](https://stackoverflow.com/questions/37419572/if-async-await-doesnt-create-any-additional-threads-then-how-does-it-make-appl)
  * [SO - async await and threading](https://stackoverflow.com/questions/40249169/async-await-and-threading)
  * [SO - async stay on current thread?](https://stackoverflow.com/questions/17661428/async-stay-on-the-current-thread)
  * [Await, Synchronization Context, and Console Apps - Stephen Toub](https://blogs.msdn.microsoft.com/pfxteam/2012/01/20/await-synchronizationcontext-and-console-apps/)
  * [There is no thread](http://blog.stephencleary.com/2013/11/there-is-no-thread.html), &nbsp;[SO - related](https://stackoverflow.com/questions/600795/asynchronous-vs-multithreading-is-there-a-difference)
* More detailed, in depth discussions
  * [Async and Await - Stephen Cleary](http://blog.stephencleary.com/2012/02/async-and-await.html)
  * [Async articles - Eric Lippert](https://blogs.msdn.microsoft.com/ericlippert/tag/async/)
  * [EduAsync series - Jon Skeet](https://codeblog.jonskeet.uk/?s=eduasync)
* [Video: Short explanation on internal implementation of async await](https://www.youtube.com/watch?v=6_GTdR0gBVE)
* Asynchronous methods in ASP.NET
  * [MSDN docs](https://docs.microsoft.com/en-us/aspnet/web-forms/overview/performance-and-caching/using-asynchronous-methods-in-aspnet-45)
  * [A simple explanation - Exception not found](https://exceptionnotfound.net/using-async-and-await-in-asp-net-what-do-these-keywords-mean/)
