---
layout: post
title: "Functional programming basic concepts"
excerpt: "Very basic understanding of functional programming concepts. No deep dive, mathematics or advanced theories. Just the basic ideas that can be applied across languages & frameworks"
date: 2020-06-20
tags: [dev, programming, functional, functionalprogramming, purefunctions]
categories: articles
comments: true
share: true
published: false
---

## Basic concepts of functional programming

Here we'll look at some very basics of Functional Programming. We'll not try to go into the theories or mathematics behind them, just the basic ideas which might be used outside pure fnctiona languages as well.

Most of the functional concepts discussed below are generally enforced in functional anguages like `Haskell`, `Clojure`, `Scala` or `F#`. Some features come out-of-the-box in those languages (like `Monads`), which needs to be constructed and enforced by developers, if used in non-functional languages like `JavaScript` or `C#`. Some features might be very difficult to impossible to be built in some non-functional programming languages, but the ideas can still be useful.

To apply even some of the concepts of funtional programming in a non-functional langage, at least the language should treat functions as first class citizens and support `higher order functions`.

1. Pure functions just take input and **works ONLY on the input to create output**
	1. Does not depend on anything outside the parameters - consts, globals, class level properties, database, network, files or anything
	2. Does not create a side effect (in practical scenarios, it *might be okay* to log something for example, but must not affect application state in any way)
    3. They are idempotent, they ALWAYS produce same result when same arguments are passed
    4. Main benefits are - reliability & simplicity
2. Pure functions are **like mathematical functions**
    1. Given same input, they always create same output like `5 + 2 = 7`
    2. You know a `sum(a, b)` function will just do sum, not read files or update DB
    3. Pure functions has the property of **Referential transparency** - if a function call is replaced with the result of a function, nothing will change in applicatin behavior
3. **Higher order functions** are functions that can take functions as inputs and/or return functions as output
    1. Functions are treated as first class citizens, they can be passed around just like data or object
4. Pure functions are **honest**
    1. Apart from above facts, they say very clearly abut outcomes
    2. If return type is `Student` it must always be a `Student object` not `null` for example. If it can also return null, we should specify in some way, like having a different class like `Maybe<Student>` which has a `HasValue` boolean property, and *maybe* throws error when `GetValue` is access but has no value (`null`)
    3. It should never throw `Exception` and it's a really a non-handleable system failure. For any errors that can be anticipated or handled, it should rather retrn data including error details than failing, like `Result<Student>` which has `IsSuccess` and `Error` properties. Or even better, a `Result<Maybe<Student>>`
5. Funtional programming kind of inherently expects the state & types to be **Immutable** e.e. -> once created, values cannot be changed
	1. To change, new data instance must be created
    2. It *should ideally* maintain a history of all value changes (depends on the implementation though)
    3. It makes tracking changes and bugs easier
    4. As a side effect, since it maintains a timeline of snapshots of the state, `Undo` can easily be done (or just travel in time)
	5. To handle this scenario efficiently, that is without copying the whole data set, there are special data structures to help -> `persistant and immutable data structures`
6. Functional programming generally comes with additional fetures like
    1. `Lazy evaluation` - expressions not actually executed/evaluated until really required (e.g. using `yield` in `C#` or `generators` in `JavaScript`)
    2. `Pattern matching` - Match object against patterns to control execution flow (example in [C# 8.0](https://docs.microsoft.com/en-us/archive/msdn-magazine/2019/may/csharp-8-0-pattern-matching-in-csharp-8-0))
    3. `Memoization` - keeping result of previous computation in memory, so that they can be used directly rather than recalculating for the same data again and again. Realize that, this is only possible becuse of the *"Referential Transparency"* property of pure functions
    
### Immutable & persistant data structures
	
They basically store data in tree structure, so that when new copy is needed, they create new node and link that up (path copying), so

1. sharing the rest of the data
2. old and new data are both available, sharing all of the common data (somewhat like the Git way)
3. they frequently look like trie - nodes represent values, paths represent keys => t -> e -> a => tea
4. if taken as node of two together, the index can be converted to binary to search in logarithim way
5. By research, 32bit keys are taken as optimal which does not make the trees too wide, or too deep
6. **See [this](https://www.youtube.com/watch?v=I7IdS-PbEgI) video** for a good intro to `Immutable & persistant data structures` with `Immutable.js`

### Monads ✊🏽

Monads are comparatively more complex concepts, and very specific to functional langages. It might be very difficult, to even impossible to implement them in other languages, based on the language's capabilities.

> **Disclaimer:** I do not use functional languages for my work, and no way I'm trying to claim that I understand `Monads` well enough. I'm just trying to put forward my basic understanding of the topic based on various discussions and forums. I just hope it might help someone like me, to at least take a stab at understanding the basic concepts of `Monads`. Also, we are going to use `C#` syntax for this section.

Monad is like a design pattern, that help us achieve mainly following things

1. Extend a type or data structure by adding additional properties and/or behaviors to it
2. Make functions on those types (being extended) callable on the extended type. Note that we are talking about functions that operate on that type, not the members/methods of that type e.g. the type might be `Employee` and a function might be `GetManagerInfo(employee)` and NOT `employee.GetManager()`
3. Make functions on those types (being extended) composable, so that they can be chained one after another in a fluent, functional way

In a pure way, it does not have anything to do with generics as such, but we can use `generics` in to create `Monad`. Understand that, though generic types always extends a type in some way (e.g. `IEnumerable<T>` or `Repository<TEntity>`), generic types do NOT mean monads, they do not provide #2 and #3 unless specifically coded to achieve them.

So, if we write the above 3 points in `C#`, we can write like below

1. Create a type `Extension<T>` with additional properties and methods
2. Do something so that we can call `SomeFunction(T)` on `Extension<T>`, without changing the function itself
3. We can write code like below (not exactly this syntax, but functionally same & structurally similar)

```cs
Extension<T>
    .SomeFunction()
    .AnotherFunction()
    .SomeOtherFunction();
```

But why would you want to do this? Say you want to take a number, multiply with 3, and add 2 to it. For simplicity, let's assume we have direct functions like `Triple(int)` and `AddTwo(int)`. You'll have to do

```cs
var result = AddTwo(
    Triple(number)
);
```

Think, you want to do more operations like this. Is it easy to read? Easy to maintain? Debug? Comment out part of the logic? What if you could write like

```cs
var result = number
    .Triple()
    .AddTwo()
    .MultiplyeBy(anotherNumber)
    .GetPercent(75);
```

Great. But you cannot do that because those methods not part of definition of `Integer` type. Well, you can write extension methods right? But still, you'll have to write a new extension method equivaent to each existing function! Moreover, if other types (like `string`, `Entity`, `Product` etc.) needs similar things, you'll have to write such extensions for all of them separately!

Let's create a **Monadic** type in `C#`, as `Maybe<T>` which is a semantic type representing any value or a null (`Just a value | null`), and adds following special capabilities

1. Easily check if it actually has a value or `null`
2. Enforce to check for `null` before using the value

This is a common monadic type, frequently called the `Maybe` or `Option` type

```cs
// A monadic type, extends any given type
public class Maybe<T>
{
    private readonly T _data;

    // EXTENDED FEATURE
    public bool HasValue => _data != null;

    // UNIT (WRAP)
    public Maybe(T t)
    {
        _data = t != null ? t : default;
    }

    // RETURN (UNWRAP)
    public T GetValue() => !HasValue
        ? throw new InvalidOperationException("Has no value")
        : _data;

    // BIND
    public void Apply(Action<T> action) => action(_data);
    public Maybe<T> Apply(Func<T, T> func) => new Maybe<T>(func(_data));
    public Maybe<V> Apply<V>(Func<T, V> func) => new Maybe<V>(func(_data));

    // AN IMPROVED BIND
    public Maybe<V> IfHasValue<V>(Func<T, V> func) => new Maybe<V>(HasValue ? func(_data) : default);
} 
```

With this **Monadic `Maybe<T>`** type, we can actually do the following

```cs
public static class TestMonadic
{
    // Helper functions
    public static int Triple(int n) => n * 3;
    public static int AddTwo(int n) => n + 2;
    public static int MultiplyBy(int n, int p) => n * p;
    public static double GetPercent(int m, int p) => (m * p) / 100;

    // Use the monadic type to compose the functions
    public static void UseMaybe()
    {
        double maybe = new Maybe<int>(10)
            .Apply(Triple)
            .Apply(AddTwo)
            .Apply((x) => MultiplyBy(x, 5))
            .Apply((x) => GetPercent(x, 75))
            .GetValue();
    }
}
```

Pretty cool 😎

So, as we can already see, a monadic type should provide following **APIs** (see comments in the `Maybe<T>` class)

* A **unit** function - which can convert a simple type to the monadic type
* A **return** function - which can convert the monadic type back to the underlying type
* One or more **bind** function(s) - which can take a function that accepts the underlying type, apply the actual data on that function, and return a monadic type of the resultant type from the function

### References

* [Learning Functional Programming with JavaScript](https://www.youtube.com/watch?v=e-5obm1G_FY)
* [Applying Functional Principles in C# - Pluralsight course](https://app.pluralsight.com/library/courses/csharp-applying-functional-principles/table-of-contents)
* [C# code in functional style - example repo](https://github.com/vkhorikov/FuntionalPrinciplesCsharp)
* [Immutable.js - internals & usage](https://www.youtube.com/watch?v=I7IdS-PbEgI)
* [Immutable data structures for functional JS](https://www.youtube.com/watch?v=Wo0qiGPSV-s)
* [Immutable.js](https://immutable-js.github.io/immutable-js/)
* [Get started with Immutable.js](https://www.freecodecamp.org/news/immutable-js-is-intimidating-heres-how-to-get-started-2db1770466d6/)
* [CRUD with Immutable.js](http://untangled.io/immutable-js-get-set-update-and-delete-data-from-maps/)
* [Object.observe (deprecated) API](https://developer.mozilla.org/en-US/docs/Archive/Web/JavaScript/Object.observe)
* [Proxy API](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)
* [Monad in plain English - SO](https://stackoverflow.com/questions/2704652/monad-in-plain-english-for-the-oop-programmer-with-no-fp-background/2704795)
* [What is Monad in Haskell - SO](https://stackoverflow.com/questions/44965/what-is-a-monad)
* [Monad intro with JavaScript](https://blog.jcoglan.com/2011/03/05/translation-from-haskell-to-javascript-of-selected-portions-of-the-best-introduction-to-monads-ive-ever-read/)
* [Monad syntax for JavaScript](https://blog.jcoglan.com/2011/03/06/monad-syntax-for-javascript/)
* [Functors, Applicatives, And Monads In Pictures](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html)
