---
layout: post
title: "How does Async-Await work - Part II"
excerpt: "A step-by-step example of converting synchronous code to asynchronous with async-await in .NET"
date: 2018-06-04
tags: [async, await, asynchronous, threading, synchronization, tasks, concurrency]
categories: articles
image:
  feature: posts/threads/goreme.jpg
  credit: Author - Göreme, Turkey
comments: true
share: true
published: true
---

This post is a continuation of **[How does Async-Await work - Part I](/articles/async-await/)**. Now that we understand the basics of how Async-Await works in .NET, we'll look at some general development scenarios and see how we can convert a simple synchronous method to asynchronous step-by-step.

#### Async-await in `ASP.NET`

Like any other code, async-await can be used in `ASP.NET` web programming as well. Let's look at an example with `MVC controller`. Consider we have the following synchronous code, which makes a call to an external website and returns the content.

```cs
public IActionResult GetData()
{
    using (var client = new HttpClient())
    {
        var response = client.GetAsync("https://arghya.xyz").Result;
        return Content(response.Content.ReadAsStringAsync().Result, "text/html");
    }
}
```

We can simply convert it into asynchronous making the basic changes - add `async` in method signature, return a `Task<T>`, use `await` to get results from asynchronous calls, and ideally change method name to add `Async` suffix.

```cs
public async Task<IActionResult> GetDataAsync()
{
    using (var client = new HttpClient())
    {
        var response = await client.GetAsync("https://arghya.xyz");
        var content = await response.Content.ReadAsStringAsync();

        return Content(content, "text/html");
    }
}
```

Here we get the response to our GET call asynchronously with `await`. Then, once we have the response, again we read the contents asynchronously. Then we return the final results.

Like any other `async` method, this does not guarantee an improvement in total execution time, BUT it does help in keeping the threads free. And since the runtime runs the `async` continuation on the _"captured context"_, the code following the `await` can still access the `HttpContext` from the original thread.

#### Multiple await

Can you use multiple await inside an `async` method? Absolutely. The compiler will create multiple states in the state-machine based on them, and at runtime the execution will suspend and resume at all `await` points. Look at the simple code below, which uses two `await` to get two different results, then combines them and returns the desired final result.

```cs
public static async Task<string> GetPersonDetailsAsync()
{
    int id = await GetIdAsync();
    string name = await GetNameAsync();
    return $"Person Id:{id}, Name:{name}";
}
```

This works fine. But we can improve it. In the above code, it executes the tasks one-by-one. We can simply start them both (ONLY when they are independent of each other) and then wait for them to complete, speeding up the overall process. The trick is to bundle the tasks with `Task.WhenAll()` and use a single `await`.

```cs
public static async Task<string> GetPersonDetails2()
{
    Task<int> idTask = GetIdAsync();
    Task<string> nameTask = GetNameAsync();
    await Task.WhenAll(idTask, nameTask); //await both the tasks
    return $"Person Id:{idTask.Result}, Name:{nameTask.Result}";
}
```

**Note:** Understand that using a `Task.WhenAll()` does not block the thread. It simply returns a `Task` that can be awaited, and will complete when all the tasks are complete. On the other hand, `Task.WaitAll()` will block the current thread until all the tasks are complete. This may lead to undesirable performances, and can also lead to deadlocks in some scenarios. More details later.
{: .notice--warning}

#### Exception handling

As discussed in [Synchronous to asynchronous in .NET](/articles/sync-to-async-in-dotnet/), if a task fails to run successfully, it terminates the task and attaches the exception in the returned `Task`. This exception can be handled when doing a `task.Wait()` or `task.Result` in a synchronous style code.

In case of `async` code, we can simply wrap the `await task` call inside a `try-catch` block.

```cs
public async static void Execute()
{
    try
    {
        var message = await GetMessageAsync(); //.Result for synchronous
    }
    catch (Exception ex) //exception will be caught here
    {
        Console.WriteLine("Oh snap! " + ex.Message);
    }
}
```

Things are different if the async method (e.g. GetNameAsync here) is void and it's difficult to handle the exception nicely. So, as discussed before, return a `Task` or `Task<T>` and avoid the void return.

> Async void methods have different error-handling semantics. When an exception is thrown out of an async Task or async Task&lt;T&gt; method, that exception is captured and placed on the Task object. With async void methods, there is no Task object, so any exceptions thrown out of an async void method will be raised directly on the SynchronizationContext that was active when the async void method started. - [MSDN](https://msdn.microsoft.com/en-us/magazine/jj991977.aspx)

#### Synchronous to asynchronous

Now we'll see how we can change traditional synchronous methods to asynchronous step-by-step and what are the options.

First, we start with the simple synchronous method that does couple of things. It calls a heavy/slow method `DoTimeTakingWork()` to get a string message. It does some other works in `DoIndependentWork()` that is independent of the message data. Then it does some processing on the returned message (for simplicity, it just calculated the length of the message) and returns the final result. Everything works synchronously, the calling thread waits at each step and takes roughly ~7000 (5000 + 2000 + few more) millisecond to complete.

```csharp
internal static int GetMessageLength()
{
    var message = DoTimeTakingWork(); //do the heavy work
    DoIndependentWork(); //do some other independent work
    var length = message.Length; //do some work on previous result
    return length; //then returns the final result
}

static void DoIndependentWork()
{
    Thread.Sleep(2000); //works for 2 seconds
}

static string DoTimeTakingWork() //the actual return value does not matter
{
    Thread.Sleep(5000); //works for 5 seconds and returns result
    return $"Current time : {DateTime.Now.ToLongDateString()}";
}
```

Just for little more clarity, showing a simple `Main` method that just calls the `GetMessageLength()` method to get the final message length. It also does some other independent work with `DoSomeOtherWork()`, and have a `Stopwatch` to calculate the total run time.

Currently the whole method runs on the same thread and takes ~8000 millisecond (7000 + 1000) to complete.

```csharp
//The calling method
static void Main(string[] args)
{
    var sw = new Stopwatch();
    sw.Start();
    var messageLength = GetMessageLength();
    DoSomeOtherWork(); //independently works for 1000 millisecond
    sw.Stop();
    Console.WriteLine($"Total Time taken = {sw.ElapsedMilliseconds} ms");
}
```

###### With `Task`

We start moving towards asynchronous with introduction of a simple `Task`. We run our heavy process `DoTimeTakingWork()` as a task, so that we can also start `DoIndependentWork()` at the same time. Then we wait for the task to complete. At this point, the current thread blocks and waits for it to complete.

```csharp
internal static int GetMessageLength()
{
    Task<string> task = Task.Run(() => DoTimeTakingWork());
    DoIndependentWork(); //start both the work
    var message = task.Result; //the thread WAITS & BLOCKS here
    var length = message.Length; //does some work on result
    return length; //then returns the final result
}
```

The method runs on the same calling thread, though the `Task` runs on a separate thread. This method completes in ~5000 millisecond, as the two pieces of work run concurrently. The `Main` method takes ~6000 millisecond (5000 + 1000 + little more for rest of the work) to complete.

###### With `async-await`

Now we'll go full-blown `async-await`. First we make the `DoTimeTakingWork()` method `async` using a `Task`. Then we call that from `GetMessageLength()` asynchronously, thus making it `async` too. Notice that we have changed the names of the methods following convention.

```csharp
internal async static Task<int> GetMessageLengthAsync()
{
    var stringTask = DoTimeTakingWorkAsync();
    DoIndependentWork(); //start the other work simultaniously, 2000 ms
    var message = await stringTask; //awaits the result ASYNCHRONOUSLY
    var length = message.Length; //THIS MAY RUN ON A SEPARATE THREAD *
    return length; //then returns the final result
}

async static Task<string> DoTimeTakingWorkAsync()
{
    var messageTask = Task.Run(() =>
    {
        //THIS RUNS ON A SEPARATE THREAD
        Thread.Sleep(5000); //works for 5 seconds
    });
    await messageTask;
    //THIS MAY RUN ON A SEPARATE THREAD *
    return $"Current time : {DateTime.Now.ToLongDateString()}";
}
```

Now that our method is `async`, we take advantage of that in the calling `Main` method and do other works concurrently. We simply start multiple works, and then wait for the asynchronous task to complete. Now the complete `Main` method runs in ~5000 millisecond as the three pieces of work run concurrently.

```csharp
//The calling method now benefits from async
static void Main(string[] args)
{
    var sw = new Stopwatch();
    sw.Start();
    var lengthTask = GetMessageLengthAsync(); //start the work
    DoSomeOtherWork(); //start other work as well, 1000 millisecond
    var messageLength = lengthTask.Result; //now wait for task to complete
    sw.Stop();
    Console.WriteLine($"Total Time taken = {sw.ElapsedMilliseconds} ms");
}
```

**Threads:** In the above process, more than one threads get involved. The _main thread_ calls the initial method, and starts all the `async` methods, runs the synchronous methods, and comes back to `Main` method. Whenever we have a separate `Task` being run, or have an `await`, they work on separate threads. Now these _separate threads_ may be same or other than the _main thread_, based on the type of application (e.g. `Console` or `WPF` or `ASP.NET` etc.) and the `ThreadPool`, these could be one or more additional threads (see [previous](/articles/async-await/) post). Since the `Task` runs on a separate thread, we can say _at least_ two threads get involved in the whole process.
{: .notice--info}

###### Almost `async` without `async-await`

Now that we've seen the fully asynchronous code with `async-await` and the benefits of it [1] performance and [2] free threads, can we achieve something similar without the compiler re-wiring our code behind the scenes? That is what happens with [async code](/articles/async-await/).

The answer is, we can have something pretty close with a `task continuation`. The whole intention is to keep working on stuffs rather than wait for each individual piece of work to complete. So, we start the initial work and create a continuation to be run when that completes. We return this continuation `Task` which can be awaited from the calling method.

```csharp
internal static Task<int> GetMessageLength()
{
    Task<string> messageTask = DoTimeTakingWorkAsync();
    var lengthTask = messageTask.ContinueWith((stringTask) =>
    {
        var message = stringTask.Result; //get result from last task
        var length = message.Length; //do some work on result
        return length; //then returns the final result
    });
    DoIndependentWork();
    return lengthTask; //returns the task
}
```

The `Main` method remains the same as in case of `async-await`, and it also performs similarly with ~5000 millisecond. BUT the main differences here are

1. The runtime does not capture the context in continuation, so that runs without context, generally on a separate thread
2. And we have made our code more clumsy compared to the simple syntax of `async-await`. It's still readable, but as we keep doing this, the whole codebase becomes less and less readable

**Note:** Using a blocking call like `task.Result` or `task.Wait()` on an `await`-able task may sometimes lead to deadlock in applications like `WPF`. The reason is [1] there is just one main UI thread that does all the UI update work, and [2] by default `async` methods try to run the continuation on the main thread. So, if the main thread is blocked (waiting for the method to complete) and the method is waiting for it to continue, it may lead to `deadlock`!
{: .notice--danger}

#### All posts in the series - Tasks, Threads, Asynchronous

* **[Synchronous to asynchronous in .NET](/articles/sync-to-async-in-dotnet/)**
* **[Basic task cancellation demo in C#](/articles/task-cancellation/)**
* **[How does Async-Await work - Part I](/articles/async-await/)**
* **How does Async-Await work - Part II**