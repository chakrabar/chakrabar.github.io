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

> WebAssembly is a new, low level binary code,  for the web!

_Wait, a what?_

That's right! So, it's
![image-right](/images/posts/webassembly/Web_Assembly.png){: .pull-right}

* A `new language`, became the offical _**"fourth language for web"**_ on [5th December 2019](https://www.w3.org/2019/12/pressrelease-wasm-rec.html.en), though it has been in development for few years now
* It's `binary`, so it is not really human readable
* It's `low level`, i.e. it's much closer (than `JavaScript` and other higher level languages) to actual machine code
* And it's `for the web`, that means (modern) web browsers can read and execute them

### So, how do I write code in `WebAssembly`?

Though there is a [text format](https://developer.mozilla.org/en-US/docs/WebAssembly/Understanding_the_text_format), you as a developer won't generally _"write"_ code in `WebAssembly`! **It's actually an `IR` or `Intermediate Representation`**, just like `ByteCode` in `Java` or `MSIL/CIL` in `.NET`. 

It's a set of binary instructions that works across common standard hardware. So rather than writing code in it, you still write code in a language that you already know, like `C/C++` or `Rust`, and a special compiler compiles your code into `WebAssembly`, and that runs inside a special `Virtual Machine` that can turn that `IR` into actual, platform specific `Machine Code` and execute them.

![Image](/images/posts/webassembly/WebAssembly_compile.png)

Yes, that's like the main idea behind WebAssembly, _**to be able to compile & run other languages on web!**_

But, why?

### Why `WebAssembly`?

Okay, so I cannot read or write code in WebAssembly! Then what is the purpose of it? Why have it as the _fouth language for the web_? We already have our `JavaScript`!

Yes, `JavaScript` works, and it works pretty well. And there are really cool `JS Frameworks` like `Angular`, `React`, `Vue` etc. In fact `JS` is so cool, that now lot of people even write back-end or server-side code also in `JS`, thanks to `Node`!

BUT, there are issues with `JavaScript`, mainly with **`performance`**! It works fine for a standard web application where small pieces of processing is done based on user interaction. Generally, the heavy processing is done on servers, and web interacts with them through services. But it does not work well when lot of real-time processing is required, think of - image & video processing, 3D gaming, AR/VR etc.

`JavaScript` is an interpreted language i.e. at the run-time, each line of code is turned into machine code as it is encountered. This happens everytime, even the same line of code is executed multiple times. This is inefficient. And there is no way to look at the code as whole at once & ompimize. Also, the garbage collector cycles take it's time to run and free up memory. Well, this is how `JavaScript` used to run traditionally. But the modern `JavaScript` engines use `JIT compiler` which has some performance improvement over it, but still there's problems, partly because of the `dynamic` nature of `JavaScript`. Read in more details in [this series by Lin Clark](https://hacks.mozilla.org/category/code-cartoons/a-cartoon-intro-to-webassembly/).
{: .notice--success}

Web has been an important medium of software user interaction, and with advanced SaaS & cloud applications, users are now enabled to do more & more stuffs on web. Users & developers, both want to the users to be able to do more stuffs on web, and faster. Apart from `JavaScript` being not-so-fast, always going over the network for heavy processing & bringing back results, makes fast processing on web application almost impossible.

`WebAssembly` aims to solve this problem. WebAssembly is a new standard of assembly (Intermediate Code) that

1. Any language can target
2. Can be executed on web

With that, developers can write their code on high-performance languages like `C/C++` or `Rust` and compile them into `WebAssembly`. This `IR` code matches very closely with machine languages across hardwares. Then the browser (with WebAssembly VMs) can directly translate that into machine code & execute. This can make the code run real fast in near native (directly running compiled machine code) speed.

For this to happen, 2 things are required. [1] Different languages need to compile into WebAssembly [2] Browsers need to be ready to run WebAssembly. For _#1_, different languages need to come up with new compilers that target WebAssembly rather than their standard IR. This is already available for `C/C++`, `Rust` and even `C#`. And very soon we'll have compilers for many other langues. And _#2_ is not an issue, as all the modern brosers already support WebAssembly viz. `Firefox`, `Chrome`, `Edge` & `Safari`. In fact, WebAssembly is a joint effort by `W3C`, `Mozilla`, `Google`, `Microsoft` & `Apple` (and of course community members). So, all the major browsers should always be up to date with latest `WebAssembly` standard.
{: .notice--info}

![Image](/images/posts/webassembly/wasm-browsers.png)