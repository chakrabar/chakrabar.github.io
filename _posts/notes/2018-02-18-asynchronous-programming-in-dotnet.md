---
layout: post
title: "Asynchronous programming in .NET"
excerpt: "Code demo of real life configuration & dependency injection in ASP.NET Core 2.0"
date: 2018-02-18
tags: [tech, csharp, dotnet, async, asynchronous]
categories: notes
modified: 2018-02-19T22:11:53-04:00
---

#### [1] Event based async model:

Invoke the async method, that returns void.
Add an event handler to the OnCompleted event.
In the event delegate (another method), capture the CompletedEventArgs.Result

So the main method starts the asynchronous process and leaves the method.
When the process is completed, an event is raised to signal completion.
If an handler is attached to it, that'd be fired and an event completion context would be passed.

```cs
private void DownloadButton_Click(object sender, RoutedEventArgs e)
{
    var client = new WebClient();
    client.DownloadStringAsync(new Uri("http://arghya.xyz/feed.xml"));
    client.DownloadStringCompleted += RssDownloaded;
}

private void RssDownloaded(object sender, DownloadStringCompletedEventArgs e)
{
    DownloadText.Text = e.Result;
}
```