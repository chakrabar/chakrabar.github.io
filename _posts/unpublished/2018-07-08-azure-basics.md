---
layout: post
title: "Azure basics"
excerpt: "Basics of using Microsoft Azure cloud platform - gist from Scott's course"
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

From pluralsight course "Developing with .NET on Microsoft Azure - Getting Started" by Scott Allen
https://app.pluralsight.com/library/courses/developing-dotnet-microsoft-azure-getting-started/table-of-contents

Platform support - ASP.NET, Java, Node, Rails, php, Python Django etc.

Azure platform & services

Cloud database & storage e.g. Azure SQL, Cosmos DB etc.
Automation - SDKs, build & CI/CD systems
Service hosting - App services & Azure functions (serverless architecture)
Virtual machines - servers on cloud, Windows, Linux, SQL Server, Containers etc.
Azure key vault to manage application secrets.

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

**Note:** If a site is deployed while it is being used, it sometimes fails to deploy as it cannot replace some of the files which are currently being used by the process. Make sure no one is using the site while it's being deployed (and stop it, if required!). This is another reason slots should be used for deployment.
{: .notice--warning}

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

## SQL database on Azure

If we use Azure as IaaS and take a VM, we can install any applications on it including database servers. Azure also provides PaaS database servers like SQL Server, Cosmos DB etc.

_**Azure SQL**_ is basically SQL Server running on Azure. There's not much of a difference for a developer, but it's little different for DBAs as most of the administrative works are done automatically by Azure.

Before creating a SQL Server database on Azure, understand that we really don't know where the database will be actually hosted, in which server or VM. Those stuffs will be taken care of by Azure itself. But we need to provide a _server name_ so that we can logically group databases together, use them in connection strings and have specific admin accounts setup. The server will have a `master` database to store all user details for all the databases under it, etc.

To create a new SQL Server database on Azure (aka Azure SQL)

* Go to "SQL databases" on the left hub menu -> click "Add"
* Provide database details like name (accoolsitedb), subscription, resource group (accoolsite) etc.
* Select starting point as blank DB / sample / restore. We did blank
* Configure server name: accool.database.windows.net, Admin: acdba, Password: Database#1234
* Select a pricing tier for the database (not the server, server is just a logical container) e.g. Basic 5 DTU, 2 GB disk-space (DTU = Database Throughput Unit combining CPU, memory & I/O)

Features of Azure SQL

* The database can be scaled up from "Pricing tier" menu (increase DTU, disk-space)
* Enable "Geo-Replication" (replicate in geographically distributed regions) for high availability & disaster recovery
* Enable data encryption from "Transparent data encryption" menu (allows to store data as "always encrypted")
* Set "Alerts" to monitor metrics (e.g. CPU usage, DTU, database size, deadlocks etc.)
* Allows to do stuffs like - restore from snapshot, export database, performance overview, query insights & diagnosis, auto tuning with index and many more options
* On the left menu, it provides connection strings in different formats like ADO.NET, JDBC, ODBC etc. Ideally you'd setup a non-admin user to be used for the connections. Now any standard tool like SSMS or [MSSQL-CLI](https://github.com/dbcli/mssql-cli) can connect to the Azure SQL DB. From settings one can restrict incoming connections through the firewall, to set of known IPs.

> Server=tcp:accool.database.windows.net,1433;Initial Catalog=accoolsitedb;Persist Security Info=False;User ID={your_username};Password={your_password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;

With admin user, create another db user

```sql
--on master DB
CREATE LOGIN devuser WITH PASSWORD = 'azsqdbdvuser#4321'

--now connect to other DB (USE db-name does not work in Azure SQL)
CREATE USER [devuser] 
FOR LOGIN [devuser]
GO

EXEC sp_addrolemember 'db_datareader', 'devuser'
EXEC sp_addrolemember 'db_datawriter', 'devuser'
EXEC sp_addrolemember 'db_ddladmin', 'devuser'
```

Relative scale - for a standard production site with ~25k page views per day, where they create ~2 queries per page (i.e. ~50k queries per day), a 10 DTU Azure SQL works just fine. Then there is premium plans with up to 4000 DTUs, if even that does not fit, use Azure data lakes.

SQL elastic pool for databases - if you are using multiple databases, you can create a pool of resources and distribute among the databases, it'll elastically adjust to uneven spikes of traffics among the databases.

**Note:** As a general practice with any appsettings, when you host your application on Azure, you do not hardcode the SQL Azure connection string in the settings. Let it have local or development connection. In your Azure App Service application settings, override the setting value with the actual SQL Azure connection string, inside Azure. Ideally you should make it a slot setting (slot = a deployment environment), so that you can connect to different databases for dev, staging, production etc.
{: .notice--info}

## Azure Cosmos DB

Azure Cosmos DB is a document type database on Azure that provides different types of APIs/flavours

* SQL
* MongoDB
* Cassandra
* Azure Table Storage (for large amount of structured by not relational data. Data is schema-less with name-value properties, with no need of joins or SPs. It is actually a storage of type "wide column store", similar to Cassandra)
* Gremlin for Graph

To use a Cosmos DB, you need an account. There is no charge for the account, but for the collections inside databases. The hierarchy is like this

Cosmos DB Account (provides URI to connect through clients)
  = Databases (optionally, one can provision database throughput in RU & corresponding price)
    = Collections (at this level we decide for price on each collection, in RU/s)
	    = Documents (stores the actual data)

**Note:** Understand that a collection is not a table. Cosmos DB is schema-less, so a collection can hold any number & types of documents with any structure of data. If we think about when to create new/separate collections, it basically comes down to computing power or capacity. Like high performance collections and low performance collections.
{: .notice--info}

RU = Request Unit. 1 RU = cost to read 1 KB document over http

I created an Azure Cosmos DB account - "accool.documents.azure.com", with `SQL` API, and an existing resource group.
Once account is created, go to 'Data Explorer' to create database & collection. I created database "accosmosdb", collection "courses" with fixed 10 GB size.

Fixed size Cosmos DB does not do partitioning. Unlimited storage DBs do partitioning, based on some supplied key (property) that all documents will have. It's a key, based on which data can be grouped and partitioned across infrastructure. e.g. /address/zipcode or /department/id. Optionally, Unique Keys can be added that'll be unique per partition key.

* Cosmos DB interaction with client happens over HTTP, and there are NuGet packages to make things easy. `Microsoft.Azure.DocumentDB` (.Core)
* To connect, create a `new DocumentClient(uri, authKey);`
* Get the connection string from Cosmos DB account overview page
* Go to Keys in left menu, and get a read OR read-write key as required. We'll take read-write key. A set of 2 keys, primary & secondary are given so that they can be rotated later. I took primary key. Use it through "Azure Key Vault";

```cs
public class CourseStore
{
    private DocumentClient _client;
    private Uri _coursesCollectionUri;

    public CourseStore()
    {
        var uri = new Uri("https://accool.documents.azure.com:443/");
        //use the key through Azure Key Valut or AppSettings
        var key = "Jt9b1dXj4GXPIM0JSjdH3F1HmU0AbPp21GBI3Tq2Smc99MnA926x034tAFltoHKR6Ta8H5O7n4c5aLwbwMsTmg==";
        _client = new DocumentClient(uri, key);
        _coursesCollectionUri = UriFactory.CreateDocumentCollectionUri("accosmosdb", "courses"); //database, collection
    }

    public async Task InsertCourses(IEnumerable<Course> courses)
    {
        foreach (var course in courses)
        {
            await _client.CreateDocumentAsync(_coursesCollectionUri, course);
        }
    }

    public IEnumerable<Course> GetAllCourses()
    {
        var courses = _client
        .CreateDocumentQuery<Course>(_coursesCollectionUri)
            .OrderBy(c => c.Title);
        return courses;
    }
}
```

* From the left menu, select Metrics to see system performance
* From the data explorer menu you can see all the details of data - see data, query them, see indexes & modify, scale up or down, write SPs in JavaScript, Triggers, User Defined Functions etc (Since we chose SQL API)

## Azure Storage

Different types of storage option for files and data, and how to interact with them.

* Blob storage - for _binary large object_, but practically any type of file can be stored. Works mostly on REST
* Table storage - a type of database/data store, that allows storing data as key-value pairs
* Queue storage - basically a messaging queue
* File storage - for apps that do file sharing (SMB 3.0 protocol)
* (Disk storage - the storage for general files for VMs, not considered a general purpose storage)

#### Blob Storage

Like creating any other resource on Azure, create new storage account for it. The hierarchy is like

Storage account
  = containers (somewhat like folders, but not exactly)
    = blob (stores actual data)

Create a new storage account with details like - location (e.g. East US), subscription, resource group, type of disk (HDD or SSD), etc. I created a new account "accoolstore.core.windows.net".

**Note:** One thing to note here is, there is actually no concept of sub-folders in containers, but the effect can be simulated by create more containers with path-like names e.g. main, main/secondary, main/secondary/tertiary etc.
{: .notice--info}

Use [Azure Pricing calculator](https://azure.microsoft.com/en-in/pricing/calculator/) to estimate costs and decide on resources.

To use the blob storage, go to the "Blob" section. Create a new container, set privacy level - private, anonymous read access, anonymous access to read and browse the whole container. I created container "images" as private.

You can interact with your data directly there on the UI or can download the free cross-platform tool [Azure Storage Explorer](https://azure.microsoft.com/en-in/features/storage-explorer/). Also you can interact with them using `C#` code from a `.NET` application.

Blob type options

* Block blob - best for large data enabled for streaming, also for efficient parallel upload
* Page blob - good for random access read-write
* Append blob - optimized access at end of file e.g. log file that'll be frequently appended

From code, we can interact with blob storage with http messages, but there is a NuGet package `WindowsAzure.Storage`, then use a `CloudBlobClient`. It needs the base URI (go to storage account, blob, properties e.g. https://accoolstore.blob.core.windows.net/), and storage credentials. SToarge credentials need storage name & access key, both ara available under left menu "Access keys".

###### Shared Access Signature (SAS)

Kind of a auth token (as URI query string) given to temporarily access a storage content. A token can be generated from the portal by specifying properties like validity range, type of access (read, write, delete etc.), allowed services level (blob, table, container etc.) connection type (http or https) etc. The token has readable start & end date etc., but has a hashed signature. So the values cannot be tampered with. For example

```bash
# SAS query string
?sv=2018-03-28&sr=b&sig=4qcaKRxvZny4QBi2aCdd2yZ3oDulloW8TPWDt635wA4%3D&st=2018-07-19T20%3A37%3A33Z&se=2018-07-19T20%3A57%3A33Z&sp=r
# uri decoded
?sv=2018-03-28&sr=b&sig=4qcaKRxvZny4QBi2aCdd2yZ3oDulloW8TPWDt635wA4=&st=2018-07-19T20:37:33Z&se=2018-07-19T20:57:33Z&sp=r
# sample full URI
https://accoolstore.blob.core.windows.net/images/c4085bbe-4ff7-4e31-8b13-3225f470f808?sv=2018-03-28&sr=b&sig=4qcaKRxvZny4QBi2aCdd2yZ3oDulloW8TPWDt635wA4%3D&st=2018-07-19T20%3A37%3A33Z&se=2018-07-19T20%3A57%3A33Z&sp=r
```

It can also be done programatically.

## Azure Functions

Azure Functions is a different type of PaaS service from Microsoft Azure that supports _"serverless computing"_. On Azure Functions service, you write small pieces of code (i.e. _lightweight functions_) that are hosted as executable functions on cloud.

> Azure Functions are an event-based serverless compute experience to accelerate your development. Scale based on demand and pay only for the resources you consume.

The concept of **Serverless computing**, from a very high level is, you run your applications without any servers. But obviously you need servers to do any processing, respond to request or store data, however small that is. The very basic idea behind "server-less" is _you don't own any servers_. So you use PaaS and SaaS combination to fulfil your application needs. For few things, you can use existing SaaS solutions and call 3rd-party APIs, for data needs you use some cloud data PaaS service. Finally for your domain specific custom things, you write small pieces of functions that you can directly host on some cloud PaaS as online executable functions. Some of the popular cloud services to host them are `Azure Functions` & `AWS Lambda`.
{: .notice--info}

Azure functions are mostly used to

* HTTP trigger - Respond to http requests (e.g. normal REST calls from client)
* On scheduled timer events
* On predefined system events (e.g. storage limit on a store has exceeded maximum threshold)
* Respond to webhooks (kind of post request from another service)

One important thing to understand for Azure Functions is, Azure offers a plan to pay only per server processing (_Consumption Plan_). That means [1] if it is not used, you need not pay. For lower use of CPU time & memory use, it's practically free! [2] you don't have to think about VMs & scaling. For small usages you'll pay only that much. If usage picks, Azure will automatically scale it in the background.

To use this, we need to create a **Function App** (though behind the scenes, it's built on top of Azure App Services). Give it a name and basic configurations like location, OS choice (Windows, Linux, Docker), storage account and it's ready! The first initial function app can be created only from the **App Service** menu! I created mine at 'acfuncs.azurewebsites.net'.

A function app can have one or more functions. While setting up functions, it has following options

1. Functions - create your own functions using C#, F#, JavaScript, Python, Bash, PowerShell etc. Any type of triggers like HTTP request, new message in Azure Storage Queue, Service Bus Queue, timer, scheduled email, service bus topic, blob, cosmos DB, webhook, GitHub comment etc.
2. Proxies - light-weight proxy between client and an Azure backend. The backend is generally some API or website on App Service. The proxy sits between the client and the Azure backend and can manipulate incoming requests or outgoing responses, like adding query string or headers, or modifying the response body. 
3. Slots - deployment slots just like Azure App Service

**Note:** The proxy function is still a functions, and the backend is optional in proxy function. If not used, a request to the proxy URI will produce response as defined in the function. If a backend is defined, then the function can use the response from the backend and make modifications on top of that.
{: .notice--info}

To create a new function within the function app, click on crate. select type of function (e.g. HTTP trigger), select language, give a name, select authorization level* & click create. It'll open an editable code.

To test the code, there itself you can run with different inputs, or directly hit the URI from browser or any REST API client.

A function can authorization levels

1. Anonymous
2. Function - function level
3. Admin - admin level, must have a host key

Both function keys & host keys are available on the left "Manage" menu of functions. All general App Service features are available in "Platform features" top menu, like application settings, authentication, service pricing plan, notifications etc.

Go to "Deployment", configure new, and select "Local Git repository". It'll create a Git repo on Azure and the URI is available in Function App properties. Mine is https://ac-dep@acfuncsv2.scm.azurewebsites.net:443/acfuncsV2.git (acfuncsV2)

Azure Function
Version 1 = .NET Framework
Version 2 = .NET Core (enable it from settings)

To create and deploy functions, create new project in Visual Studio with template Cloud > Azure Functions. Select the type of function, like HTTP trigger or blob trigger. Code it, test locally and push to Azure Git repository.

https://acfuncs.azurewebsites.net/api/SampleHttpTriggerFunc?code=rPjD0w4UiGxWFaV71MvHhT9LE1VIeK5PtLXygcH6lNhLK2dNQNUuCw==&name=arghya

#### AI & Cognitive services

Created new face app "acfaceapi" (search 'face' in create resources & create) with cognitive services. It can detect faces, estimate age and analyse expressions. All these services are exposed as REST APIs.

Rather than making RAW API calls, Microsoft provides bunch of NuGet packages with names in the lines of `Microsoft.ProjectOxford.face` etc.

#### A complete flow

User loads an image from the website hosted on App Service
It saves the image in Azure Storage (Blob)
That fires a blob-trigger hosted on Azure Function
This invokes Azure AI & cognitive service to do a face analysis on the image saved, from the blob URI
After analysis, the results get stored in Cosmos DB collection
Now the page reads the results from Cosmos DB and displays on the page

## Continuous Integration with VSTS

VSTS = Visual Studio Team Services

For source control & delivery pipeline. Free credits available @ https://visualstudio.microsoft.com/ (free for up to 5 users, unlimited stakeholders, unlimited private Git repos!)

Supports web & native apps on Windows, Linux, iOS, Android etc. Can also deploy to app stores like Apple app store, or Android PlayStore. Supports C#/.NET, Java, Node, Go among others.

Also supports project management features like Git, Unit and Load testing, reporting, CI/CD, Agile & Scrum with planning, sprints, Kanban boards etc.

###### Create a new VSTS project

Visit https://www.visualstudio.microsoft.com and create your account, mine is @ https://chakrabarty.visualstudio.com

For code, create a new repository, Git or VSTS

For Git, to add local code, just add a new remote and push. If same Microsoft account is used for Azure & VSTS, it automatically allows to push code (?)

```bash
git remote add acvsts https://chakrabarty.visualstudio.com/accoolsite/_git/accoolsite
git push -u acvsts --all # might get prompted for credentials
```

Most of the options are available in Visual Studio under View > Team Explorer

###### Build

Generally, setup a build from VSTS site.

1. Go to Build menu and create a new build definition.
2. Select the project to build (already added to VSTS Git repo), select things like project template (e.g. ASP.NET Core), agent environment (e.g. VM with Windows, or configured private server) build steps (autopopulated with - get source, NuGet restore, build, test, zip deployables, publish artifacts)
3. Check the variables like build configuration (e.g. release)
4. Enable trigger to source control integration, to fire CI from Git repo
5. And 'Save & queue'

It'll run all the build steps, show the live output from build agent in console. Once done, it'll have test results, test coverage, downloadable logs, and published artifacts as zip.

###### Release

To setup release (that is the CD part of it)

1. Create a new release
2. Select source, like - latest artifact from the CI build
3. Setup target environment, like Azure App Service
4. Enable continuous deployment trigger (trigger from CI build, for example)
5. Setup (pre-)deployment conditions like to fire on auto-trigger or manual, schedule if any. Also add delay, user approval, check gates like- no alerts from Azure monitor
6. Add any custom tasks like - getting key from Azure, running a PowerShell/Bash script, deploying at SQL database, sign android app, Gradle/Grunt/Gilp/Maven/MSBuild/Bower tasks, post to Slack, run SonarQube, build with Ant/Psake/CMake/Cake etc. & many many many more.
7. Do NOT forget to click 'Create' to save the release difinition

#### References

[All things Azure](https://docs.microsoft.com/en-in/azure/)
[Free Pluralsight courses](https://azure.microsoft.com/en-gb/training/free-online-courses/)
[Unrelated - authentication](https://docs.microsoft.com/en-gb/aspnet/core/security/authentication/social/index?view=aspnetcore-2.1)