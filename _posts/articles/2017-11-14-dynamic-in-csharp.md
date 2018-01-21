---
layout: post
title: "Dynamic in C#"
date: 2017-11-14
tags: [tech, tips, .net, code-demo]
excerpt: "A quick introduction to dynamic in C# and comparison with var and object keywords."
categories: articles
share: true
modified: 2018-01-19T08:11:53-04:00
---



# A quick glance at the dynamic keyword 

It's just a quick-and-dirty code demo, to show the basic usage of dynamic keyword and difference with other similar constructs. This does not cover any theories at all. Go to MSDN or other good resources that can give better explanations.

We'll also look at `var` and `object` to see the contrast.

## Few very basic things to note:

1. The dynamic keyword adds some dynamic capabilities to the language, which means - in brief, you can use dynamically typed objects in your code (in contrary to all the strongly typed objects you have seen in C#).

2. But `C#` still is a statically typed language in most senses. Use dynamic only if static typing is not serving your use-case.

3. From Scott Henselman's [blog](https://www.hanselman.com/blog/C4AndTheDynamicKeywordWhirlwindTourAroundNET4AndVisualStudio2010Beta1.aspx) - ***Dynamically typed objects are still statically typed, as dynamic.*** Yes, the type of those objects are `dynamic`, and internally `object` to the `CLR`. But do not break your head with that, read on.

4. Declaring something as dynamic does this one important thing - defers all the type-checks and invocation till the run-time. Basically, all the statements with dynamic stuffs are ignored at compile-time.

5. Use of `dynamic` gives this one benefit - you can use the same variable to store different types of data (can also be done with `object`) and can seamlessly invoke members on them (NOT possible with `object`), without having to type cast or using reflection!

6. As wel all have heard so many times ***With great power comes great responsibility.*** Using of dynamic gives you the responsibility of writing the correct-and-safe code, as compiler will not do that for you. If you screw up something, your application will break at run-time.

7. This was introduced mostly to work with systems and codes that does not follow `.NET` style strong typing and using `dynamic` will reduce lot of boilerplate code. It is particularly useful when working with `COM` and languages like `Python`, `Ruby` etc. So, again **use `dynamic` only if you really need to**.

8. Big cons of using dynamic - losing compile-time checks, increased probability of errors.

Now the code demo:
----

```cs
class Demo //demo class used in code
{
    public string Name { get; }
    internal string GetId()
    {
        return "Cool_ID";
    }
}
```

##### A quick glance at `var` 

This `var` is nothing but a shorthand for defining types, also called implicit typing. They are still statically and strongly typed at compile time, only thing is the type is inferred by compiler at declaration.

```cs
var x; //not allowed, it must be initialized
var z = 3; //z is inferred to be int
z = "woohoo"; //will not compile, as z IS int, not string
var y = null; //doesn't compile as the type cannot be inferred
```

#### Now we'll see `dynamic` in action.

```cs
dynamic x = 3;
x = "jassala"; //allowed, as different data types can be stored in dynamic
x = 0.5; //same as above
x += "hello"; //works fine and produces "0.5hello"
x = new Demo(); //works, obviously
x = x.GetId(); //compiles fine, as no check is done at compile time. Runs fine as well, as the code is right
x = x.GetWhatIsNotThere(); //still compiles, BUT throws RuntimeBinderException: ''string' does not contain a definition for 'GetWhatIsNotThere''
```

In the last line, the code still compiles - because compiler skips all the checks for variable declared to be dynamic!

#### The same code with `object`

```cs
object y = 3; //to show difference with object
y = "jassala"; //compiles and runs, as all types are still objects
y = 0.5; //same
y += "hello"; //same, produces "0.5hello"
y = new Demo(); //works, for same reason
y = y.GetId(); //DIFFERENCE: This gives compile error, as object does not have a definition of GetId
y = y.GetWhatIsNotThere(); //DOESN'T compile for same reason
```

#### `dynamic` in method return
This is for illustration only. Understand how unsafe and unpredictable the code can be. _Do NOT try this at home or work_ unless you unerstand exactly what you are doing.

```cs
public dynamic DynamicMethod(string s)
{
    dynamic result;
    try
    {
        int num;
        int.TryParse(s, out num);
        result = num;
    }
    catch (Exception ex)
    {
        result = ex;
    }
    return result;
}
```
