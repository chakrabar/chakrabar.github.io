---
layout: post
title: "Sample NAnt script for CI build system"
excerpt: "A sample NAnt script for common CI tasks like pulling code, building running tests, deployment and more"
date: 2018-04-12
tags: [nant, nantcontrib, dotnet, automation, buildfile, tasks, ci, cd, cicd]
categories: articles
image:
  feature: posts/nant/nant_script.png
comments: true
share: true
---

This post is a continuation of **[Automating tasks with NAnt](/articles/automating-with-nant/)**, read that first to understand what is NAnt and why to use it.

This post just shows a sample NAnt script that can be used as part of a CI (Continuous Integration) system for .NET projects. It does the following tasks

1. Prepares the build output directories
2. Pulls latest code from source control (SVN)
3. Compiles the code (.NET application with MSBuild)
4. Executes SonarQube static analysis on code
5. Runs unit tests
6. Runs code coverage
7. Compresses the artifacts into a package zip file
8. Uploads the package to a Nexus artifact repository
9. Deploy website (on existing IIS directory)
10. Logs the build execution details to a log file

#### High-level overview

I'll not go into detailed explanation of the script as it is mostly self-explanatory, I'll just mention few key points. For initial understanding of NAnt, read [this](/articles/automating-with-nant/) post, and see the references there for additional resources.

* This script has a bunch of `<property>` which are used to configure the script. To directly use it for your own project, simply update the values of properties as required. Properties are commented and grouped into different sections based on when you should update them. Basically, look for _"CHANGE"_ in the comments.
* There are two main `<target>`s, _"quick-deploy"_ & _"ci-deploy"_. The seconds one does all the tasks mentioned above, while the first one simply compiles and deploys the application. Add/modify targets to customize it for specific needs. Note, here the targets were called one-by-one rather than having nested dependency for simplicity & readability.
* Build logs are saved with `<record>` task in `Logs` directory with a name formal like `build_2018Apr08_19-10-30.log`.
* The `<tstamp>` task is used to get current timestamp. In other places, function `${datetime::now()}` is used for similar purpose.
* The solution can be built with two standard configurations `debug` and `release`. Default is `debug`, if required, pass the value as command-line arg e.g. `-D:build.config=release`.
* The scripts does code analysis with SonarQube and loads artifacts to a Nexus repository. If not required, skip those targets. To understand them, read the [SonarQube](/articles/sonarqube-for-dotnet/) and [Nexus](/articles/nexus-artifact-repository-mnager/) posts.
* All the non-NAnt commands are run simply with `<exec>` task, which is nothing but NAnt way of running command-line applications. Arguments are passed with `<arg>` tasks, where `value` is used for simple single-value arguments, and complicated ones with `line` which can basically accept _"a line of arguments"_ i.e. single or multiple values passed as-is to the executing command-line app.
* The unit test run reports & coverage reports get saved in `..BuildOutput\Argon\TestOutputs` directory, unless configured differently. See [NUnit](http://nunit.org/docs/2.2.6/consoleCommandLine.html) and [OpenCover](https://github.com/opencover/opencover/wiki/Usage) documentations for their specific syntax and options.
* The _"getSVNRevision"_ task, though looks complicated, actually just gets the current SVN revision number in `svn.revision` property, to be used later in package name. This part of script was taken from [here](http://untidy.net/blog/2006/08/21/subversion-revision-number-in-nant/).
* Finally it creates a compressed artifact package and loads to `Nexus` for later use.
* `MSBuild` is executed same way with `exec` task, directly running the `msbuild.exe`. It uses local copy of MSBuild to compile a .NET solution. Do not forget to set the correct value of `${latest.msbuild}` to the path of desired MSBuild executable.

**Note:** Since this script _"restarts"_ the IIS application pool as part of the web deployment process, the script needs to be run in _admin mode_, unless that task is skipped.
{: .notice--info}

#### The script

```xml
<?xml version="1.0" encoding="utf-8" ?>
<project name="MyApp" default="quick-deploy">
  <description>This sample NAnt script builds a .NET Application and more</description>

  <!--CHANGE this section for specific project/solution-->
  <property name="base.dir" value="C:\Projects\Build\MyApp" />
  <property name="solution.filename" value="MyApp.sln" />
  <property name="sonarqube.project.name" value="MyApp DEV" />
  <property name="sonarqube.project.key" value="myapp.trunk" />
  <property name="web.publish.dir" value="C:\Projects\Build\Publish\MyApp\Web" />

  <!--CHANGE this section as per build machine/environment configuration-->
  <property name="nunit.path" value="C:\Program Files (x86)\NUnit 2.6.4\bin\nunit-console.exe" />
  <property name="sonarqube.msbuild.path" value="SonarQube.Scanner.MSBuild.exe" />
  <property name="sonarqube.server" value="http://localhost:9000" />
  <property name="opencover.path" value="C:\Users\ArghyaC\AppData\Local\Apps\OpenCover\OpenCover.Console.exe" />
  <property name="latest.msbuild" value="C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\MSBuild\15.0\Bin\MSBuild.exe" />
  <property name="curl.path" value="C:\Users\ArghyaC\Downloads\curl-7.57.0-win64-mingw\bin\curl" />
  <property name="nexus.repo" value="http://localhost:9876/repository/raw-hosted/" />
  <property name="svn.username" value="ArghyaC" />
  <property name="svn.password" value="Passw0rd" />
  <property name="nexus.username" value="admin" />
  <property name="nexus.password" value="Passw0rd" />
  <property name="apppool.name" value="MyAppPool" />
  <property name="appserver.name" value="localhost" />

  <!--this section can be kept as is-->
  <property name="code.base" value="C:\Projects\Build\Codebase" />
  <property name="build.dir" value="${path::combine(base.dir, 'Output')}" />
  <property name="tests.dir" value="${path::combine(base.dir, 'TestOutputs')}" />
  <property name="build.config" value="debug" />

  <!--details of LOG FILE, CHANGE IF required-->
  <property name="logs.dir" value="C:\Projects\Build\Logs" />
  <tstamp property="timestamp" pattern="yyyyMMMdd_HH-mm-ss" verbose="true" />
  <property name="log.filename" value="build_${timestamp}.log"/>
  <property name="logfile.path" value="${path::combine(logs.dir, log.filename)}" />
  <record name="${logfile.path}" action="Start" level="Verbose"/>
  <echo message="Build logged to ${logfile.path}"/>

  <!--DO NOT CHNAGE this section-->
  <property name="svn.revision" value="0"/>
  <property name="zipfile.path" value="deploy.zip" />
  <property name="start.time" value="${datetime::now()}" />
  <property name="end.time" value="${datetime::now()}" />
  <echo message="Starting the process at ${start.time}" />

  <!--SVN CHECKOUT task, EDIT for each application/solution :: CHANGE IT-->
  <target name="svncheckout" description="SVN checkout the application code">
    <echo message="SVN checkout MyApp" />
    <property name="solution.dir" value="${path::combine(code.base, 'MyApp')}" />
    <echo message="SVN checkout MyApp" />
    <svn-checkout
            destination="${solution.dir}" 
            uri="https://somesource.com/svn/branch/master/MyApp" 
            revision="HEAD"
            username="${svn.username}"
      password="${svn.password}"
        />
  </target>

  <!--DEPLOY website, this has PROJECT SPECIFIC details :: CHANGE IT-->
  <target name="deploy-sites">
    <copy todir="${web.publish.dir}" overwrite="true">
      <fileset basedir="${build.dir}\_PublishedWebsites\MyApp.Web">
        <include name="**/*" />
      </fileset>
    </copy>
    <!--this needs the script to be run in ADMIN MODE-->
    <iisapppool action="Restart" pool="${apppool.name}" server="${appserver.name}" />
  </target>

  <!-- target FULL-CI-DEPLOY :: SVN checkout, BUILD, TEST, run SONARQUBE, PACKAGE for deployment, store in NEXUS, DEPLOY -->
  <target name="ci-deploy">
    <call target="clean" />
    <call target="svncheckout" />
    <call target="sonarqube-begin" />
    <call target="build" />
    <call target="run-nunit" />
    <call target="run-opencover" />
    <call target="sonarqube-end" />
    <call target="zip" />
    <call target="nexus-load" />
    <call target="deploy-sites" />
    <call target="conclude" />
  </target>

  <!-- target QUICK-DEPLOY :: ONLY - SVN checkout, BUILD & DEPLOY -->
  <target name="quick-deploy">
    <call target="clean" />
    <call target="svncheckout" />
    <call target="build" />
    <call target="deploy-sites" />
    <call target="conclude" />
  </target>

  <target name="release">
    <property name="build.config" value="release" />
    <property name="build.dir" value="${path::combine(build.dir, 'Release')}" />
    <echo message="config is ${build.config}, outputdir=${build.dir}" />
  </target>

  <target name="debug">
    <property name="build.config" value="debug" />
    <property name="build.dir" value="${path::combine(build.dir, 'Debug')}" />
    <echo message="config is ${build.config}, outputdir=${build.dir}" />
  </target>

  <target name="clean">
    <call target="${build.config}" />
    <delete dir="${build.dir}" failonerror="false" />
    <mkdir dir="${build.dir}" />
    <delete dir="${tests.dir}" failonerror="false" />
    <mkdir dir="${tests.dir}" />
  </target>

  <!--start sonarqube processing-->
  <target name="sonarqube-begin">
    <property name="nunit.results" value="${path::combine(tests.dir, 'NUnitResults.xml')}" />
    <property name="opencover.xml" value="${path::combine(tests.dir, 'opencover.xml')}" />
    <exec program="${sonarqube.msbuild.path}" verbose="true">
      <arg value="begin"/>
      <arg line='/k:"${sonarqube.project.key}"' />
      <arg line='/n:"${sonarqube.project.name}"' />
      <arg line='/v:"1.0"' />
      <arg line='/d:"sonar.host.url=${sonarqube.server}"' />
      <arg line='/d:sonar.cs.nunit.reportsPaths="${nunit.results}"' />
      <arg line='/d:sonar.cs.opencover.reportsPaths="${opencover.xml}"' />
    </exec>
  </target>

  <!--compile the project solution(s)-->
  <target name="build">
    <exec program="${latest.msbuild}" verbose="true">
      <arg line="${path::combine(solution.dir, solution.filename)}" />
      <arg value="/property:Configuration=${build.config}"/>
      <arg value="/property:OutDir=${build.dir}"/>
    </exec>
  </target>

  <!--run nunit tests, results will be loaded to sonarqube-->
  <target name="run-nunit">
    <property name="test.dlls" value="${path::combine(build.dir, 'Core.Common.UnitTests.dll')} ${path::combine(build.dir, 'MyApp.BusinessTests.dll')} ${path::combine(build.dir, 'MyApp.DataAccess.UnitTests.dll')} ${path::combine(build.dir, 'MyApp.WebTests.dll')}" />
    <exec program="${nunit.path}" verbose="true" failonerror="false">
      <arg line='/result="${nunit.results}"' />
      <arg line="${test.dlls}" />
    </exec>
  </target>

  <!--run opencover, results will be loaded to sonarqube-->
  <target name="run-opencover">
    <exec program="${opencover.path}" verbose="true" failonerror="false">
      <arg line='-register:user' />
      <arg line='-target:"${nunit.path}"' />
      <arg line='-targetargs:"${test.dlls} /noshadow"' />
      <arg line='-output:"${opencover.xml}"' />
      <arg line='-filter:"+[MyApp*]* +[Core*]* -[*Tests]* -[*Console]* -[*Contracts]* -[*Models]*"' />
    </exec>
  </target>

  <!--sonarqube end processing-->
  <target name="sonarqube-end">
    <exec program="${sonarqube.msbuild.path}" verbose="true">
      <arg value="end"/>
    </exec>
  </target>

  <!--gets the SVN revision number-->
  <target name="getSVNRevision">
    <echo message="Retrieving Subversion revision number"/>
    <exec
      program="svn"
      commandline='log "${solution.dir}" --xml --limit 1'
      output="${solution.dir}\_revision.xml" failonerror="false"/>
    <xmlpeek
      file="${solution.dir}\_revision.xml"
      xpath="/log/logentry/@revision"
      property="svn.revision" failonerror="false"/>
    <echo message="Using Subversion revision number: ${svn.revision}"/>
  </target>

  <!-- Create a zip file for package repo, from build output -->
  <target name="zip">
    <call target="getSVNRevision" />
    <property name="zipfile.name" value="${project::get-name()}_${timestamp}_R${svn.revision}.zip" />
    <property name="zipfile.path" value="${path::combine(base.dir, zipfile.name)}" />
    <echo message="zipping files from ${build.dir}" />
    <zip zipfile="${zipfile.path}" includeemptydirs="true">
      <fileset basedir="${build.dir}">
        <include name="**/*" />
      </fileset>
    </zip>
  </target>

  <!-- Upload the package to NEXUS repository -->
  <target name="nexus-load">
    <exec program="${curl.path}" verbose="true">
      <arg line="-v -u ${nexus.username}:${nexus.password} --upload-file" />
      <arg value="${zipfile.path}"/>
      <arg value="${nexus.repo}"/>
    </exec>
  </target>

  <target name="conclude">
    <property name="end.time" value="${datetime::now()}" />
    <echo message="PROCESS COMPLETED. Total time taken (hh:mm:ss) - ${((datetime::parse(end.time)) - (datetime::parse(start.time)))}" />
    <echo message="Check the detailed logs - ${logfile.path}" />
  </target>

</project>
```

**Note:** The script needs both **NAnt** & **NAntContrib** to be installed on the system. See [this](/articles/automating-with-nant/) for setup help.
{: .notice--success}

**Note:** To use the full script, you needs to have this tools installed too - [SVN](https://tortoisesvn.net/downloads.html), [NUnit](http://nunit.org/docs/2.4/installation.html), [OpenCover](https://github.com/opencover/opencover/releases), [MSBuild](https://github.com/Microsoft/msbuild/releases), [SonarQube MSBuild runner](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+MSBuild), [curl](https://curl.haxx.se/download.html).
{: .notice--success}