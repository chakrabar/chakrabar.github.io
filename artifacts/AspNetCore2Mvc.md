ASP.Net Core 2.0 MVC Application
================================

#### Web Application with `ASP.NET Core Web Application`
* Target framework: `.NET COre 2.0`
* Output type: `Console Application`

#### MVC routes are defined in `Startup.cs`

```cs
app.UseMvc(routes =>
{
	routes.MapRoute(
		name: "default",
		template: "{controller=Home}/{action=Index}/{id?}");
});
```

#### Controllers

What remains same
----

* What remains same -
	The Controllers & Views folder, Model binding, selection of View, Layout & ViewStart
	MVC specific state management -  ViewModel, ViewData, ViewBag, TempData, Session
* The basic controller action still looks the same

```cs
public IActionResult Index()
{
    return View();
}
```

What has been changed
----

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

* There's no `web.config`. Configuration can be read from a bunch of sources, including
  * Files (JSON, XML, INI)
  * Command line args
  * Environment variables etc.
  * In-memory local class object
  * Command line arguments `dotnet run key1=value1`
  * Read more about [Configuration](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/index?tabs=basicconfiguration)
  * Also look at [Options pattern](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options) in ASP.NET COre
* General convention is to keep settings in either of 
  * appsettings.json
  * appsettings.<environment>.json
  * The environment is typically set to `Development`, `Staging`, or `Production`
  * Read more about [Environments](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/environments)
* Configuration is read through injected `IConfiguration` instance. Some common methods are
  * Configuration["Profile:MachineNames:0:Domain"] configuration indexer
  * Configuration.GetSection("App:MainWindow") method
  * Configuration.GetValue<int>("App:MainWindow:Left", 80) extension
  * 
  
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
  "App": {
    "Profile": {
      "Machine": "Rick"
    },
    "Connection": {
      "Value": "connectionstring"
    },
    "Window": {
      "Height": "11",
      "Width": "11"
    }
  }
}
```

* Understand that `Configuration.GetConnectionString("ConnStrName")` methods used to get connection string, is nothing but shorthand for `GetSection("ConnectionStrings")["ConnStrName"]` 

* In controller methods, `HtmlEncoder.Default.Encode` can protect against malicious user inputs e.g. `JavaScript` code

```cs
// Requires using System.Text.Encodings.Web;
public string Welcome(string name)
{
    return HtmlEncoder.Default.Encode($"Hello {name}!");
}
```