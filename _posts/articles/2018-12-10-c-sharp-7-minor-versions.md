---
layout: post
title: "C# 7 features with minor version updates - C# 7.1, C# 7.2, C# 7.3"
excerpt: "A quick guide to new features introduced to C# in minor versions 7.1 to 7.3"
date: 2018-12-10
tags: [csharp, dotnet, code, programming, efficient, safe]
categories: articles
comments: true
share: true
published: true
---

# A quick look at the new capabilities of C# language with minor updates 7.1 to 7.3 

***This is mostly a bunch of code demos, not so much into the theories.***

If you are already familiar with `C# 6` and `C# 7`, this will help you catch up quickly on the new features of the programming language that were introduced with minor versions 7.1 - 7.3, and get you ready for upcoming [C# 8](https://blogs.msdn.microsoft.com/dotnet/2018/12/05/take-c-8-0-for-a-spin/). **If you are not comfortable with `C# 6` or `C# 7` yet, go and read the [C# 6](/articles/new-features-of-csharp-6) and [C# 7](/articles/new-features-of-csharp-7) articles first.** Note that the feature list here is not exhaustive, it covers some of the most popular and useful ones. For more details, check reference section at the bottom.

**Note:** This is the first time C# team introduced minor versions, they are not enabled in Visual Studio by default. So if you try the following code, they may not run. To use minor versions of C#, you need to configure your system as explained [here](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/configure-language-version). The easiest is to set that in Visual Studio, per project. Right-click the project on solution explorer, select properties -> Build -> Advanced. In language version dropdown, select "C# latest minor version (latest)".
{: .notice--info}

![Image](/images/posts/misc/csharp-version-in-vs.png)

## C# 7.1 features

#### `async` in `Main()`

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

#### `default` literal

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
var numberTypeTuple = (number, isEven); //automatically named as number, isEven
var fiveIsEven = numberTypeTuple.isEven; //works just fine

//you can still name them explicitly though
var numberTypeTuple2 = (Num: number, IsEven: isEven); //explicit names
var number2 = numberTypeTuple2.Num; //variable names are not used
```

Just remember that, variable names in `Tuple` are available only in compile time, they do not exist in runtime.

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

#### Non-trailing named arguments

When you are calling a method with named arguments or optional parameters, it was a rule to have all the named arguments at the end of method signature after all the required arguments.

The compiler could infer the arguments when all are positional (in same order as defined in the method), all are named (so bind arguments by name) or there are required positional arguments followed by named arguments. It was NOT allowed to have a non-named argument after a named argument. This is now supported in C# 7.2, given the compiler can map the arguments by position and/or name. Not a _"new"_ feature as such, but helps write method calls more clearly in some scenarios.

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
    FuncWithOptionals(name: "AC", age: 30, true, true);
    FuncWithOptionals(30, "AC", isActive: true, true);
    FuncWithOptionals(30, "AC", true, isAdmin: true);
}
```

#### Leading digit separator

With `C# 7.0`, digit separators were introduced for numeric literals. That means, you could use underscore `_` inside numbers to separate digits for better readability. So you could write a million as `1_000_000`.

`C# 7.2` enhances this a bit by allowing digit separator at beginning of numeric literals which can be used with `binary` or `hexadecimal` for more clarity. Examples below.

```cs
//allowed in C# 7.0
var num = 1_000_000;            
int bin = 0b0110_1001;

//ONLY allowed in C# 7.2 & above
var hex = 0x_89F0A;
var bin2 = 0b_1000_1001;
```

#### `private protected` access modifier

`C# 7.2` introduces a new composite access modifier `private protected`, with which members will be accessible only in child classes (through inheritance) in **same assembly**.

We already had a composite access modifier `protected internal` which was accessible through EITHER inheritance OR in the same assembly. The new `private protected` restricts to ONLY inheritance AND in same assembly.

Let's look at some code examples for clarity

```cs
//traditional - access through inheritance OR within assembly
public class WithProtectedInternal
{
    protected internal void M() { }
}
//in same assembly, inheritance not required
public class Test2
{
    protected void DoWork()
    {
        new WithProtectedInternal().M();
    }
}
//another assembly, only through inheritance
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

#### Conditional `ref` expression

Using `ref` you can get **reference** of an expression result rather than the value. For example, you get reference to one item in an array based on some condition. To elaborate, let's expand on the example from the official [docs](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/conditional-operator#conditional-ref-expression).

```cs
var smallArray = new int[] { 1, 2, 3, 4, 5 };
var largeArray = new int[] { 10, 20, 30, 40, 50 };

//this is how it already worked till C# 7.1
int index = 7;
int value = ((index < 5) ? smallArray[index] : largeArray[index - 5]);
value = 0; //this just updates the copy of value, not largeArray[2]

//This was not a valid syntax
//((index < 5) ? smallArray[index] : largeArray[index - 5]) = 100;

//so there is no change in the original arrays
var smallArrayStr = string.Join(" ", smallArray); //1 2 3 4 5
var largeArrayStr = string.Join(" ", largeArray); // 10 20 30 40 50

//>>>> NEW in C# 7.2, you can get a reference to original item <<<<
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

C# 7.2 introduces a bunch of features which work together towards the same goal - making value types efficient & safe to work with. We discuss those features separately in the post **[C# 7.2 value type enhancements](/articles/c-sharp-7.2-value-type-enhancements/)** (_will be published soon_).

## C# 7.3 features

This minor version update on the language does not bring any _"new"_ feature as such. Rather `C# 7.3` goes in the same direction of `C# 7.2` and adds some more features for safe & efficient code. It also adds some enhancement to some existing features. We'll briefly look at only couple of those features, see the [official docs](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-7-3) for all the features.

#### Reassignment of `ref local` variables

Before `C# 7.3`, once a `ref local` variable has been declared and initialized, it could not be re-assignd to another ref local. It's now supported.

```cs
//a simple ref return method
ref int GetPrevious(int[] arr, int idx)
{
    if (arr == null || idx <= 0)
        throw new ArgumentException();
    return ref arr[idx - 1];
}
//another ref return method
ref int GetNext(int[] arr, int idx)
{
    if (arr == null || idx >= arr.Length - 1)
        throw new ArgumentException();
    return ref arr[idx + 1];
}
//actual usage
void TestRefReturn()
{
    var arr = new[] { 1, 2, 3, 4, 5 };
    ref int arrItem = ref GetPrevious(arr, 2);
    //ONLY works with C# 7.3 and above
    arrItem = ref GetNext(arr, 2);
}
```

#### Enhanced generic constraints

Now `System.Enum` & `System.Delegate` can be used as type constraint on generic types. From the [official docs](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/where-generic-type-constraint), these are now valid generic constraints.

```cs
public class UsingEnum<T> where T : System.Enum { }
public class UsingDelegate<T> where T : System.Delegate { }
public class Multicaster<T> where T : System.MulticastDelegate { }
```

#### Tuple enhancements

Tuples can now be equated with `==` and `!=`, the comparison will work on each of the tuple members. It can also work if one side is nullable while the other is not. It even does _implicit conversion_ of member types when comparable types are found (see example below). 

But remember that the tuple variable names exist only at compile time, not at run time. At run time, the variables are still referred as `Item1`, `Item2` etc. So the variable names really does not matter as long as the sequence of members have same type. If the sequence does not match, tuples will not match even if variable names are same.

```cs
internal void TestTupleEquality()
{
    var t1 = (A: 1, B: "hello");
    var t2 = (A: 1, B: "hello");
    //making it nullable
    (int A, string B)? t3 = t2;
    //with compatible but not same type
    (long A, string B) t4 = (1, "hello");
    //with different tuple variable names
    var t5 = (X: 1, Y: "hello");

    var test1 = t1 == t2; //true
    var test2 = t1 == t3; //true
    var test3 = t1 == t4; //true
    var test4 = t1 == t5; //true

    //this DOES NOT WORK for type (sequence) mismatch
    var t6 = (B: "hello", A: 1);
}
```

## References

* [What's new in C# 7.1](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-7-1)
* [What's new in C# 7.2](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-7-2)
* [What's new in C# 7.3](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-7-3)
* [Write safe and efficient C# code](https://docs.microsoft.com/en-us/dotnet/csharp/write-safe-efficient-code)
* [C# 7 Series - Mark Zhou's Tech Blog](https://blogs.msdn.microsoft.com/mazhou/2018/03/25/c-7-series-part-10-spant-and-universal-memory-management/)