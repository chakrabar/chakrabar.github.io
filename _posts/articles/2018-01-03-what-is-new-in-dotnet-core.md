---
layout: post
title: "What is .NET Core (2.0)"
excerpt: ".NET Core 2.0 series 1/5 - What is .NET Core"
date: 2018-01-03
tags: [tech, dotnet, csharp, aspnet, dotnetcore]
categories: articles
share: true
modified: 2018-01-28T22:11:53-04:00
---

This article is part of the [.NET Core Series](/articles/dotnet-core-2.0/). Go have a look at the other articles of this series.

#### [1] What is .NET Core

`.NET Core` is a **new**, **portable**, **cross-platform**, **optimized** version of `.NET Framework` written from scratch. It's
- Open source all the source code is in [GitHub](https://github.com/dotnet)
- Cross platform (Works on `Windows`, `Linux` & `Mac OS`)
- Portable & light-weight (CoreCLR & CoreFx are very lean, and all parts/components are available as separate `NuGet` packages)
- Cloud optimized (along with above point, optimized in terms of resource usage)
- Comes with cross platform `dotnet cli` - the [command line tool](https://docs.microsoft.com/en-us/dotnet/core/tools/?tabs=netcore2x) to build, run, publish, scaffold .NET Core applications

Unlike traditional full `.NET Framework` which is very heavy, and supports all different types of applications (e.g. Web, Console, WPF, WCF, Windows Store etc.),
- `.NET Core` has only `CoreCLR` & `CoreFx` (very basic set of types, which is common for all application types, and is cross-platform).
- All components, including	these two basic building blocks - all are `NuGet` packages.

> ##### .NET Core currently supports - Console, ASP.NET, Web API (unified), SignalR, EF
> ##### .NET Core does NOT support - WPF, WCF, Xamarin, Windows phone/store apps, Silverlight etc.

#### [2] Application Building/Hosting options

To create a new .NET Core application
* Use one of the `.NET Core` project templates in VS (make sure target framework is `.NET Core` or `.NET Standard`)
* Use dotnet cli to scaffold a project e.g. `dotnet new console`

A .NET Core application can be [deployed in 2 ways](https://docs.microsoft.com/en-us/dotnet/core/deploying/index)

* **Portable** - [FDD or framework dependent deployment] very small & light-weight, depends on .net framework installed on the machine (e.g. C:\program files\dotnet)
  * General publish from VS
  * dotnet publish -f netcoreapp2.0 -c Release
* **Self-contained** - [SCD or self-contained deployment] Stand-alone, includes the necessary parts of CoreCLR & CoreFx, and other NuGet packages.
  * VS publish with specific RID (runtime identifier)
  * dotnet publish -f netcoreapp2.0 -c Release -r win81-x64

.NET Core applications can target multiple frameworks/versions and multiple runtimes (hosting environments). To enable, mention targets frameworks & RIDs in *.csproj or in dotnet cli commands.
* `csproj`

```xml
<PropertyGroup>
    <TargetFrameworks>netcoreapp2.0,net462</TargetFrameworks>
    <RuntimeIdentifiers>win81-x64;osx.10.11-x64;ubuntu.14.04-x64</RuntimeIdentifiers>
</PropertyGroup>
```

* `dotnet cli`

```bash
dotnet publish -f netcoreapp1.6 -c Release -r win10-x64
```

#### [3] Framework choices

1. The Full framework aka `.NET Framework`. It has most types and options, but runs only on Windows. It's just the same old code. >> Exe runs on Windows (OS)
2. `Mono` also has most types & options, and runs cross-platform. Same old code if already using Mono. 
  - Runs on Mono CLI (command line)
  - both of above two needs machine wide installation of the framework
3. `.NET Core` has limited types, but runs cross-platform. Code has changes from old. Also, there is choice for machine wide or app-only framework. 
	- Runs on .NET Core CLI (command line interface)
		- executes app by running the IL 
		- hosts the CLR
		- Finds entry point (Main()) and invokes
		- also has SDK support (e.g. installing Nuget)
	- The main difference with Mono is 
		- .Net Core is newly written from scratch, it has new way of doing things (e.g. ASP.NET Core)
		- It's lightweight, much more optimised
		- It's best choice for deploying to cloud, containers etc.
		- It comes with option of self-contained application (CoreCLR & CoreFX packed with published package)
4. `.NET Standard` (framework independent) is cross-platform and best for code sharing, but works only for class libraries!
	- It's not a specific framework, but a standard specification. Based on selected framework standard version, it can target multiple .net frameworks.

#### [4] The .NET Standard

So, now there are different sides/directions of framework - core (1.0), full(4.6.1), mono, windows app, xamarin, silverlight etc. 
So how to create reusable modules?
1. Portable Class Libraries (PCL) - class library assemblies where you can pre-select set of target platform/framework version(s)
  - So, you can basically check checkboxes to select what all platforms you want to target
  - With more platforms selected, you get lesser supported API surface
2. **.NET STANDARD** - (functionally replaces PCLs)
> a (versioned) specification that standardizes which APIs a .NET platform has to implement in each version.

  - [.NET Standard FAQ](https://github.com/dotnet/standard/blob/master/docs/faq.md)
  - [.NET Standard versions](https://github.com/dotnet/standard/blob/master/docs/versions.md)
  - Specific implementation of .Net Framework (like, .Net 4.6.1 OR .Net Core 1.0) targets specific .Net Standard (targets .Net Standard 1.0-1.6)
  - .Net Standard is (till now, January 2018) always backward compatible. Higher versions have more APIs, but no breaking change. 
  - With higher version .Net Standard, you get more APIs but less platform (runtime) support. So, **try to target lowest .NET Standard version that works**. 
  - Latest (till now, January 2018) is `.NET Standard 2.0` which is supported by `.NET Framework 4.6.1` AND `.NET Core 2.0`
  - **First platform specific frameworks are created, later .Net Standard is created taking most common/available subset of the frameworks**
  - Practically, ONLY when creating class libraries, .Net Standard can be targeted (which makes complete sense as that can be reused across applications, frameworks)


#### [5] Configuration & Settings

- Like older ASP.NET, still the configuration is key-value based 
- Configuration source can be `XML`, `JSON`, `in-memory`, `command line args`, INI or environment variables
- Custom sources can be created by deriving from `ConfigurationStore` class (may need to add Microsoft.Extensions.Configuration assembly)
- e.g. to create a custom JSON configuration file
  - Create a json file add populate data as simple property and value in json object format (can create complex object as well)
  - Create a custom class that corresponds to the json structure
  - Add code to read this config, in Startup.cs (add all requirde packages to project.json)
  - Setup dependency injection to get those settings as object 
  - Use multiple `AddJsonFile()` option with {EnvironmentName} to optionally use (priority/override) environment specific config file 
  - Add configuration json file to publishOptions:include , so that it is published to out directory
  - Add AddOptions() middleware
  - This can be injected as IOptions<CustomClass>
- Basically any json file can be used, and values can be read as json object properties e.g. 
`var outPath = Configuration["output:directory"];`

#### [6] .NET Core Deployment options

Deployment options for ASP.NET Core Apps:
1. Copy whole solution, use .NET Core CLI to restore packages and run
2. Build the project, move the assemblies and run - packages needs to be managed manually [build]
3. Create NuGet package and share [pack]
4. Publish - all packages will be published together, even the Core CLR if self-contained is used 
	
Publish from Visual Studio
- Can use multple profiles for publishing with multiple framework/runtimes
- Full framework creates a `.exe` for executable projects. For `ASP.NET Core` this can be run to start the web through `KESTREL` web server
- Core creates just a dll which can be run through Core CLI to start the web `dotnet MyApp.dll`
		
Publishing self-contained apps:
- Runtimes (RID) must be specified, as part of the CoreCLR will run natively (whatever part of CLR is not managed) - so will need a runtime specific version
- Has to be published for each runtime separately
- If no RID is specified, it uses RID of current machine 
- All core framework NuGet packages are added to publish folder

To run on a platform which is not natively supported (e.g. `linux-arm` or `raspbian`)
- publish as `platform` with `.NET Framework` and copy over
- install full framework OR mono (which is full equivalent) on target machine
- For ASP.NET applications, install `libuv` on target machine 
- Start with the specific CLI e.g. `> mono MyApp.dll`


This article covered the high level basic of the `.NET Core` framework. Continue to [Porting existing .NET Framework libraries to .NET Core](/articles/porting-existing-libraries-to-dotnet-core/).