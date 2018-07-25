---
layout: post
title: "Closures in JavaScript & C#"
excerpt: "Basics concept and code samples for closures"
date: 2018-07-26
tags: [closure, javascript, C#]
categories: articles
comments: true
share: true
published: false
---

## JavaScript

Functions in JavaScript form closures. Specifically, when a function creates an inner function.
A closure is the combination of a function and the lexical environment within which that function was declared.

This environment consists of any local variables that were in-scope at the time the closure was created. 

Note 1: Inner functions have access to the variables of outer functions (closure), and global ones
Note 2: Variables defined without `var` are always global, even when declared within a function */

Basically, inner functions remember variables in its outer scope, and has access to its current value!
It creates a function with own variable, just like a class in OOP with property and a single function.

```javascript
function makeFunc() {
  var name = 'Mozilla';
  function displayName() {
    alert(name);
  }
  return displayName;
}

var myFunc = makeFunc();
myFunc(); //alert Mozilla

//------------------------

function makeAdder(x) {
  return function(y) {
    return x + y;
  };
}

var add5 = makeAdder(5);
var add10 = makeAdder(10);

console.log(add5(2));  // 7
console.log(add10(2)); // 12

//------------------------

// global scope
var e = 10;
function sum(a){
  return function(b){
    return function(c){
      // outer functions scope
      return function(d){
        // local scope
        return a + b + c + d + e;
      }
    }
  }
}

console.log(sum(1)(2)(3)(4)); // log 20

//------------------------

function makeFunc() {
  var name = 'Mozilla';
  function displayName() {
    alert(name);
  }
  name = 'Google';
  return displayName;  
}

var myFunc = makeFunc();
myFunc(); //alert Google

//------------------------

function getCounter() {
       var counter = 0;
       return function() {
              return ++counter;
       }
}

var counter = getCounter();
counter();
counter();
console.log(counter()); //3
```

## C# (.NET)

In essence, a closure is a block of code which can be executed at a later time, but which maintains the environment in which it was first created - i.e. it can still use the local variables etc of the method which created it, even after that method has finished executing.

The general feature of closures is implemented in C# by anonymous methods and lambda expressions. From [Jon Skeet's SO answer](https://stackoverflow.com/questions/428617/what-are-closures-in-net)

Closures are basically a function created within another function, which captures or encloses the variables from outer function. The actually have reference to that variable (through a compiler generated internal class) and gets the latest value of it!

```cs
public class Closure
{
    public static Action WithDelegate()
    {
        int counter = 0;
        return delegate
        {
            counter++;
            Console.WriteLine("Counter = " + counter);
        };
    }

    public static Action WithLambdaAction()
    {
        int counter = 0;
        return () => Console.WriteLine("Counter => " + ++counter);
    }

    public static Func<int> WithLambdaFunc()
    {
        var counter = 0;
        return () => ++counter;
    }

    //This does NOT behave as explined in https://blogs.msdn.microsoft.com/ericlippert/2009/11/12/closing-over-the-loop-variable-considered-harmful/
    public static IEnumerable<Func<int>> ClosureInLoop()
    {
        var values = new[] { 15, 25, 35 };
        var funcs = new List<Func<int>>();

        foreach (var val in values)
        {
            funcs.Add(() => val);
        }

        return funcs;
    }

    public static void LoopIterationWithLambda()
    {
        var computes = new List<Func<int>>();

        foreach (var i in Enumerable.Range(0, 10))
        {
            computes.Add(() => i * 2);
        }

        foreach (var func in computes)
        {
            Console.WriteLine(func());
        }
    }
}

class Program
{
    static void Main(string[] args)
    {
        try
        {
            var counter3 = Closure.WithLambdaFunc();
            for (int i = 0; i < 4; i++)
            {
                Console.WriteLine("Counter :: " + counter3());
            } //prints 1, 2, 3, 4
        }
        catch (Exception ex)
        {
            Console.WriteLine("Exception!! Message: {0}", ex.Message);
        }
        Console.ReadLine();
    }
}
```