---
layout: post
title: "Basic cloud design concepts & Serverless computing"
excerpt: "To bo broken down into multiple pieces - [1] Cloud design basics [2] Serverless computing"
date: 2018-08-08
tags: [cloud, azure, serverless, functions, paas, distributedsystems, partitioning]
categories: articles
comments: true
share: true
published: false
---

# Cloud computing concepts

From [Planning a Microsoft Cloud Solution (2016)](https://www.linkedin.com/learning/planning-a-microsoft-cloud-solution-2016)

## Cloud models

1. <u>Private cloud</u> - all hosted on company's private data center. Hardware, network, software updated, security and all are managed by the company. It can be costly, and needs expertise on systems, networks etc.
2. <u>Public cloud</u> - infrastructure is managed by someone else (e.g. Amazon, Microsoft, Google), they maintain servers, patch & update them, take care of general security. For the company, cost moves from CapEx (capital expense) to OpEx (operational expense). But generally overall cost maybe lower, as you pay only for what and when you use.
3. <u>Hybrid cloud</u> - distributed between private & public clouds. Generally critical data and operations run on on-premise data centers. Other things like, non-critical workload, testing, disaster recovery leverage public clouds. If done right, it might reduce costs as well.

## General considerations before moving to cloud

1. Connectivity - you need proper connectivity to work properly with external clouds from on-premise setup
   1. Local infrastructure to connect to external systems. Modern cloud systems need modern network in place
   2. May require additional equipments like dedicated VPN gateway
   3. Bandwidth, as connecting and moving across data needs much more bandwidth (there are tools to estimate - MAPS toolkit)
2. Costs
   1. Costs may or may not be predictable (there are tools to estimate though)
   2. Pay-as-you-go services may incur high costs under high demand
   3. Overall infrastructure change might cost as well. Same for internet costs
   4. Providers like Azure, does not charge for moving-in data. But they do charge for moving-out data from Azure!
3. SLA or Service Level Agreements
   1. Check the SLA with the provider to know what you're getting
   2. For example, Azure says your web apps (App Service) will have 99.99% up-time with 2 VMs. If your infrastructure does not match, the SLA doesn't hold either
4. Workload compatibility
   1. Not everything is supported on all cloud infrastructure
   2. Outdated or very new tech, server, OS may not be supported
   3. Though few things does run without official support, they do not fall into the SLA
5. Limitations
   1. All subscriptions has default and maximum limits
   2. Make sure your subscriptions cover your needs, or change subscription or the architecture
6. Data protection
   1. Though cloud providers does general stuffs, it's up to you to protect your own data
   2. Make sure data are used properly and securely
   3. Have all the necessary backup & disaster recovery. For data as well as VMs

## Disaster recovery

The processes or precautions taken to save the data and application (or the infrastructure as a whole), when a disaster happens.

Disaster can be natural or man made

* Hurricanes, Tornado, Flood, Earthquake etc.
* Virus, accidental deletes, failed upgrades or patches etc.

#### Azure strategies for disaster recovery

1. Azure Backup - to backup in and out of Azure. Secure, reliable and automated. Can backup files and folders to whole workloads like SQL, Sharepoints and VMs. Generally from on-prem to Azure data centers. Backups are automated (like nightly jobs), but recovery is not automated.
2. Azure Site Recovery (ASR) - manages replication of on-premise VMs to data centers or data centers on <u>other regions. On failure of primary, the secondary can be activated. It is simple & automated, for easy failover and recovery</u>

###### Differences of VM backup vs ASR, mostly from [here](https://techcommunity.microsoft.com/t5/Azure/ASR-Vs-Azure-backup/td-p/89256)

* Backup ensures that your data is safe and recoverable while Site Recovery keeps your workloads available when/if an outage occurs.
* Backup is for correcting human errors such as accidental deletion or corruption introduced because of a patch etc. On the other hand, disaster recovery is when your primary data center is gone and you want to run your business from secondary data center.
* Backup typically has a data loss up to one day(once a day backup is norm across VMs) where as Site Recovery, post restore you want data loss to be as minimal as few seconds to max minutes.
* Azure Backup recovery mostly works on the source region as your production VM where as ASR recovery will let you failover to a different region than production VM region. 

##### Azure Backup

_**Backup-as-a-service**_

Backup is as simple as a secure remote copy of things that you want to secure, most commonly a directory or drive from you personal or on-prem machine. If required, they can be retrieved just as a copy of the original (e.g. copy of a folder).

Durable. Backed up daily or weekly. Can keep backup for up to 10 (99?) years. Unlike other services, there is not cost for moving this data to and from Azure.

Stores files & folders from disk. Also backups up memory by writing them to disk before backup. Restore is as simple as selecting a restore point and restore.

**Note**: One thing to note here is, Azure VMs are durable by default, they have 6 copies each. But they are only to make the Azure system stable in case something goes wrong within Azure infrastructure. These backups are NOT available for recovery if something goes wrong from user like accidental delete or failed patch.
{: .notice--info}

To do it

* Go to your resource group and add a new service
* Select 'Backup & Site Recovery (OMS)' 
* Give a name and select other options like subscription, location
* Once deployment completes, the settings will have option for both 'Backup' & 'Site Recovery' 
* Select type of source, Azure or On-prem
* Select stuffs to backup like files & folders, SQL server, Sharepoint etc.
* Follow steps to install the recovery agent and get your **vault credentials** downloaded as a file (this is important, without it you CANNOT backup)
* During installation, use vault credentials to generate **pass phrase** and save it somewhere secure (not in the same machine that needs to be backed up). Again, without this, you CANNOT recover the backup
* once done, run the recovery agent on the machine to be backed up and schedule backup
* Select files or drives to be backed up, select frequency (weekly to max 3 times a day) & retention of backup
* IF your files are tooooo big, you can do offline backup as well. That is save all in a physical disk, and send it to Microsoft for backup :D

**Note**: You can also backup your whole VM from the initial option 'Azure'. Azure can backup VMs only from the same region as the backup region.
{: .notice--info}

##### Azure Site Recovery (ASR)

_**Disaster-Recovery-as-a-service**_

It's not a backup, rather a replica of on-prem physical machine or VM. Has replication frequency of 15 min to 30 seconds! Choose **Application Consistent Snapshot** to backup in-memory data as well.

In recovery configuration, manual steps or scripts can also be added.

ASR gives a <u>Test Failover</u> option to test recovery under non-disaster situations. It does not affect the environment, or users and have no downtime.

A **failover** can be <u>planned failover</u> (like planned maintenance) or <u>unplanned failover</u> (like power outage or hardware failure). In both cases there will be some downtime before the secondary comes up. Planned one does not have any data loss whereas unplanned ones will have some data loss based on backup/replication frequency.

In both cases, you can have a **failback** once the primary system is back up. The data will be replicated back to primary.

ASR can be used as an automated way to move on-prem systems to Azure, rather than manual copy or Powershell scripts.

#### VMs

On-prem VMs will have 

* Host OS e.g. Windows Server 2012 R2 
* It'll have a VM system running on it like Hyper-V
* The VM will provision one or more guest OS like Windows Server, Linux etc.

Azure will have 

* Azure Resource Manager or ARM.
* It can host other OS or systems like Windows Server, Linux, Office Exchange, SQL Server etc.

Benefits of Azure VM

* Pay only for what you use
* Easy to setup
* Easy connection setup without fiddling with firewalls
* Super easy to scale which can also be automated (like pre-scheduled scale-up & scale-down for a defined time period)
* Can be moved in and out of on-prem


https://www.linkedin.com/learning/learning-cloud-computing-serverless-computing


## Serverless Computing

What it is? Someone else (a cloud provider) takes care of the server and platform, you just build application (and?)

* Focus on computing, not servers
* Resources & servers are allocated dynamically. <u>Automatic scaling behind the scenes</u>
* Automatic access to stuffs like - compute resources, storages, language runtime
* Pay only for active usage, no cost for idle time

*Wait, **is serverless computing basically a different name for PaaS computing?*** Well, not exactly. The concepts of serverless-computing builds on top of PaaS. It can be though of as <u>more fine grained and automated PaaS</u>. In PaaS type of architecture, we build full-scale applications that runs on cloud platforms (like Azure or AWS), and can also be configured to scale automatically based on schedule or traffic. On the other hand, in a serverless-computing architecture, the pieces of systems (like small APIs or just callable functions) are more light-weight and scale fully on demand without any configuration. They react to requests/events. When something happens, they start up in milliseconds, and can shut down if nothing else to do. So, ideally, they are <u>not always-on, and fast and fully dynamic in start, stop and scaling</u>. These features and infrastructures are provided & managed by cloud providers like *AWS Lambda, Azure Functions* etc.
{: .notice--info}

#### Types of serverless computing

The most common types of serverless-computing are of two types

* **BaaS**: Backend-as-a-service, where cloud providers give an infrastructure to have functionalities ready to do basic stuffs or interact with database without any custom code. Examples like **Google Firebase** database with APIs, weather service API etc. A smart client can call them directly. That means, if you have a smart enough client, you may not need to write any custom code for the backend at all! Similarly **Auth0** provides authorization-as-a-service.
* **FaaS**: Function-as-a-service, where (ideally) small pieces of code or functions are written that is hosted on cloud service, and respond to request by doing some processing, or database interactions. They are ideally light-weight stateless APIs, again callable by clients. The good fit example would be, where not each API does a  lot, but lot of clients access them on irregular patterns.

#### Things to consider before moving to serverless

Consider following points if want to create a serverless application, or move your existing one as serverless

* Can be done for <u>new apps</u>. Not worth the effort for old apps. Their language/platform may not be supported. Also breaking them down to function levels might be too much to do, and may not work architecturally.
* The app must have <u>value on scaling</u>. If it has fixed need, or very low demand, serverless doesn't help a lot.
* The <u>SOA or microservice</u> oriented applications would gain from serverless. The monoliths, not really.
* Also, looking at the current scenarios, serverless is <u>public only</u>. So once done, it might be difficult to move out of public clouds, or there maybe lock-in scenarios.
* Also check the provider and the functions architecture provides the required level of security, data protection & compliance (just as in case of any type of cloud computing)
* Cost (for very high usage, the cost might also shoot up). Also consider what kind of future requirements you might be having, does the current solution accommodate that?

#### The providers (FaaS)

Here we'll look at some of the most popular serverless cloud computing providers. **All provides event based service model**, monitoring, security, auto-scaling etc. They all present the functions with URLs like simple REST like APIs. Also they all provide some free credits to test and use their services to some limit.

1. **[AWS Lambda](https://aws.amazon.com/lambda/)** : Kind of started the serverless trend, and has the largest user-base. Supports `Java`, `Python`, `Node.js`. Let's you connect and collaborate with other AWS services like Dynamo DB and services hosted on AWS. Supports event driven model.
2. **[Azure Functions](https://azure.microsoft.com/en-in/services/functions/)** : Azure counterpart for serverless cloud services. More flexible in supporting `C#`, `F#`, `JavaScript/Node`, `Python`, `Bash`, `PowerShell` etc. Let's you collaborate with other Azure services like Azure Storage, Cosmos DB, SQL Azure, Cognitive services etc. it's actually a special type of *App Service* that is managed & scaled automatically, and are event driven (timer, http request, trigger based on other Azure services etc.)
3. **[Google Cloud Functions](https://cloud.google.com/functions/)** : New, but promising. Has similar features and comes with Google's promising scaling.

#### Basic design process

Basic design of serverless functions comprise of the following general steps

1. Identify and list the functions to build
2. Define event and data model for each function
3. Decide and design the data interaction of the functions like storage access, DB access, other API calls
4. Chain multiple functions to build a functions workflow if required
5. Define the security model as required
6. Also plan on the platform of choice, how to deploy, test & monitor

## Cloud architecture components

https://www.linkedin.com/learning/cloud-architecture-core-concepts/storage-level

Irrespective of specific providers and tools, the basic conceptual structure for all the cloud applications have some common elements. The basic choices in cloud application architecture 

1. Application level : The most important part of building or moving application to the cloud is, the application has to work and perform as expected
   1. The correct choice of technology and language that works
   2. Common infrastructure to be used across applications like security & authentication
   3. Application servers (e.g. App Service in Azure) and containers (e.g. Kubernetees)
   4. The UX build keeping in mind the back-end cloud system
2. Network level : Connectivity is super important for a cloud solution to work. That means you need correct and updated infrastructure, enough bandwidth, and other setup if necessary e.g. VPN
   1. DNS - to be setup properly and adjusted with firewalls
   2. Intracloud network : For IPC (inter process communication) between multiple processes hosted on cloud, like links and queues
   3. Intercloud network : Connectivity between components hosted on different cloud solutions (e.g. AWS vs Azure), like dedicated links etc.
3. Processing level : Processing and computing are core of traditional computer systems, so are the cloud systems. The options comes to the power (no of cores, GHz) and types (make, 32-bit vs 64-bit) of processor. The good part is, in almost all public cloud providers, changing the CPU is pretty easy, and can be automated as well (scaling) based on need.
   1. CPUs - how many and how big CPUs, Less can throttle requests while too big can incur huge costs
   2. Host platforms/OS - Also choose your host OS based on your need like Windows Server 2008, Windows Server 2012 R2, Red Hat Linux, Ubuntu etc. One may also choose a containerization technology (e.g. Docker or Azure App Fabric) over choosing specific platform 
4. Data level : They options for storing structured data, that comes with data management tools. Or in other words, databases.
   1. Transactional database - database that supports basic data transactions (CRUD operations, and maybe, transactions) like AWS Dynamo DB or Azure SQL, Cosmos DB
   2. Big data/analytics database - generally pulls data from transactional database (in batch or real time) to run analytics and produce insights
5. Storage level : They are basically storage spaces given by cloud systems just to store any type of data (e.g. blob storage in Azure)
   1. Physical storage - actual data storage like Firebase, which are generally accessed through APIs
   2. Logical storage - abstracted storage layer that actually connects to one or more physical storages. Generally code can access them directly as local storage
6. Testing & deployment : the tools available and choices for deployment, testing and CI & DevOps