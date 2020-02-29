---
layout: post
title: "WebAssembly - What it is & Why is it so important"
excerpt: "The what, why, how & when of WebAssembly for the dummies"
date: 2020-02-29
tags: [dev, programming, web, webassembly, wa, wasm, wasi, wat]
categories: articles
image:
  feature: posts/webassembly/office-3.jpg
comments: true
share: true
published: true
---

If you are a web developer, or a software developer in general, you must have heard of something called `WebAssembly` in last few years. If you did, but do not know actually what it is, and why you keep hearing about them - this artcle's for you. If you are already compiling your code into `wasm` and loading them into your `JavaScript` code, then, well, you'll probably not get much out of it.

If you're in the first group, jump right in. Here, we'll try to understand what is `WebAssembly`, what is the context of it, the hype vs. the fact, and a bit of the future. You might have already come across bunch of articles, blog posts, tweets, videos & even conference talks about it, but still probably have no clue what it is! Is it a new tool, a design pattern or a shiny new JavaScript framework?

## What is _WebAssembly_

> WebAssembly is a new, low level binary code,  for the web!

_Wait, a what?_

That's right! Let us take it apart, and understand piece-by-piece
![image-right](/images/posts/webassembly/Web_Assembly.png){: .pull-right}

* It's a `new language`. It became the offical _**"fourth language for web"**_ on [5th December 2019](https://www.w3.org/2019/12/pressrelease-wasm-rec.html.en), though it has been in development for few years now
* It's `binary`, so it is not really human readable (hence, _"assembly"_)
* It's `low level`, i.e. it's much closer to actual machine code (than `JavaScript` and other higher level languages)
* And it's `for the web`, that means (modern) web browsers can read and execute them

### So, how do I write code in `WebAssembly`?

Though there is a [text format](https://developer.mozilla.org/en-US/docs/WebAssembly/Understanding_the_text_format), you as a developer won't generally _"write"_ code in `WebAssembly`! **It's actually an `IR` or `Intermediate Representation`**, comparable to `ByteCode` in `Java` or `MSIL/CIL` in `.NET`. 

Basically it's a set of low level binary instructions that works across common standard hardware. Unlike higher level languages like `C#`, `Go` or `Java`, it is more closer to machine language, and can be converted into actual `machine code` very fast & efficiently. So rather than writing code in it, you still write code in a language that you already know, like `C/C++` or `Rust`, and a special compiler compiles your code into `WebAssembly`, and that runs inside a special `Virtual Machine`. This `VM` can turn the `IR` into actual, platform specific `Machine Code` and execute them. This `VM` can run on any type of platform (hardware + OS).

![Image](/images/posts/webassembly/WebAssembly_compile.png)

In a nutshell, that's like the main idea behind WebAssembly, to be able to _**run pre-compile code, written in any language, on the web!**_

Okay, but why?

### Why `WebAssembly`?

Okay, so I cannot read or write code in WebAssembly! Then what is the purpose of it? Why have it as the _fouth language for the web_? We already have our `JavaScript`!

Yes, `JavaScript` works, and it works pretty well. And there are really cool `JS Frameworks` like `Angular`, `React`, `Vue` etc. In fact `JS` is so cool, that now lot of people even write back-end or server-side code also in `JS`, thanks to `Node`!

BUT, there are issues with `JavaScript`, mainly with **`performance`**! It works fine for a standard web applications where small pieces of processing is done on the web, based on user interaction. Generally, the heavy processing is done on servers, and web interacts with them through `http` services. But it does not work well when lot of real-time processing is required, think of - image & video processing, 3D gaming, AR/VR etc.

**NOTE:** JavaScript is an interpreted language i.e. at the run-time, each line of code is turned into machine code as it is encountered. This happens everytime, even the same line of code is executed multiple times (e.g. loops). This is inefficient. And there is no way to look at the whole code at once & ompimize. Also, the garbage collector cycles take it's time to run and free up memory, stalling the applications. Well, this is how JavaScript used to run traditionally. But the modern JavaScript engines use `JIT compiler` which partially compiles the code & has some performance improvements. But still it's not the best we could get, partly because of the `dynamic` nature of JavaScript. Read in more abou them in this [series of posts by Lin Clark](https://hacks.mozilla.org/category/code-cartoons/a-cartoon-intro-to-webassembly/).
{: .notice--success}

Web has been an important medium of user interaction for software programs, and with modern SaaS & cloud applications, users are now enabled to do more & more stuffs on web. Users & developers, both want to be able to do more stuffs on web, and faster. Apart from `JavaScript` being not-so-fast, always going over the network for heavy processing & bringing back results, makes fast processing on web applications almost impossible.

`WebAssembly` aims to solve this problem. WebAssembly is a new type of assembly language (Intermediate Code) that

1. Any language compiler can target
2. Can be executed on web, stand-alone

With that, developers can write their code on high-performance languages like `C/C++` or `Rust` and compile them into `WebAssembly`. This `IR` code matches very closely with machine languages across hardwares. Now the browser (with WebAssembly VMs) can directly translate that into machine code & execute. This can make the code run real fast in near native (directly running compiled machine code) speed.

**Important:** For this to happen, 2 things are required. [1] Different languages need to compile into WebAssembly [2] Browsers need to be ready to run WebAssembly. For _#1_, different languages need to come up with new compilers that target WebAssembly rather than their standard IR. This is already available for `C/C++`, `Rust` and even `C#`. And very soon we'll have compilers for many other langues, some of them might be ready by the time you read this. And _#2_ is not an issue, as all the modern brosers already support WebAssembly viz. `Firefox`, `Chrome`, `Edge` & `Safari`. In fact, WebAssembly is a joint effort by `W3C`, `Mozilla`, `Google`, `Microsoft` & `Apple` (and of course community members). So, all the major browsers should always be up to date with latest WebAssembly standards.
{: .notice--info}

![Image](/images/posts/webassembly/wasm-browsers.png)

## Points to remember

Now that we know what is WebAssembly and what problem it aims to solve, let's look at a bunch of important points to note.

1. `WebAssembly` is still a **new & evolving** technology. Things are changing very fast, and by the time you read, many things might have changed
2. It's a **low level binary code**, meant to be **target of compilation** for higher level (ideally fast) languages. Any language that wants to generate `WebAssembly`, needs to come up with a compiler that compiles from that language to `WebAssembly` IR
3. As a language, it's **strongly typed** (unlike `JavaScript`)
4. It **runs inside a `VM`** (like `Java` runs indside `JVM` or `C#` runs inside `.NET CLR`). The `WebAssembly Virtual Machine` works like a **Stack Machine** i.e. for each operation, it `push` or `pop` something on the internal memory stack (e.g. a functions pops operands from the stack and at the end, the return value is pushed on top of stack)
5. `WebAssembly` is frequently abbreviated as `WA` or **`wasm`**. Actual WebAssembly files also have `*.wasm` extension
6. Generally the binary size is pretty **small** (compared to `JavaScript`, even when compressed) & **executes fast** because of it's near native model
7. On browser, `WebAssembly` runs with same priviledges as `JavaScript` code, so it has the same **security** levels of current browser code (e.g. it cannot start manipulating files on client system, without explicit user action)
8. It's **portable & platform agnostic**, i.e. indepndent of underlying `hardware` or `OS`
9. There is a [text representation](https://developer.mozilla.org/en-US/docs/WebAssembly/Understanding_the_text_format) of `wasm`, that is human readable and follows `s-expressions` format. Though it's totally possible to write code in `WAT`, the main idea behind this is to help `debug` WebAssembly code at run-time (it's not ready though)
10. `WebAssembly` enables web browsers to run code written in other languages than `JavaScript`. `C/C++` and `Rust` were supported from the beginning. `.NET` already supports it through [Blazor](https://dotnet.microsoft.com/apps/aspnet/web-apps/blazor), and support for many other languages should be available soon
11. Some popular compilers for `WebAssembly` are [LLVM & Emscripten](https://v8.dev/blog/emscripten-llvm-wasm). Currently **Emscripten** is the most popular compiler
12. To get the taste of first-hand `wasm` development, follow this **[Hello World](https://webassembly.org/getting-started/developers-guide/) tutorial**

### Why `Wasm` is great

1. Can have much _**higher performance**_ than JavaScript
    1. Smaller download size, mainly for the binary format
    2. As code is already in IR (Intermediate Representation) code, it is much closer to machine code. So the whole code can be translated to stable machine code quickly rather than the optimize & re-optimize cycles of JS JIT compiler (also because the types are static)
    3. No Garbage Collection by design, so `GC` cycles are saved
2. _**Code sharing**_
    1. It provides a simple and performant way of sharing `pre-compiled` (`wasm` as IR is close to machine code) code across platforms, written in `different languages`
    2. Same `wasm` code can run in front-end as well as in the back-end, like standard server side code. So basically, code sharing between front-end & back-end. **Note** that it's already possible with `JavaScript` & `Nodejs`, but that doesn't provide speed of lower level languages like `C/C++`
3. Different `wasm` modules, written in different languages, can interoperate easily, making it very easy to collaborate between developers from different backgrounds and _**interoperate otherwise incompatible modules**_ (this will become possible with implementation of [interface types](https://www.youtube.com/watch?time_continue=12&v=Qn_4F3foB3Q&feature=emb_logo))
4. Works seamlessly with `JavaSCript` i.e. `wasm` & `JavaScript` can exchange data and call each other. In fact, currently `wasm` cannot be run without `JS` becuase [1] It needs `JS` to load the `wasm` module [2] `DOM` manipulation happens through `JS` only
5. With `wasm` we can truly relize _**serverless web applications**_, that can actually run without any backing servers (it can do heavy computations, connect to database and what not!)
6. Can also _**run outside browser**_, in a safe & fast manner. `Wasm` provides a [light-weight sandboxing](https://hacks.mozilla.org/2019/03/standardizing-wasi-a-webassembly-system-interface/) by default. More about this later

### What's not-so-great (yet)

![image-right](/images/posts/webassembly/wasm-js-2.png){: .pull-right}
1. Not complete, or _"ready"_ yet. Many features are still under development or in proposal stage
2. Compilation and loading process is bit complex, and apparently no official integration into popular build tools like webpack or Angular CLI (this may change soon though)
3. By design, code needs to be compiled before distribution. So, _fix JS directly on CDN_ kind of approach does not work
4. `Internet Explorer` and older versions of other browsers does not support
5. Some conventions of `wasm` makes it more complex for developers to write code. So, high level language compilers need to provide some extra layer for developers to code easily. For example
    1. `Wasm` _**supports only numbers as method parameters**_. To use any other types, one needs to work with memory locations / pointers directly (will be easier with the [interface types](https://hacks.mozilla.org/2019/08/webassembly-interface-types/) implementation)
    2. No automatic `GC`. Memory has to be managed manually
6. Because of current not-so-optimized wasm invocation from `JS`, it doesn’t perform well if there are lot of small functions to be invoked (lot of jumping between `JS` and `wasm`)
7. `WebAssembly`, in the current form, _**cannot directly manipulate DOM**_. No direct access to the Web APIs. For that, it needs to go through `JavaScript`. Even for loading `wasm` modules, it has to depend on `JS`
8. Not really debuggable for high-level language source code (should change in future with source-maps)
9. No fully featured _**multi-threading support**_

**NOTE:** As `WebAssembly` is just in the early stage of development, specs proposals & implementations change fast. Many of the limitations mentioned above might get resolved soon.
{: .notice--info}

## `WASI` - run `WebAssembly` everywhere

If we think about about the core working principles of `WebAssembly`, it's not difficult to visualize `wasm` executing outside web browsers as well! Provided the `Wasm Virtual Machine` is available, `wasm` code can run in any system on any platform, browser or not!

Developers are already running `wasm` outside browsers for the amazing benefits it provides - fast, scalable, secure way to run the same code across all machines.

Now the committee is also pushing the idea, and working to provide an easier & standard bunch of interfaces so that - `wasm` applications written in different languages can run acorss platforms, talk to each-other seamlessly, can access necessary system APIs & securely execute inside a sandbox (memory, network, filesystem, access etc). Thasts **`WebAssembly System Interface`** or _**WASI**_. Read [this](https://hacks.mozilla.org/2019/03/standardizing-wasi-a-webassembly-system-interface/) for more details.

Now think of the implications. `WASI`, if works as expected, will open up crazy new ways of writing applications - on any language, that runs everywhere, and talk to each other fluently without any wrappers or network calls!

When `WASI` was proposed, **`Docker`** co-founder & CTO [Solomon Hykes](https://twitter.com/solomonstre) tweeted, if `WASI` was already available, `Docker` wouldn't have been necessary!

Take a moment or two, and think about that! 🤯

<blockquote class="twitter-tweet tw-align-center"><p lang="en" dir="ltr">If WASM+WASI existed in 2008, we wouldn&#39;t have needed to created Docker. That&#39;s how important it is. Webassembly on the server is the future of computing. A standardized system interface was the missing link. Let&#39;s hope WASI is up to the task! <a href="https://t.co/wnXQg4kwa4">https://t.co/wnXQg4kwa4</a></p>&mdash; Solomon Hykes (@solomonstre) <a href="https://twitter.com/solomonstre/status/1111004913222324225?ref_src=twsrc%5Etfw">March 27, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


**Disclaimer:** I'm by no means a WebAssembly expert. I'm just a developer interested in the applications & future roadmaps of WebAssembly. My whole source of knowledge was my research on wasm to see if it could solve a specific problem that I was not able to solve with current web technologies. Though I figured out that wasm, in it's current state, cannot solve the problem, but all the amazing articles and talks (few mentioned in references below) got me really excited about `WebAssembly` and all the new possibilities it opens up! 😄
{: .notice--warning}

## A few finals notes

* When can I start using `wasm`? Right now! Though it'll change a lot in future, it's already available to use and it really works!
* Will it replace `JavaScript`? No. At least not any time soon. Currently `wasm` depends on `JS` for loading & any kind of `DOM` manipulation. Though it'll change in future, `JS` is already huge and has it's own high place in web development. Practically, it's the only language tailored fit for web development, other languages - not at all. All the amazing JS frameworks also make it almost irreplacable. So, probably no existing `JS` based web applications will be re-written in `wasm`. But a time might come when new web applications might be written completely in `wasm` (in a language like `C++`, `Go`, `Kotlin` or something that doesn't even exist today)!
* What about `Docker`? Will `wasm` replace it? _"No, but imagine a future where Docker runs linux containers, windows containers and wasm containers side by side. Over time wasm might become the most popular container type. Docker will love them all equally, and run it all :)"_ - [Solomon Hykes](https://twitter.com/solomonstre/status/1111113329647325185?s=20)
* Ok, so what about `JS developers`? Is it like all the JS developers will now have to learn a completely new language (maybe more than one?) just to keep up with web development? Well, learning a new language might be really helpful, mainly to benefit from the performance of lower level languages. But it may not be necessary. For instance, there is already a popular language **[AssemblyScript](https://github.com/AssemblyScript/assemblyscript)** that can compile `TypeScript` (a subset actually) into `WebAssembly`.

### References

* [The WebAssembly home page](https://webassembly.org/)
* [GitHub repos](https://github.com/WebAssembly)
* [Mozilla WebAssembly docs](https://developer.mozilla.org/en-US/docs/WebAssembly)
* [A cartoon intro to wasm](https://hacks.mozilla.org/category/code-cartoons/a-cartoon-intro-to-webassembly/)
* [Wasm Hello World guide](https://webassembly.org/getting-started/developers-guide/)
* Tools
    * [WasmExplorer](http://mbebenita.github.io/WasmExplorer/)
    * [WebAssemblyStudio](https://webassembly.studio/)
    * [Emscripten](https://emscripten.org/)
* [Wasm Text Format WAT](https://developer.mozilla.org/en-US/docs/WebAssembly/Understanding_the_text_format)
* [WASI](https://hacks.mozilla.org/2019/03/standardizing-wasi-a-webassembly-system-interface/)
* [AssemblyScript](https://github.com/AssemblyScript/assemblyscript)
* Videos
    * [WebAssembly Demystified - Jay Phelps](https://www.youtube.com/watch?v=6Y3W94_8scw)
    * [Bringing WebAssembly outside the web with WASI by Lin Clark](https://www.youtube.com/watch?v=fh9WXPu0hw8)
    * [Interface Types - Lin Clark](https://www.youtube.com/watch?time_continue=12&v=Qn_4F3foB3Q&feature=emb_logo)
* Also look at this [WebAssembly logo contest](https://github.com/WebAssembly/design/issues/112) from 2015, for fun