---
layout: post
title: "Azure basics"
excerpt: "WIP"
date: 2018-07-08
tags: [graphql, api, graph, webservice]
categories: articles
image:
  feature: posts/misc/og-image-lg.jpg
comments: true
share: true
published: false
---

# Azure for .NET developers

(From Scott Allen's Pluralsight course)

Platform support - ASP.NET, Java, Node, Rails, php, Python Django etc.

Azure platform & services

Cloud database & storage e.g. Azure SQL, Cosmos DB etc.
Automation - SDKs, build & CI/CD systems
Service hosting - App services & Azure functions (serverless architecture)
Virtual machines - servers on cloud, Windows, Linux, SQL Server, Containers etc.

Subscription: One account can have multiple subscriptions. Subscriptions can have separate billing e.g. for one client
Resource group: A logical grouping of resources e.g. for one project. Can show resources & billing collectively.

Create an Azure account @ https://azure.microsoft.com/en-us/
Access the Azure potal @ https://portal.azure.com

## Create a resource: VM

Name: ac-win-vm-1
VM Disk type: SSD (costlier than HDD, but faster)
Credentials: dev/Develop#1234
Subscription: My free trial
Resource group* : Create new: BasicStudy
Location: Central India
Already have a Windows license: No

Next: OK

Select VM configuration:
B1s: 1 VCPU, 2 disks, 800 Max IOPS, 4GB SSD, Charge: INR 885/Month

Also select
High Availability setup, storage type, virtual network, subnet, public ip (ac-win-vm-1-ip: 104.211.90.102), diagnostics, backup etc.

Run validations: Valid, price 1.1897 INR/hr

Create! and it'll be ready in minutes. See more details [here](https://docs.microsoft.com/en-gb/azure/virtual-machines/windows/quick-create-portal?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json#custommachine).

>> That one VM will appear as a bunch of resources like - VM, disks, network interface, IP address, virtual network etc.

#### Programming Azure

Azure works as a programmable data center!

Most of what is available on the web GUI are accessible via REST APIs. It also provides SDKs in many languages to create, update and tear down resources. See more details [here](https://docs.microsoft.com/en-gb/azure/#pivot=sdkstools).

There are Azure SDKs in .NET, Java, Go, Python, Node and REST APIs. There are Azure CLI and Azure PowerShell tools. Also extensions for VS Code, Docker etc.

Let's work with the Azure CLI 2.0 (cross-platform on Windows, macOS & Linux)

Install it from [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) or you can also use the Cloud Shell (online CLI) from within the Azure portal, or @ https://shell.azure.com/

Type `az` to see the available Azure commands
To login, type `az login` and follow instructions. Once logged in, it shows the logged in user's subscription details

```javascript
[
  {
    "cloudName": "AzureCloud",
    "id": "c1f3b99d-3daa-4e54-901e-381c66xxxxxb",
    "isDefault": true,
    "name": "Free Trial",
    "state": "Enabled",
    "tenantId": "54755850-e86a-45bd-a661-21f62dxxxxxc",
    "user": {
      "name": "mymail@live.com",
      "type": "user"
    }
  }
]
```

Let's do some more Azure management work with the CLI

```bash
# commands on Azure CLI (use -h for help on commands)
# set a subcription to be used for the commands
az account set -s subscription-name
# to activate interactive mode (with intellisense)
az interactive # press Ctrl+D to exit
# deallocate my VM (stopping wouldn't stop billing)
az vm deallocate -g BasicStudy -n ac-win-vm-1
# Some Azure storage will be still used, but relatively cheaper
# Now delete the whole resource group wil all peripherals
az group delete --name BasicStudy
```

#### Different ways of deploying & hosting

There are mainly 4 types of cloud hosting in general, based on - what we do, and what is done by someone else i.e. the cloud service

* **On-Prem** or On the premise: From hardware, to OS, the application all are done by customer
* **IaaS**: The infrastructure or mainly the hardware is done by cloud service (the server/virtualization, power, storage, network cables, physical housing etc.)
* **PaaS**: The infrastructure & the platform (the OS and app framework & runtime, like CLR or JVM) are taken care of by cloud service. Customer just builds & deploys the application and data
* **SaaS**: The complete software is provided by the cloud service provider e.g. Office 365, Dropbox, Gmail, Salesforce, Jira etc. and is (generally) accessible directly through a web browser. The main difference of SaaS and standard software, from a user's point-of-view is, using SaaS does not needs specific system (hardware + OS + runtime) to use them, any machine with a web browser & internet connection is enough.

In Azure world, the VMs are IaaS. Though it comes with pre-installed OS, patching, updating & managing that is on the user. This is good choice when many custom apps are required, or the plan it to host multiple mix of applications on the same server. You just RDP into the machine and work. One great advantage, for example, over on-prem is, VMs can be programmed to scale automatically as per need e.g. low computation during night, high computation during office hours.

Azure App Service is a typical example of PaaS. On top of the infrastructure, the OS and app runtime are maintained by Azure. It's super easy to deploy websites & services on them with simple IDE or VCS integration. Basic CI/CD is built-in and simple to use. App Services can also be scaled manually or automatically based on load. One can also deploy SQL Server, MySQL or other types of databases.

#### Azure App Service

The best solution for hosting web (we'll use ASP.NET, but other tools/technologies are fine too) is Azure PaaS for web, i.e. **Microsoft Azure App Services**. It is an Azure service for hosting apps. Among others, some of the main use cases of app services are

* Web applications
* Web services & APIs
* Mobile back-ends, with push notification support
* Logic apps for automated data collection etc.

To start using, first create a new App Service. Select from left menu or hub.

1. Click on Add
2. Select a template (e.g. Web App + SQL)
3. Provide a domain name (which forms part of public domain name) e.g. accoolsite
4. Select resource group, OS (Windows, Linux, Docker), App Service Plan etc.

An **App Service Plan** or _pricing tier_ ?? is like a configuration that determines server location, server configuration, computing power, price etc. They also list other features like custom domain, manual vs automatic scaling etc. There are different plan types & templates for dev/test, production & advanced (isolated systems). If multiple apps share the same plan, they share the same server too.

I create plan: ac-web-plan-1 (Basic: 1 Small).

**Note:** For ana analogy, the relationship between app service & app service plan is like a hosted web site & the hosting machine.
{: .notice--info}

Azure App Service can be **scaled up** or **scaled out** when needed. Apps in the same App Service Plan scale together. In scaling up, Azure seamlessly migrates to more powerful machine. In scaling out ??, it adds more VMs.

When the app service is ready, it'll show up in dashboard and have link to it's public Azure site e.g. https://accoolsite.azurewebsites.net

#### Web deployment on Azure

First, lets create a simple .NET Core MVC web app locally on VS. Use user based authentication, and in-app store for now.

Create an environment in Azure for deploying the app. We'll do it through CLI in the portal _Cloud Shell_.

The Azure hierarchy for App Service is like, Azure <- Resource Group <- App Service Plan <- App Service

```bash
az login #cloud shell is automatically logged in
az account set -s "Free Trial" #select the subscription
az account list-locations --query [].name #just check all location names
az group create -n accoolsite --location centralus #create a resource group
az appservice plan create -g accoolsite -n ac-coolsite-plan --sku B1 #create a plan
az webapp create -g accoolsite -n ac-coolsite-wa -p ac-coolsite-plan #create app service
```

Now that the environment is created, let's publish the website on Azure. The basic options of deploying are

* Use FTP to send the published artifact files
* Or upload a zip file
* Use VS default publish option "Microsoft Azure App Service" (super easy)
* Deploy from VCS repository like Git

###### Azure App Service & Git & Deployment

We'll do it from Git with a deployment configuration. Enterprise Git, VSTS, GitHub or BitBucket works, but we'll create our own Git repository inside the Azure App Service.

* Go to Azure portal, and go to the App Service.
* On the left menu (of App Service), go to Deployment section and add deployment credentials (ac-dep/Deploy#1234). The username has to be globally unique
* On left menu, go to "Deployment Options" > Select "Configure required settings" > and choose "Local Git Repository"
* Once saved, go to App Service overview. It'll show
  * Git/Deployment username: ac-dep 
  * Git clone url: https://ac-dep@ac-coolsite-wa.scm.azurewebsites.net:443/ac-coolsite-wa.git

Now let's deploy our website which is ready to be published.
  
```bash
cd /c/Arghya/The-website-directory
git init #initialize git
# add the remote: git remote add repo-name repo-clone-uri
git remote add acazure https://ac-dep@ac-coolsite-wa.scm.azurewebsites.net:443/ac-coolsite-wa.git
git add . #stage the changes & then commit
git commit -m "Initial commit" #all this can be done simply from VS "add code to source code" command
# push the code
git push acazure master #enter password when prompted
# apart from normal push related messages, it'll show lot of messages from Azure through custom hooks
```

## Configuration, monitoring, scaling & debugging Azure

#### Deployment slots

Deployment slots are like a copy (another instance) of the App Service. You can create deployment slots like **Dev**, **Staging**, **Production** for example. Then you can deploy the app to any of the available deployment slots, and test/monitor them. Once we are ready, we can promote (via swapping) them to the next level. If something goes wrong, they can be swapped back. This is just a common deployment workflow, but they can be used in any other way as well.

Creating a deployment slot is actually similar to creating another App Service, just that it'll have some ties with the main production App Service. We dont actually pay for a App Service, we pay for the App Service Plan which includes hardware. As long as we use the same App Service Plan, we do not pay extra for the deployment slots. Deployment slots can have their own settings, git repo, app version etc. But the app will have it's own url so that you can run and check the deployed version. See documentation [here](https://docs.microsoft.com/en-us/azure/app-service/web-sites-staged-publishing).

To create a deployment slot in Azure App Service, 

* Go to the App Service
* Select "Deployment slots" on left menu
* Click "Add" button on the top (not available in free subscription)
* Give it a name e.g. "Staging" and select a configuration source
* Select the production App Service as the configuration source. It'll copy initial configurations from there (e.g. App Service Plan, resource group etc.)

To deploy site to this deployment slot, add a Git repo to this deployment slot and push code to this Git repo. The default name of the primary App Service is always _**production**_.

If there is an application settings in the configuration (e.g. inside `App.config` or `appsettings.json` in ASP.NET), that can be overwritten in Azure. Go to the App Service, select "Application settings" on left menu & Add an application setting with the same key. So, when the app will be deployed in Azure, it'll use this value of the setting rather than the config file.

You can select the "Slot Setting" check-box which will make this setting sticky to the slot. Meaning, when this slot is swapped to production, this settings value will not go to production. Obviously, production slot or App Service can have it's own value.

Image:: 

**Swap builds**: To promote (actually swap option in Azure) a build, go to the deployment slot, select "Swap" on the top menu. Select source & destination, and go. It can also be done from Azure CLI. Generally there is no downtime in swap. Azure has load balancers in front of the App Service and slots, so it just starts redirecting the traffic accordingly.

```bash
az webapp deployment slot swap -g ResourceGroupName -n AppServiceName --slot FromSlotName --target-slot TargetSlotName
```

Basic benefits of deployment slot swap:

1. New changes can be staged and tested before moving to production
2. Production deployments are very fast
3. In case something still goes wrong, older version can be swapped back into production

#### Monitoring

Go to the App Service, select the one you are interested in and see the dashboard. There are couple of charts showing some statistics over time, like - http requests, errors, data in and out, average response time. This charts can be pinned to dashboard as tiles.

Another place to explore much more details is the Azure monitoring tool _"**Monitor**"_. It is generally added as favorite to hub menu, or just search for it. Now select any resource (e.g. an App Service or VM) and see all HTTP and server details like - requests, response time, http status codes, server CPU use, memory use, threads, connections, assemblies, per generation garbage collection etc.

If something looks alarming e.g. high CPU, memory usage, it needs to be mitigated. Like reducing load on the server, or scaling up or out.

###### Setting up alerts

Azure can monitor the metrics explained above for events like VM shutdown, web deploy etc. and send alerts over mail, text etc. For this we need to set up some alerts, and provide details like which resource and what metrics/activity to monitor. Also the events or metric thresholds (e.g. CPU usage > 80% for 5 min) that would fire the alert.

Alerts can also invoke webhooks (push based callback POST API) or run logic apps.

Alerts can be set from Alerts menu under Monitor.

#### Scaling App Services

* Vertical scaling - Adding up more computation power to same server, or reducing to scale up or down
* Horizontal scaling - Adding or removing more servers, to scale out or in

Scaling in Azure basically means change in App Service Plan. You can spend more for a server with higher computation powers, to scale up.

Or, we can use multiple instances of the similar servers to scale out. That will cost N x App Service Plan. Some plans support auto scaling as well - based on load (a metric, described above) or scheduled (e.g. scale out during business hours). You can also choose to keep the server shut down for specific durations to save money, see details [here](https://azure.microsoft.com/en-in/features/autoscale/). **Autoscale** also support scale in (reduce number) rules.

While scaling apps, Azure takes care of allocating suitable infrastructure, setting it up for use, copying code/binaries to it and running the app in new server(s). It also takes care of putting the new server(s) behind load balancer properly so that incoming traffic is routed correctly.

## Diagnostics & trouble shooting

###### Diagnostic logs

Diagnostic logs available on left menu inside App Service. One can choose have have application logging (e.g. System.Diagnostics in .NET) and store them in file-system (which auto turns off in 12 hr) or in an Azure blob storage, level of logging can be chosen i.e. Error, Warning, Information, Verbose.

Web server logs (e.g. IIS logs) can also be used, and can be configured like how many days to keep the logs. It also has failed request tracing.

Azure provides ways to download the logs through FTP or Azure CLI...

###### Application insights

This let's collect telemetry data, like - details of each request & response, performance details, queries fired & performance, full stack-trace of failures etc. It can profile the application and detect problem areas. This is available for many platforms (.NET, Node etc.) and may incur additional costs.

You can go to failure menu inside Application Insight to see failed requests, exceptions etc. Or see the performance menu to get an general performance overview of the application.

###### Diagnose and solve problems

This menu gives options to another bunch of tools to investigate, diagnose problems (e.g. Analyze memory dump from w3wp.exe process) and generate reports.

###### Project Kudu

Go to the App Service -> go to "Development tools" on left menu -> select "Advanced Tools" -> select "Go". It'll launch a page https://app-service-name.scm.azurewebsites.net/ environment and REST API details.

This **"Project Kudu"** gives a lot of information about the server and environment details like

* System details (OS, CLR version, uptime etc.), app settings, environment variables, system paths (Windows) etc.
* Process explorer similar to task manager in local Windows system
* Installed & available extensions e.g .NET Core, Node, IIS Manager, WordPress etc etc.
* Tools like file system explorer and "push zip to deploy"
* Debug console to access the file system and run command/powershell in the server

**Note:** When a new web app is deployed to a Azure App Service, the Kudu service inspects the app and figures out what type of app it is e.g. .NET, .NET Core, Node etc. Based on that it creates deployment scripts and deploys the app. If required (e.g. custom copy of files), the deployment script can be edited and updated.
{: .notice--info}

#### Remote debugging Azure apps

Now, if nothing else works, one may want to debug the application running on Azure. And that is possible! We can enable _**Remote Debugging**_ to remotely debug the app running on Azure App Service locally from visual studio.

* Go to the App Service -> Application settings -> Debugging
* Turn remote debugging on, and select the Visual Studio version you want to use -> save

Now to actually debug the application, hit up your local VS and open the solution

* VS menu -> View -> Cloud explorer 
* Select the account icon -> login -> select subscription
* Now it'll show all your resources. You can also explore the files and logs!
* To debug, right click on App Server and select "Attach Debugger" (be sure to already have set break-points)
* If it shows the alert, go to Options -> Debugging -> General -> uncheck "Enable Just My Code", and attach again
* It'll now start the Azure site and be ready to hit break-points!!

## Databases on Azure

If we use Azure as IaaS and take a VM, we can install any applications on it including database servers. Azure also provides PaaS database servers like SQL Server, Cosmos DB etc.

_**Azure SQL**_ is basically SQL Server running on Azure. There's not much of a difference for a developer, but it's little different for DBAs as most of the administrative works are done automatically by Azure.

Before creating a SQL Server database on Azure, understand that we really don't know where the databse will be actually hosted, in which server or VM. Those stuffs will be taken care of by Azure itself. But we need to provide a _server name_ so that we can logically group databases together, use them in connection strings ?? and have specific admin accounts setup.

* Go to "SQL databases" on the left hub menu -> click "Add"
* Provide database details like name (accoolsitedb), subscription, response group (accoolsite) etc.
* Select starting point as blank DB / sample / restore. We did blank
* Configure server name: accool.database.windows.net, Admin: acdba, Password: Database#1234
* Select a pricing tier for the database (not the server) e.g. Basic 5 DTU, 2 GB disk-space (DTU is like a data throughput unit combining CPU, memory & I/O)

* The database can be scaled up from "Pricing tier" menu (increase DTU, disk-space)
* Enable "Geo-Replication" (replicate in geographically distributed regions) for high availability & disaster recovery
* Enable data encryption from "Transparent data encryption" menu
* Set "Alerts" to monitor metrics (e.g. CPU usage, DTU, database size, deadlocks etc.)
* Do Restore from snapshot, export database, performance overview, query insights diagnosis, auto tuning with index and many more options














#### References

[All things Azure](https://docs.microsoft.com/en-in/azure/)
[Free Pluralsight courses](https://azure.microsoft.com/en-gb/training/free-online-courses/)