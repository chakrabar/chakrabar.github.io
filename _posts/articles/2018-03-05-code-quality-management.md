---
layout: post
title: "Code quality management - common problems & how to solve them"
excerpt: "Common problems with software code quality and how SonarQube can help manage them"
date: 2018-03-05
tags: [tech, codequality, staticanalysis, codemetrics, sonar, sonarqube]
categories: articles
image:
  feature: posts/sonarqube/lynda-checklist.jpg
  credit: Lynda.com
  creditlink: https://www.lynda.com/Developer-Programming-Foundations-tutorials/Foundations-Programming-Software-Quality-Assurance/126119-2.html
comments: true
share: true
---

**Quality of software**

If you have ever worked on a large software application that does complex processing and is being developed by a big team, you'd know the complexities of maintaining high quality of the software. We as software development teams do lot of things to maintain the application with high standards but still almost all teams struggle with it. In this post we'll look at some basic problems with software code, and some ways that can help us tackle the problems.

Before going into _"solutions"_ let's look at **some of the common quality issues** we face with a software application.

There are basically two types of issues of a software application - the ones faced by the users and others that the developers have to deal with.

1. Usability issues that are faced by the users
    1. Incorrect behaviour of the application (visible bugs)
    2. Non-reliable application (crashing or freezing)
    3. Poor performance (lags & slowness)
    4. Complex and/or non-user-friendly interface (usability issues)
2. Quality issues of the code faced by the development team
    1. Poorly written code (bad naming practices, no input check, unhandled exceptions etc.)
    2. Overly complex code (lot of nested loops or if-else)
    3. Un-structured or spaghetti code (long procedural code that does lot of things in non-structured way)
    4. Non-modular code (modular: structured in well-defined modules/packages/layers that are pluggable)
    5. Code duplication (exactly same or almost same code, copy-pasted in multiple places)
    6. Code with security issues (e.g. non-parameterized queries, direct storage of user credentials)
    7. And many more...

While there are standard approaches to find and reduce both type of issues in a software applications, here we'll talk about managing the code quality issues.

Some of the common practices for reducing first type of issues are - well written functional tests, manual tests, load testing & profiling the application, early & frequent review with end users, regular code reviews etc.

#### Common practices to manage code quality

What is code quality?

There is no standard definition of _"code quality"_, and there is no single metric that can measure it. Code that does not suffer from the issues mentioned above can be considered good code. Well, it's almost impossible to have a large codebase with no issues at all, but lesser the issues - better the codebase. My general definition of high quality code would be

> In addition to being functionally correct and performant, a high-quality code is easy to read, understand, debug, build, test, maintain, reuse, replace, extend and is just-enough documented.

There are bunch of practices that has been established and time-tested by the development community to help tackle the code quality issues. None of them are _"silver bullet"_, and the success rate depends on multiple factors like - skillset and maturity of the developers, team priorities, outlook of the organization, pragmatic approach towards standard coding practices, choice of tools & platforms etc.

we'll briefly look at some of the common practices to manage code quality issues

* Training and guidance for the developers
* Set of basic guidelines (ruleset/checklists) and enforcement of the same
* Team code reviews (where everything is discussed from rules to design)
* Unit tests (coverage is important, so is actual meaningful tests)
* Automated functional tests for E2E correctness
* Regular code audit for design smells
* Senior developers actively working hands on with the team
* Security hackers to test vulnerabilities
* Code analysis tools

#### Tools for code analysis

If we look at the list above, we'll see all of them depend heavily on human perfection, except the last one. But like all humans, developers are also not perfect and for the same reason, many a times the above measures does not work as expected.

In real world, developers juggle between different tasks, management has changing priorities, there's always hurry to ship products, technical tasks doesn't always get high priority and get dumped as "backlogs" (some backlogs just remain backlogs forever before they are lost into eternity), good developers are moved to new projects before they could actually make the last one _really awesome_!

So, one thing that (in many ways) overcomes these problems, are tools for automatically checking code that look for issues, code smells etc. If configured properly, they can do stuffs like

* Code reviews (against set rules)
* Run unit tests
* Find possible bugs
* Suggest improvements
* Calculate code complexities, etc.

Most of the modern IDEs can do pretty good analysis (e.g. Eclipse for Java, Visual Studio for .NET, PyCharm for Python), and there are great extensions and 3rd party tools available for the same (e.g. [IntelliJ Idea](https://www.jetbrains.com/idea/index.html), [ReSharper](https://www.jetbrains.com/resharper/), [ESLint](https://eslint.org/), [Coverity Scan](https://scan.coverity.com/)).

#### Static Code Analysis

Static Code Analysis tools are a type of applications that can read through static code (without executing the code) and find problems. They generally have a set of rules for good-code (also called _"coding rules"_ or _"style guidelines"_) like, _"class names should have Pascal casing", "methods should not be more than 20 lines", "input parameters should be null checked", "exception should not be swallowed"_ etc.

Static code analysis tools check if the current code follow those rules or not. For each deviation from those rules, it shows errors/warnings/suggestion as configured. For example, [SonarQube](https://www.sonarqube.org/) or [Coverity Scan](https://scan.coverity.com/) has static code analysis for many languages. And there are language specific tools like [IntelliJ Idea](https://www.jetbrains.com/idea/index.html), [FindBugs](http://findbugs.sourceforge.net/) for Java, [StyleCop](https://github.com/StyleCop), [FxCop](https://msdn.microsoft.com/en-us/library/bb429476(v=vs.80).aspx) for .NET etc.

How does static code analysis help?

In a big project, it's pretty tedious to review code line-by-line. Also, for codebase of any size, it is inevitable to miss many code issues because of human limitations, time constraints, low-readability code etc. Static code analysis tools do those common code inspections automatically and very fast with no miss. Using static code analysis tools can save lot of manual effort, make (basic) code review fast & easy, generally give feedback immediately, and catch many issues which are very hard to find with manual inspection (e.g. code duplication across codebase).

#### Continuous code quality analysis and why it matters

Another huge benefit of the code analysis tools is that, they can be integrated into IDEs and build systems. So, effectively we can get _"continuous code quality analysis"_. With continuous code quality analysis, developers get immediate feedback on their code (as they type code in IDE or at build), and can fix the issues before merging or pushing the code into main centralized repository. This prevents _"bad code"_ from entering the system and keeps the codebase clean & more maintainable. In turn it reduces code review time and chances of bugs.

Continuous code analysis tools are ideally integrated into **Continuous Integration (CI)** systems (e.g. [Travis CI](https://travis-ci.org/), [Shippable](https://www.shippable.com/), [VSTS](https://www.visualstudio.com/team-services/), [Jenkins](https://jenkins.io/), [GoCD](https://www.gocd.org/) etc.), so that analysis is done on each commit of code, and results are maintained as historical data.

Ultimately it results in

* _**High quality product**_
* _**Reduced cost**_
* _**Faster shipping of product**_

----

Now that we understand the importance of _continuous code quality analysis_, we'll look at the very popular tool **[SonarQube](https://www.sonarqube.org/features/clean-code/)** and the great features it comes with.

SonarQube was formerly known as _"Sonar"_, and it's probably the most popular tool for continuous code analysis now. Initially popular with Java, now they support 20+ programming languages.

## Benefits of SonarQube

* Open source (Source code on [GitHub](https://github.com/SonarSource/sonarqube))
* Multi language support (Java, C#, Python, JavaScript, C++, TypeScript, XML, PL/SQL and [more](https://www.sonarqube.org/features/multi-languages/))
* IDE integration - SonarLint for [Visual Studio](https://www.sonarlint.org/visualstudio/), [Eclipse](https://www.sonarlint.org/eclipse/) etc.
* Build tool integration - MSBuild, Ant, Maven, Gradle, TFS etc.
* Extensibility with [plugins](https://docs.sonarqube.org/display/PLUG) & custom rules
* Authentication & authorization [choices](https://docs.sonarqube.org/display/PLUG/Plugin+Library) - LDAP, Azure, Google, GitHub, GitLab and more.
* Source control - SVN & Git supported out of the box, more with plugins

#### Code analysis features

* Static analysis on source code
* Code complexity
* Lines of code
* Code duplication
* Code issues (bugs, vulnerabilities, code smells)
* Unit test results
* Code coverage (Unit tests)
* Historical analysis
* Leak period analysis (issues from last commit)
* Technical debt analysis

There's no doubt that SonarQube's an excellent tool, but like all tools, it also has its limitations (which are pretty obvious actually)

* Cannot see user interface issues
* Cannot analyse architectural or high level design aspects
* Does not participate in load or stress testing

In the next section, we'll see **[how to setup & use SonarQube for continuous analysis of .NET projects](/articles/sonarqube-for-dotnet/)**.