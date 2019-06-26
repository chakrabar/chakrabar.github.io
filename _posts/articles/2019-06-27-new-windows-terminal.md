---
layout: post
title: "The great new Windows Terminal"
excerpt: "The new Windows Terminal is now available for download (preview) in the Microsoft store. It's sooo cool"
date: 2019-06-27
tags: [dev, programming, windows, wsl, terminal, commanline, cmd]
categories: articles
comments: true
share: true
published: true
---

# The new _Windows Terminal_

***This is an incomplete post. Please come back later for updates.***

Now the preview version is available to download from Microsoft store. You can also build from the source code from GitHub.

This is like a whole new version of the classic `cmd` commandline terminal, which is super cool, open source & comes with bunch of really helpful productivity features.

Some of the main features:

1. Multiple **tab** support
2. Accelerated text rendering
3. Modern text support including `unicode`, `emojis`, `ligature fonts`
4. Configurable **themes**, colors, background image - including _animated GIF_
5. Fully **configurable terminals** - use any terminal / commandline app through settings
6. Editable (& sharable) `JSON` based settings file
7. Configurable commands & key shortcuts (throgh settings)
8. Copy-paste (I still have trouble using it seamlessly across terminals though)
9. And many more
10. Also it's open source & in full development, so it'll get better

![Image](/images/posts/misc/ac_win10_terminal_2.png)

### Settings hint that might help configure your own terminal

```js
{
    "globals" : 
    {
        // defaults...
        "keybindings" : 
        [
            // defaults...
            {
                "command" : "newTabProfile0",
                "keys" : 
                [
                    "ctrl+n" // open new default tab & go to tab
                ]
            },
            // defaults...
        ],
        "requestedTheme" : "light", // theme for title bar etc.
        "showTabsInTitlebar" : false,
        "showTerminalTitleInTitlebar" : false
    },
    "profiles" : 
    [
        {
            "acrylicOpacity" : 0.85,
            "backgroundImage" : "ms-appdata:///roaming/pubg42.jpg", // background image
            "backgroundImageOpacity" : 0.94999998807907104,
            "backgroundImageStretchMode" : "uniformToFill",
            "closeOnExit" : true,
            "colorScheme" : "Arghya", // custom color scheme
            "commandline" : "powershell.exe", // program command to run
            "cursorColor" : "#14EA09",
            "cursorHeight" : 25,
            "cursorShape" : "vintage",
            "fontFace" : "Fira Code", // custom font (with ligature)
            "fontSize" : 10,
            "guid" : "{61c54bbd-c2c6-5271-96e7-009a87ff44bf}",
            "historySize" : 9001,
            "icon" : "ms-appx:///ProfileIcons/{61c54bbd-c2c6-5271-96e7-009a87ff44bf}.png",
            "name" : "Windows PowerShell",
            "padding" : "10, 5, 10, 0", // cutom padding
            "snapOnInput" : true,
            "startingDirectory" : "C:\\Codebase", // directory to open by default
            "useAcrylic" : false
        },
        {
            "acrylicOpacity" : 0.7,
            "closeOnExit" : true,
            "colorScheme" : "One Half Dark", // system color scheme
            "commandline" : "cmd.exe",
            "cursorColor" : "#14EA09", // green cursor
            "cursorHeight" : 25,
            "cursorShape" : "vintage", // classic cursor
            "fontFace" : "Consolas",
            "fontSize" : 9,
            "guid" : "{0caa0dad-35be-5f56-a8ff-afceeeaa6101}",
            "historySize" : 9001,
            "icon" : "ms-appx:///ProfileIcons/{0caa0dad-35be-5f56-a8ff-afceeeaa6101}.png",
            "name" : "Command Prompt",
            "padding" : "0, 0, 0, 0",
            "snapOnInput" : true,
            "startingDirectory" : "%USERPROFILE%",
            "useAcrylic" : true
        },
        // Other profiles...
    ],
    "schemes" : 
    [
        {
            "background" : "#262C6B", // background color
            "black" : "#0A2D75",
            "blue" : "#0965E5",
            "brightBlack" : "#555753",
            "brightBlue" : "#116CD6",
            "brightCyan" : "#34E2E2",
            "brightGreen" : "#1AF40E",
            "brightPurple" : "#AD7FA8",
            "brightRed" : "#DD3E1F",
            "brightWhite" : "#DBFDFF",
            "brightYellow" : "#F9F90E",
            "cyan" : "#06989A",
            "foreground" : "#ADCFFF", // default font color
            "green" : "#15A531",
            "name" : "Arghya", // color scheme name
            "purple" : "#75507B",
            "red" : "#F90078",
            "white" : "#21EF4A",
            "yellow" : "#C4A000"
        },
        // defaults...
    ]
}
```

It is not super stable yet, I've faced few crashes myself. But, it's just an early preview, so I'm sure it'll get stable in some time and hopefully the team will add more handy features in the future.

### Some useful links

* [Download](https://www.microsoft.com/en-us/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab)
* [Source code](https://github.com/microsoft/terminal)
* [Settings docs](https://github.com/microsoft/terminal/blob/master/doc/cascadia/SettingsSchema.md)
* [Hanselman's tips](https://www.hanselman.com/blog/YouCanNowDownloadTheNewOpenSourceWindowsTerminal.aspx)