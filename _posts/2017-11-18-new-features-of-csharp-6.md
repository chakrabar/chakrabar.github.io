---
layout: default
title: "Features demo of C# 6"
date: 2017-11-18
tags: tech tips notes code-demo
---



# A quick glance at the new capabilities of C# 6 

***This is a code demo, not elaborate theories.***

If you are like me who got little late in catching up with the latests and greatests of the `C#` language, this will help you catch up quickly.

There are no theoretical or academic details here. If you are interested in them, there are tons of resources in MSDN and all over internet, just do your reserach. This article will directly introduce you to some of the most useful features of C# 6 in a code & code-only manner (with a bit of explanation in the comments). 

C# 6 does not add new framework features, rather gives a bunch of cool new language constructs. All the enhancements we'll see, will help developers write cleaner & more precise code.

## These new stuffs are covered in code below

1. Auto read-only property
2. Auto property initializer
3. Expression-bodied functions
4. Using static
5. Null-conditionals OR null-propagation operators
6. String interpolation
7. Exception filters
8. nameof() Expressions
9. Await in Catch & Finally blocks
10. Index initializers

### So, here you go

```cs

using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using static CSharpLearning.DemoType; //see [4] using static

namespace CSharpLearning
{
    public class DemoType
    {
        public string Name { get; set; }
        public int Count { get; set; }
        public static string AddSmiley(string message)
        {
            return message + " :)";
        }
    }

    public class CSharp6
    {
        //1. Auto read-only property
        //Makes a read-only property, that can be set ONLY in constructor or initializer
        //Earlier, declaring only get accessor was not allowed, private set; was allowed though
        public string ReadOnlyProperty { get; } //See [3] for an alternative syntax
        //constructor can Set ReadOnlyProperty
        public CSharp6()
        {
            ReadOnlyProperty = "Value";
        }

        //2. Auto property initializer
        //Set initial value of properties in declaration with an assignment sign
        public int AutoInitializedProperty { get; set; } = 10;
        public IList<decimal> AutoInitializedListProperty { get; } = new List<decimal>(); //Read-only properties can also be initialized
        public Func<string, string> WelcomeMessageGeneratorProp { get; set; } = (name) => $"Welcome {name}."; //See Expression-bodied function & String Interpolation        

        //3. Expression-bodied functions
        //Shorten code for simple Methods or Read-Only Properties with lambda expressions
        public int DoubleOf(int num) => 2 * num;
        //Another representation of read-only property
        public string AnotherReadonlyProp => "Hello there"; //[1] Alternative syntax

        //4. Using static
        //Import static methods of a class by declaring - using static Namespace.Class;
        //After that use all the STATIC methods directly wothout mentioning class-name
        void SomeMethod()
        {
            var cuteMessage = AddSmiley("Hello devs"); //class-name not used, like - SomeUtility.AddSmiley("Hello devs");
        }
        //Note a: This way imported methods can only be used as static methods, not as extensions - even if they are
        //Note b: When a class is imported like this, nested classes are also imported

        //5. Null-conditionals OR null-propagation operators
        //When this (.?) is used, compiler will always check for null before invoking the next member. If null is encountered, null is returned as result
        void NullDemoMethod()
        {
            DemoType demoObject = null;
            var name = demoObject.Name; //this will throw null reference exception as DemoProperty is null
            var name2 = demoObject?.Name; //Now if left hand side of ?. is null, null will be set as final value
            //var result = SomeObject?.Details?.Address?.City; //like this, null check is performed and propagated. Any null will result in null being returned.
            var safeName = demoObject?.Name ?? "No name"; //setting default value with null coalescing operator
            int count = demoObject?.Count ?? 0; //here, since left side is not-nullable, it MUST be given a default value TO COMPILE
        }
        //Note: With null conditional operator, the expression is EVALUATED JUST ONCE AND CACHED. So it wil not face a situation where the check has reached 
        //nth level and then some other code sets (n-1)th part to null. It will still work as it has been cached already. Really helpful for delegate invokation.

        //6. String interpolation
        //New syntax for string formatting. String expression is prefixed with $, then variables/expressions are placed within {braces}
        //Functionally replaces (still available though) String.Format. 
        void StringFormattingDemo()
        {
            var obj = new DemoType { Name = "AC", Count = 10 };
            var messageOld = string.Format("Name is {0}, count is {1}", obj.Name, obj.Count); //old style formatting with positional parameters - still works
            var messageNew = $"Name is {obj.Name}, count is {obj.Count}"; //New cool syntax. Intellisense works fine within string expression
            var itsCool = $"Name starts with {obj.Name.Take(1)}. Percentage: {(obj.Count / 100):F2}"; //takes any expression and string formatting too!
        }

        //7. Exception filters
        //You can now filter catch blocks with conditions. Only when the condition (and Exception type) matches, the block will execute. Else, skip.
        //Condition is checked before entering block. If none of the catch blocks match (Exception type + condition), program skips all and throws.
        void ExceptionFilterDemo()
        {
            var obj = new DemoType { Name = "AC", Count = 10 }; //assume it is done in some other code
            try
            {
                var message = $"The initial is - {obj.Name.First()}";
            }
            catch (NullReferenceException ex) when (obj == null) //will NOT execute if obj.Name is null
            {
                Log("Told you to populate the object first");
            }
            //NOTE: Interesting, if obj.Name is null, stack will show original ex thrown in original point, as execution never reached the catch block.
        }

        void Log(string msg) { } //dummy log methods
        public async Task LogAsync(Exception ex) { await Task.Delay(2000); }

        //8. nameof() Expressions
        //Gets you the simple (unqualified) string name of a variable, type, or member.
        //The basic idea is to use an expressions rather than a hard-coded string value, so that code survives through refactorin/rename
        public void NameofDemo()
        {
            var obj = new DemoType { Name = "AC", Count = 10 };
            //examples
            var n1 = nameof(obj); //produces "obj"
            var n2 = nameof(obj.Name); //produces "Name"
            var n3 = nameof(DemoType); //produces "DemoType"
            //var n4 = nameof(obj.GetType()); //Compile error: Expression does not have a name
            //var n5 = nameof(typeof(DemoType)); //Compile error: Expression does not have  a name
            //sample use case
            Log($"Value of {obj.Name} was null"); //Upgrade from hard coded "Name" in log message "Value of Name was null"
            //Note the huge benefit here - if name of Name property is changed, the log message will automatically be updated!
        }

        //9. Await in Catch & Finally blocks
        //Now await can be used in finally as well as catch blocks
        public async void AwaitDemo()
        {
            try
            {
                await System.IO.File.ReadAllLinesAsync("path to file");
            }
            catch (Exception exception)
            {                
                await LogAsync(exception); //Catch operation also can be async now!!
            }
        }

        //10. Index initializers
        //C#6 introduces initializing indexed collections and dictionaries like lists
        public void IndexInitializerDemo()
        {
            Dictionary<int, string> students = new Dictionary<int, string>
            {
                [101] = "Sumit",
                [102] = "Deepmala",
                [110] = "Priyanka"
            };
            //This may look very similar to the OLD syntax as below
            Dictionary<int, string> oldStudents = new Dictionary<int, string>
            {
                {5, "Neha" },
                {8, "Vikas" }
            };
            //Well, functionally they are very similar. There is a inherent difference though
            //The old syntax uses the 'Add' method on the Type whereas the new syntax actually uses the indexer
        }
        //Note: If you have a custom collections that does not have an Add() method, you can create an extension method and use collection initializers.

        //REMOVED feature - Primary constructor
        //Idea was to define constructor parameters with class declaration. It was never released.
        //public class CoolNew (string name, int code)
        //{
        //    public string AreaName { get; set; } = name;
        //    public int AreaCode { get; } = code;
        //}
    }
}


```

It'll be good idea to copy-paste the whole code, call the methods and debug.

Once done, continue to **[features demo of C# 7](/2017/12/17/new-features-of-csharp-7.html)**.

**Note: Source of knowledge for this article are MSDN, StackOverflow and other forums.