---
layout: post
title: "Porting ASP.NET Web API apps to ASP.NET Core 2.0"
excerpt: ".NET Core 2.0 series - 5/5 - Porting existing ASP.NET Web API apps to ASP.NET Core 2.0"
date: 2018-01-22
tags: [tech, mvc, webapi, dotnet, csharp, aspnet, dotnetcore]
categories: articles
share: true
modified: 2018-01-28T22:11:53-04:00
---

This article is part of the [.NET Core Series](/articles/dotnet-core-2.0/). Go have a look at the other articles of this series, and run through the previous topics if not done already!

#### _Note: This article is still WIP. It'll be updated with More content soon._

What is **Web API** in .NET?

Web API in _classic .NET_ was the framework for building modern [RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer) web services. In `ASP.NET Core` The `MVC` web application framework and `Web API` framework are unified, creating a single framework for both.

So, What is the difference between `MVC` and `Web API`?

The difference is pretty much same like the difference between a web site and a RESTful service! A web gives a nicely formatted graphical user interface, whereas a web service provides plain data as requested. So, the main distinguishing points are:

1. Web API provides data than user interface. Code wise, it does not return a `View` to the end user. So, it has controller but no view.
2. It follows the `REST` principles. But, that responsibility mostly lies with the developer. With wrong implementation, it can easily violate the REST pattern.
3. It respects the general http standards like http verbs, content negotiation, headers and stuffs.

Since we have already covered ASP.NET Core web (MVC) development in last article, here we'll see just the Web API specific parts.

#### [1] A simple Web API Controller

A Web API `Controller` is very similar to a `MVC` controller. Most significant differences are

1. Returns plain data or `http` code specific results, but not `View`
2. Specifies `http verb` & [attribute route](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/routing) for each action, which should be in line with REST principles

In the sample below, `DataStore` is some virtual source of `Item` data. Where `Item` is a simple class with an integer `Id` and other properties. 

```cs
[Route("api/Items")] //base route for all actions in this controller
public class ItemController : Controller
{
    private readonly ItemContext DataStore; //some source of data

    //GET http://example.com/api/items
    [HttpGet]
    public IEnumerable<Item> GetAll() //return type is data type
    {
        return DataStore.Items.ToList();
    }

    //GET http://example.com/api/items/10
    [HttpGet("{id}")] //uri part after base route will map to id parameter
    public IActionResult GetById(long id)
    {
        var item = DataStore.Items.FirstOrDefault(t => t.Id == id);
        if (item == null)
            return NotFound(); //http 404

        return new ObjectResult(item); //this returns with 200 OK
    }
    //return type is IActionResult as there are different return items

    //POST http://example.com/api/items, item in request body
    [HttpPost]
    public IActionResult Create([FromBody] Item item)
    {
        if (item == null)
            return BadRequest(); //http 400

        DataStore.Items.Add(item);
        DataStore.SaveChanges();

        //http 201 created //also adds location header to get the newly created item
        return CreatedAtRoute("GetTodo", new { id = item.Id }, item); 
    }
}
```

#### [2] Result types in ASP.NET Core 2.0 MVC

Web API supports many different result types to format and send data in different forms and with different http codes.

1. `JsonResult` returns `application/json`
2. `ContentResult` returns `text/plain`
3. `ObjectResult` supports content negotiation. When any `object` type is returned from API, they are wrapped inside an ObjectResult. `StatusCode` can be specified too.
4. `StatusCodeResult` for returning a plain status code.

#### [3] There are lot of short-hand methods and types for http-code-specific result
1. 200 - `OkResult` or `OkObjectResult` to include object data
2. 404 - `NotFoundResult` or `NotFoundObjectResult`
3. 201 - `CreatedAtRoute`, with route to newly created object in header
4. 400 - `BadRequest` 
5. 204 - `NoContent`
6. etc. Read more about them [in the docs](https://docs.microsoft.com/en-us/aspnet/core/mvc/models/formatting).

#### [4] Adding media type formatters

By default, ASP.NET Core 2.0 (Web API too) supports only `JSON` as output formatter. If `XML` support is required, add in `Startup`. Same should be followed for any other media types as well.

```cs
//using using Microsoft.AspNetCore.Mvc.Formatters;
services.AddMvc(options =>
{
    //to use XmlSerializer, which you should
    options.OutputFormatters.Add(new XmlSerializerOutputFormatter());
    //OR, to use DataContractSerializer instead
    options.OutputFormatters.Add(new XmlDataContractSerializerOutputFormatter());
    //use only one
});
```

#### [5] The Produces filter

A `Produces` ResultFilter can be added action, controller or global level to force output to be specific format ignoring content negotiation. Following code forces a `XML` formatted result

```cs
[Produces("application/xml")] //will force to send XML ignoring ConNeg
public IActionResult GetById(long id)
{
    var item = _context.Items.FirstOrDefault(t => t.Id == id);
    if (item == null)
    {
        return NotFound(); //http 404
    }
    return new ObjectResult(item); //200 OK
}
```

To enforce `JSON`, there is also the `JsonResult` return type and `Json(object)` shorthand method.

#### [6] The cool uri format trick 

In some environments (including development & testing), it's very helpful to be able to specify the expected mime type without modifying http headers. For that, ASP.Net Core provides a in-built `ResultFilter`, `Produces`. When this is used, the mime type can be specified in the request URI as an extension, like `.json` or `.xml`.

* OLD ASP.NET Web API :: type as query string
  * GET [http://myapp.com/api/SomeEntity?format=xml](http://myapp.com/api/SomeEntity?format=xml)

```cs
//public static class WebApiConfig
public static void Register(HttpConfiguration config)
{
    config.Formatters.JsonFormatter.MediaTypeMappings
        .Add(new QueryStringMapping("format", "json", "application/json"));
    config.Formatters.XmlFormatter.MediaTypeMappings
        .Add(new QueryStringMapping("format", "xml", "application/xml"));
}
```

* New ASP.NET Core 2.0 :: type as extension
  * GET [http://myapp.com/api/SomeEntity/5.xml](http://myapp.com/api/SomeEntity/5.xml)

```cs
//Startup
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc(options =>
    {
        //json is available by default
        options.FormatterMappings
            .SetMediaTypeMappingForFormat("xml", "application/xml");
    });
}

//Controller
[FormatFilter] //added here at action level
[HttpGet("{id}.{format?}")] //OPTIONAL format like .json or .xml
public IActionResult GetByIdWithFormat(long id)
{
    var item = _context.TodoItems.FirstOrDefault(t => t.Id == id);
    return new ObjectResult(item);
}
```


----

This article covered the process of porting existing `ASP.NET Web API` REST services `.NET Core 2.0` services. This concludes the [.NET Core Series](/articles/dotnet-core-2.0/).