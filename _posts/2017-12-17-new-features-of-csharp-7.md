---
layout: default
title: "New features of C# 7.0"
date: 2017-12-17
tags: tech tips notes code-demo
---



# A quick glance at the new capabilities of C# 7.0 

***This is a code demo, not elaborate theories.***

If you are familiar with `C# 6` but not `C# 7`, this will help you catch up quickly. **If you are not not comfortable with `C# 6` yet, go and read the [New features of C# 6](/2017/11/18/new-features-of-csharp-6.html) article first.**

There are no theoretical or academic details here. If you are interested in them, there are tons of resources in MSDN and all over internet, just do your reserach. This article will directly introduce you to some of the most useful features of C# 7 in a code & code-only manner. 

C# 7 does not add a lot of new framework features, but builds on top of C# 6 to give more language constructs. Apatrt from making the laguage easier to read and write, it also takes C# little more closer to functional languages. Some of the new concepts and constructs might take a while to wrap your head around them, so it's suggested that you do more research and write some code to get familiar with them.

## These new stuffs are covered in code below

1. Inline initialization of out parameter
2. Tuple enhancements
    1. Example of new syntax - method returning tuple
    2. Tuple Deconstruction
    3. Deconstruction of user defined types
3. Discards
4. ...
5. ...
6. ...

### So, here you go

```cs

using System;
using System.Linq;
using System.Threading.Tasks;

namespace CSharpLearning
{
    public class CSharp7
    {
        //dummy log methods
        void Log(string msg) { }
        public async Task LogAsync(Exception ex) { await Task.Delay(2000); }

        //quick recap - usage of ref & out
        public void MethodWithRef(ref string str) { } //ref parameter may not be assigned inside, that argument must be initialized before call
        public void MethodWithOut(out string str) { str = "whatever"; } //out parameter must get assigned inside, that argument may not be initialized

        public void OutRefBasic()
        {
            string str1 = "initial value";
            string str2;
            MethodWithRef(ref str1);
            MethodWithOut(out str2);
        }

        //1. Inline initialization of out parameter
        //out variables can be declared inline in the method call statement, with specific type or var. 
        public void OutInitializationDemo()
        {
            var isStringAnInteger = int.TryParse("0100", out int result); //This. out var result - also works
            if (isStringAnInteger)
                Log($"The number is {result}"); //the variable can be used normally as if it was declared before calling
            //if (int.TryParse("0100", out int result)) is a more cleaner way to write it
            //When used like this, the declaration "leaks" into outer scope of the if-statement
        }

        //2. Tuple enhancements
        //Tuples are light-weight data structures that can be declared/created inline and used to return multiple values
        //C# 7 makes tuples more of a first-class citizen with cleaner syntax, named-elements
        //This needs System.ValueTuple NuGet, which comes by default in .NET Framework 4.7, .NET Core 2.0, .NET Standard 2.0
        public void TupleDemo()
        {
            //OLD syntax, still valid
            var tuple1 = Tuple.Create("str", 1); //this is a Tuple<string, int>
            var tuple2 = new Tuple<string, int>("str", 1);
            var str1 = tuple2.Item1; //elements are auto-named as Itemn, which cannot be changed

            //NEW syntax
            var tuple3 = ("str", 1); //Install System.ValueTuple NuGet, if required
            string item1 = tuple3.Item1; //Item1 since no name was given

            //Tuples with NAMED elements
            //IMportant - these names exist in compile time only. They cannot be fetched e.g. through Reflection in runtime
            (string Name, int Id) tuple4 = ("name", 1);
            var name = tuple4.Name;
            var tuple5 = (ObjeName: "yoyo", ObjId: 3);
            var id = tuple5.ObjId;

            //Something that might create confusion. This works, but shows warning.
            (string FirstName, string LastName) tuple6 = (Ignored1: "fname", Ignored2: "lname"); //shows green squiggly lines for warning!
            //here, the names "Ignored1" & "Ignored2" will be ignored as "FirstName" & "LastName" will get precedence as target type
            var firstName = tuple6.FirstName; //tuple6.Ignored1 does NOT exist

            //Something even less expeted!
            //DECONSTRUCTION of the tuple - see below [2.2]
            (string Name, int Id) = ("name", 1);
            string copyOfName = Name; //DECONSTRUCTED variable declaration!
        }

        //2.1 Example of new syntax - method returning tuple
        public (char first, char last) GetEndCharacters(string input) //names of the return fields does not matter, they are just variable names
        {
            if (string.IsNullOrEmpty(input))
                throw new ArgumentException($"Parameter {nameof(input)} cannot be null or empty!");
            return (input.First(), input.Last());
        }

        //2.2 Tuple Deconstruction - directly assign tuple fileds to a set of variables
        //You can directly stores values of Tuple fields in separate variables by putting them inside parenthesis 
        public void TupleDeconstructionDemo()
        {
            var data = "some string data";
            (char first, char last) = GetEndCharacters(data);
            var message = $"The end characters in {data} are {first} and {last}";
            var (first1, last1) = GetEndCharacters(data); //works just like above
        }

        //DECONSTRUCTION - think of it someting opposite of construction. NOT DESTRUCTION, which wipes out the thing from memory!
        //Like CONSTRUCTOR assembles multiple pieces of data into one structure, DECONSTRUCTOR breaks a structure into multiple data elements :)

        //2.3 Deconstruction of user defined types
        //Needs to declare Deconstruct() method(s) with out parameters, for as many elments it needs to be deconstructed to
        public class CoolClass
        {
            public string Name { get; set; }
            public int Id { get; set; }

            public CoolClass(string name, int id) => (Name, Id) = (name, id); //Constructor

            public void Deconstruct(out string name, out int id) //Deconstructor
            {
                name = Name;
                id = Id;
            }
        }
        //With the Deconstructor() method present in a Type, it can be directly deconstructed (assigned to set of variables)
        public void UserDefinedTypeDeconstructorDemo()
        {
            var cool = new CoolClass("cool", 5);
            (string name, int id) = cool; //well, isn't it really cool?
            var (coolName, coolId) = cool; //another approach for the same!
        }

        //3. Discards :D
        //Discards are a variable (denoted with _) to receive all the throw-away-ble values that you do not care about.
        //Discards are write-only variable, they cannot be read from or used in any other way. They're basically a bin for collecting all unimportant values.
        //The _ variable does not need a declaration and one variable can take many values, all at once. Mostly used in deconstruction, out variables etc.
        public (char fisrt, char last, int length, bool hasNumeric) GetDetails(string input)
        {
            if (string.IsNullOrEmpty(input))
                throw new ArgumentException($"Parameter {nameof(input)} cannot be null or empty!");
            return (input.First(), input.Last(), input.Length, input.Any(c => char.IsNumber(c)));
        }
        public void DiscardDemo()
        {
            var (_, _, length, _) = GetDetails("My string"); //Here I'm only interested in length, so I DISCARD other values
            var isTooLong = length > 100; //this works, obviously. BUT _ cannot be used anymore to retrieve values
            //string something = _.ToString(); //DOES NOT work. _ is write-only & non-usable otherwise
            _ = isTooLong.ToString(); //This works fine. This is again reuse of discard
        }
        //If you have your own _ variable for real use, be careful not to confuse the two!
        public void DiscardAndLocalVariableDemo()
        {
            string _ = "my value"; //a local variable with name _ , this is not a discard
            var (_, _, length, _) = GetDetails("My string"); //these are discards
            var message = $"Value of local _ is : {_}"; //This works. Here _ is the local variable
        }
    }
}

```

It'll be good idea to copy-paste the whole code, call the methods and debug.