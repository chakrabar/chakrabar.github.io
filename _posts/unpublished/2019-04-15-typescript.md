---
layout: post
title: "Basics of TypeScript"
excerpt: "For internal purposes only"
date: 2019-04-15
tags: [coding, programming, wf, wwf, workflow]
categories: articles
comments: true
share: true
published: false
---

## `TypeScript`

Source of information is [Pluralsight course](https://app.pluralsight.com/library/courses/typescript-in-depth/table-of-contents).

What is `TypeScript`

* A programming language, a superset of `JavaScript`
* It compiles to plain `JavaScript`, that can run on modern browsers (client-side) or in `node` (server-side)
* Provides type-safety and higher level programming constructs over `JavaScript`
* Note that any `JavaScript` is already valid `TypeScript`
* Since `TypeScript` has type safety and a compiler that turns it into `JavaScript`, it can detect errors and problems at the compiling stage
* Compiled plain ol' `JavScript` can be `ES6` the most modern version of `JavaScript` right now, or `ES5` which is most widely supported on browsers, or even down to `ES3` standards!
* Cross-platform and open source (on [GitHub](https://github.com/Microsoft/TypeScript))

### Installation & Setup

* Download and install [nodejs](https://nodejs.org/en/)
* Make sure `npm` is installed and working (try commands on a terminal like `npm -v` or `npm version`)
* Easiest way to install TypeScript is with npm `npm install -g typescript`, see details [here](https://www.typescriptlang.org/index.html#download-links). Check with `tsc -v`

Note: When installed globally through npm `npm install -g`, on Windows it gets installed in `C:\Users\<user>\AppData\Roaming\npm\node_modules\typescript`

TypeScript supports many IDEs, or rather TypeScript is very well supported by many IDEs, including but not limited to: `Visual Studio`, `Visual Studio Code`, `Sublime`, `WebStorm` etc.

### VS Code

VS Code gives excellent code completion, compilation, debugging and other supports for `TypeScript`.

* Create a file `tsconfig.json` in the root directory which will be detected by the VS Code tasks (The presence of a tsconfig.json file in a directory indicates that the directory is the root of a TypeScript project - from [docs](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html))
* Now when you press `Ctrl + Shift + B`, it can run the compiler one time to generate the `.js` files form `.ts` files OR can run compiler in ***watch*** mode, which will compile the code every time a code is saved. See details [here](https://code.visualstudio.com/docs/editor/tasks#vscode)
* Open command palette by pressing `Ctrl + Shift + P`
* Type and search for `task`
* Select **`Run Build Task`** (shortcut `Ctrl + Shift + B`)
* Configure default build task (it'll create a `tasks.json` file inside `.vscode` directory) and select `tsc build - tsconfig.json` 

#### `.vscode\tasks.json`

```js
{
    "version": "2.0.0",
    "tasks": [
        {
            "type": "typescript",
            "tsconfig": "tsconfig.json",
            "problemMatcher": [
                "$tsc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ]
}
```

#### `tsconfig.json`

```js
{
    "compilerOptions": {
        "module": "es6", //default: commonjs for es5
        "noImplicitAny": true,
        "removeComments": true,
        "preserveConstEnums": true,
        "sourceMap": true,
        "target": "es5",
        "watch": true,
        "outDir": "js"
    },
    "files": [
        "app.ts"
    ]
}
```

### Debug configuration

In VS Code

* Go to `Debug` left menu
* Click on configure (small gear icon)
* Select environment as `Node.js`
* It'll add another new file `.vscode\launch.json`
* Now you can click the green arrow or 'Play' sign to start debugging

#### `.vscode\launch.json`

```js
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "Launch",
            "program": "${workspaceFolder}/app.ts", // ${file}
            "outFiles": [
                "${workspaceFolder}/**/*.js"
            ],
            "sourceMaps": true
        },
        { // another debug configuration
            "type": "node",
            "request": "attach",
            "name": "Attach",
            "port": 9229
        }
    ]
}
```

### Basic TypeScript

* Variable declaration with `var, let, const`
* Types `Boolean, Number, String, Array, Enum, Any, Void`
* Type interface - we can define type of variables. If not defined, the type is inferred and the variable stick to it

TypeScript addition: `Tuple`

### Functions

* Can define parameter and return types
* Can define function types
* Supports arrow functions, optional parameters, default values, rest parameters (variable number of parameters that are collected as array) from `ES6`
* Supports required parameters & function overloading

```js
// an arrow function
const checkLegalAge = (age: number) : string => age >= 18 ? 'Welcome' : 'Sorry, you are underage';

// function type (userAge: number) => string
const ageCheckFunc: (userAge: number) => string = checkLegalAge;

console.log(ageCheckFunc(15)); // Sorry, you are underage

// function types

const checkLegalAge = (age : number) : string => age >= 18 ? 'Welcome' : 'Sorry, you are underage';

const ageCheckFunc : (userAge : number) => string = checkLegalAge;

console.log(ageCheckFunc(15)); // Sorry, you are underage

// parameters (in TypeScript, all parameters are required by default)
// required, default (with default value, which is also treated as optional), and optional
// NOTE: a required parameter cannot follow an optional parameter
const getWelcomeMessage = (name : string, age : number = 18, country? : string) =>
    `Hello ${name}, ${age} years, from ${country ? country : 'your country'}`;

// all valid calls to the function
getWelcomeMessage('AC', 30, 'India');
getWelcomeMessage('AC');
getWelcomeMessage('AC', 30);
getWelcomeMessage('AC', null, 'India');
// getWelcomeMessage(); not allowed, name is required parameter

// REST PARAMETER : any number of parameters that'll be captured in an array
// can appear only after required parameters (if any)
const printFirstN = (n : number, ...users : string[]) : void => {
    if (n > 0 && users && users.length) {
        let count = 0;
        while (count < n && count < users.length) {
            console.log(users[count]);
            ++count;
        }
    }
}

printFirstN(2, 'Penny', 'Sheldon', 'Amy', 'Leonard'); // Penny Sheldon
```

### Interfaces

> Well, it's getting too long now. Obviously, I cannot write about a new programming language that I'm just learning, in a precise blog post. Rather go over to this [GitHub repo](https://github.com/chakrabar/Solutions/tree/master/src/LibraryManager) to see some real and simple code examples.