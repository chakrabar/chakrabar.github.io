---
layout: post
title: "New features of C# 8.0"
excerpt: "For internal purposes only"
date: 2018-12-23
tags: [coding, programming]
categories: articles
comments: true
share: true
published: false
---

#### Nullable references

Weren't they nullable by nature?

Yes, in a way. But that has got us into all these null related mess for over 50 years now. `Null` is not an instance of of any type, it is nothing. Being able to assign it to specific types (reference types), we've got us into this mess.

To slowly get us out of this mess, while still keeping the decade old convention of being able to assign `null` to reference types, here comes nullable reference types. This also immensely helps to clearly state your intent as a programmer, whether you allow `null` to be assigned to your `reference type` variable or not.

```cs
//A nullabe string (syntax similar to nullable value type)
string? s = null; //allowed
//All traditionally expressed reference types are non-nullable
string s2 = null;  //throws warning
```

So any time you declare a reference type the traiditional way, they become _non-nullable_. So any time compiler detects you are assigning a nul to it, or something that can potentially be null (e.g. a nullable reference), compiler throws a warning! For this, the compiler can pretty much follow large control flows as well.

But that will break most of the existing code? Yes, that's why compiler throws warning, not error. Also, this feature is not turned on by default. You can opt-in, per project.

## References

* https://blogs.msdn.microsoft.com/dotnet/2018/11/12/building-c-8-0/
* https://blogs.msdn.microsoft.com/dotnet/2018/12/05/take-c-8-0-for-a-spin/
* https://github.com/dotnet/csharplang/wiki/Nullable-Reference-Types-Preview
* https://github.com/dotnet/csharplang/tree/master/proposals
* https://github.com/dotnet/csharplang/blob/master/proposals/default-interface-methods.md
* https://github.com/dotnet/csharplang/issues/288
* https://github.com/dotnet/csharplang/issues/406