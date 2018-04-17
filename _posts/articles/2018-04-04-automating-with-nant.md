---
layout: post
title: "Automating build & tasks with NAnt"
excerpt: "Building .NET projects and much more with good old NAnt"
date: 2018-04-04
tags: [nant, nantcontrib, dotnet, automation, installation, setup, build, tasks]
categories: articles
image:
  feature: posts/nant/nant_script2.png
comments: true
share: true
---

###### What is NAnt

`NAnt` is an automated build tool for `.NET` based applications. That means, you can write some scripts in NAnt and let it build your projects without any manual intervention. You can also trigger the script from batch jobs for CI (Continuous Integration) systems.

But why? There is `Visual Studio` which can build (compile) all .NET projects, what is the need for a separate build tool?

To build a project/solution from Visual Studio, you need to open it, open the project, click on `Build` etc. That is doable in the development environment where the developer is actually coding the application with Visual Studio IDE. But, if we want to

1. Trigger the build based on some pre-defined events (e.g. nightly job or a code check-in)
2. Build on a system without Visual Studio installed on it
3. Do other stuffs like - creating output directories, running unit tests, deploying a site on IIS etc.

Visual Studio does not prove to be very useful. There we use some tool to automatically kick-off a build process, and automate other tasks around it. These tools are build tools like [NAnt](http://nant.sourceforge.net/) or [MSBuild](https://msdn.microsoft.com/en-us/library/dd393574.aspx).

#### Why use NAnt for building .NET projects

NAnt is a build tool for .NET application (based on [Ant](http://ant.apache.org/) for Java), that can help in building .NET application, run unit tests, do configuration and lot more things. It is actually older than `MSBuild` and arguably, the most widely used build tool in the .NET community.

Though NAnt is used mostly as a build tool, because of its generic nature and lot of pre-configured tasks, it can be used simply as an automation tool as well! Some of the main features or NAnt are

* Free to use, and [open source](https://github.com/nant/nant)
* Runs on `Windows` and `Linux` systems (based on [Mono](https://www.mono-project.com/) framework)
* Build scripts are basically `XML` files in specific format (with `.build` extension)
* Workflow is created with a series of _"tasks"_
* It can run `NAnt` specific tasks, and any other command-line tools with `<exec>` task

NAnt also comes pre-included with a bunch of open source libraries. Most recent versions of NAnt distribution includes - [NUnit](https://sourceforge.net/projects/nunit/) for unit testing, [NDoc](http://ndoc.sourceforge.net/) for documentation, [SharpZipLib](http://www.icsharpcode.net/OpenSource/SharpZipLib/) for binary compression etc.

###### But, is NAnt still relevant?

From the [NAnt official site](http://nant.sourceforge.net/) we can see the last release `v0.92` was in 2012, and it never made it to `1.0`. Is it still usable?

Though the project was not updated recently, it is completely stable and usable for production release. All the useful tasks are already included in NAnt and the community supported [NAntContrib](http://nantcontrib.sourceforge.net/) project. And anything else can be run simply as a normal command-line application with `exec` task. And, probably, most teams still use NAnt behind their build or CI system.

#### Installing NAnt

NAnt has binary and source distributions. Binary is enough to setup and run builds. We'll install it on a `Windows` based system, for other platforms or more details, see documentation [here](http://nant.sourceforge.net/release/latest/help/introduction/installation.html).

* Download the current version from [here](http://nant.sourceforge.net/) - see "Releases" menu on the left, e.g. nant-0.92-bin.zip
* Extract the contents to a directory where you have access. e.g. _"C:\nant"_
* Add this path to _"System Environment PATH"_ for easy access. Once done, command `nant -help` should work
* OR, in case you do not want to add another new path, create a wrapper over `nant.exe` on a system accessible path. For instance, create a batch file as `nant.bat` and save it inside (for example) `C:\Windows` with following content (make sure the exe path is correct)

```bash
@echo off
"C:\nant\bin\NAnt.exe" %*
```

**Note:** The NAnt configuration is present in _"C:\nant\bin\NAnt.exe.config"_ (based on your installation path). You can check and change some configurations, if needed e.g. default .NET framework. Also, **global properties** can be set in this file.
{: .notice--info}

###### Installing NAntContrib

NAntContrib is another community driven tool that adds lot of additional features (tasks) to the NAnt like adding binaries to GAC, checking-out source code, creating IIS virtual directory etc. At some point, you'll definitely need it to complement NAnt.

* Download the latest version from [here](http://nantcontrib.sourceforge.net/) - see "Releases" menu on the left, e.g. 0.92
* Download the required package, e.g. `nantcontrib-0.92-bin.zip`
* Save them in a directory with access, e.g. `C:\nantcontrib-0.92\bin`
* Now _"ideally"_ you _"should"_ be able to use it by adding a `<loadtasks>` in your build file with the location of the NAntContrib assemblies, like

```xml
<loadtasks>
    <fileset>
        <include name="C:/nantcontrib-0.92/bin/**/*.dll" />
    </fileset>
</loadtasks>
```

But that may not work, as it didn't for me!

**Note:** You'll most probably get an error like "Failure scanning CollectionGen.dll for extensions. Could not load file or assembly 'Microsoft.VSDesigner, Version=7.0.3300.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a' or one of its dependencies. The system cannot find the file specified."
{: .notice--danger}

Unfortunately the solution is not clear from the official documentation. Here _**you'll need some weird wizard skills to make it work!!**_ Thanks to posts like [this](http://byatool.com/lessons/how-to-installusing-nantcontrib-because-the-documentation-sucks/) and [this](https://sourceforge.net/p/nantcontrib/bugs/84/) for the solution.

**Solution** to _"CollectionGen.dll"_ error

1. Copy `CollectionGen.dll`, and the two `SLiNgshoT` assemblies in a new `tasks` folder inside `nant\bin` directory e.g. _"C:\nant\bin\tasks"_
2. Copy all the other (main) assemblies, including `NAnt.Contrib.Tasks` directly inside `nant\bin` directory e.g. _"C:\nant\bin"_
3. Now these tasks will be part of all the NAnt scripts by default

#### Anatomy of NAnt build files

NAnt runs based on `.build` files. Each files define a bunch of tasks grouped in targets, and optionally some properties. Then NAnt can run one or more targets based on instructions. Here we'll look at some basic constructs of a build file.

Each `NAnt` build file is an `XML` file with bunch of command and configuration.

###### `<project>`

* Build files will have `<project>` as the root element
* Basically each file is a NAnt project
* Project can have optional `description` and `basedir` which will be base root path for all relative paths
* Project generally defines the `default` target to be executed

###### `<property>`

* A project can define as many properties as it wants
* Properties are like variables with a `name` and `value`
* Property names are case-sensitive & can include alphabets, numbers, hyphen, underscore and dot (.). The value of a property can be used as `${property.name}`. See the property documentation [here](http://nant.sourceforge.net/release/0.85/help/fundamentals/properties.html)
* Value of a property can be passed as command line argument while running NAnt as `-D:prop.name=value`. Properties can also be set to be read-only by specifying `readonly="true"`
* Changing value of a property in the script is like declaring the same property again with new value

**Note:** The properties those are set with command-line arguments become `read-only`, and cannot be modified inside the NAnt script
{: .notice--info}

###### `<target>`

* A `<target>` is a single or set of tasks, with a name & optional `description` and other attributes. Generally, a target represents a complete step in the whole build process
* Target can optionally have `unless` or `if` attributes to control execution based on some condition. If `if` evaluates to `true`, the target executes, else that is skipped. The `unless` works the opposite way
* The optional `depends` attribute creates a dependency on another target, such that current target will not run until that target is executed i.e. the depends-on target will be run before running current task

###### Expressions & functions

* A NAnt script can also have [expressions](http://nant.sourceforge.net/release/0.85/help/fundamentals/expressions.html) like `if` condition
* And simple function call like `${datetime::now()}`.
* Functions can be called as `${prefix::func-name(arg1, ..., argN)}`, see [reference](http://nant.sourceforge.net/release/0.92/help/fundamentals/functions.html)
* Functions can use properties by the name like `${path::combine(src.dir, 'app.sln')}`

###### Tasks

* A `task` is the smallest unit of work in NAnt, it is just one command within a `<target>`. A task can do stuffs like creating or deleting a directory, building a set of C# files or projects, send email, calling another `target` etc. There is big bunch of useful tasks included in NAnt by default.
* Almost anything that can be run from the command line, can be executed with a `<exec>` task
* Each task has an optional `failonerror` boolean attribute which says where the build should fail on failure of this task or not. The default is to fail the build `failonerror="true"`
* Many more additional tasks are available with the community driven project [NAntContrib](http://nantcontrib.sourceforge.net/) like interacting with source control, building with `MSBuild` etc.
* Custom tasks can also be built with C# or VB. See the abstract base class [here](https://github.com/nant/nant/blob/master/src/NAnt.Core/Task.cs), and some [sample code](https://stackoverflow.com/questions/140616/is-there-a-nant-task-that-will-display-all-property-name-values) to print all the properties, including the default ones!

#### Running NAnt scripts

Let's look at a very basic NAnt script, and what options we have for running that

```xml
<?xml version="1.0"?>
<project name="Hello World" default="time">
  <property name="user.name" value="friend" />
  <target name="hello" description="Says hello to user">
    <echo message="Hello ${user.name}" />
  </target>
  <target name="time" description="Says current time" depends="hello">
    <echo message="The current time is ${datetime::now()}" />
  </target>
</project>
```

When a NAnt script is run with `nant` command

* NAnt looks for a file with `.build` extension. If there are more than one, it takes `default.build`. One can also specify `-buildfile:<build-filename>` (or `-f:`) command line argument> If it cannot find/specify the build file, it'll exit with error.
* The default `<target>` is specified as `default` attribute at `<project>` level, or it can be passed as runtime argument just by specifying the name
* There are many options for running a NAnt script (in the following examples, nant is being run in the same directory where the build files are)

```bash
# single or default build file and default target
$ nant
# single or default build file with specific target
# can specify multiple targets, separated by space
$ nant hello
# with specific build file and default target, -f: also works
$ nant -buildfile:hello.build
# with specific build file and specific target
$ nant -f:hello.build hello
# passing a property value, enclose in quotes in value has spaces
$ nant -D:user.name=Arghya
```

**Note:** If no default target is mentioned in .build file and no target specified as command argument, NAnt will simply exit without doing anything.
{: .notice--warning}

#### Building a .NET project

There are multiple options for building a .NET project with NAnt.

The `<csc>` task can be used to compile one or more C# files with the C# compiler. Similar tasks are available for `VB`, `F#` etc. When [csc](http://nant.sourceforge.net/release/0.85/help/tasks/csc.html) task is used, it'll build the files as mentioned in `<sources>` and create assemblies specified in `target` and `output` attributes.

Specifying a big list of files is cumbersome and will need to be updated every time the actual code is updated. But, generally we have `*.csproj` or `*.sln` files that defines the whole hierarchy of files, it's dependencies etc. So, it makes complete sense to use that directly rather than manually creating the same setup again in the `.build` file.

There is a `<solution>` task that can directly build a solution or one or more project files. See more details [here](http://nant.sourceforge.net/release/0.91/help/tasks/solution.html).

The limitation of the `<solution>` task is, it cannot build solutions that are created Visual Studio versions later that 2003! So, for those projects, the easiest solution is to use the `<msbuild>` task from NAntContrib. See this [NAntCOntrib MSBuild reference](http://nantcontrib.sourceforge.net/release/0.85/help/tasks/msbuild.html).

**Note:** While building a newer (VS 2012+) solution file (generally a MVC web project) with NAntContrib `msbuild` task, you may face this error: "[msbuild] err or MSB4019: The imported project "C:\Program Files (x86)\MSBuild\Microsoft \VisualStudio\v11.0 \WebApplications \Microsoft.WebApplication.targets" was not found. Confirm that the path in the \<Import\> declaration is correct, and that the file exists on disk."
{: .notice--danger}

This happens for a target mismatch with MSbuild versions, as explained [here](https://stackoverflow.com/questions/3980909/microsoft-webapplication-targets-was-not-found-on-the-build-server-whats-your) and [here](https://stackoverflow.com/questions/17433904/v11-0-webapplications-microsoft-webapplication-targets-was-not-found-when-file-a). There are two basic solutions to problem.

1. To solve for a specific project/solution, add the [Web.targets](https://www.nuget.org/packages/MSBuild.Microsoft.VisualStudio.Web.targets/) `NuGet` to the project/solution.
2. To solve it for all builds on the machine. Simply copy the targets from a compatible MSBuild location to target MSBuild directory. For example, copy from _"C:\Program Files (x86)\ MSBuild\Microsoft\VisualStudio \v10.0\WebApplications"_ to _"C:\Program Files (x86)\ MSBuild\Microsoft\VisualStudio \v11.0\WebApplications"_

###### Running MSBuild.exe directly with \<exec\>

The problem with NAntContrib `<msbuild>` task is, it is not updated to work with newer solution or project formats & framework versions. The .NET framework and Visual Studio keeps updating almost every year, and so does the MSBuild. So, at some point the msbuild task will get outdated, if it isn't already!

The solution is to directly run the `MSBuild.exe` as a command-line executable with `<exec>` command. This way we need not care about the .NET framework or the solution version, we just need to update the path to the exe every time we need to upgrade. See the sample target below.

```xml
<property name="build.dir" value="C:\Publish\MyApp" />
<property name="build.config" value="debug" />
<property name="pathto.solution" value="C:\CodeBase\MyApp\MyApp.Core.sln" />
<property name="pathto.msbuild" value="C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\MSBuild\15.0\Bin\MSBuild.exe" />

<target name="build">
  <exec program="${pathto.msbuild}" verbose="true">
    <arg line="${pathto.solution}" />
    <arg value="/property:Configuration=${build.config}"/>
    <arg value="/property:OutDir=${build.dir}"/>
  </exec>
</target>
```

#### Alternatives to NAnt

For building your project to automating everyday tasks, you are not bound to NAnt. There are plenty of other tools which can do the same things. Some of the options are

1. [MSBuild](https://msdn.microsoft.com/en-us/library/dd393574.aspx) is a close competitor, uses similar XML based build files. It is created and maintained by Microsoft, and the `.csproj` files are basically MSBuild files! But NAnt (with NAntContrib) is even older than MSBuild, and in some cases it seems to be slightly more flexible.
2. [Rake](https://github.com/ruby/rake) (aka Ruby Make) is a `Ruby` based build tool, that can [also be used for .NET](http://www.codemag.com/article/1006101/Building-.NET-Systems-with-Ruby-Rake-and-Albacore). Some drawbacks are - needs installing Ruby and related stuffs, scripts are also written in Ruby which is not native to .NET developers.
3. [psake](https://github.com/psake/psake) another build tool written in `PowerShell`. Good thing is .NET is natively supported and language is similar to simple command-line.
4. [Cake](https://cakebuild.net/) is another good option. The good things about it are - C# code for build and fully cross-platform. I'll probably explore more on it and see it as a possible replacement of my current build tool.

Meanwhile I'll continue to use NAnt mostly because the infrastructure is already setup and working fine in my build system.

For a complete and runnable NAnt script that does bunch of stuffs for a CI build system, see this **[complete runnable NAnt script](/articles/sample-nant-script-for-ci/)**.

#### References

* [NAnt help reference](http://nant.sourceforge.net/release/latest/help/)
* [NAntContrib](http://nantcontrib.sourceforge.net/)
* [NAnt fundamentals](http://nant.sourceforge.net/release/0.85/help/fundamentals/)
* Quick start articles on - [4guysfromrolla](http://www.4guysfromrolla.com/articles/120104-1.aspx), [Code magazine](http://www.codemag.com/Article/0507081/Building-.NET-Applications-with-NAnt), [DZone](https://dzone.com/articles/getting-started-nant-automatio)
* All tasks list - [NAant core](http://nant.sourceforge.net/release/latest/help/tasks/), [NAntContrib](http://nantcontrib.sourceforge.net/release/latest/help/tasks/)
* MSBuild reference - [Using MSBuild](https://msdn.microsoft.com/en-us/library/dd393573.aspx), [Command-Line Reference](https://msdn.microsoft.com/en-us/library/ms164311.aspx)