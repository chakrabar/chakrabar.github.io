---
layout: post
title: "WebAssembly - What it is & Why is it so exciting"
excerpt: "The what, why, how & when of WebAssembly for the dummies"
date: 2020-02-29
tags: [dev, programming, web, webassembly, wasm, wasi, wat]
categories: articles
comments: true
share: true
published: true
---

If you are a web developer, or a software developer in general, you must have heard of somethig called `WebAssembly` in last few years. If you did, but do not know actually what it is, and why you keep hearing about them - this artcle's for you. If you are already compiling your code into `wasm` and loading them into your `JavaScript` code, then, well, you'll probably not get much out of it.

If you're in the first group, jump right in. Here, we'll try to understand what is `WebAssembly`, what is the context of it, the hype vs. the fact, and a bit of the future. You might have already come across bunch of articles, blog posts, videos & even conference talks about it, but still have no clue if it's a new tool, a design pattern or a shiny new JavaScript framework!

## What is _WebAssembly_

> WebAssembly is new, low level binary code,  for the web!

_Wait, a what?_

That's right! So, it's
![image-right](/images/posts/webassembly/Web_Assembly.png){: .pull-right}

* A `new language`, became the offical _**"fourth language for web"**_ on [5th December 2019](https://www.w3.org/2019/12/pressrelease-wasm-rec.html.en), though it has been in development for few years now
* It's `binary`, so it is not really human readable
* It's `low level`, i.e. it's much closer (than `JavaScript` and other higher level languages) to actual machine code
* And it's `for the web`, that means (modern) web browsers can read and execute them

### So, How do I write code in `WebAssembly`?

Though there is a [text format](https://developer.mozilla.org/en-US/docs/WebAssembly/Understanding_the_text_format), you as a developer won't generally _"write"_ code in `WebAssembly`! **It's actually an `IR` or `Intermediate Representation`**, just like `ByteCode` in `Java` or `MSIL/CIL` in `.NET`. It's a set of binary instructions that works across common standard hardware. So rather than writing code in it, you still write code in a language that you already know, like `C/C++` or `Rust`, and a special compiler cpmpiles your code into `WebAssembly`, and that runs insode a special `Virtual Machine` that can turn that `IR` into actual, platform specific `Machine Code` and execute them.

![Image](/images/posts/webassembly/WebAssembly_compile.png)