---
layout: default
title: "Dynamic in C#"
date: 2017-11-14
tags: tech tips notes code-demo
---



# A quick glance at the dynamic keyword 

It's just a quick-and-dirty code demo, to show the basic usage of dynamic keyword and difference with other similar constructs. This does not cover any theories at all. Go to MSDN or other good resources that can give better explanations.

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
class Temp //demo class
{
    public string Name { get; }
    internal string GetId()
    {
        return "Cool_ID";
    }
}

//In the actual class, method
var z = 3; //z is statically typed as int at compile time. var is just syntactic sugar, comiler will infer the actual type
z = "jassala"; //will not compile, as z IS int, not string

dynamic x = 3;
x = "jassala"; //allowed, as different data types can be stored in dynamic
x = 0.5; //same as above
x += "ekigo"; //works fine and produces "0.5ekigo"
x = new Temp(); //works, obviously
x = x.GetId(); //compiles fine, as no check is done at compile time. Runs fine as well, as the code is right
x = x.GetWhatIsNotThere(); //still compiles, BUT throws RuntimeBinderException: ''string' does not contain a definition for 'GetWhatIsNotThere''

object y = 3; //to show difference with object
y = "jassala"; //compiles and runs, as all types are still objects
y = 0.5; //same
y += "ekigo"; //same, produces "0.5ekigo"
y = new Temp(); //same
y = y.GetId(); //DIFFERENCE: This gives compile error, as object does not have a definition of GetId
y = y.GetWhatIsNotThere(); //same
```