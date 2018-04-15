---
layout: post
title: "nAnt"
excerpt: "Automating tasks with NAnt"
date: 2018-04-04
tags: [nant, nantcontrib, dotnet, automation, installation, setup, buildfile, tasks]
categories: articles
comments: true
share: true
published: false
---

# A build tool for .NET and more

NAnt is a build tool for .NET application (based on [Ant](http://ant.apache.org/) for Java), that can help in building .NET application, run unit tests, do configuration etc.

* Free to use, and [open source](https://github.com/nant/nant)
* Runs on build files which are XML based, and has `.build` extension
* Workflow is created with a hierarchy of _"tasks"_
* It can run `NAnt` specific tasks, and any other command can be run with `<exec>` task, based on the `OS`

NAnt uses a number of open source third party libaries.  Recent versions are included included with the NAnt distribution, for example - [NUnit](https://sourceforge.net/projects/nunit/) for unit testing, [NDoc](http://ndoc.sourceforge.net/) for documentation, [SharpZipLib](http://www.icsharpcode.net/OpenSource/SharpZipLib/) for binary compression etc.

#### Installation

The working system must meet [system requirements](https://github.com/nant/nant#prerequisites).

NAnt has binary and source distributions. Binary is enough to setup and run builds. See details [here](http://nant.sourceforge.net/release/latest/help/introduction/installation.html).

* Download the current version from [here](http://nant.sourceforge.net/) - see "Releases" menu on the left, e.g. nant-0.92-bin.zip
* Extract the contents to a directory where you have access. e.g. _"C:\nant"_
* Add this path to _"System Environment PATH"_ for easy access. Once done, command `nant -help` should work
* OR, in case you do not want to add another new path, create a wrapper over `nant.exe` on a system accessible path. So, create a batch file as `nant.bat` and save it inside (for example) `C:\Windows` with following content (make sure the path is correct)

```bash
@echo off
"C:\nant\bin\NAnt.exe" %*
```

**Note:** The NAnt configuration is present in _"C:\nant\bin\NAnt.exe.config"_ (based on your installation path). You can check and change some configurations, if needed e.g. default .NET framework. Also, **global properties** can be set in this file.
{: .notice--info}

###### Installing NAntContrib

NAntContrib is another tool that adds lot of additional features (tasks) to the NAnt like adding binaries to GAC, checking-out source code, creating IIS virtusal directory etc. One of the most useful one is the `msbuild` task .NET projects/solutions created with Visual Studio with version higher than 2003!

* Download the latest version from [here](http://nantcontrib.sourceforge.net/) - see "Releases" menu on the left, e.g. 0.92
* Download the required package, e.g. `nantcontrib-0.92-bin.zip`
* Save them in a directory with access, e.g. `C:\nantcontrib-0.92\bin`
* Now ideally you "should" be able to use it by adding a `<loadtasks>` in your build file with the location of the NAntContrib assemblies, like

```xml
<loadtasks>
    <fileset>
        <include name="C:/nantcontrib-0.92/bin/**/*.dll" />
    </fileset>
</loadtasks>
```

**Note:** You'll most probably get an error like "Failure scanning CollectionGen.dll for extensions. Could not load file or assembly 'Microsoft.VSDesigner, Version=7.0.3300.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a' or one of its dependencies. The system cannot find the file specified."
{: .notice--danger}

 Here _**you'll need some weird wizard skills to make it work!!**_ There are two ways of getting it to work, none of which is clear from the documentations! Thanks to posts like [this](http://byatool.com/lessons/how-to-installusing-nantcontrib-because-the-documentation-sucks/) and [this](https://sourceforge.net/p/nantcontrib/bugs/84/) for the solution.

**Option 1:** Adding `<loadtasks>` to each main build file

1. Copy `CollectionGen.dll`, and the two `SLiNgshoT` assemblies in a new folder `bin\tasks`
2. Copy all the other (main) assemblies, including `NAnt.Contrib.Tasks` in a new folder `bin\lib`
3. Add `<loadtasks>` to the begining of your build file targetting the `lib` directory that has the tasks

```xml
<loadtasks>
    <fileset>
        <include name="C:/nantcontrib-0.92/bin/lib/**/*.dll" />
    </fileset>
</loadtasks>
```

**Option 2:** (Preferred) Adding NAntContrib tasks to NAnt default tasks list

1. Copy `CollectionGen.dll`, and the two `SLiNgshoT` assemblies in a new `tasks` folder inside `nant\bin` e.g. _"C:\nant\bin\tasks"_
2. Copy all the other (main) assemblies, including `NAnt.Contrib.Tasks` directly inside `nant\bin` e.g. _"C:\nant\bin"_
3. Now these tasks will be part of all the `nant` scripts by default

#### Build files

Each `NAnt` build file is an `XML` file with bunch of command and configuration.

* Files will have one **`<project>`** root element
* And one or more `<property>` and `<target>`
* A **`<property>`** is a avariable with name & value. Property names are case-sensitive & can include alphabets, numbers, hyphen, underscore and dot (.). The value of a property can be used as `${property.name}`. See the property documentation [here](http://nant.sourceforge.net/release/0.85/help/fundamentals/properties.html).
* Value of a property can be passed as command line argument while running NAnt. Properties can be set to be readonly by specifying `readonly="true"`
* A **`<target>`** is a single or set of tasks, with a name & optional `description` and other attributes. Generally, a `<target>` represents a step in the whole build process
* Target can optionally have `unless` or `if` attributes to control execution based on some condition. If `if` evaluates to `true`, the target executes, else that is skipped. The `unless` works the the opposite way
* The optional `depends` attribute creates a dependency on another target, such that current target will not run until that target is executed

Example - TO CHANGE <<

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

* NAnt looks for a file with `.build` extension. If there are more, it takes `default.build`. One can also specify `-buildfile:<build-filename>` command line argument
* The default `<target>` is specified as `default` attribute at `<project>` level, or it can be passed as runtime argument
* A `task` is just one command within a `<target>`. A task can do stuffs like creating or deleting a directory, building a set of C# files or projects, send email, calling another `target` etc. There is big bunch of useful tasks included in NAnt by default.
* Almost anything that can be run from the command line, can be executed with a `<exec>` task
* Each task has an optional `failonerror` boolean attribute which says where the build should fail on failure of this task or not. The dafault is to fail the build `failonerror="true"`
* Many more additional tasks are available with another tool [NAntContrib](http://nantcontrib.sourceforge.net/) like intercating with source control, building with `MSBuild` etc.
* Custom tasks can also be built with C# or VB. See the abstract base class [here](https://github.com/nant/nant/blob/master/src/NAnt.Core/Task.cs), and [sample code](https://stackoverflow.com/questions/140616/is-there-a-nant-task-that-will-display-all-property-name-values) to print all the properties, including the default ones!
* Basic NAnt run options (assuming the build file is in directory c:\\builddirectory)

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

* A NAnt script can also have [expressions](http://nant.sourceforge.net/release/0.85/help/fundamentals/expressions.html) like `if` condition or simple function call like `${datetime::now()}`.
* Functions can be called as `${prefix::func-name(arg1, ..., argN)}`
* Functions can use properties simply by the name like `${path::combine(src.dir, 'app.sln')}`

**Note:** If no default target is mentioned in .build file and no target specified as command argument, NAnt will simply exit without doing anything.
{: .notice--warning}

#### Building a project

The `<csc>` task can be used to compile one or more C# files. Similar tasks are available for `VB`, `F#` etc.

But, generaly we have `*.csproj` or `*.sln` files that defines the whole hierarchy of files, it's dependencies etc. There is a `<solution>` task that can directly build a solution or one or more project files.

**Note:** The `solution` task does not work for projects that created after VS2003. For that, best choice is the [msbuild task from NAntContrib](http://nantcontrib.sourceforge.net/release/0.85/help/tasks/msbuild.html)
{: .notice--info}

#### The 'msbuild' task

-- details & example --

**Note:** While building a newer (VS2012+) solution file (generally a MVC web project) with NAntContrib `msbuild` task, you may face this error: "[msbuild] err or MSB4019: The imported project "C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v11.0\WebApplications\Microsoft.WebApplication.targets" was not found. Confirm that the path in the \<Import\> declaration is correct, and that the file exists on disk."
{: .notice--danger}

This happens for a target mismatch with MSbuild versions. See details [here](https://stackoverflow.com/questions/3980909/microsoft-webapplication-targets-was-not-found-on-the-build-server-whats-your) and [here](https://stackoverflow.com/questions/17433904/v11-0-webapplications-microsoft-webapplication-targets-was-not-found-when-file-a). Again, there are two basic solutions to problem.

1. Option #1 - Solving per project/solution - adding the [Web.targets](https://www.nuget.org/packages/MSBuild.Microsoft.VisualStudio.Web.targets/) to the project/solution.
2. Option #2 - (Preferred) Solving for all builds on the machine. Simply copy the targets from a compatible MSBuild location to target MSBuild directory. For example, copy from _"C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v10.0\WebApplications"_ to _"C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v11.0\WebApplications"_

#### References

* [NAnt help reference](http://nant.sourceforge.net/release/latest/help/)
* [NAntCOntrib](http://nantcontrib.sourceforge.net/)
* [NAnt fundamentals](http://nant.sourceforge.net/release/0.85/help/fundamentals/)
* Quick start articles on - [4guysfromrolla](http://www.4guysfromrolla.com/articles/120104-1.aspx), [Code magazine](http://www.codemag.com/Article/0507081/Building-.NET-Applications-with-NAnt), [DZone](https://dzone.com/articles/getting-started-nant-automatio)
* [All NAnt built-in functions](http://nant.sourceforge.net/release/0.85/help/functions/index.html)
* [Sample ?](https://dzone.com/articles/getting-started-nant-automatio)
* All tasks list - [NAant core](http://nant.sourceforge.net/release/latest/help/tasks/), [NAntContrib](http://nantcontrib.sourceforge.net/release/latest/help/tasks/)
* MSBuild reference - [Using MSBuild](https://msdn.microsoft.com/en-us/library/dd393573.aspx), [Command-Line Reference](https://msdn.microsoft.com/en-us/library/ms164311.aspx)