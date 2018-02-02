---
layout: post
title: "Porting .NET libraries to .NET Core (2.0)"
excerpt: ".NET Core 2.0 series - 2/5 - Porting existing libraries to .NET Core"
date: 2018-01-05
tags: [tech, dotnet, csharp, aspnet, dotnetcore, aspnetcore]
categories: articles
share: true
comments: true
modified: 2018-01-28T22:11:53-04:00
---

This article is part of the [.NET Core Series](/articles/dotnet-core-2.0/). Go have a look at the other articles of this series, and run through the previous topics if not done already!

Here we'll talk about porting an existing `.NET Framework` project to `.NET Core`.

Before we look into the process, let's briefly discuss about what, when, why.

#### [1] Which applications can be ported to .NET Core

These are the following type of projects that I've been able to port, with increasing order difficulty. But please note that **portability varies on case to case basis, depending on what APIs and features are being used, what NuGet packages are referred etc.**

1. Class libraries are good candidate, can be ported with lesser troubles
2. Console applications
3. Unit test projects 
4. Web API projects
5. ASP.NET MVC projects
6. EF (with EF Core 2.0, there are some limitations though)

etc.

#### [2] What cannot be ported

1. WCF
2. WPF
3. Xamarin
4. Windows store apps
5. Web-form web applications
6. SingalR ([scheduled for 2.1](https://github.com/aspnet/Home/wiki/Roadmap))

etc.

#### [3] Should you port?

Porting an application to `.NET Core` is great when it

1. Should run cross-platform OR on non-Windows platforms
2. Is portable and deployed to cloud
3. Is light-weight and any performance improvement matters
4. Follows `microservices` pattern
5. Needs to run on containers like `Docker`

<u>But there are few things to consider before you decide to port your happy healthy application</u>

1. Porting _may_ need a huge amount of development effort
2. Testing the same end-to-end may take even more effort and time
3. Some of the existing functionalities may not work any more. Based on what the feature does, it can be a _complete show stopper_
4. If the application will ever run on a Windows system only, porting to core might not give lot of benefits
5. The development team needs to get up-to-date with this new and evolving technology
6. Since it is still evolving, the code might need more changes in future. More tests again.
7. The development setup works best with latest infrastructures, e.g. `VS 2017 v15.3+`. For some teams/individuals, the setup cost can become a bummer!
8. If the application depends on other projects which are not ported to core, it will not help (and if they are from the not supported list [2] like WCF, they cannot be ported)
9. The full `.NET Framework` is not going anywhere, and it'll always be maintained and updated
10. If the only compelling reason is _"It's new and cool, and everyone is talking about it"_, just drop the idea already

#### [4] General changes for porting any project

Though the `C#` code written for old `.NET Framework` works fine with `.NET Core`, the whole project does not work as is with Core. The main reasons are

1. Some of the features are not available in `.NET Core`
2. Some features are available with new APIs (the interface, class, method, parameters etc.)
3. Some functionalities have moved to new `namespace` because of new granular NuGet distribution
4. Some old NuGet packages are not compatible with `.NET Core`

There are actually a lot of changes required to make it work. And, **some of the code might not work at all** as not everything from full framework is supported in Core. In some cases, you'll have to find alternatives, sometimes you may have to drop some functionality completely. So, that pretty much can be a blocker!

If you want to access the portability of your application, you can use the [API Portability Analyzer](https://docs.microsoft.com/en-us/dotnet/standard/portability-analyzer) tool, but I didn't find it very useful!

Looking at what NuGet packages are used - whether their latest version support `.NET Core` is one indicator. Also, while trying to actually port you'll get to know if that porting will work or not.

#### [5] <u>General steps for porting a project to .NET Core</u>

1. The `*.csproj` has changed a lot. So, better create a new project with required framework and version. Then add the code to it.
2. Since the project file does not list the reference `C#` files, just copying them over to the new project folder automatically includes them in the project.
3. Now try to build it. Obviously it'll fail as it does not have the other project & NuGet references. Add them.
4. Add the required project references. Make sure they already are .NET Core projects, else the port will not work.
5. Add the required NuGet packages. Make sure the NuGet package targets `.NET Core` or `.NET Standard`. Sometimes you might have to add multiple packages what used to be part of a single package.
    1. Else look for alternative packages which can do the same job. 
    2. Many 3rd party packages have unofficial .NET Core port, which works fine. 
    3. Sometimes you need to select the "include pre-release" options in NuGet package manager to get latest version that works. Make sure the pre-release is stable.
    4. If none of this works, _you might be stuck !_
6. Because of the NuGet changes, some of the `namespace` would have changed. Find out the new namespace and do a "Replace All" in your project.
7. Some of the APIs (the interface, class, method, parameters etc.) might have changed as well. Make necessary changes in the code so that it complies with the new APIs.
8. `app.config` or `web.config` does not work as they used to. Ideally move all the required settings to `appsettings.json` file or other configuration sources. Make necessary code changes to use settings from the new source.
9. No `AssemblyInfo` class is created by default. If required, add it (e.g. for assembly level settings like "internals visible to")
10. BUILD IT. If everything goes fine, _**you have done it!**_ Do not forget to test the new project with unit tests, manual tests, performance tests etc.
 
#### [6] Tips & Gotchas

1. For class libraries, though targeting `.NET Core` works just fine, it's better to target `.NET Standard` as that'll make the library more reusable across frameworks.
2. Always start with the least dependant project (with no reference to other projects) and the work down the dependency graph as base projects get ported successfully.  
3. If you have a `NUnit` test project, target it to `.NET Core` as the [VS Test Explorer cannot discover](http://hermit.no/net-core-setup/) the tests if the project targets `.NET Standard`! Test projects are simply class libraries.
4. For **NUnit** project to work properly, add the following NuGet packages
- NUnit
- NUnit3TestAdapter
- Microsoft.NET.Test.Sdk
5. There has been [lot of changes](https://msdn.microsoft.com/en-us/magazine/dn890367.aspx) in **Entity Framework Core**, which makes it pretty hard to port an EF project.
- Firstly, it is not recommended to port an EF project to EF Core. Do it, only if really necessary.
- `EntityFramework 6.2.0` works fine with `.NET Core` projects. But that'll not work cross-platform.
- Some features of `EF 6.2` are not available in `EF Core 2.0`. For instance, _"lazy loading"_
- `edmx` is not supported in `EF Core 2.0`. Do a fresh `code-first` migration to create you DB from your models. If you need a `database-first`, use following command to scaffold the entity classes
```bash
$ Scaffold-DbContext "ConnectionString" Microsoft.EntityFrameworkCore.SqlServer -OutputDir ModelsFolder
```
- There is an [existing issue](https://github.com/aspnet/EntityFrameworkCore/issues/10678) which creates problem in generating the names correctly. Until that is fixed in `EF Core 2.1`, there are some [alternative tools](https://stackoverflow.com/questions/48150867/how-to-customise-entity-name-casing-in-ef-core-with-database-first) available for the same purpose.
- Also, by default it does not singularize entity names and pluralize table names at it used to. To fix that, add an instance of `IPluralize` interface to the entry point project, and inject it through `IDesignTimeServices`. A sample implementation below.

```cs
using Humanizer; //needs Humanizer.Core 2.2.0 NuGet package
using Microsoft.EntityFrameworkCore.Design;
using Microsoft.Extensions.DependencyInjection;

namespace MyProject.Web
{
    public class MyDesignTimeServices : IDesignTimeServices
    {
        public void ConfigureDesignTimeServices(IServiceCollection services)
        {
            services.AddSingleton<IPluralizer, MyPluralizer>();
        }
    }

    public class MyPluralizer : IPluralizer
    {
        //used for naming tables
        public string Pluralize(string name) => name?.Singularize(false) ?? name;
        //used for naming entities
        public string Singularize(string name) => name?.Singularize(false) ?? name;
    }
}
```

#### [7] Some common packages

My personal experience with some common packages. While some of them are purely `.NET Core` related, few of them are just generic changes in their latest releases.

* Unity v5.5.5 has changed namespace from `Microsoft.Practices.Unity` to `Unity`. The `[Dependency]` attribute is present in `Unity.Attributes` namespace.
* SharpZipLib has a new pre-release version `1.0.0-alpha2` that is .NET Core compatible
* AutoMapper v6.2.2 has changed it's mapping APIs (since version 4)

```cs
//OLD:
ConfigurationStore myConfigurationStore = new 
    ConfigurationStore(new TypeMapFactory(), MapperRegistry.Mappers);
IMappingEngine myMapper = new MappingEngine(_myConfigurationStore);
                    
myConfigurationStore.CreateMap<Source, Destination>()
    .ForMember(d => d.Prop, opt => opt.MapFrom(s => s.Property))
    .ForMember(d => d.Something, opt => opt.Ignore());

//NEW:
MapperConfigurationExpression myConfigurationExpressions = new MapperConfigurationExpression();
myConfigurationExpressions.CreateMap<Source, Destination>()
    .ForMember(d => d.Prop, opt => opt.MapFrom(s => s.Property))
    .ForMember(d => d.Something, opt => opt.Ignore());
MapperConfiguration mapperConfig = new MapperConfiguration(_myConfigurationExpressions);
IMapper myMapper = new Mapper(mapperConfig);
```

* Service model related basic stuffs are present in `System.ServiceModel.Primitives` NuGet package
* NUnit 3 has [bunch of changes](https://github.com/nunit/docs/wiki/Breaking-Changes), for example

```cs
//this attribute has been dropped from NUnit 3
[ExpectedException] 
[Test]
public void MyTest() { }
//now use
Assert.Throws<Exception>(() => --statement--);
//one sample change in Assert
Assert.IsNullOrEmpty(result); //is removed, use
Assert.That(result, Is.Null.Or.Empty);
```

* In place of `Ionic.Zip` use the native [System.IO.Compression](https://docs.microsoft.com/en-us/dotnet/standard/io/how-to-compress-and-extract-files)
* Use `EPPlus.Core 1.5.4` for working with Excel as original/official EPPlus doesn't support .NET Core yet
* `System.Runtime.Caching` is not supported in .NET Core yet. Change to [Microsoft.Extensions.Caching.Memory](https://docs.microsoft.com/en-us/aspnet/core/performance/caching/memory) v2.0.0 NuGet package. Register memory cache `.AddMemoryCache()` in `Startup`, and get `IMemoryCache` injected in client classes.
* For many framework NuGets, you have to find the new (sometimes multiple) package(s)

And many more small changes...

----

This article covered the process of porting existing `.NET Framework` class libraries to `.NET Core` or `.NET Standard`. Continue to [Understanding ASP.NET Core 2.0](/articles/understanding-aspnet-core-2/). 