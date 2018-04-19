---
layout: post
title: "Getting started with building desktop apps with Electron"
excerpt: "Desktop apps with HTML, CSS & JavaScript"
date: 2018-04-19
tags: [tech, apps, desktop, electron, html, css, javascript]
categories: articles
comments: true
share: true
published: false
---

# Quick facts

* `Electron` was created by `GitHub` and open sourced (initially called _"Atom Shell"_)
* Uses `HTML`, `CSS` & `JavaScript` for building cross-platform native desktop apps

* Some popular apps built on Electron - VS Code, Atom editor, GitHub desktop, slack, postman client etc.

# How does it work

* Uses `Chromium` for rendering displays
* Uses `Node.js` for JavaScript backend

# Capabilities of Electron

* Electron has access to the OS native APIs (`Windows`, `Linux` & `macOS`)
* Electron can acces stuffs like - filesystem, media, recent documents, tool bars, menus, notifications, app windows, power management, network, accessible processes etc.
* Can access all modules of `Node.js`

# Installation

* Install node from [the official site](https://nodejs.org/en/download/)
* Create a new directory `C:\Arghya\Repos\MyDesktopApp`
* Initiate `package.json` with `npm init -y`
* Get all dependencies with `npm install electron -D` (previously `electron-prebuilt`) - it'll download a pre-built electron for the current system (e.g. Windows x64), with Node & chromium
* Set the entry point in `package.json` as `"main": "src/main.js"`
* Add a npm script to specify running electron in current project directory, with `"start": "electron ."`

```javascript
{
  "name": "MyDesktopApp",
  "version": "1.0.0",
  "description": "",
  "main": "src/main.js", //entry point
  "scripts": {
    "start": "electron ." //run electron, in current directory
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

**Tip:** While running the electron app, press `ctrl + shift + i` (or `F12`) to open the chromium debugger console!
{: .notice--info}

# More

* Electron has multiple processes running all the time. SOme of the main ones are
  * Main - always one _"Main"_ process. Loads first and controlls other stuffs, and can spawn new process.
  * Renderer - creates the display, There can be many of them. Can also create other _"Renderer"_ process
  * Modules available for each process in listed [here](https://github.com/electron/electron/tree/master/docs)
  * Main & Renderer processess communicate to each other via `IPC` or inter-process-communication
  * Each process has it's own IPC module and they interact through a `pub-sub` model. For example, if a Renderer detects a click event, it reports to Main. Main in tern broadcasts the event to all it's child renderer processes, and whoever has subscribed to that event can raise an event-handler