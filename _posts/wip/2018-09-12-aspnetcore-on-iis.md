---
layout: post
title: "Hosting an ASP.NET Core web application on IIS"
excerpt: "A quick guide to deploying your new ASP.NET Core app on existing IIS web server"
date: 2018-09-12
tags: [aspnet, aspnetcore, web, iis, deployment]
categories: articles
comments: true
share: true
published: false
---

https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/iis/?view=aspnetcore-2.1&tabs=aspnetcore2x
https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/aspnet-core-module?view=aspnetcore-2.1#log-creation-and-redirection

0. Publish as FDD, Select "Portable" https://docs.microsoft.com/en-us/dotnet/core/deploying/#framework-dependent-deployments-fdd

1. Enable/Install IIS https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/iis/?view=aspnetcore-2.1&tabs=aspnetcore2x#iis-configuration

2. Go to the download [Microsoft Visual C++ latest Redistributable](https://support.microsoft.com/en-in/help/2977003/the-latest-supported-visual-c-downloads) page
Download the latest version (`x64` or `x86`)

3. Go to [.NET Downloads](https://www.microsoft.com/net/download) page
Click Download .NET Core Runtime, under `.NET Core` menu
From the page notes

The .NET Core Runtime & Hosting Bundle contains everything you need to run existing .NET Core apps, including hosting ASP.NET Core apps. The bundle includes the .NET Core runtime, the ASP.NET Core runtime, and if installed on a machine with IIS it will also add the [ASP.NET Core IIS Module](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/aspnet-core-module?view=aspnetcore-2.1).
{: .notice--info}

4. Install the Microsoft Visual C++ latest Redistributable package, double click on installer or run from command prompt

5. Install Microsoft .NET Core Windows Server Hosting package. To avoid installing the x86 components, run the installer from command prompt with `OPT_NO_X86` switch

```
C:\Dir_with_installer> dotnet-hosting-2.1.4-win.exe OPT_NO_X86=1
```

6. Restart the system or restart IIS with following commands in admin mode - `net stop was /y` followed by `net start w3svc`

```bash
c:\SomeDirectory>net stop was /y
The following services are dependent on the Windows Process Activation Service service.
Stopping the Windows Process Activation Service service will also stop these services.

   World Wide Web Publishing Service

The World Wide Web Publishing Service service is stopping.
The World Wide Web Publishing Service service was stopped successfully.

The Windows Process Activation Service service is stopping.
The Windows Process Activation Service service was stopped successfully.

c:\SomeDirectory>net start w3svc
The World Wide Web Publishing Service service is starting.
The World Wide Web Publishing Service service was started successfully.
```

7. Open IIS. Search for IIS or run `inetmgr` from run command (Windows + R). Remember you need to be an admin on the system to access IIS

8. 

9. Create a `logs` folder to host ASP.NET Core Module stdout logs when stdout logging is enabled

10. Under the server node on the left, right-click on `Sites` and select `Add Website`. Give the site a name (e.g. mycoreweb.com), use the same name for `host` and select the folder where the app is published.

To make it available over LAN, add another binding for your website with the IP address (e.g. check [this](https://stackoverflow.com/questions/21896534/accessing-a-local-website-from-another-computer-inside-the-local-network-in-iis) post)

**Note**: To run the app locally with a custom domain name (e.g. `mycoreweb.com` in our example), we need to add it to the Windows hosts file so that browsers can find it. Basically it's like adding this domain to local IP address in local DNS. For that, open any text editor in admin mode, open the file C:\Windows\System32\driver\etc\hosts, and add the custom domain name with local IP address i.e. `127.0.0.1 mycoreweb.com` in a new line, and save.
{: .notice--info}

```bash
Local:
Type: http
Ip Address: All Unassigned
Port: 80
Host name: mycoreweb.com

LAN:
Type: http
Ip Address: <Network address of the hosting machine ex. 192.168.0.10>
Port: 8808 <make sure this port is allowed to have incoming traffic>
Host name: <Leave it blank>
```

To make sure firewall allows inbound requests for your configured port. Check [this](https://stackoverflow.com/questions/12008606/connecting-to-windows-localhost-iis-from-another-computer) and [this]() Stack Overflow posts for help.

11. `<aspNetCore processPath="dotnet" arguments=".\EntryPointProject.dll" stdoutLogEnabled="true" stdoutLogFile=".\logs\stdout" />`

12. In the Application Pool, set Managed pipeline mode: "Integrated", .NET CLR version: "No Managed Code" ~gyan, core runs as a process outside IIS

13. If you get **HTTP Error 502.5 - Process Failure** that basically means IIS could not find .NET Core. The options are

  1. Add the .NET Core path to environment PATH (e.g. C:\ProgramFiles\dotnet)
  2. Mention the full path to `dotnet.exe` in (for FDD) or `.\My.Web.Project.exe` (for SCD) `aspNetCore processPath` of `web.config`
https://stackoverflow.com/questions/41590493/iis-fails-to-run-asp-net-core-site-http-error-502-5/41590600
https://stackoverflow.com/questions/38624453/asp-net-core-1-0-on-iis-error-502-5

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <handlers>
      <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModule" resourceType="Unspecified" />
    </handlers>
    <aspNetCore processPath="C:\Program Files\dotnet\dotnet.exe" arguments=".\My.Web.Project.dll" stdoutLogEnabled="true" stdoutLogFile=".\logs\stdout" />
  </system.webServer>
</configuration>
```

14.
Error:
  An assembly specified in the application dependencies manifest (My.Web.Project.deps.json) was not found:
    package: 'Microsoft.ApplicationInsights.AspNetCore', version: '2.1.1'
    path: 'lib/netstandard1.6/Microsoft.ApplicationInsights.AspNetCore.dll'
  This assembly was expected to be in the local runtime store as the application was published using the following target manifest files:
    aspnetcore-store-2.0.0-linux-x64.xml;aspnetcore-store-2.0.0-osx-x64.xml;aspnetcore-store-2.0.0-win7-x64.xml;aspnetcore-store-2.0.0-win7-x86.xml
	
https://stackoverflow.com/questions/46491957/asp-net-core-2-missing-applicationinsights
https://github.com/dotnet/coreclr/issues/13542

  1. Install .NET Core full [SDK](https://www.microsoft.com/net/download/archives) (not the runtime) on the server
  2. Add this flag to your main `csproj` <PublishWithAspNetCoreTargetManifest>false</PublishWithAspNetCoreTargetManifest>
  3. If it is a framework dependent deployment, publish your app with specific runtime ID (e.g. win-x64 or linux-x64)
