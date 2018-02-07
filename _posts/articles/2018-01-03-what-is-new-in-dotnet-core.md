---
layout: post
title: "What is .NET Core (2.0)"
excerpt: ".NET Core 2.0 series - 1/5 - What is .NET Core"
date: 2018-01-03
tags: [tech, dotnet, csharp, aspnet, dotnetcore, aspnetcore]
categories: articles
share: true
comments: true
modified: 2018-02-06T16:10:53+05:30
---

This article is part of the [.NET Core Series](/articles/dotnet-core-2.0/). Go have a look at the other articles of this series.

For those who are not familiar with the **.NET Framework**, it is an application development framework built by Microsoft for building almost any kind of application, ranging from Command line applications, Desktop applications, Web applications, Mobile application (Xamarin), Web services to automation and reporting tools etc. [.NET framework](https://www.microsoft.com/net/download/dotnet-framework-runtime) comes with multiple language support of `C#`, `VB`, `F#` and many more. It also comes with very powerful IDE [Visual Studio](https://www.visualstudio.com/), to develop, build, test & deploy applications. .NET Framework works fully on Windows systems, and with limited capabilities on some other systems with help of the `Mono` framework.

`.NET Core` is the new (not a replacement) version of `.NET` for the modern world. Though it follows many concepts & patterns of _classic_ .NET, it should rather be treated as a completely new & independent application development platform.

#### [1] What is .NET Core

`.NET Core` is a **new**, **portable**, **optimized**, **cross-platform** version of `.NET Framework` written from scratch. It's
- Open source with all the source code available on [GitHub](https://github.com/dotnet)
- Cross platform (Works on `Windows`, `Linux` & `MacOS`)
- Portable & light-weight (the Core runtime CoreCLR and base class library CoreFx are very lean, and all the parts/components are available as separate `NuGet` packages)
- Cloud optimized (along with above point, optimized in terms of resource usage)
- Comes with cross platform `dotnet cli` - the [command line tool](https://docs.microsoft.com/en-us/dotnet/core/tools/?tabs=netcore2x) to create, scaffold, build, run, publish .NET Core applications

Unlike the traditional full `.NET Framework` which is very heavy, and supports all different types of applications (e.g. Web, Console, WPF, WCF, Windows Store etc.),
- `.NET Core` has only `CoreCLR` & `CoreFx` (very basic set of types, which is common for all application types, and is cross-platform).
- All components, including	these two basic building blocks - all are `NuGet` packages.

> ##### .NET Core currently supports - Console, ASP.NET, Web API (unified), EF
> ##### .NET Core does NOT support - WPF, WCF, Xamarin, Windows phone/store apps, WebForm, Silverlight etc.

#### [2] Application Building/Hosting options

To create a new .NET Core application
* Use one of the `.NET Core` project templates in VS (make sure target framework is `.NET Core` or `.NET Standard`)
* Use dotnet cli to scaffold a project e.g. `$ dotnet new console`

A .NET Core application can be [deployed in 2 ways](https://docs.microsoft.com/en-us/dotnet/core/deploying/index)

* **Portable** - [FDD or framework dependent deployment] very small & light-weight, depends on .net framework installed on the machine (e.g. C:\program files\dotnet)
  * General publish from VS
  * $ dotnet publish -f netcoreapp2.0 -c release
* **Self-contained** - [SCD or self-contained deployment] Stand-alone, includes the necessary parts of CoreCLR & CoreFx, and other NuGet packages.
  * VS publish with specific RID (runtime identifier)
  * $ dotnet publish -f netcoreapp2.0 -c release -r win81-x64

.NET Core applications can target multiple frameworks/versions and multiple runtimes (hosting environments). To enable, mention targets frameworks & RIDs in *.csproj or in dotnet cli commands.
* In `csproj`

```xml
<PropertyGroup>
    <TargetFrameworks>netcoreapp2.0;net462</TargetFrameworks>
    <RuntimeIdentifiers>win81-x64;osx.10.11-x64;ubuntu.14.04-x64</RuntimeIdentifiers>
</PropertyGroup>
```

* With `dotnet cli`

```bash
$ dotnet publish -f netcoreapp1.6 -c release -r win10-x64
```

#### [3] Framework choices

.NET Core comes with a bunch of choices for target framework.

1. The Full framework aka `.NET Framework`. It has most types and options, but runs only on Windows. It's just the same old code. Exe runs on Windows
2. `Mono` also has most types & options, and runs cross-platform. Same old code if already using Mono. 
  - Runs on Mono CLI (command line)
  - both of above two needs machine wide installation of the framework
3. `.NET Core` has limited types, but runs cross-platform. Code structure has changes from the old conventions. Also, there is choice for machine wide or app-only framework. 
	- Runs on .NET Core CLI (command line interface)
		- Executes app by running the IL from the assemblies
		- Hosts the CLR or runtime
		- Finds entry point `Main()` and invokes
		- Also has SDK support (e.g. installing Nuget)
	- The main difference with Mono is 
		- .Net Core is newly written from scratch, it has new way of doing things (e.g. ASP.NET Core)
		- It's lightweight, much more optimised
		- It's best choice for deploying to cloud, containers etc.
		- It comes with option of self-contained application (CoreCLR, CoreFX and other NuGet packages packed with published package)
    - Honsetly, though Mono is fully supported, Microsoft will be concentrating mostly on the Core
4. `.NET Standard` (framework independent) is cross-platform and best for code sharing, but works only for class libraries!
	- It's not a specific framework, but a standard specification. Based on selected framework standard version, it can target multiple .net frameworks.

#### [4] The .NET Standard

So, now there are different sides/flavours of the framework - core (e.g. v1.0), full(e.g. v4.6.1), mono, windows app, xamarin, silverlight etc. 
So how to create reusable modules? Options are

1. Portable Class Libraries (PCL) - class library assemblies where you can pre-select "profile" for set of target platform/framework version(s)
  - So, you can basically check checkboxes to select what all platforms you want to target
  - With more platforms selected, you get lesser supported API surface
2. **.NET STANDARD** - (functionally replaces PCLs)

> a (versioned) specification that standardizes which APIs a .NET framework has to implement in each version.

  - [.NET Standard FAQ](https://github.com/dotnet/standard/blob/master/docs/faq.md)
  - [.NET Standard versions](https://github.com/dotnet/standard/blob/master/docs/versions.md)
  - Specific implementation of .Net Framework (like, .Net 4.6.1 OR .Net Core 1.0) targets specific .Net Standard (e.g. .Net Standard 1.6)
  - .Net Standard is always backward compatible. Higher versions have more APIs, but no breaking change. 
  - With higher version .Net Standard, you get more APIs but less platforms (runtimes) that are supported. So, **try to target lowest .NET Standard version that works**. 
  - Latest (as of January 2018) is `.NET Standard 2.0` which is supported by `.NET Framework 4.6.1`, `.NET Core 2.0`, `Mono 5.4` etc.
  - On initial releases, first platform specific frameworks were created, later a .Net Standard version was created taking most usable subset of all the frameworks! But going forward, the order might be reveresed.
  - Practically, **only for class libraries** .Net Standard can be targeted (which makes complete sense as those are supposed be reused across applications, runtimes & frameworks)

<u>So, how does it work actually?</u>

When `.NET Standard 2.0` is tergeted in any class library, it basically adds the NuGet package `NetStandard.Library` v2.0 . That comes with and refers to the `netstandard.dll` which has all the compatible base classes (kind of BCL) defined. These types in this dll are _more of contract or API definitions rather than real implementation_. When running, it internally does "type forwarding" to a compatible implementation in any of the available framework version.

See the videos [.NET Standard intro](https://www.youtube.com/watch?v=YI4MurjfMn8) and [Under the hood of .NET Standard](https://www.youtube.com/watch?v=vg6nR7hS2lI) for better understanding.

**Some key points about .NET Standard 2.0**

1. To run a class library with specific .NET Standard version, a framework is needed that supports that standard version. For example, to run .NET Standard 2.0 library, either .NET Framework 4.6.1 or .NET Core 2.0 is required.
2. Because of backward compatibility, a .NET Standard 2.0 class library can refer to a project or package that targets a lower version of standard, e.g. .NET Standard 1.3 ; that'll just work fine.
3. .NET Standard 2.0 comes with a `Compatibility Shim`, which allows to refer packages which are not complianat with .NET Standard, but targets .NET Framework 4.6.1 . This way the project will build fine (with warning, see below). But whether it'll actually run depends on what APIs the package is using. If the used APIs are part of .NET Standard, it'll run all fine.
4. In cases like this, you get a warning. It warns you about the compatibility issue stated above.

> Warning NU1701: Package 'Xyz v1.2.3' was restored using '.NETFramework,Version=v4.6.1' instead of the project target framework '.NETStandard,Version=v2.0'. This package may not be fully compatible with your project.

It's okay to leave the warning as a remider of future implications. Or it can be supressed by adding `NoWarn` to the specific package like (in .csproj)

```xml
<PackageReference 
  Include="Microsoft.AspNet.WebApi.Client" Version="5.2.2" 
  NoWarn="NU1701" />
```

The `Compat Shim` does the magic through internal type forwarding between .NET Standard and .NET Framework DLLs. 

<figure>
	<img src="/images/posts/NetStandardWithNetFrameworkLibrary.png" alt=".NET Standard with Framework lib">
	<figcaption>
    <a href="https://www.youtube.com/watch?v=vg6nR7hS2lI" title=".NET Standard library referencing .NET Framework library via CompactShim" target="_blank">.NET Standard library referencing .NET Framework library via CompatShim</a>.
  </figcaption>
</figure>


#### [5] Configuration & Settings

- Like older .NET, still the configuration is key-value based 
- Unlike older .NET, there is no `app.config` and there is no predefined config sections like `appSettings`. Configuration settings in .NET Core is much more flexible.
- Source of configuration can be any `XML`, `JSON`, `in-memory`, `command line args`, `INI` file or environment variables
- Configuration source is defined in Program.cs, and registered with DI. Then the configuration values can be used across the application
- See the **[practical configuration & DI](/articles/practical-configuration-and-di-in-aspnet-core/)** article to see real life code demo on how to use configuration settings in .NET Core


#### [6] .NET Core Deployment options

1. Manual deployment options for .NET Core 
    1. Copy whole solution, use .NET Core CLI to restore packages and run
    2. Build the project, move the assemblies and run - packages needs to be managed manually [build]
    3. Create NuGet package and share [pack]
    4. Publish - all packages will be published together, even the Core CLR if self-contained is used 
	
2. Publish from Visual Studio
  - Can use multple profiles for publishing with multiple framework/runtimes
  - Full framework creates a `.exe` for executable projects. For `ASP.NET Core` this can be run to start the web through `KESTREL` web server
  - Core creates just a dll which can be run through Core CLI to start the web `$ dotnet MyApp.dll`
		
3. Publishing self-contained apps:
  - Runtimes (RID) must be specified, because part of the CoreCLR will run natively (and whatever part of the CLR is not managed, will need a native version) - so will need a runtime specific version
  - Has to be published for each runtime separately
  - If no RID is specified, it uses RID of current machine 
  - All core framework and other NuGet packages are added to publish folder

4. To run on a platform which is not natively supported (e.g. `linux-arm` or `raspbian`)
  - publish as `platform` with `.NET Framework` and copy over the files
  - Install full framework OR mono (which is full equivalent) on target machine
  - For ASP.NET applications, install `libuv` on target machine 
  - Start with the specific CLI e.g. `$ mono MyApp.dll`

----

This article covered the high level basic of the `.NET Core` framework. Continue to [Porting existing .NET Framework libraries to .NET Core](/articles/porting-existing-libraries-to-dotnet-core/).