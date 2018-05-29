---
layout: post
title: "Easier asynchronous with Async-Await - Part I"
excerpt: "A quick introduction to Async-Await in .NET"
date: 2018-05-28
tags: [async, await, asynchronous, threading, synchronization, tasks, concurrency]
categories: articles
image:
  feature: posts/threads/pxhere.jpg
  credit: pxhere
  creditlink: https://pxhere.com/en/photo/1032879
comments: true
share: true
published: true
---

**Note:** This article is currently incomplete & in-progress. It'll be updated soon.
{: .notice--danger}

The keyword pair [Async-Await](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/) was introduced in `C# 5` as a way of writing clear (synchronous-like) code for asynchronous operations. With use of `async-await`, a method can be written which looks very similar in structure to standard synchronous code, but can work asynchronously without keeping the calling thread busy.

Here, we'll look at a typical `async` method, and see how they work practically, without going too deep in the implementation details.

#### A typical example of Async-Await

First let's look a typical asynchronous code written with async-await. This sample method, on a high level,  makes a heavy/long-running call to get a message string and then returns the length of the message.

The method `GetMessageLengthAsync()` calls a long-running (takes considerable time) method `GetTimeTakingMessageAsync()` to get a message string. While that is being processed, it does some other work `DoIndependentWork()` which is independent of the message value. Then it suspends the flow and _asynchronously_ waits for the message work to complete. Once completed, it does some processing that is dependent on the previous result (calculate the message length), and finally returns the result.

```csharp
public async Task<int> GetMessageLengthAsync()
{
    Task<string> stringTask = GetTimeTakingMessageAsync(); //calls a long running method
    DoIndependentWork(); //does some independent work meanwhile
    string message = await stringTask; //suspends & awaits the result
    int length = message.Length; //once done, does some work on awaited result
    return length; //then returns the final int result
}
```

Note that we are taking the `GetMessageLengthAsync()` as the sample `async` method. The `GetTimeTakingMessageAsync()` that is called from inside, is just another `async` method, but that is not part of the discussion.

The single most important thing to understand here is, _**the method suspends the current work-flow at `await` and waits for the long-running work to complete asynchronously, without blocking the calling thread**_. That means, the thread that originally had called `GetMessageLengthAsync()`, gets free at this point and _leaves the method_. Read along for more details.

Well, what happens after that? Before the original calling thread leaves at `await`, the runtime captures the current `SynchronizationContext` and the `TaskScheduler` and internally creates a continuation workflow. Now the calling thread leaves the method, and is free to do other tasks. Meanwhile the long-running work that was started, keeps running in the background. Note that the `GetTimeTakingMessageAsync` must also be an `async` method for our code to work. But the actual implementation does not matter.

Once that long-running (_get the message_) work is complete, the runtime uses the captures context and/or scheduler to schedule the remaining work of the method, as appropriate. And eventually the rest of the method completes and returns the final `int` value.

#### Basic characteristics

The method looks pretty similar to a synchronous method in structure but there are few significant differences. The main things to notice about the method are

1. The method signature is marked with `async`
2. The method uses `await` on a `Task` to get the result
3. The return type is `Task<int>` rather than `int`
4. The method name has `Async` as a suffix (GetMessageLengthAsync)

The points 1-3 makes the method asynchronous. The `async` enables the `await` functionality in the method. You CANNOT use `await` without using the `async` declaration on the method signature. On the other hand, a method can be declared as `async` without using `await` in the method body. It does work, but the just runs synchronously.

The `await` is the part which actually turns the method asynchronous! When the execution hits the line with `await` - it first checks if the work has completed. If yes, it simply continues as normal synchronous method. Else the current executing thread _suspends_ the work-flow of the method, captures the current context, and _returns from the method_ to the calling method. So, the main thread can continue executing other works in the calling method, and the awaited task continues in the background.

The return type of a `async` method can only be `Task`, `Task<T>` or `void`. As a general suggestion, avoid using `void` return type, as that makes the method non-awaitable, i.e. the method cannot be used with `await` from another method. If there is a result to return, use `Task<TResult>` or just use `Task`.

Now, interestingly, notice our sample method for once. The method actually returns just an integer (message.Length), but the return type is `Task<int>`! How does that work? This piece of magic is done by the `async` keyword. In simple terms, it just means that, the method returns an integer, but does that _asynchronously_ after the control was already returned from the method. See the next section.

The `Async` suffix to the method is totally optional, but it is the general convention to name `async` methods that way so that it's easier to read and notice.

#### The control flow

Before we explore the control flow in a `async` method, let's first understand what happens to the code when it is compiled. In compilcation, the compiler basically (taking it in an overly-simplified way) creates a continuation for the part of code, that is after `await`. This continuation is implementade as an auto-generated class with a _state-machine_. The state-machine has 2 states - incomplete, complete (just giving some names for easy understanding). Initially, when the work is started, it goes to _incomplete_ state. Later, when the awaited work is complete, the state-machine goes to _complete_ state, and the next part of the code is kicked off.

So, we'll take our sample code as an example. But the working principle stays same for any `async` method.

![Image](/images/posts/threads/workflow.png)

1. First an external method, say `CallingMethod()` calls our `GetMessageLengthAsync()` method. The thread that makes this call, let's call it the _main thread_ (the green lines).
2. It makes a call to another `async` method on the same thread, just like any normal code execution.
3. Now, this method `GetTimeTakingMessageAsync()` being an `async` method, returns control immediately, leaving the work of getting message to be done in background.
4. Now it makes a call to an method that does not depend on message. This is called synchronously on the main thread.
5. After completion, it comes back to `GetMessageLengthAsync()`.
6. Now the execution hits the `await`, which says it'll _wait asynchronously_ for the message. At `await`, the _main thread_ suspends the work on this method, and _returns_ to `CallingMethod()` where it is free to do other stuffs. Just before this, CLR captures the synchronization context*.
7. At a later point of time, when the message is fully received, the continuation of the rest of the code is invoked. Now depending on the *captured context, the rest of the code might run on the same _main thread_, or another thread. See next section.
8. It then executes the rest of the code, and upon completion, returns the final result to `CallingMethod()`.

#### Threads in Async-Await

A very common question around `async-await` is, are there any _new thread_ created for ths asynchronous process to execute?

The answer is, NO. There might or might not be additional threads involved. If additional threads are required, generally threads are taked from the managed `ThreadPool`.

So, when is thread pool thread used? If not, how does the work complete without a thread!

Before answering this, first lets understand that generally there are two types of work that are done asynchronously - **CPU bound**, and **I/O bound**. A CPU bound work is something that does some heavy computation, like running some formula along a huge set of data. This needs continuous CPU cycles. This will need a dedicated thread, and will _generally_ use a `ThreadPool` thread. On the other hand, I/O bound work is something that depends on something outside the CPU system. For example, time to get response from a web service, or to write data to a disk. This type of work _may not_ need any dedicated threads (the network or disk driver may handle it by themselves). Only few time-slices of thread(s) are used just for start/stop/progress notification.

Now that we have understood (at least on a high level), what happens during compilation & run-time, we know the framework is actually doing bunch of additional works to make it work. It is creating task continuation and scheduling them. It is managing the threads etc. Because of all these, additional time and resource are used. So, it is not advisable to add `async` to very lightweight works, as the overhead might outweigh the benefits.

###### Captured Context*

So, we talked about _"captured context"_ multiple times. What does it mean actually?

In simple words, it means that the CLR remembers the execution context before suspending the work at `await`, and tries to re-apply it on the continuation. So, for example, if the _main thread_ was the UI thread in a `WPF` application, the continuation runs on the same UI thread. For `ASP.NET`, it might not be the same thread, but it'll get back the same `HttpContext`. On the other hand, say for a `Console` application, there is no such context. Then the continuation runs on any thread taken from the `ThreadPool`.

**Note: Things to remember**<br />
[1] It's the return type that makes it awaitable. Because the method returns `Task` or `Task<T>`, the method can be await-ed from another method. Not because it has `async` in signature, or it uses `await` inside it's body.<br />
[2] Async all the way - if a method towards the end of a calling hierarchy is `async`, it's better to make the whole hierarchy `async`, up till the entry point method. This helps in keeping the whole process un-blocked, also code follows same standard. For some discussion, see [this](https://msdn.microsoft.com/en-us/magazine/jj991977.aspx) and [this](https://stackoverflow.com/questions/29808915/why-use-async-await-all-the-way-down).<br />
[3] Do not make all methods async - just do not make all your existing code `async`, just becaue you can. As discussed above, that might degrade performance in some scenarios. Can also lead to deadlocks if done incorrectly.
{: .notice--warning}

**Note:** The standard classes provided by the framework for I/O works, all supports `async` operations. These are well designed, and should be used in place of the synchronous counterparts whenever they make sense. For example, `File.WriteAllBytesAsync()` or `HttpClient().GetStreamAsync()`.
{: .notice--info}

###### Other articles in the series

* **[Synchronous to asynchronous in .NET](/articles/sync-to-async-in-dotnet/)**
* **[Basic task cancellation demo in C#](/articles/task-cancellation/)**

#### References

* Asnc Await documentation - [MSDN](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/), &nbsp; [Control flow](https://docs.microsoft. com/en-us/dotnet/csharp/programming-guide/concepts/async/control-flow-in-async-programs), &nbsp; [TAP - Task-based Asynchronous Pattern](https://docs.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap)
* How does it work? From StackOverflow QA - [CLR implementation](https://stackoverflow.com/questions/4047427/c-sharp-async-how-does-it-work/4047607#4047607), &nbsp; [int to Task&lt;int&gt;](https://stackoverflow.com/questions/13159080/how-does-taskint-become-a-int), &nbsp; [The wrong implementation](https://stackoverflow.com/questions/14455293/how-and-when-to-use-async-and-await)
* Async-Await & threads
  * [SO - how applications are responsive if there are no threads](https://stackoverflow.com/questions/37419572/if-async-await-doesnt-create-any-additional-threads-then-how-does-it-make-appl)
  * [SO - async await and threading](https://stackoverflow.com/questions/40249169/async-await-and-threading)
  * [SO - async stay on current thread?](https://stackoverflow.com/questions/17661428/async-stay-on-the-current-thread)
  * [Await, Synchronization Context, and Console Apps - Stephen Toub](https://blogs.msdn.microsoft.com/pfxteam/2012/01/20/await-synchronizationcontext-and-console-apps/)
  * [There is no thread - Stephen Cleary](http://blog.stephencleary.com/2013/11/there-is-no-thread.html), &nbsp; [SO - related](https://stackoverflow.com/questions/600795/asynchronous-vs-multithreading-is-there-a-difference)
* More detailed, in depth discussions by experts - [Stephen Cleary](http://blog.stephencleary.com/2012/02/async-and-await.html), &nbsp; [Eric Lippert](https://blogs.msdn.microsoft.com/ericlippert/tag/async/), &nbsp; [Jon Skeet](https://codeblog.jonskeet.uk/?s=eduasync)
* [Video: Short explanation on internal implementation of async await](https://www.youtube.com/watch?v=6_GTdR0gBVE)
* Asynchronous methods in ASP.NET - [MSDN docs](https://docs.microsoft.com/en-us/aspnet/web-forms/overview/performance-and-caching/using-asynchronous-methods-in-aspnet-45), &nbsp; [Exception not found](https://exceptionnotfound.net/using-async-and-await-in-asp-net-what-do-these-keywords-mean/)