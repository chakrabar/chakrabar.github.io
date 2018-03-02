---
layout: post
title: ".NET artifacts repository management with Nexus"
excerpt: "Setup & configuration of Sonatype Nexus as a artifacts repository manager for .NET projects"
date: 2018-03-02
tags: [tech, artifact, repository, nexus, package, nuget, dotnet]
categories: articles
comments: true
share: true
---

An artifacts what?

For those not familiar with the very popular _"artifact repositories"_ used all over the development community, it might be a pretty unsual thing. But they have been used in many open source and commercial software projects for many years - for a reason. Probably not super popular with the .NET community (_well, we have NuGet, don't we?_), but they are being used increasingly in many .NET projects. And the popular ones now come with .NET support by default.

So what is an **artifacts repository manager**. Going by the standard definition

> An artifacts repository is basically an application server that can store software artifacts in an organised and versioned manner, and that can be used for software projects as and when required. They are generally wired into application IDEs and build systems to store or fetch project artifacts (binaries, metadata etc.)

In all software projects, we have to deal with bunch of artifacts like code binaries, metadata files, configurations, xml files etc. In projects with size moderate+ they can become a handful with versions, inter dependencies and all. It can make everyday development, sharing code, building software pretty cumbursome. Here an _artifacts repository manager_ comes to rescue. It simple _manages repositories of artifacts_, where artifacts can be in-house or third party (e.g. NuGet packages for .NET projects).

So, it mainly does two things

* Manage & distribute software artifacts (project binaries and more)
* Proxy remote repositories (e.g. NuGet/Maven) to a local/network server (optional tbh)

Put together, they help improve development, build and distribution of software systems.


#### Sonatype Nexus artifacts repository manager

[Sonatype Nexus](https://www.sonatype.com/nexus-repository-sonatype) is one of the most popular artifacts repository managers available in market today. It was initially made popular with `Maven` projects for `Java`, but now they are used in almost all sort of software projects like `.NET`, `Ruby`, `Node`, `Python` etc. as well as any kind of raw binary files. `Nexus` has got an enterprise version as well as a free OSS version available to choose from.

Nexus can run on all major OS (Windows, Linux, Mac), comes with greate compatibility with different tech stacks (Java, .NET, Ruby, Node, Python, Docker etc.) and has lot of useful features (grouped repositories, security, monitoring, reporting and many more).

Alternatives? There are some popular alternatives as well

* [JFrog Artifactory](https://jfrog.com/artifactory/)
* [Proget](https://inedo.com/proget)
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
* Extract to `C:/Nexus` (or any folder of choice with access). This extracted folders are basically the Nexus application.
* It'll have two sub [directories](https://help.sonatype.com/display/NXRM3/Directories)
	* The installation directory `nexus-3.9.0-01` (conventionally `$install-dir`)
	* The data directory `sonatype-work` (conventionally `$data-dir`)
* In command prompt, go to folder  $install-dir\nexus-3.9.0-01\bin
* Run command `nexus.exe /run`
	* It will show all the log outputs. 
* If it starts successfully, it'll display at the end
    * **Started Sonatype Nexus OSS 3.9.0-01**

**Note:** If it fails with an error like _"address already in use"_, that means the default URI `http://localhost:8081` is being used by some other application. So, we need to change the port.
{: .notice--info}

To change the application port (I changed mine to _9876_)

* Go to  $install-dir\etc\nexus-default.properties
* Change the line application-port=8081 to another port (e.g. 9876)

Once started successfully, the user interface can be accessed at http://localhost:9876/ (if not changed, default port is 8081)

![Image](/images/posts/nexus/nexus-oss-3.9.0.01.png)

If you log in with the admin user, you'll see additional option. The settings menu is on top with the gear sign.

![Image](/images/posts/nexus/snapshot-delete-task.png)

**Note:** For a production deployment, Nexus needs to be installed as a service, so that it can auto-restart and do other stuffs for more fail-safe operation. See instructions [here](https://books.sonatype.com/nexus-book/3.0/reference/install.html#service-windows).


#### Nexus and NuGet

* Proxy
* Hosted

Benefits

1. Host for own (personal/team/org) NuGet packages
2. One single source (nuget-group) or URI for local & global ([nuget.org](https://www.nuget.org/)) packages
3. Less network & storage use for nuget.org packages (packages are downloaded only once and cached)
4. Custom or LDAP based access control

#### Packaging NuGet libraries

[.NET Package Repositories with NuGet - book](http://books.sonatype.com/nexus-book/3.0/reference/nuget.html)

[Packaging and publishing your NuGet](https://docs.microsoft.com/en-gb/nuget/quickstart/create-and-publish-a-package-using-the-dotnet-cli)

* Create a new project or use an existing project (I used one of my existing `.NET Standard 2.0` class library project `LinqExtensions`)
* Add required minimal metadata needed for creating `nuspec` (an `xml` metadata file that describes the `NuGet` package. See below) in the `LinqExtensions.csproj` project file (inside existing `PropertyGroup` node)

```xml
<PackageId>LinqExtensions</PackageId>
<Version>1.0.0</Version>
<Authors>Arghya C</Authors>
<Company>Arghya C</Company>
```

* Run the `pack` [dotnet cli](https://docs.microsoft.com/en-us/dotnet/core/tools/?tabs=netcore2x) command **from the project folder**

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

```bash
# command structure
$ dotnet nuget push {nupkg-name} -k {nuget-api-key} -s {repository-source-uri}
# actual command I issued
$ dotnet nuget push LinqExtensions.1.0.0.nupkg -k a0fdd1a1-af65-3ac9-ab28-c0b1bfadc82a -s http://localhost:9876/repository/nuget-hosted/
```

And voila! Our `NuGet`  is now available on our `Nexus` manager

![Image](/images/posts/nexus/hosted-nuget.png)

#### Integrating with Visual Studio

```bash
VS > Tools > Options > NuGet Package Manager > Package Sources
# Give a meaningful name and the Nexus repository source
Name: Nexus-nuget-group
Source: http://localhost:9876/repository/nuget-group/
```

![Image](/images/posts/nexus/vs-nuget-source.png)

And now (along with nuget.org packages) my `LinqExtensions` package is also available as `NuGet` for all my projects!

![Image](/images/posts/nexus/my-nuget-found.png)

#### References

* [What is a artifact repository?](https://blog.sonatype.com/2009/04/what-is-a-repository) 
* [Sonatype Nexus](https://www.sonatype.com/nexus-repository-sonatype)
* [The Nexus book](https://books.sonatype.com/nexus-book/3.0/reference/install.html)
* [Official installation guide](https://help.sonatype.com/display/NXRM3/Installation)
* [Basic setup example](http://www.vineetmanohar.com/2010/06/getting-started-with-nexus-maven-repo-manager/)
* [The user interface](https://help.sonatype.com/display/NXRM3/User+Interface)
* [Admin and configuration](https://help.sonatype.com/display/NXRM3/Configuration)
* [Install as service for production](https://books.sonatype.com/nexus-book/3.0/reference/install.html#service-windows)
* [.NET Package Repositories with NuGet - site](https://help.sonatype.com/display/NXRM3/.NET+Package+Repositories+with+NuGet)