---
layout: post
title: "Modern web development concepts"
excerpt: "WIP - Modern web development concepts"
date: 2020-06-05
tags: [dev, programming, web, webdev, webassembly, wa, wasm, wasi, wat]
categories: articles
comments: true
share: true
published: true
---

**NOTICE:** This is a `Work-In-Progress` post, please do not read it yet! 🐌🐌🐌
{: .notice--warning}

# Basic idea of some modern front-end concepts

## Virtualization

* UI virtualization means that detailed visualization of items is deferred until we scroll the items into visible area of the web page or the `ViewPort`. The term might have come from the concept of `Windowing` the elements i.e. lazily load elements which are within the current window
* The basic idea is, when there is long list of items to be shown, maybe with complex visulization using many DOM elements, the performance & memory load is improved by only displaying the detailed elements which are presently visible. Others are not rendered completely. The moment they enter the viewport, they get fully rendered with available data and logic
* The main difference between `virtualized list/grid/table` and `infinite-scroll` is - 
    * Infinite-scroll shows only a limited number of elements at start. So first load is fast. But after that, it keeps accumulating DOM elements, making the overall DOM heavier and heavier
    * Virtualization on the other hand, always has only a fixed number of visible elements rendered, not more. This is generally achieved by discarding DOM elements once they move out of visible screen OR reusing the same DOMs with different data!
* Some examples - [SlackGrid](https://github.com/mleibman/SlickGrid/) , [Infinite-scroll](https://github.com/metafizzy/infinite-scroll), [Variable height virtual list for Vue](https://github.com/tangbc/vue-virtual-scroll-list#variable-height)

## IntersectionObserver

* The `Intersection Observer API` provides a way to `asynchronously observe` the `changes in the intersection` of a `target element` with an `ancestor element` or with a top-level document's `ViewPort`
* The most common use is to trigger something when an element enters the ViewPort, to implement lazy loading, infinite scrolls and the likes
* [Mozilla docs](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)

## Canvas (and SVG)

The `Canvas` is a comparatively new `HTML5` standard for painting images/graphics on the web! Canvas is just a rectangular area on the page, generally defined with an id, and height & width. The basic HTML Canvas element `<canvas />` has nothing in it. `JavaScript` can be used to utilize Canvas's drawing APIs to paint anything that is required. Those painting APIs are basically set of commands like - draw a line, draw a rectangle, write some text etc. So, based on the JavaScript code, the browser executes the commands, and an image is drawn pixel-by-pixel on the Canvas area. Once the commands are executed, we just have the resultant image as a single HTML element i.e. the original canvas!

`SVG` (Scalable Vector Graphics) on the other hand is a document structure for storing some graph representation. The `<svg>` HTML element contains other drawing components as child HTML elements, which are generally geometric shapes like `path, rectangle, circle` etc. SVG is generally a collection of DOM elements to define a graph, represented as n HTML text, and can be stored like any other text/XML file and be inserted anywhere in HTML like any othe element. No JavaSCript code is necessary to render an SVG on a web page. `JavaScript` can interact with any elements within this the SVG just like any other HTML element, and `CSS` can be used to style them!

### Similarities

1. Both can draw things on a web page, and are supported by modern browsers
2. Both can be scaled & animated & have events, BUT in different ways

### Differences

1. Canvas renders as a single HTML element, whereas an SVG is a collection of HTML elements
2. Canvas cannot be stored as a document. It needs the JavaScript commands to actually draw what is intended. SVG on the other hand is like a XML document which has all the drawing components configured as child elements. For static SVG, no JavaSCript is required, and CSS can be used to style the elements.
3. SVG can be animated in terms of CSS animation OR moving the elements with JavaScript. Elements can be added or removed as well, through JavaScript code. For Canvas to animate, we need to write code to define different frames and each frame has to be displayed by redrawing the canvas.
4. SVG is truly scalable. SVG scales seamlessly as the viewport or the spcific area of the page is scaled up or down, not much of additional effort is required. Also the SVG and overall document size remains same even if the graphics are zoomed in indefinitely. It can also be scaled using code. Canvas can be scaled only by redrawing the whole image again. And larger canvas means more pixels, which is more heavy on memory and resources.
5. For SVG, JavaScript events can be attached to each/any elements just like simple HTML elements. For canvas, events can be added to pixels or areas, based on code and calculations!
6. In general (though it depends on other factors), SVG works best when there are smaller number of drawing elements, and they need simple changes to styles, animate or to scale. Canvas works best for complex interactive drawings applictions like games, photo editing etc. where control over each pixel is more important.

## WebGL

`WebGL` is a cross-platform, modern web standard for high performance interactive 2D and 3D graphics, through low-level graphics API based on OpenGL ES standard, exposed to ECMAScript (JavaScript) via the `HTML5 Canvas` element. WebGL brings plugin-free 3D to the web, implemented right into the browser. Major browser vendors Apple (Safari), Google (Chrome), Microsoft (Edge), and Mozilla (Firefox) are members of the WebGL Working Group.

This conformance makes it possible for the API to take advantage of hardware graphics acceleration provided by the user's device, i.e. it can take advantage of `GPU rendering` for fast, efficient, parallel drawing without putting much load on the CPU.

### What is GPU

GPU or `Graphics Processing Unit` in a computer, on a high level, is a processing chip consisting of a collection of many, low power computing units specially designed to efficiently handle graphics computation and drawing. But they can also be used for parallel, small computations. GPUs generally have many cores, in the orders of hundreds.

CPU or `Central Processing Unit` on the other hand is much more sophisticated, general purpose processing unit. Typical CPUs have a few (2-12) very powerful cores.

It's also important to understand that a _"graphics card"_ is not a GPU. GPU is just a chip, which is sometimes directly embedded in the motherboard (e.g. laptops) or SoC (System on a Chip, for mobiles), or can be put on a ditachable graphics card. A graphics card, along with a GPU, also has dedicated memory, connecting ports, cooling systems etc.

GPUs are specially disegned to handle graphical processings, which may include maths, game physics, display rendering etc. But they can also do smaller standard tasks like general computations. Since GPUs have many cores, they genrally can handle many threads concurrently, and can handle many tasks in parallel.

So, in brief

> `HTML5 Canvas` can use `WebGL` APIs, which can use `GPU rendering`

A `canvas` can be coded directly using the Canvas APIs or using WebGL APIs. Generally, programming the WebGL APIs is more complex than the standard Canvas APIs ([example](https://developers.google.com/web/updates/2019/08/get-started-with-gpu-compute-on-the-web)), but they bring in bunch of additional features, the huge performance boost of GPU rendering being the most useful. But, there are standard libraries available in the market to simplify this tasks, for example the [three.js](https://github.com/mrdoob/three.js/) library. Some useful discussions [here](https://stackoverflow.com/questions/21603350/is-there-any-reason-for-using-webgl-instead-of-2d-canvas-for-2d-games-apps).

> In some cases, the GPU acceleration is automatic; no special coding is required at all. For example, 3D CSS and Canvas 2D both run whether or not the PC has a GPU. If a GPU is found, the work gets passed to the GPU instead of the CPU. GPUs can render CSS transforms to create displays like pop-overs and animations, Canvas rendering of fonts, images, and vector graphics, and compositing, or blending layers, scrolling, and zooming. [source](https://smartbear.com/blog/develop/making-the-most-of-gpu-acceleration-in-your-web-ap/)

## Workers

The `Worker` is the main interface of the `Web Worker API`.

### Web Worker

The `Web Worker` is a common web standard for background processing in `JavaScript`. Basically what it enables is, running a script (a something.js file), which ideally does a heavy computational work, on a separate thread so that the `main UI thread` (one per browser tab) can stay free and responsive, so that users do not feel any lag while using the page.

Web Workers can run any script, but it has few restrictions

1. It cannot access the `DOM` directly
2. It can run `XMLHTTPRequest` aka `ajax` requests, but the `Response` remains empty!
3. Not all `Window` functions are available. [See details](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Functions_and_classes_available_to_workers)
4. A worker can spawn new therads in terms of new `Worker` instance, but not all browsers can handle that (e.g. [Safari](https://bugs.webkit.org/show_bug.cgi?id=22723))
5. Some of the older browsers can only take `string` or primitives as `message`. Most of the modern browsers can take a signle `object` as message though

To use a `Worker`, a script needs to create an instance giving another script path in same domain. Then the other script runs in a background thread, and can send data to main thread using `postMessage()` API, and vice versa. The main thread can listen to the messages using `onmessage()` API on the instance.

See docs [here](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API).

### What are Service Workers then?

They are pretty different. The are mainly used to provide `offline app experience`.

They mainly work based on network availability and cached data and browser local storages like [IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API) etc. The `Service Worker` is generally a single instance per browser, which can serve any number of tabs. Some good discussions [here](https://stackoverflow.com/questions/38632723/what-can-service-workers-do-that-web-workers-cannot).

> [ServiceWorkers](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API) essentially act as proxy servers that sit between web applications, and the browser and network (when available). They are intended to (amongst other things) enable the creation of effective offline experiences, intercepting network requests and taking appropriate action based on whether the network is available and updated assets reside on the server. They will also allow access to push notifications and background sync APIs.

## OffscreenCanvas

> You can render your graphics off the main thread with `OffscreenCanvas`, using a `HTMLCanvasElement` and `Web Worker`!

When doing heavy processing for Canvas e.g. for rendering high quality games etc. it might take a performance hit on the app, and may make the UI thread unresponsive.

The OffscreenCanvas API solves that problem by providing a native way to offload heavy canvas operations to backgrounds threads (through Worker, which needs to be configured through code). This can make the UI thread free, while the canvas is rendered in background, without using the DOM directly. Then it has a bunch of helper methods to transfer the generated graphics to the main canvas on page, or to mirror the OffscreenCanvas directly to the visible canvas. Docs [here](https://developer.mozilla.org/en-US/docs/Web/API/OffscreenCanvas).

**NOTE:** The `OffscreenCanvas` is not well supported in all browsers yet (June 2020).

## Web Components

Web Component in itself is not a technology, but a set of DOM APIs to enable developers to build `custom HTML tags` something like `<my-weather-widget></my-weather-widget>` which can work on any `JavaScript` based application, and interact with any library/framework just the way a standard native HTML tag woud do.

The main pieces that are generally put together to build `Web Component` are

1. `customElements` - to define a custom `HTML` tag with a backing `ES6` class, to support desired functionality
2. `shadow DOM` - a special area within the DOM to encapsulate and insulate the main page from the custom HTML internals, so that the `CSS` and `JS` of the element does not affect the outer page & vice-versa
3. `template` element - a new HTML which can hold any `HTML` along with `CSS` and `JS`, but does not actually render on the page. This is sometimes used to hold the `HTML` for the `customElement`, but it's not really required for building `Web Component` as such!

As these are native APIs supported by most modern browsers, and `polyfills` being available for others, they can be built with plain vanilla `JavaScript`. There are few good libraries as well, to write more functional maintainable code like `LitElements`, `Angular Elements` etc. Check [this](https://github.com/chakrabar/webcomponents) repo for some simple examples.

---

## All References

### Virtualization, OffScreenCanvas etc.

* https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API
* https://developers.google.com/web/updates/2018/08/offscreen-canvas
* https://medium.com/ingeniouslysimple/building-a-virtualized-list-from-scratch-9225e8bec120
* https://dev.to/nishanbajracharya/what-i-learned-from-building-my-own-virtualized-list-library-for-react-45ik

### WebGL & GPU

* https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API
* https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API/Tutorial/Getting_started_with_WebGL
* https://developers.google.com/web/updates/2019/08/get-started-with-gpu-compute-on-the-web
* https://smartbear.com/blog/develop/making-the-most-of-gpu-acceleration-in-your-web-ap/
* https://github.com/mrdoob/three.js/


### HTML Template strings & related APIs

* https://stackoverflow.com/questions/494143/creating-a-new-dom-element-from-an-html-string-using-built-in-dom-methods-or-pro
* https://stackoverflow.com/questions/3103962/converting-html-string-into-dom-elements
* https://developer.mozilla.org/en-US/docs/Web/HTML/Element/template
* https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentHTML
* https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentElement
* https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentText
* https://developer.mozilla.org/en-US/docs/Web/API/DOMParser
* https://javascript.info/template-element
* https://www.html5rocks.com/en/tutorials/webcomponents/template/
* https://wesbos.com/template-strings-html

### WebComponents

* https://css-tricks.com/an-introduction-to-web-components/
* https://developers.google.com/web/fundamentals/web-components
* https://custom-elements-everywhere.com/
* https://lit-element.polymer-project.org/guide/start
* https://html.spec.whatwg.org/multipage/custom-elements.html#custom-elements
* https://angular.io/guide/elements
* https://github.com/webcomponents/webcomponentsjs/tree/v1#browser-support
* https://www.webcomponents.org/community/articles/web-components-best-practices
* https://stackoverflow.com/questions/53244494/correct-way-to-apply-global-styles-into-shadow-dom, https://stackoverflow.com/questions/35694328/how-to-use-global-css-styles-in-shadow-dom
* https://stackoverflow.com/questions/55126694/how-to-create-litelement-without-shadow-dom
* https://lit-element.polymer-project.org/guide/styles#configurable
* https://dev.to/thepassle/web-components-from-zero-to-hero-4n4m
* https://medium.com/codingthesmartway-com-blog/angular-elements-a-practical-introduction-to-web-components-with-angular-6-52c0b3076c2c

### JavaScript, functional programming etc.

* [What the heck is the event loop anyway? | Philip Roberts](https://www.youtube.com/watch?v=8aGhZQkoFbQ)
* [Franziska Hinkelmann: JavaScript engines - how do they even?](https://www.youtube.com/watch?v=p-iiEDtpy6I)
* [Anjana Vakil: Immutable data structures for functional JS](https://www.youtube.com/watch?v=Wo0qiGPSV-s)
* [An introduction to functional programming](https://codewords.recurse.com/issues/one/an-introduction-to-functional-programming)
* [immutable-js](https://github.com/immutable-js/immutable-js)
* [Web Perf - Google dev](https://developers.google.com/web/fundamentals/codelabs/web-perf)
* [Build with jake](https://jakejs.com/docs-page.html#item-overview-jakefiles)
* [Accessibility for all](https://web.dev/accessible/), [Accessibility - web fundamentals](https://developers.google.com/web/fundamentals/accessibility)
* [ARIA](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA) vs [Semantic HTML](https://developer.mozilla.org/en-US/docs/Learn/Accessibility/HTML)
* [Disability & Assistive Technologies - W3C](https://www.w3.org/WAI/people-use-web/)

### Flux, Redux, RxJS, Immutable...Oh my!

* https://github.com/facebook/flux/tree/master/examples/flux-concepts
* https://facebook.github.io/flux/docs/in-depth-overview/
* https://redux.js.org/introduction/prior-art#immutable
* https://immutable-js.github.io/immutable-js/
* https://rxjs-dev.firebaseapp.com/guide/overview
* https://dzone.com/articles/building-redux-like-apps-using-rxjs
* https://martinfowler.com/eaaDev/EventSourcing.html
* https://cycle.js.org/
* https://github.com/reduxjs/redux-devtools
* https://github.com/jas-chen/rx-redux
