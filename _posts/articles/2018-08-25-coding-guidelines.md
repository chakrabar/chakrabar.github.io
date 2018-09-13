---
layout: post
title: "Basic coding guidelines for OOP"
excerpt: "A general coding guidelines for Object Oriented Programming, like C# and others"
date: 2018-08-25
tags: [coding, standards, guidelines, bestpractices]
categories: articles
comments: true
share: true
published: true
---

**Note:** I wrote this coding specifications for entry/mid level developers of my team, during 2013-14. It was meant to help developers write better code, and also to help peers review code more efficiently. Parts of it might be outdated now, but most of the ideas and practices still apply. Sharing the document as-is.
{: .notice--info}

<hr />

This document describes briefly, the coding standards & practices all the developers (including automation engineers, and anyone who writes code) should follow while developing a software application.

This document is written for any developer _**working with a modern object oriented programming language**_. But at few places it is slightly biased towards `C#`, those have been marked specifically.

This document is not exhaustive; it describes only the most important set of guidelines to follow. This document will be updated as and when necessary.

<hr />

1. **Readability & Maintainability** – The most important aspects of a long lived codebase.
   1. Writing a compiling and running code is inevitable, write code that
        1. Others (humans) can read and understand easily
        2. Can be changed or extended with least side effects
   2. Follow _"newspaper style"_ of writing code. Just looking at the structure and names, a fellow developer should understand the overall functionality
   3. Comments – try to avoid. Write code that is self-explanatory
        1. But, do write some comments to explain the complex parts, which cannot be understood well enough just by reading. Write xml comments if possible, so that compiler can use that. `C#`
        2. Keep a `//TODO:` task where more work is needed
   4. KISS – keep it short and simple. Do not write long methods or classes. [See DRY section for more on this]
   5. Minimize the scope of everything. [See Scoping for more on this]
   6. Use functional shorthands/Lambda expressions and Linq methods for higher readability. But avoid them if the internal logic is becoming way too complex, and code is losing readability. (Linq is for `C#`)
   7. Use ternary operator (condition ? truthy logic : falsy logic) for simple  if-else
   8. Cache variables (create local reference) for multiple uses. e.g. define locally and use miltiple times, var lastCustId = _custService. getCustomersByPostcode(postcode). Last(). Profile. AccountDetails. Id; Improves performance and readability, reduces duplication
   9. Remove all kind of redundancy from code [e.g. if (isSuccess == true) return true; else return false; => return isSuccess;].
   10. Don’t litter the whole codebase with business logic in bits and pieces
        1. Core business (domain) logic should be within a single module
        2. Do NOT write business login in `JavaScript`, except UI validation
        3. Do NOT write UI logic within core application classes (styles, uri, control state etc.)
        4. Application (or behavioural) logic should be confined in specific module (e.g. controllers for MVC applications)
        5. Separate out logic to maintain state (session, cache etc.) from core business logic

2. **Structure** – good code starts with good structure
   1. Follow this structure/flow for classes
        1. Constants, Read-only fields
        2. Private fields
        3. Properties
        4. Constructor
        5. Public methods, Internal/package-private methods
        6. Protected methods
        7. Private methods
   2. Group code entities logically into Projects/Packages/Modules/Folders
        1. Separate logical layers of application into different modules/projects (e.g. Domain, Business logic/Application, UI, data access, services, Infrastructure, Utilities etc.)
        2. Within folders, use sub folder hierarchically to group related/similar entities
   3. Use separate files for each class, struct, interface, enum etc. Name of the file and the enclosing entity must be sam5. (e.g. class Employee in Employee.cs / Employee.java)

3. **Naming** – name that describes the purpose
   1. Name of all entities should be meaningful. They should clearly convey the purpose. Project/Package, Namespace, Classes, Interfaces, Properties, Methods, Fields, local variables and even method parameters must follow the convention
   2. Follow proper casing
        1. Namespace, Class, Interface, Property, Method, Event, Enumeration Type – must follow `PascalCasing`
        2. Method parameter, local variables – must follow `camelCasing`
        3. Test cases, enumeration fields, constants, configuration and resource keys – may contain Underscore_Separator
        4. Use underscored camel case for _privateFields [See scoping section]
        5. Use caps for CONSTANTS or application level variables. Pascal casing can also be used as per platform specific standards [See scoping section]
   3. Start Interface names with ‘I’, followed by a meaningful name in Pascal case
   4. Do NOT use Hungarian naming like strFirstName, intCount, ProductTypeEnum etc.

4. **Scoping** - Minimize scope of everything
   1. Define and use everything within the minimum scope possible
   2. Use proper access modifier for all code entities. Use private, protected, internal/package-private, protected-internal (`C#`), public - in this precedence
   3. Use naming that visually describes scope like _privateField, CONSTANT etc.
   4. Use readonly/immutable when a field’s value should not be change after initialization
   5. Use only get, for properties that should not be updated from outside
   6. Don’t forget the **"law of Demeter"**, entities should have minimum knowledge of outside world. [See SOLID section for more on this]

5. **DRY** – it is not an option, DRY is the law
   1. Do not repeat code. Think a million times before copy-pasting code
   2. A piece of knowledge (logic, in other word) should exist only in one place within the codebase/application
   3. Reuse code as much as possible
        1. If a piece of code can be reused by other components, extract them out into a public method
        2. If a piece of code is executed multiple times within a class, extract it out into a private method. Do it, even if it has just few lines
   4. Always write short methods, short classes
   5. One single method should not have too many logic, long conditional flow or too many parameters (exception can be, for example, mapper methods for third-party/legacy entities)
   6. One single class should not have many methods [See SRP in SOLID]
   7. Methods over 20 lines and classes over 100 lines should be thought through, for possible "separation of concerns"

6. **SOLID principles** - follow religiously, they will battle out the evils of bad code
   1. Strictly follow _Single Responsibility Principle (SRP)_ when writing methods, classes, modules, projects., packages or any other code entities. They all must have one and only one reason to change
   2. Always follow _Open Closed Principle (OCP)_. Write classes and other code entities that are easy to extend without modification. Prefer polymorphism over conditional logic, prefer composition over inheritance, use extension methods (`C#`) for external or legacy classes, use Visitor and Decorator patterns when applicable
   3. Remember _Liskov’s Substitution Principle (LSP)_ when inheriting. Don’t create sub classes that violate the expected behaviour of base class
   4. Segregate interfaces based on logically related functionalities, follow _Interface Segregation Principle (ISP)_. In other words, remember SRP when defining abstracts, contracts, APIs
   5. Try to follow _Dependency Inversion Principle (DIP)_ whenever possible (at least when writing new code modules). Code on abstractions not on implementation, implementation should depend on abstraction not the other way round, make high level modules (logic and policy defining modules that are fundamental to the system) independent of low level modules (implementation specific details like UI, database), keep things as loosely coupled as possible. Use IoC containers for binding modules at run time

7. **Tests** – developers miss them way too often!
   1. All parts of codebase has to be unit-testable
        1. Follow all the coding practices mentioned in this doc
        2. Code on abstraction (read interface), so that they can be mocked and injected
        3. Do not read database, files or call service within non-mockable/static classes
   2. Write test for all the business logic
        1. Try to have good coverage. But, more importantly
        2. Do have meaningful set of tests that confirm functional correctness of the application
   3. After any code change, run all the tests and fix them if required (do not alter test such a way that expected behaviour is altered, just to make them pass)
   4. Write related tests before code check-in [see general practices and process section]

8. **Exceptions and logging** – do them properly
   1. Always handle exceptions
        1. Use `Try-Catch-Finally` whenever required
        2. Use exception filter attribute/annotation whenever possible
        3. Do NOT swallow exceptions. Do a `throw;` rather than `throw ex`
        4. Show proper message in UI as applicable
        5. Do NOT use exception as control flow logic
        6. Do it for `JavaScript` code as well, especially when calling a service
   2. Log properly
        1. Log exception and other significant event details
        2. Follow the same convention for logging all over the application

9. **Cautions!** – get rid of the bad practices people follow around us
   1. Get rid of hard coding
        1. Do not use magic strings, magic numbers
        2. If required – use xml/config/resource files for global (configurable) constants
        3. Constants class can be used if the values are not configurable, but use them if absolutely necessary
        4. Define locally used constant values as private constants within the class
   2. Be cautious when using `static` keyword
        1. Use static methods if they do not use instance (object specific) values
        2. Do NOT use static properties (they can create deep rooted bugs)
   3. Do NOT use partial classes (`C#`), unless it is a asp.net web form code behind file
   4. Avoid nested classes
   5. Do NOT randomly use Design Pattern names, which do not represent anything!

10. General practices and process
    1. Always design first, then code. Take time to understand the requirement, discuss with the team, design properly, and then start coding. Do NOT touch the code until the whole thing has been designed at low level
    2. Always get the code reviewed before checking in. NO code check-ins without thorough code. Review -> review comments -> changes per comments. (Or, commit, get reviewed, then push to master, as per team standards). Before committing, always build the project, run the unit tests, run the application and do a quick sanity
    3. Write related tests before code check-in and get them reviewed as well. NO code check-in without related tests
    4. Refactor. It’s a continuous process, do it often (when adding new code, or fixing bugs, or doing general clean up). But do it precisely and methodically, otherwise there is a high chance of breaking existing functionalities. Run tests, to make sure everything is working as expected
    5. Learn and use established Design Patterns, whenever they fit
    6. <u>Performance</u> – last but not the least! Always consider **performance and scalability** issues while writing code. [Details of good coding practices for performance & scalability are beyond the scope of this document!]

`C#` denotes some C# language specific constructs. If you are using some other language for development, similar constructs available for that language/platform can be used.