---
layout: post
title: "Porting ASP.NET MVC apps to ASP.NET Core 2.0"
excerpt: ".NET Core 2.0 series 4/5 - Porting existing ASP.NET MVC Web apps to ASP.NET Core 2.0"
date: 2018-01-14
tags: [tech, mvc, dotnet, csharp, aspnet, dotnetcore]
categories: articles
share: true
modified: 2018-01-28T22:11:53-04:00
---

This article is part of the [.NET Core Series](/articles/dotnet-core-2.0/). Go have a look at the other articles of this series, and run through the previous topics if not done already!

Here I'll talk about my experience of converting an existing ASP.NET MVC application (.NET Framework) to the new [ASP.NET Core 2.0](https://docs.microsoft.com/en-us/aspnet/core/) application. It is rather a port or [migration](https://docs.microsoft.com/en-us/aspnet/core/migration/proper-to-2x/index) than an upgrade, as it involved bunch of code changes and restructuring. Some of the concepts has changed dramatically, and some existing functionalities are not available anymore.

Before going into the porting process, let us briefly discuss what changes we are going to make, and what to expect overall. Here, when we say ASP.NET Core or .NET Core, we refer to the **.NET Core 2.0** framework.

#### [1] MVC Application with `ASP.NET Core` Web Application

Some of the distinctive features of new `ASP.NET Core` web projects

* Target framework: `.NET Core 2.0`
* Output type: `Console Application`
* Not bound to heavy `System.Web` dll
* Builds as a console application. Not tied to `IIS` and can be integrated with most web servers like `Apache`, `Nginx`, `Docker` etc.
* And all general notions of `.NET Core` - cross platform, open source, better optimized code written from scratch, lightweight & portable, NuGet based framework


#### [2] The ASP.NET Core MVC application - What remains same

* The core ideas and concepts of MVC as an architecture (obviously)
* The project structure - the Controllers & Views folder
* Model binding, model validation, selection of View etc.
* Layout & ViewStart
* MVC specific state management -  ViewModel, ViewData, ViewBag, TempData, Session
* Razor based views - there are some syntax changes & enhances though
* Model `data-annotations`
* The basic controller action still looks the same (can be `async` as well, see later)

```cs
public IActionResult Index()
{
    return View();
}
```


#### [3] General changes for MVC project (compared to ASP.NET 4 or older)

* The project builds as a `Console` project
* It has a `Program` class with `Main()` method - this is called just once when the app starts for the first time
* The [`Startup`](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/startup) class. It is called from `Main` and has the initial configuration code for the application (which used be mostly in `Application_Start()` method of `Global.asax`) are in `Startup` class. The main methods are
  * ConfigureServices() - optional, inject & configure services, manages lifetime 
  * Configure() - required, configure the `http pipeline` by adding necessary [MiddleWare](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware?tabs=aspnetcore2x). A middleware basically works on the `HttpContext` and then invokes next middleware in the pipeline or sends back response directly
  * The `Startup` can accept constructor dependencies defined by the host, like `IHostingEnvironment`, `IConfiguration` etc.
* With the `ConfigureServices()` method, `Dependency Injection` is built in
* The `wwwroot` folder. All static files are kept here, files outside this folder are not be served directly. The name of the folder can be changed though
* As the app runs with an internal web server (Kestrel) by default, it needs a full powered web server (like IIS, Nginx, Apache etc.) as a reverse proxy for production [hosting](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/?tabs=aspnetcore2x)
* Support for popular front-end or [client-side](https://docs.microsoft.com/en-us/aspnet/core/client-side/index) development tools like `Gulp`, `Grunt`, `Bower` etc. Support is available both in VS and dotnet cli
* Bundling & minification needs additional [tools or NuGet](https://docs.microsoft.com/en-us/aspnet/core/client-side/bundling-and-minification?tabs=visual-studio%2Caspnetcore2x)
* **There is no `ApiController`** for `Web API`. [Web API](https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-web-api) is built using the same infrastructure like MVC
* There is no `[FromUri]` attribute. Use `[FromQuery]` or attribute routing instead.
* The `ActionFilter`s are still there, but has changed form. Following needs to be replaced
	`System.Web.Http.Filters` with `Microsoft.AspNetCore.Mvc.Filters`
	`HttpActionContext` with `ActionExecutingContext` for `OnActionExecuting()`
* `Razor` views have `Tag Helper`, comparable to what used to be `HTML Helper`.
* [Anti forgery](https://docs.microsoft.com/en-us/aspnet/core/security/anti-request-forgery) is automatic and more customizable
* MVC routes are defined in `Configure()` method of `Startup.cs`

```cs
app.UseMvc(routes =>
{
	routes.MapRoute(
		name: "default",
		template: "{controller=Home}/{action=Index}/{id?}");
});
```

* Controller actions are preferably `async` where there is a slow/network work

```cs
public async Task<IActionResult> Details(int? id)
{
	if (id == null)
		return NotFound();

	var movie = await _context.Movie
		.SingleOrDefaultAsync(m => m.ID == id);
	if (movie == null)
		return NotFound();

	return View(movie);
}
```

* For all stuffs `MVC` like `[HttpPost]`, `[FromBody]` etc. base namespace changed from `System.Web.Mvc` to `Microsoft.AspNetCore.Mvc`
* In controller methods, `HtmlEncoder.Default.Encode` can protect against malicious user inputs e.g. `JavaScript` code

```cs
// Requires using System.Text.Encodings.Web;
public string Welcome(string name)
{
    return HtmlEncoder.Default.Encode($"Hello {name}!");
}
```

* And many other changes & additions


#### [4] Configuration/Setting files in ASP.NET Core

When a new ASP.NET Core (MVC) Web Application is created, the following Configuration/Setting files are created

* `ProjectName.csproj` - the project settings, mainly target Framework(s), and NuGet packages
* `appsettings.json` - similar to `app.config`, runtime configuration like log profile, connection strings etc. This can be environment specific.
* `bundleconfig.json` - static files (css, js etc.) bundling and minifications settings
* `Properties`/`launchSettings.json` - app launch and debug profiles, IIS settings, start mode (IISExpress, Kestrel etc.), Environment (Development, Production etc.) etc.

Now the `csproj` files are pretty clean and simple. See one sample file below, it's quite self explanatory. 
```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>netcoreapp2.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.All" Version="2.0.3" />
    <PackageReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Design" Version="2.0.1" />
  </ItemGroup>

  <ItemGroup>
    <DotNetCliToolReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Tools" Version="2.0.1" />
  </ItemGroup>

</Project>
```


#### [5] Configuration

* There's no `web.config` (unless app is run with IIS). Configuration can be read from a bunch of sources, including
  * Files (JSON, XML, INI)
  * Environment variables etc.
  * In-memory local class object
  * Command line arguments `dotnet run key1=value1`
* Read more about [Configuration](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/index?tabs=basicconfiguration)
* Also look at [Options pattern](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options) in ASP.NET Core
  
* General convention is to keep settings in 
  * appsettings.json, or specific to environment like
  * appsettings.{environment}.json , read about [6] Environments later
  * The environment is typically set to `Development`, `Staging`, or `Production`
  * Read more about [Environments](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/environments)
  
* Configuration is read through injected `IConfiguration` instance. Some common methods are (assume IConfiguration instance is Configuration)
  * Configuration["Profile:MachineNames:0:Domain"] configuration indexer
  * Configuration.GetSection("App:MainWindow") method
  * Configuration.GetValue<int>("App:MainWindow:Left", 80) extension
  * ??
  
* One can bind a whole settings to class hierarchy/structure as

```cs
var builder = new ConfigurationBuilder()
	.SetBasePath(Directory.GetCurrentDirectory())
	.AddJsonFile("appsettings.json");

var config = builder.Build();

//AppSeetings is a user defined class that matches appsettings App section
var appConfig = new AppSettings();
config.GetSection("App").Bind(appConfig);
//Or alternatively
var appConfig = config.GetSection("App").Get<AppSettings>();
```

* Sample `appsettings.json`

```javascript
{
    "Logging": {
        "IncludeScopes": false,
        "LogLevel": {
            "Default": "Warning"
        }
    },
    "ConnectionStrings": {
        "MovieContext": "myCoolConnectionStringHere"
    }
}
```

* Understand that `Configuration.GetConnectionString("ConnStrName")` methods used to get connection string, is nothing but shorthand for `GetSection("ConnectionStrings")["ConnStrName"]` 


#### [6] Environments

ASP.NET Core configuration is **environment-aware**. That means configuration, settings and code can be written specific to specific application deployment environment. By default it understands 3 environments - `Development`, `Staging` & `Production` (names are NOT case-sensitive in `Windows` & `Mac OS`, but IS case-sensitive in `Linux`), but environment can be set to anything, e.g. 'Pre-Staging'.

To set `ASPNETCORE_ENVIRONMENT` variable
* If nothing is set, by default it's taken as `Production`
* Can be set through `dotnet cli` commands (commands vary per platform) per run of the application

```bash
set ASPNETCORE_ENVIRONMENT=Development
dotnet run
# or
set ASPNETCORE_ENVIRONMENT=Production && dotnet run
# or, to ignore launchsettings.json profiles
dotnet run --no-launch-profile
# or, to run with specific profile in launchSettings.json
dotnet run --launch-profile "Development"
```
* Machine wide value can be set in Windows environment variables in 

**Control Panel > System > Advanced system settings > Environment variables**

* Local application settings can be set in `Properties`\\`launchSettings.json`. This takes precedence over system values. See sample `launchSettings.json` below. Remember that
  * commandName Can be : `IIS`, `IISExpress` or `Project` (Kestrel)
  * ASPNETCORE_ENVIRONMENT can be `Development`, `Staging`, `Production` or anything
  * When app is run with `dotnet run`, it takes first profile with `commandName`:`Project`
  * These different profiles are available as **VS debuf profile** under VS debug menu
  * Profile & environment can be updated from `Project` > `properties` > `Debug`

```json
{
  "iisSettings": {
    "windowsAuthentication": false,
    "anonymousAuthentication": true,
    "iisExpress": {
      "applicationUrl": "http://localhost:60237/",
      "sslPort": 0
    }
  },
  "profiles": {
    "IIS Express": {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "MvcMovie": {
      "commandName": "Project",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      },
      "applicationUrl": "http://localhost:60238/"
    }
  }
}
```


#### [7] Environment specific code

Now settings can be environment specific. In all cases, when app is running with <Dev_2> environment, it'll use `SettingsDev_2` class or `ConfigureDev_2()` etc. whenever they exist. Otherwise they'll fall back to default options.

* Startup{EnvironmentName} e.g. StartupStaging class
* public void ConfigureDevelopmentServices(IServiceCollection services)
* public void ConfigurePreStaging(IApplicationBuilder app, IHostingEnvironment env)
* appSetting.Production.json

Also, code can be written to specific environment, both in `cs` and views

```cs
if (env.IsDevelopment())
{
	app.UseDeveloperExceptionPage();
	app.UseBrowserLink();
}
if (env.IsProduction() || env.IsStaging() || env.IsEnvironment("Dev_2"))
{
	app.UseExceptionHandler("/Error");
}
```

```html
@inject Microsoft.AspNetCore.Hosting.IHostingEnvironment hostingEnv

<p> ASPNETCORE_ENVIRONMENT = @hostingEnv.EnvironmentName</p>

<environment exclude="Development">
    <p>&lt;environment exclude="Development"&gt;</p>
</environment>
<environment include="Staging,Production,Staging_2">
    <p>
        &lt;environment include="Staging,Production,Staging_2"&gt;
    </p>
</environment>
```


#### [8] Areas

* Areas are just folders to keep bunch of related controllers, views, models together
* Individual areas do not have `MyAreaRegistration` class
* For all of the areas to work, add following in `Startup.Configure()`

```cs
routes.MapRoute(
	name: "areas",
	template: "{area:exists}/{controller=Home}/{action=Index}/{id?}"
); //if 1st part of uri matches existing area, it'll find controller there
```

This article covered the process of porting existing `ASP.NET MVC` web applications to `.NET Core 2.0` MVC application. Continue to [Porting Web API services to ASP.NET Core 2.0](/articles/porting-existing-webapi-to-aspnet-core-2.0/).