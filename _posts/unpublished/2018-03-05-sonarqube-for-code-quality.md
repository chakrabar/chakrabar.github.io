---
layout: post
title: "SonarQube for quality analysis of software codes"
excerpt: "let's see..."
date: 2018-03-05
tags: [tech, dotnet, codequality, staticanalysis, codemetrics, unittests, codecoverage, sonar, sonarqube]
categories: articles
comments: true
share: true
published: false
---

# SonarQube benefits

open source
multi language support (Java, C#, Python, JavaScript, C++, TypeScript, XML) [details](https://docs.sonarqube.org/display/PLUG/Plugin+Library)
features
    static analysis on source code
    Static analysis on compiled code (selected languages, line Java, C#)
    code complexity ??
    duplication ??
    code coverage ??
    historical analysis ??
VS (Eclipse etc) integration - SonarLint
Build tool integration - MSBuild, Ant, Maven, Gradle etc.
extensibility

Authentication & authorization [options](https://docs.sonarqube.org/display/PLUG/Plugin+Library) - LDAP, Azure, Google, GitHub, GitLab etc.
Source control - SVN & Git are supported out of the box
From site

Supported project types and build systems

* Easy analysis of any existing Visual Studio Solution or MSBuild project
* Native integration with any existing build in TFS or VSTS

Metrics

* Code coverage by tests: SonarC# supports the import of Microsoft Visual Studio, dotCover, OpenCover, and NCover 3 test coverage reports.

Custom Rules

* SonarC# supports custom rules written in Roslyn, and packaged via the SonarQube Roslyn SDK project.

#### SonarQube infrastructure

SonarQube needs three components to work, `SonarQube server`, `SonarQube Scanner` and language pugin

* SonarQube Scanner - language specific scanner that actualy runs analysis on the code
* SonarQube server - where data is sent back after analysis, where the processed data is analysed further and reports, insights are created
* Language plugin - language specific plugin for the the server (e.g. SonarC#, SonarJava, SonarPython etc.). Some of the language plugins are installed on the server by default, while others can be installed separately. [Details here](https://docs.sonarqube.org/display/SONAR/Marketplace).

#### SonarQube setup process for development

We'll setup the infrastructure on a `Windows` system. They can be setup on `Linux` and `Mac OS` as well. We'll run analysis on a `.NET` project, so we'll use `SonarC#` plugin and MSBuild runner. Seeting up and running for other languages and project types will have a similar but different setup process.

<u>First, let's install and run the **SonarQube server**</u>

1. Download [SonarQube here](https://www.sonarqube.org/downloads/). I got the current latest _"SonarQube 7.0"_
2. Extract the contents of the zip file to a directory with access (e.g. `C:\sonarqube`)
3. Go inside `bin` folder and find the correct directory as per the system (e.g. `bin\windows-x86-64`)
4. Run the `StartSonar.bat` bat file (double-click or run from command line)
5. It may take a few minutes to run. Once started successfully, it'll show a message like _"jvm 1    \| 2018.03.08 17:01:12 INFO  app[][o.s.a.SchedulerImpl] SonarQube is up"_
6. Browse the user interface at [http://localhost:9000](http://localhost:9000)
7. You can login with default admin credentials `admin/admin`
8. Press `Ctrl + C` on the command line window to stop the server

![Image](/images/posts/sonarqube/sonar-started.png)

**Note:** For SonarQube 7 you need to have **Java 8 runtime** installed on the system. Otherwise you'll get an error like _" Unable to execute Java command.  The system cannot find the file specified"_. If not already installed, download and install [JRE 8](http://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html).
{: .notice--danger}

<u>The C# plugin **SonarC#**</u>

With current versions of SonarQube server, SonarC# is pre-installed. With my `SonarQube 7`, I got `SonarC# 6.7.1`

Once SonarQube server is setup & running, <u>let's install **SonarQube Scanner**</u> on the machines where analysis will be performed. That means, the development machines if the developers want to run locally and the build machines where the build system will run the code analysis. We'll use it for a .NET project, so we'll use the [SonarQube Scanner for MSBuild](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+MSBuild). Installation and setup guides for other project/build types (e.g. Maven, Gradle, Ant etc.) are available [here](https://docs.sonarqube.org/display/SCAN).

1. You should already have SonarQube 6.7+ and SonarC# 6.7+ (I have 7 and 6.7.1 respectively) on SonarQube server
2. On SonarQube Scanner machine - install [.NET Framework 4.6+](https://www.microsoft.com/net/download/dotnet-framework-runtime), [Java runtime 8](http://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html)
3. Download [SonarQube Scanner for MSBuild](https://github.com/SonarSource/sonar-scanner-msbuild/releases). It'll download a zip file _"sonar-scanner-msbuild-{version}.zip"_. I downloaded version 4.0.2.892
4. Unblock the zip file (Right-click on file -> Properties -> Unblock)
5. Unzip the _"sonar-scanner-msbuild-{version}.zip"_ on to local directory, e.g. inside `C:\sonarqube\bin\scanner`
6. Add the path `C:\sonarqube\bin\scanner` to system environment variables. This path has the MSBuild scanner exe
7. Edit `SonarQube.Analysis.xml` to specify the following parameters:
    1. sonar.host.url - URL to your SonarQube server, if not http://localhost:9000
    2. sonar.login - Analysis token of a user with Execute Analysis permissions. Required only if Anonymous does not have them
8. If required, restrict access to SonarQube.Analysis.xml by setting appropriate file permissions

#### Running MSBuild Scanner for a project

Once all the above setup id completed successfully, we are ready to actually analyse our code and store details in a SonarQube project.

**Note:** We should create separate SonarQube project for each of our codebase, and everytime the analysis reports should be sent to same project for historical analysis. So while running following commands, the project key (which uniquely identifies a SonarQube project) and project display name (optional, used for display on SonarQube UI) should ke kept constant throughout the lifetime of the codebase.
{: .notice--info}

Now we'll run analysis against our code, and upload results to our SonarQube project. For that go to the desired project (code) directory and **run the following 3 commands in sequence**. Ideally, the 3 commands should be saved in a `.bat` file in project folder, and run from there.

```bash
# begin analysis - prepare for the analysis
# syntax: SonarQube.Scanner.MSBuild.exe begin /k:"unique-project-key" /n:"project-display-name" /v:"project-version"
SonarQube.Scanner.MSBuild.exe begin /k:"myproject" /n:"My Project" /v:"1.0"
# build the project/solution
msbuild solutionName.sln /t:rebuild
# end analysis - this will actually start the analysis!
SonarQube.Scanner.MSBuild.exe begin
# /d:sonar.host.url=https://myserver.com:9000 on begin
```

**Tip:** For the `end` analysis command, it'll try to fetch `blame` data from the source control (Git & SVN are pre-configured). If your source control needs a **VPN or proxy**, set them up before running the `end` command.
{: .notice--success}

**Tip:** To run `msbuild` command from any location, add the path of `MSBuild.exe` to the system environment variables. For example, the MSBuild version 15 that comes with Visual Studio 2017, can be found at _"C:\Program Files (x86)\Microsoft Visual Studio\2017\\{Edition}\MSBuild\15.0\Bin"_.
{: .notice--success}

The process can take quite a while (the `end` command). While it took around 2 minutes to process a codebase of ~20k lines, for my codebase of ~125k lines (that includes ~55k lines C# source code, JavaScript code & libraries, XML files etc.), it took almost 30min to process completely. But that time inludes scanning SVN for blame data over VPN network. In the process, it'll create a new folder `.sonarqube` inside current folder, to write all details. And after the scanner has completed the work, the server may need some more time to process and show updated results.

![Image](/images/posts/sonarqube/analysis-complete.png)

We can now got the the project in SonarQube server and see the details of the analysis.

<figure class="half">
	<a href="/images/posts/sonarqube/project-dashboard.png">
        <img src="/images/posts/sonarqube/project-dashboard.png" alt="dashboard" title="dashboard">
    </a>
	<a href="/images/posts/sonarqube/project-issues.png">
        <img src="/images/posts/sonarqube/project-issues.png" alt="issues" title="issues">
    </a>
    <a href="/images/posts/sonarqube/project-issue-details.png">
        <img src="/images/posts/sonarqube/project-issue-details.png" alt="issue-details" title="issue-details">
    </a>
    <a href="/images/posts/sonarqube/project-measures.png">
        <img src="/images/posts/sonarqube/project-measures.png" alt="measures" title="measures">
    </a>
	<figcaption>Project analysis details</figcaption>
</figure>

#### Production deployment

For development or testing, SonarQube server can be installed on almost any machine, but for production deployment there are recommended configurations. For example, we can use the _embedded database_ for development or evaluation purposes, but production instance should have a proper databse configured. See details [here](https://docs.sonarqube.org/display/SONAR/Requirements#Requirements-SupportedPlatforms).

* Hardware - At least: 2GB of available RAM, few GBs of available disk space with high I/O speed
* Java - Needs to have Java runtime installed (Oracle JRE 8 or OpenJDK 8)
* Database - Either of: PostgreSQL 8/9, MS SQL Server 2014/2016 with JDBC driver, Oracle 11G/12C, MySQL 5.6/5.7
* Browser - The user interface can be accessed through: IE11, Edge, Chrome, Firefox, Safari

For the production installation, see [instructions here](https://docs.sonarqube.org/display/SONAR/Installing+the+Server).

#### SonarLint code analysis for Visual Studio

The simplest way to use SonarQube code analysis at development time is to add SonarQube extension **[SonarLint](https://www.sonarlint.org/visualstudio/)** to Visual Studio (currently available for VS 2015 and 2017). To install

* Open Visual Studio
* Go to > `Tools` > `Extensions & updates...`
* Go to `Online` left menu
* Search for "sonarlint"
* Click "Download". The extension will install after VS restart. I installed current latest version `3.10.0.3095`

![Image](/images/posts/sonarqube/sonarlint.png)

Once installed, it'll show code analysis details as you code

![Image](/images/posts/sonarqube/sonarlint-hint.png)

You can click on the rule-id link to go to the rule details/help page.

Actually this squiggly lines for error/warning or the bulb-tips are already part of the Visual Studio infrastructure, SonarLint basically integrates it's rule-set with the VS in-built rules.

**Note:** As of March 2018, [SonarLint DOES NOT support .NET Core project](https://jira.sonarsource.com/browse/MMF-484). This feature is currently under test and hopefully, it will be available soon. On the other hand, [SonarQube Scanner for MSBuild 2.3](https://www.sonarsource.com/resources/product-news/news.html#2017-04-13-sonarqube-scanner-for-msbuild-2-3-released) DOES support .NET Core projects.
{: .notice--danger}

Another good practice is to run code analysis on each build, so that you get to know the code quality on build and compare with previous state for progress. Once setup properly, Visual Studio will show code analysis results in the `Error List` bottom tab of the editor. This is not enabled by default, we have to enable it for projects which we want to analyse on build. 

To **enable SonarLint code analysis on build** for a project

1. Enable code analysis on build for the project
    1. Right-click on project and go to `Properties`
    2. Select `Code Analysis` on left menu
    3. Check the _"Enable Code Analysis on Build"_ option, and save.
2. Enable SonarQube rules
    1. Expand project references, right-click _"Analysers"_ and select _"Open Active Rule Set"_
    2. OR, On `Code Analysis` left menu from above, select _"Microsoft All Rules"_ and _"Open"_
    3. Select the rules you want (or leave with default selection)
    4. You can change action level if required (e.g. Warning to Error) - if you make any change, they get saved in the project folder as `ProjectName.ruleset` XML file

![Image](/images/posts/sonarqube/code-analysis-on-build.png)

![Image](/images/posts/sonarqube/rule-editing.png)

Notice that the code analysis settings are saved in the `project.csproj`, inside a `PropertyGroup`

```xml
<PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Debug|AnyCPU' ">
    ...
    <CodeAnalysisRuleSet>AllRules.ruleset</CodeAnalysisRuleSet>
    <RunCodeAnalysis>true</RunCodeAnalysis>
</PropertyGroup>
```

SonarQube (or SonarLint) comes with a strong set of rule set. But, if you realy need, you can customize own rules with the [SonarQube Roslyn SDK](https://github.com/SonarSource/sonarqube-roslyn-sdk). The source code for current set of ruls is available [here](https://github.com/SonarSource/sonar-csharp/tree/master/sonaranalyzer-dotnet/src/SonarAnalyzer.CSharp/Rules).

**Note:** As an alternative, one can install the [FxCop NuGet package](https://www.nuget.org/packages/Microsoft.CodeAnalysis.FxCopAnalyzers/) on a project, which also add bunch of standard rules to the analyzer.
{: .notice--info}

#### References

* [SonarQube - continuous code quality](https://www.sonarqube.org/)
* [SonarC#](https://www.sonarsource.com/products/codeanalyzers/sonarcsharp.html)
  * [Sonar C# docs](https://docs.sonarqube.org/pages/viewpage.action?pageId=1441900)
  * [SonarC# on GitHub](https://github.com/SonarSource/sonar-csharp/tree/master/sonaranalyzer-dotnet)
  * [SonarC# rules](https://rules.sonarsource.com/csharp/RSPEC-3877)
  * [SonarQube Roslyn SDK](https://github.com/SonarSource/sonarqube-roslyn-sdk)
* [SonarLint for Visual Studio](https://www.sonarlint.org/visualstudio/)
  * [Linking SonarLint with remote SonarQube](https://www.sonarlint.org/visualstudio/howto.html#one-time-analysis)
* [Quick crappy walkthrough](http://www.dotnetcurry.com/visualstudio/1306/sonarcube-visual-studio-2015-tfs-build)
* [Download JRE 8](http://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html)
* [Download SonarQube](https://www.sonarqube.org/downloads/)
* [Install a plugin](https://docs.sonarqube.org/display/SONAR/Installing+a+Plugin)
* [Sample C# project](https://github.com/SonarSource/sonar-scanning-examples/tree/master/sonarqube-scanner-msbuild/CSharpProject)
* [SonarQube Scanner for MSBuild](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+MSBuild)
  * [C# MSBuild scanner setup](https://docs.sonarqube.org/display/SCAN/Scanning+on+Windows)
  * [MSBuild docs](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild-concepts)
  * [MSBuild commandline reference](https://msdn.microsoft.com/en-us/library/ms164311.aspx)
  * [Configuring SonarQube Scanner for MSBuild](https://github.com/SonarSource/sonar-.net-documentation/blob/master/doc/appendix-2.md)
* [runsettngs](https://msdn.microsoft.com/en-us/library/jj159530.aspx?f=255&MSPPError=-2147217396)