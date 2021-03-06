---
layout: post
title: "Understanding ASP.NET Core 2.0"
excerpt: ".NET Core 2.0 series - 3/5 - Understanding ASP.NET Core 2.0"
date: 2018-01-12
tags: [tech, mvc, dotnet, csharp, aspnet, dotnetcore, aspnetcore]
categories: articles
share: true
comments: true
modified: 2018-02-07T22:11:53+05:30
---

This article is part of the [.NET Core Series](/articles/dotnet-core-2.0/). Go have a look at the other articles of this series, and run through the previous topics if not done already!

The **ASP.NET** is the _classic_ web application development framework based on `.NET` framework. An `ASP.NET Core` application is nothing but an ASP.NET Web application that targets `.NET Core`. While the class files and code structure (it's still good ol' C#), remains pretty much the same, the web infrastructure has changed a lot since the _classic ASP.NET_! Like the rest of .NET Core, ASP.NET Core is also written from scratch and has all the attributes of [.NET Core](/articles/what-is-new-in-dotnet-core/).

The new infrastructure is very light-weight and highly configurable. From the web host, request pipeline, environments, routing strategy, error-handling and logging to dependency injection, everyhting is built in and customizable. The web application can be hosted on all major platforms (Windows, Linux, MacOS), with all major web servers (IIS, Nginx, Apache, Docker etc.) and the front end can be build with any front-end framework of choice (plain html-JavaScript, Angular, React or anything) with support for modern web tools like Node, Gulp, Grunt, Bower etc.

A web application in `ASP.NET Core 2.0` can be `MVC` or `Razor Page` web application. Or, as many of the modern web based applications are, it can be simply a bunch of RESTful APIs, with an independent user interface. Even a `Web API` RESTful web service is actually a `MVC` web application. 

#### [1] ASP.NET Core project

- VS Solution shows whatever is there in the project folder
  - Proj file doesn not have included files
  - All files are inluded in project
	
- In root of web project, there is `wwwroot` folder
  - Only files in this folder are served
  - Files outside this folder are not accessible
  - This folder can be renamed or changed though

- While creating a new Web Application in VS 2017, you cen choose between the flavours
  - [ASP.NET Core Razor Pages](https://docs.microsoft.com/en-us/aspnet/core/mvc/razor-pages/index?tabs=visual-studio)
  - [ASP.NET Core MVC](https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/start-mvc?tabs=aspnetcore2x)
  - [ASP.NET Core Web API](https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-web-api)
	
			
#### [2] Program.cs & Main()

- There's no `Global.asax` anymore
- Entry point to application is `Main()` method in `Program.cs`
- `Main()` uses `Startup` class to setup application configuration, and then

- Main() builds a web host `IWebHost` using `WebHostBuilder` class options 
  - It defines the web server, KESTREL is the default internal web server (`Http.Sys` or something else can also be used)
  - Calls the ConfigureServices() & Configure() methods in Startup
  - It can also integrate with IIS or other web servers (relay between internal/KESTREL & external web server)
- Then `Run()` the host (to start the application)
  - **From this point the console becomes a ASP.NET CORE application, and starts listening to http requests**

```cs
//Program.cs
public static void Main(string[] args)
{
    BuildWebHost(args).Run();
}

public static IWebHost BuildWebHost(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>() //Invokes ConfigureServices & Configure
        .Build();
```


#### [3] The Startup.cs class

A simple class with two methods that the runtime calls
  - ConfigureServices(IServiceCollection services) - this is optional, to inject required services to container
  - Configure(IApplicationBuilder app, IHostingEnvironment env) - this is required, to setup http pipeline. **Any service that hass been registered in above method, can be injected directly in this method parameters**.
  - Also note that Startup class is a good place to add any custom code that needs to run on application startup, traditionally which used to be inside `Gloabl.asax`.

**Dependency injection**
		
The main purposes of `ConfigureServices()` method is to setup dependency injection (DI is "almost" enforced here)
- Choice of service lifetimes
  - Transient: Creted each time they are requested
  - Scoped: Once per client request
  - Singleton: One per lifetime of application
- Own IoC can be used, but ASP.NET Core comes with default IoC
- Default IoC can be used through the `IServiceCollection`

```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<MyDbContext>(opt => opt.UseInMemoryDatabase("MyDbName"));
    services.AddMvc(); //inject all services related to MVC
    services.AddScoped<IRepository, Repository>();
    services.AddTransient<ISomeService, SomeService>();
    services.AddSingleton<IConfigBuilder, FileConfigBuilder>();
}
```

<u>To use an registered service</u>
- Do constructor injection in Controller & other types those are invoked through DI
- Through `HttpContext` with 
   ctx.RequestServices.GetService<ISomeService>();
- Method parameter injection in middleware `Invoke()`
- Use `@inject` in views (..!!)

See the **[practical configuration & DI](/articles/practical-configuration-and-di-in-aspnet-core/)** article to see real life code demo on how DI & configuration settings are used in .NET Core.

	
#### [4] Request processing & HTTP Pipeline & Middleware

`Startup.Configure()` gets injected with a `IApplicationBuilder` interface - which can be used to configure the HTTP PIPELINE
- Pipeline defines how the application will respond to incoming http request 
- A **middleware** XYZ is added to pipeline with method `UseXYZ()`
- By default it's empty, and it needs to be built up with stuffs/blocks (e.g. MIDDLEWARE)
  - e.g. MVC framework, Authentication block, static file serving strategy, routing, response compression, uri rewritting etc.
- Custom (OWIN like) middleware can be created
		
**Request processing**

Request comes from browser to IIS (external web server) 
- Which invokes the dotnet runtime CLR
- It invokes entry point Main() method in application and executes (first time only)
- This starts internal web server KESTREL (Main() only configured KESTREL)
- It sends the request through middleware pipeline
- If it has `UseMvc()`, then it'll look for a `route` match. Based on match, it'll invoke `action` on a `controller`
- Finally processed result is routed back
- Note: functionality of the web server (internal) is also accessible by the middleware through specific feature interfaces 


Traditionally ASP.NET was heavily dependent on **System.Web** which was tightly coupled with `IIS`
  - Now it does not use the heavy `System.Web` and no dependency on `IIS`
  - But `IIS` is actually pretty good a web server, the issues mostly come with `System.Web`
	
**Note: Basically there are two web servers - External & internal**
> * External can be IIS or Nginx or Apache or Docker or some standard web server
> * Internal can be KESTREL (pronounced "kes-tral") or something else (well, mostly KESTREL)


#### [5] Building the PIPELINE

The pipeline can be built within `Configure()` method on the `app` object using `Run()`, `Use()` etc.
Middlewares need to be piped/chained to one another, else the rest will not be executed!
e.g. (here the `context` is a new entity, not the old one from `System.Web`)

```cs
app.Use(async (context, next) => 
{
	await context.Response.WriteAsync("Response text");
	await next(); //to next middleware in pipeline
});
```


----

##### Wait a minute! What is a middleware?

----

In `ASP.NET Core` a middleware is a piece of code that can define how the application will handle any incoming http request. In an ASP.NET Core web application, all http requests are passed through a pipeline which processes the request and produces the response. This pipeline is nothing but a series of middlewares. Or in other words, a middleware is nothing but a piece of the http request processing pipeline. Practically, every `middleware` has access to the `httpContext` object and all it's properties. It can manipulate the details, and then pass on to the next middleware in the pipeline, or return the response directly (called _short-circuiting_).

Understand that the application wouldn't do anything unless proper middlewares were setup. 

One of the most common middlewares is the `MVC` middleware, which is configured with the `.UseMvc()` call in `Configure()` method of the `Startup.cs`. There are lot of in-built middlewares like this, and also it's pretty simple to create a [custom middleware](https://docs.microsoft.com/en-gb/aspnet/core/fundamentals/middleware/index?tabs=aspnetcore2x).

All the middlewares have to be setup in the `Configure()` method, and they will process request in the same order they are registered. A middleware can be inline, or defined in a separate class. Following is an example of an inline middleware (a dummy one rather than something really useful) that _short-circuits_ the pipeline and returns a response directly if the user requests for a `.pdf`.

```cs
//In Configure() method of Startup.cs
app.MapWhen(context => context.Request.Path.Value.EndsWith(".pdf"),
    appBuilder =>
    {
        appBuilder.Run(async context =>
        {
            context.Response.StatusCode = 400; //Bad request
            await context.Response.WriteAsync("We do not serve PDFs yet!");
        });
    });
```

Custom middlewares are good place to handle the cross-cutting concerns of the application like logging, exception handling etc. They can pretty well replace global filters. It could also be a good idea to [replace http handlers and modules](https://docs.microsoft.com/en-gb/aspnet/core/migration/http-modules) with middlewares!

Note: An **`IApplicationBuilder`** instance is provided to `Startup.cs` by `Program.cs`, which allows to add middlewares to configure the request pipeline of the application.

----




#### [6] Side note: The relationship between OWIN, KATANA & KESTREL (and ASP.NET vNext, ASP.NET Core et al.)


1. **OWIN** :: (Open Web Interface for .Net) is a set of standards (mostly borrowed from `Ruby` & `Node.js`), which defines the set of interfaces to be 
  - Implemented by web applications & servers so that they are not coupled to each other. Any framework & server, as long as they implement those standards can work with each other. (e.g. Server should provide a response body stream & application should write on that :)
		
2. **KATANA** :: Katana was Microsoft's initial implementation of OWIN middlewares (that can sit between a full web server and the web application),
  - Available as a NuGet package. 
		
3. **ASP.NET Core** :: That time (early 2015) `ASP.NET vNext` was built, which later became `ASP.NET 5`, and finally `ASP.NET Core` - which can run on 
- `.NET core 1.0` as well as `.NET standard framework 4.6`. This could work with KATANA.
- Then came the **unified ASP.NET 6** combining MVC & Web API (MVC doesn't depend on `IHttpHandler` anymore) and with self-host capabilities.
- Meaning, now they can run as console apps, as microservices & can be hosted on `Azure Service Fabric` and the likes (read `Docker`). 
		
4. **KESTREL** :: KESTREL came in as a successor of KATANA. It has all OWIN capabilities & got rid of `System.Web` completely!!
- Many .NET Core/KESTREL middleware are available as NuGet which complies with `OWIN middlewares`. KESTREL does not use the full capabilities of IIS, but on th other hand it is compatible with many web servers & OS! 
- It replaces the now deprecated "Helios" project (which besides many the good things like KESTREL, relied on System.Web). 
- KESTREL is based on open source **libuv** (pronounced "lee-bu-vee") [project](https://github.com/libuv/libuv), which is a cross-platform library for asynchronous I/O, 
  - Written in `C`. (Originally built & used in `Node.js` ;)
  - KESTREL is production ready, but it is not a full-blown web server. Recommended model is to use it behing `IIS`, `Ngnix` or some other web server.

> A vague analogy ~ CLS:OWIN, CLR:KATANA, .NET Core:KESTREL (well, it's just me)

**Ideal web server setup for ASP.NET Core:**

* For internal/intranet only applications, we can use `KESTREL` as stand alone web server
* For public/internet applications, KESTREL should sit behind a full fledged web server (as reverse proxy) like IIS/Ngnix/Apache/Docker
* If HTTPS is needed, it should be done through a external web server (like above). **KESTREL works on plain HTTP only**.
* Even in simple requirement also, having a full web server will give more security, scalability, features, load balancing etc.
* If different ports are to be shared on same server, KESTREL cannot do that. It serves just one port in server. 

----


#### [7] MVC & Web API

Read the [next article]((/articles/porting-aspnet-apps-to-aspnet-core-2.0/)) for more information on following stuffs.

**MVC is a middleware in ASP.NET Core application**. 
- `services.AddMvc()` in `ConfigureServices()` method in Startup.cs can add necessary stuffs for MVC (available in aspnet mvc core NuGet).
- MVC routes needs to be added inside `Configure()` with overload of `app.UseMvc()`
- Same way, add support for static files like `app.UseStaticFiles()` from different dll.
- **web.config is GONE.** There are multiple options for settings
  - Config can be put in an external file like json, xml etc.
  - Though ASP.NET Core doesn't depend on web.config, to deploy the app in IIS, it is required!
- Developer exception page is not Yellow anymore and contains data in better format
- There can be different `Configure()` & `ConfigureServices()` methods for environments `development`, `staging` & `production`
  - e.g. ConfigureDevelopment(), ConfigureServicesProduction()
  - In debug mode, it can be changed from Debug tab of project properties
		
MVC & Web API systems (controller, model binding, routing etc.) are unified now
- (they WERE always very similar, only MVC relied on `System.Web` and Web API didn't )

- Controller methods now return `IActionResult`
- Controller methods can return View(), access ViewBag and use _Layout.cshtml, _ViewStart.cshtml etc.
- Along with `@Razor`, views now support something called `TagHelpers` which is extension to normal HTML syntax

TagHelpers: 
  - TagHelpers (like html directives) comes as new HTML tag or new attribute to existing HTML tag 
  - To add TagHelper to use throughout the application, create a file "_ViewImports.cshtml" in root View folder
  - Use @addTagHelper directive syntax and specify helpers (or all with * wildcard)
  - Predefined TagHelpers are in "Microsoft.AspNet.Mvc.TagHelpers" namespace, which is part of MVC NuGet package
  - To use in specific view, import in that view file 
  - Some predefined tags: <environment> <link asp-href-include> <img asp-append-version> etc.
  - e.g. <form asp-controller="Home" asp-action="Feed" method="POST">...</form>
		
In ASP.NET 5/Core, there is no `ApiCOntroller`. Since it's unified
- Web API also inherits `Controller`
- The main difference is, rather than `IActionResult`, Web API controller returns the model object


#### [8] View Components

A [reusable display module](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/view-components)
- A partial view with simple controller!!
- Can be rendered with @TagHelper or
    @await Component.InvokeAsync ("ViewComponentName")
- The ViewComponent class has two specialties, apart from that it's pretty much like a controller 
  - Be derived from `ViewComponent`
  - have a `Invoke()` method that returns `IViewComponentResult` (the method returns a View) or the `async` version of the method
  - The view needs to be added inside a `Views/ViewComponents` folder
  - Default view for `Invoke()` is `Default.cshtml`
		

#### [9] Project dependencies for ASP.NET Core web application

**Sources:**

- `NuGet` - required for all applications, for server side packages
- `Bower` - optional, for client side packages
  - Right click on project and select "manage bower packages"
  - Will create a folder "Bower" under "Dependencies" - will have name & version of each packages
  - Will also create a "bower_components" folder at root level - will have the actual files
- `NPM` - optional, For `Node.js` and client side build tools like `Gulp`, `Grunt`
    
**Tasks:**

- **Bundling & minifications** are not there in ASP.NET Core
  - This is done, be default, with [VS bundling & minification tools](https://docs.microsoft.com/en-us/aspnet/core/client-side/bundling-and-minification?tabs=visual-studio%2Caspnetcore2x)
  - Download bundler & minifier from VS gallery
  - Add new item - "bundleconfig.json"
  - Simply specify input files array & output file name for bundling & minification 
  - This is also recognized by task runner explorer 
- These type of **front end tasks** can also be done through `Gulp` or `Grunt` tasks 
  - That run on `node.js` (with different plugins available) - adding package in next step
  - Tasks written in `javascript` to call plugins etc.
  - Tasks can be run through build events & from VS
- **GULP tasks**:
  - Add new item to project >> "Gulp configuration file" >> gulpfile.js (do not change name of this file)
  - Add all the file paths (source & destination)
  - Create tasks with gulp plugins as js (like minify js, minify css, copy all css etc.)
  - We can also group multiple tasks in single task 
	- To see tasks, Go to : VS > View > Other Windows > Task Runner Explorer
	- From the same window, we can also bind tasks to build events (like, after build)
        
**Nuget packages:**

- Specified in project.csproj file (only top level packages)
- Used ONLY for server side packages
- Can be added 
  - Directly in project.csproj (with intellisense), or 
  - Through NuGet package manager
- Packages are not stored locally, all are kept centrally e.g. C:/Users/{user}/.nuget/packages

NPM packages:

- Add new item to project - "npm configuration file" - package.json
  - Add `Gulp` with version, and required Gulp packages under "devDependencies"
  - New folder created under "Dependencies" as "npm"


#### [10] ASP.NET Core Deployment options

See the [What is .NET Core](/articles/what-is-new-in-dotnet-core/) article for understanding general deployment options for .NET Core applications.

With default `Publish` from Visual Studio
- Core creates just a dll which can be run through Core CLI to start the web `$ dotnet WebApp.dll`
	
Once again, **KESTREL** is a performant simple web server
- It has a managed dll
- And natively it depends on `Libuv` (lee-bu-vee) which is not managed - it is included in shared section of the platform 
- For other packages with native dependent modules, a folder will be created "runtimes" with child folder per RID 
  - RID = runtime identifier
  - e.g. win7-x64 : OS & processor family 
	
Deploying to **IIS** (for security, load balancing and other advanced features)
- `AspNetCoreModule` has to be installed on the machine (get it form GitHub)
  - Also includes DotNetCore, so it can host portable apps 
- It then works as a `reverse-proxy`, sends requests to KESTREL and then sends response back to client 
- To depoy
  - Create new website
  - Give path to published portable app (or self-contained?)
  - In AppPool select .NET CLR Version = "No Managed Code" (as CLR will run in it's own process, IIS need not host CLR)
- Hosting on AZURE is very similar, which runs an IIS on a VM with `AspNetCoreModule` installed 
	
**Nginx** (read out as "engine-ex") is a production web server for `Linux`/`Mac OS`
- Can be run with the published data in very similar fashion
- In general also, published data as type:platform can be run on any machine given it as DotNetCore installed 

----

This article covered the high level basic of the `ASP.NET Core` applications. Continue to [Porting ASP.NET MVC applications to ASP.NET Core 2.0](/articles/porting-aspnet-apps-to-aspnet-core-2.0/).