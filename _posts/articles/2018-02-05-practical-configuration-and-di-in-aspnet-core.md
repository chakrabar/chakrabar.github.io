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

#### [1] DI - Service Registration in .NET Core

`.NET Core` comes with in-built dependency injection (DI). Though practically optional, conventionally it is expected to build the whole application based on DI. That means, different modules and layers to not depend on each other directly, rather connect via some abstraction (good ol' Dependency Inversion Principle). Talking of `code` you register all your dependency to the .NET Core `IoC container` at application startup and get them injected in the client code when required.

The `Startup.cs` class has a `ConfigureServices` method where all the services are registered. This method is called by the `Main()` method and it passes the `IServiceCollection` to the method, which is used to register services. A service can be registered with 3 different type of scopes or lifetimes. 

- Choice of service lifetimes
  - **Transient**: Creted each time they are requested
  - **Scoped**: Once per http request
  - **Singleton**: One per lifetime of application

Following code shows the standard ways of registering services in .NET Core 2.0

```cs
//Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    //with framework provided extension methods
    services.AddDbContext<MyDbContext>(opt => opt.UseInMemoryDatabase("MyDbName"));
    services.AddMvc(); //inject all services related to MVC
    //simple scoped service registration
    services.AddScoped<IRepository, Repository>();
    services.AddTransient<ISomeService, SomeService>();
    services.AddSingleton<IConfigBuilder, FileConfigBuilder>();
    //any type that needs injection/to be injected
    services.AddScoped<JustAnotherEntity>();
    //instance registration example, sp is IServiceProvider
    services.AddTransient<MyService>(sp => new MyService(
        "Some string argument",
        sp.GetService<ISomeService>()));
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

Constructor injection is the ideal way to inject dependency in a class as it is conventional, very descriptive and readable. Also it makes the intention very clear that _the code will not function if those dependencies are not provided_.

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

I can think of two reasons for using this

1. To create service instances conditionally at runtime
2. To minimize the parameters of constructor. Only `IServiceProvider` can be injected in constructor, then later the actual services can be instantiated as and when necessary.

**<u>Service instance injection inside the ConfigureServives method</u>**

In some occasions, it might be required to get a service instance injected inside the `ConfigureServices` method itself! The way to do that is again using `IServiceProvider`. The `IServiceCollection` available to the method has an extension to create an `IServiceProvider` instance. See below

```cs
//Startup.ConfigureServices()
//using Microsoft.Extensions.DependencyInjection;
var someService = services.BuildServiceProvider()
    .GetService<ISomeService>();
```

**Note:**
1. Before using this, the `ISomeService` and it's dependecy graph must be registered
2. For the `Configure` method, any service that has been registered, can be directly injected as method parameters

#### [4] Dependency Injection in Filters

Let's say I have an `ActionFilter` like this, which simply adds a custom header to the response

```cs
public class MyHeaderFilterAttribute : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        context.HttpContext.Response.Headers.Add("MyHeader", "HeaderValue");
    }
}
```

This can be used as an attribute in `controller` or `action` method

```cs
[MyHeaderFilter]
public ActionResult Get()
{
    return new OkObjectResult(new { Id = 123, Name = "Hero" });
}
```

Now, if I have a constructor dependency in the filter, it cannot be used simply as an attribute. The dependency also cannot be injected.

```cs
public class MyHeaderFilterAttribute : ActionFilterAttribute
{
    ISomeService _service;

    public MyHeaderFilterAttribute(ISomeService someService)
    {
        _service = someService;
    }

    public override void OnActionExecuting(ActionExecutingContext context)
    {
        _service.DoStuff();
        context.HttpContext.Response.Headers.Add("MyHeader", "HeaderValue");
    }
}
```

Dependency injection to `Filter` in `ASP.NET` is little tricky. As filters cannot be used directly as attributes anymore, we need to get some work-arounds. The good news is the framework provides us some work-arounds.
There are basically 3 ways to handle this.

**<u>As ServiceFilter</u>**

For this, 
- register the filter with container in `Startup`
- use `ServiceFilter` attribute with type of desired filter that needs service injection

```cs
//In Startup.ConfigureServices()
services.AddScoped<MyHeaderFilterAttribute>();

//In controller
[ServiceFilter(typeof(MyHeaderFilterAttribute))]
public ActionResult GetWithStatus()
{
    return new OkObjectResult(new { Id = 123, Name = "Hero" });
}
```

**<u>As TypeFilter</u>**

For this, 
- registration of the filter is NOT REQUIRED
- use `TypeFilter` attribute with type of desired filter that needs service injection
- this doesn't use the DI container directly, rather it internally uses frameworks's `ObjectFactory` to inject the instance. More details [here](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/filters)
- with TypeFilter, additional constructor arguments can also be passed along with injection

```cs
[TypeFilter(typeof(MyHeaderFilterAttribute))]
public ActionResult GetWithStatus()
{
    return new OkObjectResult(new { Id = 123, Name = "Hero" });
}
```

**<u>With TypeFilter wrapper type</u>**

Though the above two way works, they are little cumbersome and the syntax is not clean. To just use the filter directly as attribute, without DI registration we can wrap the actual filter as a `TypeFilter` instance. Then it can be used directly as attribute.Note, this does not allow constructor parameters.

We'll create a simple `ExceptionFilter` to log exceptions, which has got a dependency on some service.

```cs
public class LogErrorAttribute : TypeFilterAttribute
{
    public LogErrorAttribute() 
        : base(typeof(LogErrorFilterImplementation))
    {
    }

    public class LogErrorFilterImplementation : ExceptionFilterAttribute
    {
        private static ISomeService _service;

        public LogErrorFilterImplementation(ISomeService someService)
        {
            _service = someService;
        }

        public override void OnException(ExceptionContext context)
        {
            if (context.Exception != null)
            {
                //do something with injected service
                _service.DoStuff();
                //log the exception
            }
        }
    }
}
```

This can be used just as an attribute without any registration.

```cs
//In controller
[LogError]
public JsonResult Get()
{
    //do some processing
    return Json(new Item { Id = 123, Name = "Hero" });
}
```

Pretty neat I'd say :)

#### [5] Dependency injection with HttpContext

When `HttpContext` is built, it gets it's own copy of `IServiceProvider` as `RequestServices`. So whoever has access to a valid `HttpContext` like a `controller` or `filter`, can use that to get a service instance. We'll see an example using a controller action.

```cs
public IActionResult Get()
{
    var svc = (ISomeService)Request.HttpContext.RequestServices
        .GetService(typeof(ISomeService));
    svc.DoStuff(); //use the service as usual

    return new OkObjectResult("Everyone gets a service provider!");
}
```

#### [6] Basic Configuration setup in .NET Core 2.0

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

#### [7] Adding additional configuration source

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

#### [8] Strongly typed configuration

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

#### [9] IOptions injection for configuration instance

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

#### [10] Injection of strongly typed configuration without IOptions

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
