---
layout: post
title: "Hosting an ASP.NET Core web application on IIS"
excerpt: "A quick guide to deploying your new ASP.NET Core app on existing IIS web server"
date: 2018-09-12
tags: [aspnet, aspnetcore, dotnetcore, web, iis, deployment]
categories: articles
comments: true
share: true
published: true
---

In this post, we'll see how to host your brand new shiny `ASP.NET Core` web application on a Windows `IIS` server, so that it can be used at production level.

We'll not cover anything about .NET Core development in general, or about IIS as a general purpose web server on `Windows` computers or the application publish/deployment process. If you have already creates some ASP.NET Core application and run them locally, and you have hosted .NET Framework web applications on IIS, this post will show you how to quickly host your ASP.NET Core application on IIS. The purpose of existence of this post is, the process is different from that for full .NET Framework applications. Along the way, we'll cover some basic concepts and explain few things in brief.

If you are not looking for hosting your application on Windows IIS, head over to this [official documentation](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/?view=aspnetcore-2.1&tabs=aspnetcore2x) to learn how to host it on `Linux`, `MacOS`, `Docker` or `Azure`. If you want to learn about .NET Core, check out this **[series on .NET Core development](/articles/dotnet-core-2.0/)**.

Just before we jump in, one might be wondering, when ASP.NET Core application already comes with its own web server [Kestrel](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/?view=aspnetcore-2.1&tabs=aspnetcore2x), why would anyone host it on IIS! Kestrel is very lean (and fast), and does only what is required, i.e. serve a HTTP request with a response. On the other hand, mature web servers like `IIS`, `Nginx`, `Apache` etc. have much more functionality built into them. So, it is always advised to use Kestrel behind a full web server as _reverse proxy_, for production use. That means, the web server will receive the request from internet, and forward it to Kestrel after some general preliminary work and send the response back from Kestrel.

![Image](/images/posts/misc/kestrel-to-internet2.png)

With a full web server, viz. IIS, you can do stuffs that are not possible with Kestrel. Some examples are - running multiple apps on same server sharing same port (e.g. mysite.com and othersite.com both listening to HTTP port 80), public domain & SSL certificate management, security & authentication, request limiting, caching, URL re-writing etc.

<hr />

The basic steps to host an ASP.NET Core application on IIS are

1. Publish or package your application
2. Install/Enable the IIS server
3. Install .NET Core Runtime & Hosting Bundle
4. Create a new website for your application
5. Configure

#### [1] Package your application

Build your application. Once successfully built, publish your ASP.NET Core web application. The 2 basic ways to **publish** are

1. Right-click the project in `Visual Studio` and select publish
2. Use the `dotnet CLI` command for [publish](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-publish?tabs=netcore21), i.e. `dotnet publish`

Whichever way you prefer among the two above, there are 2 types of deployment

1. **[FDD](https://docs.microsoft.com/en-us/dotnet/core/deploying/#framework-dependent-deployments-fdd)** or Framework-Dependent Deployment. Here, only your app, NuGet and other related packages & configurations are packed in the published package. It needs the .NET Core runtime to be present on the target machine to run. The published package is platform independent and can run on any supported platform like `Windows`, `Linux` or `MacOS`. Practically, FDD also has options like
    1. **Portable** - totally platform independent (there are some catches though, see later). Total binary size is pretty small, comes with your own DLLs, some .NET Core base types and a few specific to different runtimes
    2. **Runtime specific** - not the ideal framework dependent, and comes with bunch of runtime specific files (that can handle the catch situations of portable option above). Binary size much larger than the portable one
2. **[SCD](https://docs.microsoft.com/en-us/dotnet/core/deploying/#self-contained-deployments-scd)** or Self-Contained Deployment. Here the whole (the required parts) .NET Core runtime and [Kestrel](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel?view=aspnetcore-2.1) web server is packed with your application code. So, you can directly deploy the package on a machine and run, it does not need the .NET Core runtime to be pre-installed on the machine. This is a platform specific deployment, so you must choose the target runtime at the time of publish e.g. `win-x64` or `linux-x64`. The binary size much larger compared those FDD options

**Note:** In .NET Core, you specify a target platform with a `runtime ID`. A runtime ID is nothing but a combination of a specific OS and processor architecture e.g. win-x86, linux-x64 or osx-x64.
{: .notice--info}

**Note:** Actual size of the published binaries will vary based on the type, size & complexity of the project. And they'll also change, probably, with future releases of .NET Core. But, just to give an idea, for one of my not-so-large ASP.NET Core MVC project with .NET Core 2.0, the sizes are - 10 MB (FDD Portable), 40 MB (FDD Win-x64) and 100 MB (SCD Win-x64).
{: .notice--info}

#### [2] Setup IIS web server

Now you need IIS web server up & running on your Windows desktop or server. If you already have a running IIS, skip this step. Then you need to have `ASP.NET Core Module` installed on the system, that comes as part of the _.NET Core Runtime & Hosting Bundle_.

Install the IIS. IIS comes as part of Windows OS, so you basically need to enable them. Follow instructions from [here](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/iis/?view=aspnetcore-2.1&tabs=aspnetcore2x#iis-configuration) and enable it from Windows features in Control Panel. Note that, you'll need to have **Administrator role** to install and configure IIS.

#### [3] Install .NET Core Runtime & Hosting Bundle

Now we'll need the **`ASP.NET Core Module`** configured for IIS, which comes as part of _.NET Core Hosting Bundle_. This enables IIS to serve ASP.NET Core applications. Remember that ASP.NET Core web applications run as separate process with Kestrel web server. IIS needs this module to handle this setup.

1. Next we'll get _.NET Core Hosting Bundle_, which needs _Microsoft Visual C++_ to run. It'll be auto downloaded during installation. But, if you are working on a machine that does not have active internet connection, you first need to install it separately. So, if you have active internet on target machine, skip step 2 & 3. Go to the download the latest version (`x64` or `x86`) of [Microsoft Visual C++ latest Redistributable](https://support.microsoft.com/en-in/help/2977003/the-latest-supported-visual-c-downloads).
2. Install the Microsoft Visual C++ latest Redistributable package downloaded, double click on installer or run from command prompt.
3. Now, we'll install the _.NET Core Hosting Bundle_. Go to [.NET Downloads](https://www.microsoft.com/net/download) page. Click _Download .NET Core Runtime_, under `.NET Core` menu. It'll download the required installers.
4. Install the Microsoft .NET Core Hosting Bundle. To avoid installing the x86 components on a x64 machine, run the installer from command prompt with `OPT_NO_X86` switch e.g.

```yaml
C:\Dir_with_installer> dotnet-hosting-2.1.4-win.exe OPT_NO_X86=1
```

Now, restart the system or restart IIS so that the ASP.NET Core module is recognized by IIS. You can do that without system restart with following commands in admin mode - `net stop was /y` followed by `net start w3svc`. It'll show messages similar to following.

```yaml
c:\SomeDirectory>net stop was /y
The following services are dependent on the Windows Process Activation Service service.
Stopping the Windows Process Activation Service service will also stop these services.

   World Wide Web Publishing Service

The World Wide Web Publishing Service service is stopping.
The World Wide Web Publishing Service service was stopped successfully.

The Windows Process Activation Service service is stopping.
The Windows Process Activation Service service was stopped successfully.

c:\SomeDirectory>net start w3svc
The World Wide Web Publishing Service service is starting.
The World Wide Web Publishing Service service was started successfully.
```

Just in case you thought what is this _.NET Core Hosting Bundle_ that you just installed, the download page says it clearly.

> The .NET Core Runtime & Hosting Bundle contains everything you need to run existing .NET Core apps, including hosting ASP.NET Core apps. The bundle includes the .NET Core runtime, the ASP.NET Core runtime, and if installed on a machine with IIS it will also add the [ASP.NET Core IIS Module](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/aspnet-core-module?view=aspnetcore-2.1).

Now you might be wondering, if you published your application as SCD, why do you need this ASP.NET Core runtime? Shouldn't you package be self-sufficient? The answer is yes, you app can run on its own (with Kestrel on Windows and with Kestrel/other web servers on other OS). But this runtime and module is required by IIS so that it can host your ASP.NET Core application, which was built to host full .NET Framework ASP.NET applications.

#### [4] Create a new website on IIS

If you have hosted any ASP.NET application on IIS before, you already know how to create a new website. If you have not done it before, follow step by step instructions from [here](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/iis/?view=aspnetcore-2.1&tabs=aspnetcore2x#create-the-iis-site).

Here, we'll quickly create a new website on IIS for our application and do so minimal configuration.

1. Open IIS. Search for IIS or run `inetmgr` from run command (Windows + R). Remember you need to be an admin on the system to access IIS.
2. Under the server node on the left, right-click on `Sites` and select `Add Website`. Give the site a name (e.g. mycoreweb.com), use the same name for `host` and select the directory where the app is published as `Physical path`.
3. It'll also create a new `app_pool` with the same name as the website host. You can use the same, select a different one or rename it if you want. It's important you configure your application pool and set _pipeline mode = Integrated_, _**.NET CLR Version = No Managed Code**_. This basically tells the IIS it need not load full .NET Framework CLR, and the application will run on it's own process. This is optional though.

Make sure your app_pool and the website are running, and you can browse your ASP.NET Core application now.

#### [5] Configure

You should already have a running application, but few more configuration will help you manage & troubleshoot your application.

ASP.NET Core applications do not have a `web.config` (to know about configuration in .NET Core applications, read [this](/articles/practical-configuration-and-di-in-aspnet-core/)), but when you publish it for Windows, a web.config gets generated. This is required by IIS and it has some minimal configuration.

It's good you enable **logs**. Then it'll create simple text logs for events like application start-up and errors, comes handy when trying to find problems. To enable logging

1. Create a `logs` directory in the application root i.e. in the published folder that IIS points to. This directory will host the ASP.NET Core Module stdout logs when stdout logging is enabled (see config below)
2. Turn on the logs by changing this flag `stdoutLogEnabled="true"` in web.config

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <handlers>
      <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModule" resourceType="Unspecified" />
    </handlers>
    <aspNetCore processPath="dotnet" arguments=".\Web.Project.dll" stdoutLogEnabled="true" stdoutLogFile=".\logs\stdout" />
  </system.webServer>
</configuration>
```

This is a FDD config, so the `processPath` points to the `dotnet.exe` that'll run the process, and host the main web dll (e.g. Web.Project.dll). On the other hand, for SCD on Windows, it'll create an executable for the main web (e.g. Web.Project.exe) and that'll be the target process.

<hr />

#### Some common problems

[1] If you get **HTTP Error 502.5 - Process Failure** that basically means IIS could not start the process, for which most common reason is - it could not find .NET Core executable (discussed just above). To fix this, the options are

1. Add the .NET Core path to system environment PATH (e.g. C:\ProgramFiles\dotnet)
2. Mention the full path to `dotnet.exe` in (for FDD) or `.\Web.Project.exe` (for SCD) in `aspNetCore processPath` attribute of `web.config`

[2] If you get an error that says **dependencies manifest was not found**, that means, well as it says. Some manifest files are missing that is used by the system. The error might look like this

```yaml
Error:
  An assembly specified in the application dependencies manifest (Web.Project.deps.json) was not found:
    package: 'Microsoft.ApplicationInsights.AspNetCore', version: '2.1.1'
    path: 'lib/netstandard1.6/Microsoft.ApplicationInsights.AspNetCore.dll'
  This assembly was expected to be in the local runtime store as the application was published using the following target manifest files:
    aspnetcore-store-2.0.0-linux-x64.xml;aspnetcore-store-2.0.0-osx-x64.xml;aspnetcore-store-2.0.0-win7-x64.xml;aspnetcore-store-2.0.0-win7-x86.xml
```

This [Stack Overflow post](https://stackoverflow.com/questions/46491957/asp-net-core-2-missing-applicationinsights) and this [GitHub issue](https://github.com/dotnet/coreclr/issues/13542) talks about the problem. To fix this, you have to either of the following three options

1. Install .NET Core full [SDK](https://www.microsoft.com/net/download/archives) (not just the runtime) on the server
2. If it is a Framework Dependent Deployment, publish your app with specific runtime ID (e.g. win-x64 or linux-x64) rather than _"Portable"_
3. Add this flag to your main `web.csproj` and publish

```xml
<PublishWithAspNetCoreTargetManifest>false</PublishWithAspNetCoreTargetManifest>
```

<hr />

#### Accessing the website locally and on network

This is not related to .NET Core, but if you are struggling to access your IIS hosted website on the same machine or over the LAN, you need to set the bindings properly and tweak the `hosts` file.

The hosts file bit is only required if you're trying to access the website with a custom domain name like _mycoolsite.com_. The hosts file works like the first level DNS lookup, so you can setup any domain name to IP mapping here, for local use.

[1] To run the app locally with a custom domain name (e.g. `mycoolsite.com` in our example), add it to the Windows hosts file so that browsers can find it. For that, open any text editor in admin mode, open the file hosts file, and add the custom domain name with localhost IP address i.e. `127.0.0.1` on a new line, and save

```yaml
# in C:\Windows\System32\driver\etc\hosts
127.0.0.1 mycoolsite.com
```

[2] To make it available over LAN, add another binding for your website with the IP address of the hosting machine and a port (check [this](https://stackoverflow.com/questions/21896534/accessing-a-local-website-from-another-computer-inside-the-local-network-in-iis) post for help). So you'll actually have two bindings for your site

```yaml
#For Local:
Type: http
Ip Address: All Unassigned
Port: 80
Host name: mycoolsite.com

#For network LAN:
Type: http
Ip Address: <Network address of the hosting machine ex. 190.168.10.50>
Port: 8080 <make sure this port is allowed to have incoming traffic>
Host name: <Leave it blank>
```

![Image](/images/posts/misc/iis-bindings.png)

Also, make sure that the port that you used (e.g. 8080) is allowed to receive inbound requests and is not blocked by firewall.