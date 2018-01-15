---
layout: default
title: "New features of C# 6"
date: 2017-11-18
tags: tech tips notes code-demo
---



# A quick glance at the new capabilities of C# 6 

## It's a code demo, not theories behind it

If you are like me who got little late in catching up with the latest and greatest of the `C#` language, this will help you catch up quickly.

This is not theoretical or academic details. If you need that, there are tons of resources in MSDN and all over internet, just do your reserach. This article will directly introduce you to some of the most useful features of C# 6 in a code & code-only manner. So, here you go

```cs
using System;
using System.Collections.Generic;
using static CSharpLearning.DemoType; //see static using

namespace CSharpLearning
{
    public class DemoType
    {
        public string Name { get; set; }
        public static string AddSmiley(string message)
        {
            return message + " :)";
        }
    }

    public class CSharp6
    {
        //Auto read-only property
        //Makes a read-only property, that can be set ONLY in constructor or initializer
        //Earlier, declaring only get accessor was not allowed, private set; was allowed though
        public string ReadOnlyProperty { get; }
        //ctor can access ReadOnlyProperty
        public CSharp6()
        {
            ReadOnlyProperty = "Value";
        }

        //Auto property initializer
        //Set initial value of properties in declaration with an assignment sign
        public int AutoInitializedProperty { get; set; } = 10;
        public IList<decimal> AutoInitializedListProperty { get; } = new List<decimal>(); //Read-only properties can also be initialized
        public Func<string, string> WelcomeMessageGeneratorProp { get; set; } = (name) => $"Welcome {name}."; //See Expression-bodied function & String Interpolation        

        //Expression-bodied functions
        //Shorten code for simple functions (or properties) with lambda expressions
        public int DoubleOf(int num) => 2 * num;
        //Another representation of read-only property
        public string AnotherReadonlyProp => "Hello there";

        //Using static
        //Import static methods of a class by declaring - using static Namespace.Class;
        //After that use all the STATIC methods directly wothout mentioning class-name
        void SomeMethod()
        {
            var cuteMessage = AddSmiley("Hello devs"); //class-name not used, like - SomeUtility.AddSmiley("Hello devs");
        }
        //Note 1: This way imported methods can only be used as static methods, not as extensions - even if they are
        //Note 2: When a class is imported like this, nested classes are also imported

        //Null-conditionals OR null-propagation operators
        public DemoType DemoProperty { get; set; }
        void NullDemoMethod()
        {
            var name = DemoProperty.Name;
        }
    }
}

```