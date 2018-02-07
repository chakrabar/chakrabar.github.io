---
layout: post
title: "Practical configuration & DI in ASP.NET Core 2.0"
excerpt: "Code demo of real life configuration & dependency injection in ASP.NET Core 2.0"
date: 2018-01-22
tags: [tech, mvc, csharp, dotnetcore, aspnetcore, configuration, di]
categories: articles
share: true
comments: true
modified: 2018-02-05T22:11:53-04:00
---

This article is an extension to the [.NET Core Series](/articles/dotnet-core-2.0/). Go have a look at the articles of this series, and run through the previous topics if not done already!

In this article, we'll look at some real life code to see how configuration and DI are used in `.NET Core 2.0` code.

#### [1] Service Registration in .NET Core

`.NET Core` comes with in-built dependency injection (DI). Though practically optional, conventionally it is expected to build the whole application based on DI. That means, register all your dependency to the .NET Core `IoC container` at application startup and get them injected in the client code when required.

The `Startup.cs` class has a `ConfigureServices` method where all the services are registered. This method is called by the `Main()` method and it passes the `IServiceCollection` to the method, which is used to register services.

- Choice of service lifetimes
  - **Transient**: Creted each time they are requested
  - **Scoped**: Once per http request
  - **Singleton**: One per lifetime of application

Following code shows standard way of registering services in .NET Core 2.0

```cs
//Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<MyDbContext>(opt => opt.UseInMemoryDatabase("MyDbName"));
    services.AddMvc(); //inject all services related to MVC
    services.AddScoped<IRepository, Repository>();
    services.AddTransient<ISomeService, SomeService>();
    services.AddSingleton<IConfigBuilder, FileConfigBuilder>();
}
```

#### [2] Dependency Injection in .NET Core

The standard and ideal way to get a service instance (any dependency type that has been registered in above method) injected in a class is to use `Constructor Injection`. In simple words, add the dependencies as parameter to the constructor of the class, the DI framework will inject the actual instances as arguments at runtime.

_**Note**: The DI framework can only inject constructor dependencies when the class is instantiated through DI, if a new instance is created directly with `new` or `Reflection`, DI will play no role in the process!_

Following code shows simple constructor injection in ASP.NET Core controller.

```cs
public class TestController : Controller
{
    ISomeService _someService;

    //constructor injection
    public TestController(ISomeService service)
    {
        _someService = service;
    }

    public IActionResult Get()
    {
        //use injected service
        var result = _someService.SomeMethod();
        return new OkObjectResult(result);
    }
}
```

#### [3] Injecting service without constructor injection

Constructor injection is the ideal way to inject dependency in a class as it is conventional, very descriptive and readable and also it makes the intention very clear that _the code will not work if those dependencies are not provided_.

But there can be cases, where one might need to get a service instance by explicit injection. This can be achieved by asking for an instance directly from `IServiceProvider`. This IServiceProvider itself can be injected through dependency injection. See code below

```cs
public class TestController : Controller
{
    IServiceProvider _serviceProvider;

    //constructor injection of IServiceProvider
    public TestController(IServiceProvider sp)
    {
        _serviceProvider = sp;
    }

    public IActionResult Get()
    {
        //get service instance from IServiceProvider
        ISomeService service = _serviceProvider.GetService(typeof(ISomeService));
        var result = service.SomeMethod();
        return new OkObjectResult(result);
    }
}
```

#### [4] Basic Configuration setup in .NET Core 2.0

- Like older .NET, still the configuration is key-value based 
- Unlike older .NET, there is no `app.config` and there is no predefined config sections like `appSettings`. Configuration settings in .NET Core is much more flexible.
- Source of configuration can be any `XML`, `JSON`, `in-memory`, `command line args`, `INI` file or environment variables
- Configuration source is defined in Program.cs, and registered with DI. Then the configuration values can be used across the application

Older .NET Core

```cs
//Program.cs
public static IWebHost BuildWebHost()
{
    return new WebHostBuilder()
        .UseKestrel()
        .UseContentRoot(Directory.GetCurrentDirectory())
        .ConfigureAppConfiguration((builderContext, config) =>
        {
            IHostingEnvironment env = builderContext.HostingEnvironment;

            config.AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true, reloadOnChange: true);
        })
        .UseStartup<Startup>()
        .Build();
}
```

.NET Core 2.0

```cs
//Program.cs
public static IWebHost BuildWebHost(string[] args) =>
    WebHost.CreateDefaultBuilder(args) //adds appsettings.json & appsettings.env.json
        .UseStartup<Startup>() // Invokes ConfigureServices & Configure on Startup
        .Build();
```

Sample default `appsettings.json` from ASP.NET Core 2.0 project

```javascript
{
  "Logging": {
    "IncludeScopes": false,
    "Debug": {
      "LogLevel": {
        "Default": "Warning"
      }
    },
    "Console": {
      "LogLevel": {
        "Default": "Warning"
      }
    }
  }
}
```

Reading **string value** from appsettings file with `IConfiguration`

```cs
using Microsoft.Extensions.Configuration;

public class TestController : Controller
{
    IConfiguration _configStore;

    public TestController(IConfiguration configuration,)
    {
        _configStore = configuration;
    }

    public IActionResult Get()
    {
        //get config value with IConfiguration
        var logLevel = _configStore["Logging:Debug:LogLevel:Default"];
        return new OkObjectResult($"Log level: {logLevel}");
    }
}
```

#### [5] Adding additional configuration source

```cs
//Program.cs
//using Microsoft.Extensions.Configuration;
public static IWebHost BuildWebHost(string[] args)
{
    WebHost.CreateDefaultBuilder(args)
        .ConfigureAppConfiguration((builderContext, config) =>
        {
            config.AddXmlFile("AppData/AppConfigs.xml", 
                optional: true, reloadOnChange: true);
        })
        .UseStartup<Startup>()
        .Build();
}
```

Sample `AppConfigs.xml` configuration XML

```xml
<?xml version="1.0" encoding="utf-8" ?>
<AppConfigs>
  <EnableGuestAccess>true</EnableGuestAccess>
  <CustomLogHeader>My Cool Log Header</CustomLogHeader>
  <DeleteLogsAfterDays>45</DeleteLogsAfterDays>
</AppConfigs>
```

#### [6] Strongly typed configuration

```cs
public class AppConfigs
{
    public bool EnableGuestAccess { get; set; }
    public string CustomLogHeader { get; set; }
    public int DeleteLogsAfterDays { get; set; }
}
```

Read a whole configuration file or a section of it (Json/XML) into a compatible onject instance. Here, it is important that the structure of the configuration section matches the type of the object.

```cs
public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public IConfiguration Configuration { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        //Read compatible config file (AppConfigs.xml) into AppConfigs instance
        //Here compatible means a matching structure with Type AppConfigs
        services.Configure<AppConfigs>(Configuration);
        //OR read only a section of config file into a compatible onject
        services.Configure<LogSetup>(Configuration.GetSection("Logging"));
        //Here it'll read from appsettings.json as that has section "Logging"
    }
}
```

#### [7] IOptions injection for configuration instance

The configuration registered in above code can be injected with an IOptions wrapper of the created instance type e.g. `IOptions<AppConfigs>` for our code sample

```cs
public class TestController : Controller
{
    AppConfigs _appConfigs;

    public TestController(IOptions<AppConfigs> configOptions)
    {
        //Use value of injected IOptions wrapper
        _appConfigs = configOptions.Value;
    }

    public IActionResult Get()
    {
        //use the typed config like any other object
        var logHeader = _appConfigs.CustomLogHeader;

        return new OkObjectResult($"Log header: {logHeader}");
    }
}
```

#### [8] Injection of strongly typed configuration without IOptions

```cs
public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public IConfiguration Configuration { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        var appConfigs = new AppConfigs();
        Configuration.Bind(appConfigs);
        //Register the config object as singleton
        services.AddSingleton(appConfigs);
    }
}
```

Inject that singleton object directly by type

```cs
public class TestController : Controller
{
    AppConfigs _appConfigs;

    //directly inject the singleton config object
    public TestController(AppConfigs appConfigs)
    {
        _appConfigs = appConfigs;
    }

    public IActionResult Get()
    {
        //use the typed config like any other object
        var logHeader = _appConfigs.CustomLogHeader;

        return new OkObjectResult($"Log header: {logHeader}");
    }
}
```

One good [read](https://joonasw.net/view/aspnet-core-2-configuration-changes).
