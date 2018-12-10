---
layout: post
title: "C# 7 minor version features - C# 7.1, C# 7.2, C# 7.3"
excerpt: "A quick guide to new features intoduced to C# in minor versions 7.1 to 7.3"
date: 2018-12-10
tags: [csharp, dotnet, code, programming]
categories: articles
comments: true
share: true
published: false
---

# A quick look at the new capabilities of C# language with minor updates 7.1 to 7.3 

***This is a minimalistic code demo, not elaborate theories.***

If you are alredy familiar with `C# 6` and `C# 7`, this will help you catch up quickly on the new features of the programming language that were introduced with minor versions 7.1 - 7.3, and get ready for upcoming [C# 8](https://blogs.msdn.microsoft.com/dotnet/2018/12/05/take-c-8-0-for-a-spin/). **If you are not not comfortable with `C# 6` or `C# 7` yet, go and read about** [C# 6](/articles/new-features-of-csharp-6) **and** [C# 7](/articles/new-features-of-csharp-7) **articles first.** Note that the feature list here is not exhaustive, it covers some of the most popular and useful features. For more details, check reference section at the bottom.

**Note:** This is the first time C# team introduced minor versions, they are not enabled in Visual Studio by default. So if you try the following code, they will not run. To use minor versions of C#, you need to configure your system as explained [here](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/configure-language-version). The easiest is to set that in VS for a project. Right-click the project on solution explorer, select properties -> Build -> Advanced. In language version dropdown, select "C# latest minor version (latest)".
{: .notice--info}

![Image](/images/posts/misc/csharp-version-in-vs.png)

## C# 7.1 features

#### Async in Main()

Now the entry point method `Main()` in your project can also have `async` modifier and so can use `await` in the method body. If you are not comfortable with `async-await`, you can learn about them [here](/articles/async-await).

```cs
static async Task<string> Main()
{
    //Directly await an await-able async method
    return await GetBigMessageAsync();
    //or do other stuffs before returning, as required
}
```

If the Main method does not return anything, you can use `async Task Main()`.

#### Default literal

When setting the default value of any type, you can simply use `default` literal, without specifying the type.

```cs
//Rather than
Dictionary<string, Func<int, bool>> funcIndex = default(Dictionary<string, Func<int, bool>>);
//You can now do
Dictionary<string, Func<int, bool>> funcIndex = default;
//Obviously the following doesn't work
var inferredVariable = default; //not for implicitly-typed variables
```

#### Inferred tuple element names

Now, if you use variables to construct a tuple, by default the tuple elements take the name of those variables.

```cs
//let's say you have couple of variables
int number = 5;
bool isEven = false;
//then you use them to construct a tuple
var numberTypeTuple = (number, isEven); //automatially named as number, isEven
var fiveIsEven = numberTypeTuple.isEven; //works just fine

//you can still name them explicitly though
var numberTypeTuple2 = (Num: number, IsEven: isEven); //explicit names
var fiveIsEven2 = numberTypeTuple2.IsEven; //variable names are not used
```

#### Generic pattern matching

C# 7.1 enhances the pattern matching introduced in [C# 7](/articles/new-features-of-csharp-7) to work with generic types. So, now you can pattern-match on generic type arguments.

```cs
//Types Cat & Dog implement interface IAnimal
public string GetSound<T>(T animal) where T : IAnimal
{
    switch (animal)
    {
        case Cat cat:
            return "meow";
        case Dog dog:
            return "bhow";
        default:
            return "...";
    }
}
```

## C# 7.2 features

#### Non-trailing named arguements

When you are calling a method with named arguments or optional parameters, it was a rule to have all the named arguments at the end of method signature after all the required arguments.

The compiler could infer the arguments when all are positional (in same order as defined in the method), all are named (so bind arguments by name) or there are required positional arguments followed by named arguments. It was NOT allowed to have a non-named argument after a named argument. This is now supported in C# 7.2, given the compiler can map the arguments by position and/or name.

Let's look at some sample to understand what works & what doesn't.

```cs
void FuncWithOptionals(int age, string name, bool isAdmin = false, bool isActive = false)
{
    //use age, name, isAdmin, isActive
}

void TestMethod()
{
    //WORKED since C# 4.0
    //standard positional arguments
    FuncWithOptionals(30, "AC", true, true);
    //all named & positional - redundant, but more readable
    FuncWithOptionals(age: 30, name: "AC", isAdmin: true, isActive: false);
    //out of position, but all are named - so works
    FuncWithOptionals(name: "AC", age: 30, isActive: false, isAdmin: true);
    //same as above, only optional ones are omitted
    FuncWithOptionals(age: 30, name: "AC");
    //with a (trailing) named argument
    FuncWithOptionals(30, "AC", isActive: false);

    //Works ONLY WITH 7.2 and above <<<<
    //isAdmin in non-trailing named argument, but in position/order
    FuncWithOptionals(30, "AC", isAdmin: true, true);

    //DOES NOT WORK - as mix of named and non-positional arguments
    OptionalParams(name: "AC", age: 30, true, true);
    OptionalParams(30, "AC", isActive: true, true);
}
```

#### Leading digit separator

With `C# 7.0`, digit separators were introduced for numeric literals. That means, you could use underscore `_` inside numbers to separate digits for better readability. So you could write a million as `1_000_000`.

`C# 7.2` enhances this a bit by allowing digit separator at begining of numeric literals which can be used with `binary` or `hexadecimal` for more clarity. Examples below.

```cs
//allowed in C# 7.0
var num = 1_000_000;            
int bin = 0b0110_1001;

//ONLY allowed in C# 7.2 & above
var hex = 0x_89F0A;
var bin2 = 0b_1000_1001;
```

#### Private Protected access modifier

`C# 7.2` introduces a new composite access modifier `private protected`, with which members will be accessible only in child classes (through inheritance) in **same assembly**.

We already had a composite access modifier `protected internal` which was accessible through EITHER inheritance OR in the same assembly. The new `private protected` restricts to ONLY inheritance in same assembly.

Let's look at some code examples for clarity

```cs
//traditional - access through inheritance OR within assembly
public class WithProtectedInternal
{
    protected internal void M() { }
}
//in same assembly
public class Test2
{
    protected void DoWork()
    {
        new WithProtectedInternal().M();
    }
}
//another assembly
public class Test2 : WithProtectedInternal
{
    protected void DoWork()
    {
        M();
    }
}

//NEW in C# 7.2
//access through inheritance within same assembly
public class WithPrivateProtected
{
    private protected void M() { }
}
//in same assembly (it's not internal)
public class Test : WithPrivateProtected
{
    protected void DoWork()
    {
        M(); //works
        new WithPrivateProtected().M(); //does NOT work
    }
}
//another assembly - DOES NOT COMPILE
public class Test : WithPrivateProtected
{
    protected void DoWork()
    {
        M(); //inaccessible due to protection level
    }
}
```

#### Conditional ref expression

Using `ref` you can get **reference** of an expression result rather than the value. For example, you get reference to one item in an array based on some condition. To elaborate, let's expand on the example from the official [docs](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/conditional-operator#conditional-ref-expression).

```cs
var smallArray = new int[] { 1, 2, 3, 4, 5 };
var largeArray = new int[] { 10, 20, 30, 40, 50 };

//this is how it worked till C# 7.1
int index = 7;
int value = ((index < 5) ? smallArray[index] : largeArray[index - 5]);
value = 0; //this just updates the copy of value, not largeArray[2]

//This was not a valid syntax
//((index < 5) ? smallArray[index] : largeArray[index - 5]) = 100;

//so there is no change in the original arrays
var smallArrayStr = string.Join(" ", smallArray); //1 2 3 4 5
var largeArrayStr = string.Join(" ", largeArray); // 10 20 30 40 50

//NEW in C# 7.2, you can get a reference to original item <<<<
ref int refValue = ref ((index < 5) ? ref smallArray[index] : ref largeArray[index - 5]);
refValue = 0; //here, actually largeArray[2] got updated to 0!

//and this is a valid syntax, that updates smallArray[2] to 100!
index = 2;
((index < 5) ? ref smallArray[index] : ref largeArray[index - 5]) = 100;

//so we get...
smallArrayStr = string.Join(" ", smallArray); //1 2 100 4 5
largeArrayStr = string.Join(" ", largeArray); // 10 20 0 40 50
```

### Safe efficient enhancements to value type

> C# 7.2 is another point release that adds a number of useful features. One theme for this release is working more efficiently with value types by avoiding unnecessary copies or allocations. - [docs](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-7-2)

#### In modifier on parameter

It means that a parameter is passed by reference but cannot be modified by the calling method. This is more useful for readability of code specifying the intent that it is used as `in`put parameter only, not to be modified.

```cs
//intended use of in modifier
public double GetSquare(in double value) => value * value;
//following code does not compile, as it tries to set value of in-parameter
public double GetQube(in double value)
{
    value = 5; //error: in double variable is readonly
    return value * value * value;
}
```

#### Ref readonly modifier on parameter

We already have `ref` and `out` modifiers on parameters. 

## Struct

What is a `struct` in .NET?

Similar to `class`, a `struct` is a compound data type that can have multiple data pieces (properties) and behaviors (methods, events, indexers etc.). A struct is a `value type`, means it is passed around as a complete value and assigning it to a new variable makes a complete new copy of the data (unlike class which is a reference type, and are passed around as references. Copying a class instance just creates a new reference to the same memory address).

Generally struct objects are stored as a single chunk of memory in the stack (not always, e.g. when they are used as property of a class object).

Class & struct can both have methods, properties, indexers & events. They both can implement interface (though casting a struct to interface type boxes it to a reference). But, there are some differences

1. Classes can inherit from another class, struct cannot inherit
2. Struct cannot have explicit (defined in code) parameter-less constructor
3. Struct cannot have a destructor
4. Struct has default implementation of `Equals()` that matches by value equality of data/properties

So, when should you use struct? 

In .NET, generally classes are preferred. But struct can have some benefits (e.g. performance & memory overhead) in some scenarios, as allocating and deallocating memory including garbage collector is generally heavier than using struct in that place. Another scenario, e.g. creating a 10 item array with reference type creates 11 references, for the 10 items and one for the array. When the same is created for struct, only a single reference is created for the array which holds the whole chunk of data with 10 structs. But having heavy structs can choke the stack memory, and passing around them as parameters can be expensive as that copies the whole data every time.

Structs are specifically useful when there are **large number of short lived, small piece of data that follows value semantics** i.e. data that actually represents a single value (e.g. a decimal number or a coordinate point), rather than a complex object (e.g. a person with height, weight, age, name, gender, job, address etc.) - are said to follow value semantics. Another important point is, value type should equate when their values are same and need not have same reference.

There are some rough rules for when to consider struct over class:

1. It logically represents a single value, similar to primitive types (integer, double, and so on).
2. It has an instance size smaller than 16 bytes (small, with few value type data)
3. It is immutable (well, depends on actual implementation)
4. It will not have to be boxed frequently (not casted frequently, e.g. to an interface)

* https://stackoverflow.com/questions/13049/whats-the-difference-between-struct-and-class-in-net
* https://stackoverflow.com/questions/14147340/immutable-class-vs-struct
* https://stackoverflow.com/questions/7484735/c-struct-vs-class-faster
* https://stackoverflow.com/questions/1951186/which-is-best-for-data-store-struct-classes
* https://stackoverflow.com/questions/521298/when-to-use-struct

## References

* [What's new in C# 7.1](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-7-1)
* [What's new in C# 7.2](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-7-2)
* [What's new in C# 7.3](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-7-3)
* [Write safe and efficient C# code](https://docs.microsoft.com/en-us/dotnet/csharp/write-safe-efficient-code)
* [The ‘in’-modifier and the readonly structs in C#](https://blogs.msdn.microsoft.com/seteplia/2018/03/07/the-in-modifier-and-the-readonly-structs-in-c/)
* [Span<T> Struct](https://docs.microsoft.com/en-us/dotnet/api/system.span-1?view=netcore-2.2)