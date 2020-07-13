---
layout: post
title: "The great new Windows 10 Terminal"
excerpt: "The new Terminal for Windows 10 is now available for download (preview) in the Microsoft store. It's sooo cool"
date: 2019-06-27
tags: [dev, programming, windows, wsl, terminal, commanline, cmd]
categories: articles
comments: true
share: true
published: true
modified: 2020-07-13T17:15:00+05:30
---

## The new _Windows Terminal_

Now the preview version for Windows 10 terminal is available to [download](https://www.microsoft.com/en-us/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab) from Microsoft store. You could also build it from the source code on [GitHub](https://github.com/microsoft/terminal).

So, what's this? And why would you care?

This is like a whole new version of the classic `Command Prompt` command-line terminal on Windows, which is super cool, open source & comes with bunch of really helpful productivity features. If you are a developer, love/need to use multiple command line applications (e.g. `cmd`, `PowerShell`, `Git`, `Bash` etc.) then this is for you. You can install this new terminal, and configure to suit your taste, and this can <u>host all you command terminals in one place</u>, separated by tabs, their individual settings, themes etc. You can even have background images, including ***animated GIFs !!***

Some of the main features:

1. Multiple **tab** support
2. Accelerated text rendering
3. Modern text support including `unicode`, `emojis`, `ligature fonts`
4. Configurable **themes**, colors, background image - including _animated GIF_
5. Fully **configurable terminals** - use any terminal / command-line app
6. Editable (& sharable) `JSON` based settings file
7. Configurable commands & key shortcuts (through settings)
8. Configurable copy-paste (I still have trouble using it seamlessly across terminals though)
9. And many more
10. Also it's open source & in full development, so it'll get better

![Image](/images/posts/misc/terminal.png)

#### Settings hints to help configure your own terminal

**Note:** See update later for the latest settings. The profile settings has been updated since `v0.11`.
{: .notice--warning}

So when you [download](https://www.microsoft.com/en-us/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab) and install it from the Microsoft Store, it's all ready for use. But to use it to it's potential and you taste, you need to make some adjustments.

I guess in future they'll provide some UI for the settings, but currently it's a bare bone `JSON` file. To make any changes, you'll have to update the JSON file manually. For that, click on the drodown button on the top-right corner and select `Settings`. It'll open the `profile.json` file that hosts all the settings. It's just a text file, so you can edit it with any text editor and save it. That's it! It'll immediately update the terminal with the changes.

**Note:** You need Windows 10 version 18362.0 or higher for the new Windows Terminal to work. If you don't have that, update your Windows 10 OS first.
{: .notice--info}

The settings file mainly have four sections

1. The global **app level settings** - how the terminal window behaves, for example how it shows tabs
2. The **key bindings** - key shortcut for common tasks like open a specific profile on a new tab
3. The **profiles** - settings for a new tab including command to run, display settings, color settings etc.
4. The **color schemes** - color settings or themes for profiles are set through `schemes`. 

For the different commands / tools (e.g. `PowerShell`, `Bash` etc.) you want to run through the terminal, you need to set them up in **profiles**. Each profile can have it's own command to run. You can also set a bunch of other settings in profile like what should be the starting directory, font face, font size, background color or image, opacity, icon, cursor style etc.

Each profile should have it's own `Guid` that they are identified with. If needed, you can generate your new Guids online. The `defaultProfile` at the root level specifies that profile to run on a new tab by default, specified by that profile Guid.

Now for the theme, you can use one of those default color schemes, add your own or can update an existing one. All the color schemes are basically palettes for set of ANSI default colors. To create your own style, just update the color values with #hex codes for your favorite colors. Then add the scheme to a profile using the `name` of the scheme.

Following is a part of my own settings which might help you understand it better. See the settings documentation [here](https://github.com/microsoft/terminal/blob/master/doc/cascadia/SettingsSchema.md) for more details.

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
        "requestedTheme" : "dark", // theme for title bar etc.
        "showTabsInTitlebar" : false,
        "showTerminalTitleInTitlebar" : false
    },
    "profiles" : 
    [
        {
            "acrylicOpacity" : 0.85,
            "backgroundImage" : "ms-appdata:///roaming/pubgHD.jpg", // background image
            "backgroundImageOpacity" : 0.94999998807907104,
            "backgroundImageStretchMode" : "uniformToFill",
            "closeOnExit" : true,
            "colorScheme" : "ArghyaMatte", // custom color scheme
            "commandline" : "powershell.exe", // program or command to run
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
            "background" : "#012456", // background color
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
        { // a custom color scheme
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
            "name" : "ArghyaMatte", // the scheme name
            "purple" : "#75507B",
            "red" : "#F90078",
            "white" : "#21EF4A",
            "yellow" : "#C4A000"
        },
        // defaults...
    ]
}
```

The preview version is not super stable yet, I've faced few crashes myself. But, it's just an early preview, so I'm sure it'll get stable in some time and hopefully the team will add lot more handy features in the future.

# Update for settings, v0.11

The settings got a bunch of updates including some breaking changes in [version 0.11](https://github.com/microsoft/terminal/issues/5458). Now the `profile.json` settings file is called `settings.json`, usually found in `AppData\Local\Packages\Microsoft.WindowsTerminal_key\LocalState`. Check the following links for more details

* [Windows Terminal Preview v0.11 Release](https://devblogs.microsoft.com/commandline/windows-terminal-preview-v0-11-release/)
* [Windows Terminal Status Updates](https://github.com/microsoft/terminal/issues/3600)

If you want to apply the same settings as above, your new `settings.json` will look like

```json
// This file was initially generated by Windows Terminal 1.0.1811.0
// To view the default settings, hold "alt" while clicking on the "Settings" button.
// For documentation on these settings, see: https://aka.ms/terminal-documentation
{
    "$schema": "https://aka.ms/terminal-profiles-schema",
    "defaultProfile": "{61c54bbd-c2c6-5271-96e7-009a87ff44bf}",
    "copyOnSelect": false,
    "copyFormatting": false,

    // Apply custom settings
    // To learn more about profiles, visit https://aka.ms/terminal-profile-settings
	"theme" : "dark", // CHANGED from requestedTheme
	"showTabsInTitlebar" : true,
	"showTerminalTitleInTitlebar" : true,
	"wordDelimiters" : " ./\\()\"'-:,.;<>~!@#$%^&*|+=[]{}~?\u2502",
	"alwaysShowTabs" : false,
	"initialCols" : 120,
	"initialRows" : 30,
    "profiles":
    {
        "defaults":
        {
            // Put settings here that you want to apply to all profiles.
        },
        "list":
        [
            {
				"acrylicOpacity" : 0.85,
				"backgroundImage" : "ms-appdata:///roaming/pubgHD.jpg", // background image
				"backgroundImageOpacity" : 0.94999998807907104,
				"backgroundImageStretchMode" : "uniformToFill",
				"closeOnExit" : true,
				"colorScheme" : "ArghyaMatte", // custom color scheme
				"commandline" : "powershell.exe", // program or command to run
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
				"background" : "#012456", // background color
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
        ]
    },
    // Add custom color schemes to this array.
    // To learn more about color schemes, visit https://aka.ms/terminal-color-schemes
    "schemes":
	[
		{ // a custom color scheme
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
            "name" : "ArghyaMatte", // the scheme name
            "purple" : "#75507B",
            "red" : "#F90078",
            "white" : "#21EF4A",
            "yellow" : "#C4A000"
        },
        // other schemes...
	],

    // Add custom keybindings to this array.
    // To learn more about keybindings, visit https://aka.ms/terminal-keybindings
    "keybindings":
    [
        { "command": {"action": "copy", "singleLine": false }, "keys": "ctrl+c" },
        { "command": "paste", "keys": "ctrl+v" },
        { "command": "find", "keys": "ctrl+shift+f" },
        { "command": { "action": "splitPane", "split": "auto", "splitMode": "duplicate" }, "keys": "alt+shift+d" },
		// Open a new tab with default profile
        { "command": { "action": "newTab", "index": 0 }, "keys": "ctrl+n" }
		// See default key bindings at https://github.com/microsoft/terminal/issues/3600
    ]
}
```

### Some useful links

* [Download](https://www.microsoft.com/en-us/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab)
* [Source code](https://github.com/microsoft/terminal)
* [Settings docs](https://github.com/microsoft/terminal/blob/master/doc/cascadia/SettingsSchema.md)
* [Scott Hanselman's tips](https://www.hanselman.com/blog/YouCanNowDownloadTheNewOpenSourceWindowsTerminal.aspx)
* [Fluent terminal](https://github.com/felixse/FluentTerminal)
* [Powerline setup as GIT terminal - docs](https://docs.microsoft.com/en-us/windows/terminal/tutorials/powerline-setup)
* [More customization - Hanselman](https://www.hanselman.com/blog/HowToMakeAPrettyPromptInWindowsTerminalWithPowerlineNerdFontsCascadiaCodeWSLAndOhmyposh.aspx)
