---
layout: post
title: "Practical configuration & DI in ASP.NET Core 2.0"
excerpt: "Code demo of real life configuration & dependency injection in ASP.NET Core 2.0"
date: 2018-01-22
tags: [tech, csharp, dotnetcore, aspnetcore, configuration, di, aspnetcore20]
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
  - **Scoped**: Created once per http request
  - **Singleton**: Created once per lifetime of application

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

As mentioned above, the ideal way to define dependency is the constructor. But there can be some cases when one might want to use the `IServiceProvider`

1. To create service instances conditionally at runtime
2. To minimize the parameters of constructor. Only `IServiceProvider` can be injected in constructor, then later the actual services can be instantiated as and when necessary.

**<u>Service instance injection inside the ConfigureServives method</u>**

In some occasions, it might be required to get a service instance injected inside the `ConfigureServices` method itself! The way to do that is again using `IServiceProvider`. The `IServiceCollection` available to the method has an extension to create an `IServiceProvider` instance. See below

```cs
//using Microsoft.Extensions.DependencyInjection;
//Startup.ConfigureServices()
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

Now, if I have a constructor dependency in the filter, it CANNOT be used simply as an attribute. The dependency also cannot be injected.

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
There are basically few ways to handle this.

**<u>Using ServiceFilter</u>**

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

**<u>Using TypeFilter</u>**

For this, 
- registration of the filter is NOT REQUIRED
- use `TypeFilter` attribute with type of desired filter that needs service injection
- this doesn't use the DI container directly, rather it internally uses frameworks's `ObjectFactory` to inject the instance. More details [here](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/filters)
- with TypeFilter, additional constructor arguments can also be passed along with injection (e.g. Arguments = new object[] { 1, "yo" } )

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

Also, one can implement own **`IFilterFactory`** as outlined [here](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/filters#ifilterfactory).

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

----

**_Notes:_** _Few things to watch out for, specially when converting older projects to .NET Core_.

Before we move on to Configurations, let's quickly talk about some (decided) <u>limitations of the current dependency injection framework in .NET Core</u>.

1. It doesn't support named instances like some other popular DI frameworks like Unity, StructureMap etc.
2. It doesn't support setter or property injection like Unity, Ninject etc.

From [GitHub](https://github.com/aspnet/DependencyInjection/issues/473), Microsoft does not have a plan to support named instances. There are some workarounds though. This [Stack Overflow QA](https://stackoverflow.com/questions/39174989/how-to-register-multiple-implementations-of-the-same-interface-in-asp-net-core/44177920#44177920) shows some viable alternatives.

The idea is to cleverly use an extension method of `IServiceCollection` that allows registering a function of type `Func<IServiceProvider, TService>`, to actually register a factory function of type `Func<string, TService>`. Now that factory can be injected by the DI framework.

```cs
//using Microsoft.Extensions.DependencyInjection;
//Startup.ConfigureServices()
services.AddTransient(servicrProvider =>
{
    Func<string, ISomeService> accesor = key =>
    {
        switch (key)
        {
            case "My":
                return servicrProvider.GetService<MyService>();
            case "Other":
                return servicrProvider.GetService<OtherService>();
            default:
                throw new KeyNotFoundException();
        }
    };
    return accesor;
});
//And the default standard instance
services.AddTransient<ISomeService, MyService>();

//To use a named instance
//A controller constructor as example
public TestController(Func<string, ISomeService> serviceAccessor)
{
    ISomeService svc = serviceAccessor("Other");
}
```

----

#### [6] Basic Configuration setup in .NET Core 2.0

There is a significant change from **classic `.NET`** to **`.NET Core`** on how configuration and settings are handled. There are no preset `app.config` or `web.config` files to dump the config values (well, there is `appsettings.json` but that is practically optional). It comes with lot of flexibility and options on how the config settings can be managed. See the Configuration & settings section of [Porting ASP.NET MVC applications to ASP.NET Core 2.0](/articles/porting-aspnet-apps-to-aspnet-core-2.0/) for a basic understanding of how it works in (ASP).NET Core.

Here are some key points

- Like older .NET, still the configuration is key-value based
- Unlike older .NET, there is no `app.config` and there is no predefined config sections like `appSettings`. Configuration settings in .NET Core is much more flexible, and the settings can be structured in any form.
- Source of configuration can be any `XML`, `JSON`, `INI` file or `in-memory`, `command line args` or environment variables
- The configuration sources (e.g. appsettings.json) can be updated without restarting the application
- Configuration source is defined in Program.cs, and registered with DI in Startup. Then the configuration values can be used across the application

In the earlier releases of .NET Core, any file or other sources of configuration needed to be explicitly mentioned in Program.cs. Example below

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
            //Adding all configuration sources
            config.AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true, reloadOnChange: true);
            config.AddEnvironmentVariables();
            if (args != null)
            {
                config.AddCommandLine(args);
            }
        })
        .UseStartup<Startup>()
        .Build();
}
```

Starting with **.NET Core 2.0**, the inclusion of main `appsettings.json`, environment specific `appsettings.{env-name}.json` and other standard configuration sources are implicit. The `CreateDefaultBuilder()` method takes care of that. See the [code on GitHub](https://github.com/aspnet/MetaPackages/blob/rel/2.0.0/src/Microsoft.AspNetCore/WebHost.cs).

```cs
//Program.cs
public static IWebHost BuildWebHost(string[] args) =>
    WebHost.CreateDefaultBuilder(args) //adds appsettings.json, appsettings.env.json
        .UseStartup<Startup>() // Invokes ConfigureServices & Configure on Startup
        .Build();
```

A sample default `appsettings.json` from ASP.NET Core 2.0 project

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

**Reading _simple values_ from appsettings file with `IConfiguration`**

The `IConfiguration` is auto-registered with the DI and can be injected in any class. This IConfiguration has a string based indexer that allow reading values with JSON-style keys. Sample code below.

```cs
using Microsoft.Extensions.Configuration;

public class TestController : Controller
{
    IConfiguration _configuration;

    public TestController(IConfiguration configuration,)
    {
        _configuration = configuration;
    }

    public IActionResult Get()
    {
        //get config value with IConfiguration
        var logLevel = _configuration["Logging:Debug:LogLevel:Default"];
        //from array with index
        var firstServerName = _configuration["Servers:0:Name"];
        //casting with (optional) default value
        var country = _configuration.GetValue<string>("Address:Country", "India");

        return new OkObjectResult($"Log level: {logLevel}");
    }
}
```

#### [7] Adding additional configuration source

Apart from the default `appsettings.json`, any other source of configuration can also be added. This does not override the initial config, rather appends the new values. Following code shows example of adding an `xml` file as an additional configuration source.

```cs
//using Microsoft.Extensions.Configuration;
//Program.cs
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

Sample `AppConfigs.xml` configuration XML. Here, the name and structure of the XML file doesn't really matter. Once added, the values can be read the same way as shown above.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<AppConfigs>
  <EnableGuestAccess>true</EnableGuestAccess>
  <CustomLogHeader>My Cool Log Header</CustomLogHeader>
  <DeleteLogsAfterDays>45</DeleteLogsAfterDays>
</AppConfigs>
```

#### [8] Strongly typed configuration

Another useful cool feature of `.NET Core` is, a complete configuration file or any part of it can be easily casted to simple `POCO` object, and then can be passed around as strongly-typed configuration. Rather than injecting the `IConfigurationRoot` or `IConfiguration` with all the settings, it is better to have related settings bound to a meaningful POCO and inject where it makes sense. 

For this, create a class that is compatible with the section of configuration file that needs to be casted. Following is a class that is used to store the configuration values from the XML shown above.

```cs
public class AppConfigs
{
    public bool EnableGuestAccess { get; set; }
    public string CustomLogHeader { get; set; }
    public int DeleteLogsAfterDays { get; set; }
}
```

Read a whole configuration file or a section of it (Json/XML) into a compatible object instance, and register with the DI container. Here, it is important that the structure of the configuration section matches the type of the object. Following code inside the `ConfigureServices()` method shows how a configuration is casted to an object and registered with the DI container.

**Note**: When this `services.Configure()` method is used, it'll look in all configuration files available in the reverse order of how they were added. So, the source that was added last will be checked first and so on. It'll take the first match, complete file or by section name, checking the properties recursively.

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
        //Here it'll read from appsettings.json as that has matching section "Logging"
    }
}
```

Also **note**, the strongly-typed configuration object registered like this, behaves more like a singleton registration.

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

Instead of `IOptions<T>`, the `IOptionsSnapshot<T>` can also be used. See the last section.

#### [10] Injection of strongly typed configuration without IOptions wrapper

If we want to use that strongly-typed configuration object without the `IOptions<>` wrapper, we'd need to create an instance of that object in `Startup` and register that as `singleton`. Following code shows an example.

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
        //create a new instance of the object
        var appConfigs = new AppConfigs();
        //bind that object from the config section
        Configuration.Bind(appConfigs);
        //or Configuration.GetSection("key").Bind(appConfigs);
        //Register the config object as singleton
        services.AddSingleton(appConfigs);
    }
}
```

Now, once that object has been registered as **`singleton`**, it can be injected with the DI framework into any class by the config object type. 

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

**<u>Runtime reload of configuration values</u>**

When working with configuration, sometimes it is required to update configuration values. Reloading of config values, when a configuration source has been updated, doesn't work out of the box always. To be specific, strongly-typed configurations in `ASP.NET Core 2.0` are not automatically reloaded. 

One options is to simply restart the application, so that the `Startup` is run again, and it reloads the config again. But, it is not a practical solution. All users will lose their state, session, data etc., and there could be an application downtime.

One solution to this problem is to [use IOptionsSnapshot](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options#reload-configuration-data-with-ioptionssnapshot). This internally uses the `IOptionsMonitor` based triggers to re-read configuration values from source, whenever that is updated.

To use, simply use `IOptionsSnapshot<>` in place of `IOptions<>` where a strongly-typed config is being injected. Or to **inject the reloadable config object** type directly, register an instance of the IOptionsSnapshot's `Value` for the config object's type. This is done with an extension method to register a service, like `Func<IServiceProvider, TConfig>`.

```cs
public SomeController(IConfiguration configuration, IOptions<AppConfigs> options,
    IOptionsSnapshot<AppConfigs> optionsSnapshot, AppConfigs appConfigs)
{
    //IOptions does NOT auto-update
    var c1 = options.Value.LogCount;
    //object config does NOT auto-update
    var c2 = appConfigs.LogCount;

    //simple values from IConfiguration DOES auto-update
    var c3 = configuration["log:count"];    
    //IOptionsSnapshot DOES auto-update
    var c4 = optionsSnapshot.Value.LogCount;
}

//using Microsoft.Extensions.DependencyInjection;
//Startup
//Registering IOptionsSnapshot instance as config object
services.AddScoped(isp =>  //IServiceProvider
    isp.GetService<IOptionsSnapshot<AppConfigs>>().Value);

//Now the AppConfigs injected instance auto-updates
public SomeController(AppConfigs appConfigs)
{
    //this RELOADS on source data change
    var logCount = appConfigs.LogCount;
}
```

**Note:** This is important to add the configuration sources with `reloadOnChange: true` (see section 7 above) option. Without this, the config values will not reload without an application restart.

Hope this helps in understanding the confiduration system and DI in ASP.NET Core 2.0 little more easily. Check back [official documentation](https://docs.microsoft.com/en-gb/aspnet/core/fundamentals/configuration/index?tabs=basicconfiguration) and [GitHub](https://github.com/aspnet) for latest developments on all topics. 