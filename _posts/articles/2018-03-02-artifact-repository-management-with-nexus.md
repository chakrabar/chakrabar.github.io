---
layout: post
title: "Nexus artifacts repository manager for .NET with NuGet"
excerpt: "Setup & configuration of Sonatype Nexus from scratch, as an artifacts repository manager for .NET projects"
date: 2018-03-02
tags: [tech, artifacts, nexus, package, nuget, dotnet, repository]
categories: articles
comments: true
share: true
---

_**"An artifacts what??!!!"**_

For those not familiar with the very popular _"artifact repositories"_ used all over the development community, it might be a pretty unsual thing. But they have been used in many open source and commercial software projects for many years - for a reason. Probably not super popular with the .NET community (_well, we have NuGet, don't we?_), but they are being used increasingly in many .NET projects. And the popular ones now come with .NET support by default.

So what is an **artifacts repository manager**? Going by the standard definition

> An artifacts repository is basically an application server that can store software artifacts in an organised and versioned manner, and that can be used for software projects as and when required. They are generally wired into application IDEs and build systems to store or fetch project artifacts (binaries, metadata etc.)

In all software projects, we have to deal with bunch of artifacts like code binaries, metadata files, configurations, xml files etc. In projects with size moderate+ they can become a handful with versions, inter dependencies and all. It can make everyday development, sharing code, building software pretty cumbursome. Here an _artifacts repository manager_ comes to rescue. It simple _manages repositories of artifacts_, where artifacts can be in-house or third party (e.g. NuGet packages for .NET projects).

So, it mainly does two things

* Manage & distribute software artifacts (project binaries and more)
* Proxy remote repositories (e.g. NuGet/Maven) to a local network server (optional tbh)

Put together, they help improve development, build and distribution of software systems.


#### Sonatype Nexus artifacts repository manager

[Sonatype Nexus](https://www.sonatype.com/nexus-repository-sonatype) is one of the most popular artifacts repository managers available in market today. It was initially made popular with `Maven` projects for `Java`, but now they are used in almost all sort of software projects like `.NET`, `Ruby`, `Node`, `Python` etc. as well as any kind of raw binary files. `Nexus` has got an enterprise version as well as a free OSS version available to choose from.

Nexus can run on all major OS (Windows, Linux, Mac), comes with greate compatibility with different tech stacks (Java, .NET, Ruby, Node, Python, Docker etc.) and has lot of useful features (grouped repositories, security, monitoring, reporting, automated mails and many more).

Alternatives? There are some popular alternatives as well

* [JFrog Artifactory](https://jfrog.com/artifactory/)
* [Proget](https://inedo.com/proget) etc.
* For .NET projects, a local automated [NuGet server](https://www.nuget.org/) can also be used, but they do not come with the bunch of features and flexibility of standard artifact repositories


#### Nexus basic concepts & terminology

Nexus generally has two types of repositories. So, any repository is **either of**

* Release repository - release specific, stable and incremented only on new release
* Snapshot repository - current, evolving, development repository

Repository Coordinates (or identifiers)

* Group Id: Identifier for a group of related artifacts (e.g. company.web.library)
* Artifact Id: The artifact identifier (e.g. ViewModels)
* Version: The standard version like 1.2.3.4 or 1.0.9-preview

Collectively they are called GAV coordinates.

These coordinates generally translate into a URI to point to the binaries, something like http://myrepo/my-group/library-name/package-1.0.dll

<u>Users and admin</u>

Nexus can have many user with different access rights. The default initial user is admin. To change or add new users, sign in with default admin credentials as `admin/admin123`. Now you can add new users, grant access to LDAP etc.


#### Installation

Since we'll be using Nexus for .NET project, we'll install it on a Windows system (becuase most .NET projects are still built on Windows systems. Not because this is the only system I have access to on day-to-day basis). To install on a different system, follow the [official installation guide](https://help.sonatype.com/display/NXRM3/Installation). 

I'm using the current latest OSS version `3.9.0-01`. Follow this step-by-step instructions

* Download the correct version [from here](https://www.sonatype.com/download-oss-sonatype)
* Extract to `C:/Nexus` (or any directory of choice with access). This extracted directorys are basically the Nexus application.
* It'll have two sub [directories](https://help.sonatype.com/display/NXRM3/Directories)
	* The installation directory `nexus-3.9.0-01` (conventionally `$install-dir`)
	* The data directory `sonatype-work` (conventionally `$data-dir`)
* In command prompt, go to directory  `$install-dir\bin`
* Run command `nexus.exe /run`
	* It will show all the log outputs. 
* If it starts successfully, it'll display at the end
    * **Started Sonatype Nexus OSS 3.9.0-01**

![Image](/images/posts/nexus/nexus-started.png)

**Note:** If it fails with an error like _"address already in use"_, that means the default URI `http://localhost:8081` is being used by some other application. So, we need to change the port.
{: .notice--info}

To change the application port (I changed mine to _9876_)

* Go to  $install-dir\etc\nexus-default.properties
* Change the line application-port=8081 to another port (e.g. 9876)

Once started successfully, the user interface can be accessed at http://localhost:9876/ (if not changed, default port is 8081)

![Image](/images/posts/nexus/nexus-oss-3.9.0.01.png)

If you log in with the admin user, you'll see additional option. The settings menu is on top with the gear icon.

![Image](/images/posts/nexus/snapshot-delete-task.png)

**Note:** For a production deployment, Nexus needs to be installed as a service, so that it can auto-restart and do other stuffs for more fail-safe operation. See instructions [here](https://books.sonatype.com/nexus-book/3.0/reference/install.html#service-windows).
{: .notice--info}

Also, to get automated mail alerts, setup the SMTP server in settings.

#### Nexus and NuGet

Wait wait, we do have `NuGet` which does the job of package management, then why do we need a fancy _"artifacts repository manager?"_ 

Well, the global NuGet server does it's job , but if we want to manage our own packages (for versioning, distribution and reuse etc.) we need to setup a local network NuGet server. While that is pretty doable, wouldn't it be great if you could have just one server infrastructure to hold & manage all (local & global) artifacts for all your development teams (think Maven, NuGet, npm, bower, raw binaries etc.) with additional benefits like monitoring, house keeping, security etc. If your answer is yes, Nexus is there to help you out. 

Using a `Nexus` server comes with bunch of benefits...

1. Host for own (personal/team/org) NuGet packages (pretty much like a local NuGet server)
2. One single source (see `nuget-group` below) or URI for all your local & global ([nuget.org](https://www.nuget.org/)) packages
3. Less network & storage use for nuget.org packages (in Nexus, packages are downloaded only once and cached. If the same package is needed again, the cached package is served & it does not download them from source again)
4. Custom or LDAP based access control
5. Manage other packages like `npm`, `bower` etc. with the same infrastructure and standards
6. Store and manage even raw binaries (e.g. any *.dll)
7. All other features of Nexus like monitoring, logging, reporting, security, custom tasks etc.

Nexus installation comes with preset NuGet settings. There are three repositories already setup for three different purposes, more can be added if required (To start with, I'm good with the preconfigured ones). 

* Proxy - To proxy the nuget.org global repository. See `nuget.org-proxy` in below image of default instance of Nexus
* Hosted - To host own packages, this works like a local NuGet server. `nuget-hosted`
* Group - A group to easily access multiple repositories, from a single source. The default `nuget-group` groups both the proxy and host above, and it can be configured to add more repositories.

![Image](/images/posts/nexus/initial.png)


#### Packaging NuGet libraries

To have our own NuGet packages hosted in our Nexus server, we need to `package` our .NET libraries as NuGet and then `publish` them to the Nexus server.

The packaging of custom NuGet packages doesn't have much to do with Nexus, they are the usual packaging process of NuGet as detailed in [Packaging and publishing your NuGet](https://docs.microsoft.com/en-gb/nuget/quickstart/create-and-publish-a-package-using-the-dotnet-cli) document. 

Here we'll see a quick step-by-step process of packaging libraries as NuGet. The process basically involves creating a `nuspec` file that defines the package metadata, and calling NuGet tools to create the actual package in `nupkg` format.

**Note:** I'm using a .NET Standard library with `dotnet cli` tools for the packaging. The process might be slightly different for creating .NET Framework libraries with NuGet console.
{: .notice--info}

* Create a new project or use an existing project (I used one of my existing `.NET Standard 2.0` class library project `LinqExtensions`)
* Add required minimal metadata needed for creating `nuspec` (an `xml` metadata file that describes the `NuGet` package. See below) in the `LinqExtensions.csproj` project file (inside existing `PropertyGroup` node). Alternatively a separate `nuspec` file can be created and linked.

```xml
<PackageId>LinqExtensions</PackageId>
<Version>1.0.0</Version>
<Authors>Arghya C</Authors>
<Company>Arghya C</Company>
```

* Run the `pack` [dotnet cli](https://docs.microsoft.com/en-us/dotnet/core/tools/?tabs=netcore2x) command **from the project directory**

```bash
$ dotnet pack
```

* It'll create the `NuGet` package `LinqExtensions.1.0.0.nupkg` inside `bin/Debug`. It's done and ready for use!
* Just to check, we can look at the `nuspec` file. The `nupkg` file is nothing but a compressed file. Uncompress it with your favourite tool and see the `LinqExtensions.nuspec` file. The contents of the file is shown below

```xml
<?xml version="1.0" encoding="utf-8"?>
<package xmlns="http://schemas.microsoft.com/packaging/2013/05/nuspec.xsd">
  <metadata>
    <id>LinqExtensions</id>
    <version>1.0.0</version>
    <authors>Arghya C</authors>
    <owners>Arghya C</owners>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description>Package Description</description>
    <dependencies>
      <group targetFramework=".NETStandard2.0">
        <dependency id="Newtonsoft.Json" version="11.0.1" exclude="Build,Analyzers" />
      </group>
    </dependencies>
  </metadata>
</package>
```


#### Publishing NuGet libraries

To host a NuGet package on server, we need to `publish` the local `nupkg` file to a NuGet enabled server (e.g. a network NuGet server or Nexus). Again, publishing to Nexus is not much different than standard NuGet server, only we need to setup the `NuGet API key`. The [NuGet chapter of the Nexus book](http://books.sonatype.com/nexus-book/3.0/reference/nuget.html) explains the process.

First we need to get the API key. The key is unique to each user. Login to the Nexus web application, go to the user section and click on the left menu `NuGet API key`. See image below. Remember, **do not share** your API key as that is the unique id given to you to verify with the Nexus server. (I've changed mine after taking the screensot. You have never seen my key!)

![Image](/images/posts/nexus/nuget-api-key.png)

**Note:** For the NuGet API key to work, first the admin needs to enable the _"NuGet API-Key Realm"_. To do that, go to Realms section in Security menu and add it to the active realms. See image below.
{: .notice--warning}

![Image](/images/posts/nexus/nuget-realm.png)

Now, all you need to do is run `nuget push` command from the package directory with the key obtained above, to the repository source. We'll be pushing the package to our `nuget-hosted` repository.

```bash
# command structure
$ dotnet nuget push {nupkg-name} -k {nuget-api-key} -s {repository-source-uri}
# actual command I issued
$ dotnet nuget push LinqExtensions.1.0.0.nupkg -k a0fdd1a1-af65-3ac9-ab28-c0b1bfadc82a -s http://localhost:9876/repository/nuget-hosted/
```

And voila! Our `NuGet`  is now available on our `Nexus` manager

![Image](/images/posts/nexus/hosted-nuget.png)


#### Integrating with Visual Studio

Now that we have our artifacts as NuGet package on our Nexus repository manager, we can use it simply as NuGet source for development and build purposes. Here, we'll setup our Nexus repository as NuGet source for Visual Studio.

Go to the Packagae Sources, and add a new source pointing to the `nuget-group` so that we can access the global nuget.org packages as well as our custom packages from the same source.

1. Go to `VS` > `Tools` > `Options` > `NuGet Package Manager` > `Package Sources`
2. Add a new source (the big bold green plus)
3. Since it'll access the global packages as well, only this source would be enough for projects

```bash
# Give a meaningful name and the Nexus repository source
Name: Nexus-nuget-group
Source: http://localhost:9876/repository/nuget-group/
```

![Image](/images/posts/nexus/vs-nuget-source.png)

And now (along with nuget.org packages) my `LinqExtensions` package is also available as `NuGet` for all my projects!

![Image](/images/posts/nexus/my-nuget-found.png)


#### References

* [What is artifact repository?](https://blog.sonatype.com/2009/04/what-is-a-repository) 
* [Sonatype Nexus](https://www.sonatype.com/nexus-repository-sonatype)
* [The Nexus book](https://books.sonatype.com/nexus-book/3.0/reference/install.html)
* [Official installation guide](https://help.sonatype.com/display/NXRM3/Installation)
* [Basic setup example](http://www.vineetmanohar.com/2010/06/getting-started-with-nexus-maven-repo-manager/)
* [The user interface](https://help.sonatype.com/display/NXRM3/User+Interface)
* [Admin and configuration](https://help.sonatype.com/display/NXRM3/Configuration)
* [Install as service for production](https://books.sonatype.com/nexus-book/3.0/reference/install.html#service-windows)
* [.NET Package Repositories with NuGet](https://help.sonatype.com/display/NXRM3/.NET+Package+Repositories+with+NuGet)