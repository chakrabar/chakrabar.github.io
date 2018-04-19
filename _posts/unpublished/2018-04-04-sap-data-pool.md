---
layout: post
title: "SAP data extraction for external analysis"
excerpt: "What is SAP. Different modules like MM, PP, SD, FICO etc. SAP Connector. Pulling SAP data with .NET application"
date: 2018-04-04
tags: [tech, SAP]
categories: articles
comments: true
share: true
published: false
---

# Initial questions

1. Is it an existing system which we are trying to enhance, or it needs to be built from scratch?
2. If existing, how the data is being extracted now and any issue with current setup.
3. What is the scale of data? e.g. average data volume accumulated daily
4. Is data extraction required periodically, like a nightly-job? Or like a continuous stream of live data?
5. Any specific format for output data? Specific types, like XML/JSON etc?
6. What is the type of "Data pool" that was mentioned?
7. Any kind of archieving of data required?
8. The analytics system does it's own logging or additional logging required? If required, do we need 9. detailed logging or just basics?
9. What is the current server infrastructure being used (type of hardware, OS etc.)
10. What type of databases (if any) are being used within the set of applications (e.g. Oracle, SQL Server, MongoDB etc.)
11. If there are existing infrastructures for any application tech-stack(s) like Java, .NET, Node etc.

#### What is ERP

#### What is SAP

* Initial understandings...
* System Application and Products
* SAP has it's own set of database/tables. The native data manipulation language is ABAP, a SAP specific language
* SAP table names are generally cryptic
* Apart from physical data tables, SAP also contains logocal tables like _pooled_ & _cluster_
* SAP Object types
  * **Transparent:** Main data, that can be fetched using SQL query
  * **Cluster:** Program & control data as dictionary, ABAP needed for fetching
  * **Pooled:** Program & control data from multiple pooled-table, ABAP needed for fetching

#### Some SAP modules

* MM - Material Management is one of the vital modules in SAP ERP software and MM application module supports the procurement and inventory functions occurring in day-to-day business operations.
* PP - Production Planning is the process of aligning demand with manufacturing capacity to create production and procurement schedules for finished products and component materials.
* SD - The SAP SD module is one of the primary ERP module developed by SAP. SAP service and distribution deals in better management of sales and customer distribution data and processes in organizations.
* FICO - The best configuration for internal as well as external accounting processes; represents FI (Financial Accounting) plus CO (Controlling). It is an important core module of ERP processes, wherein real time financial transactions are integrated with various parallel SAP modules for best results.

#### SAP data store

#### SAP Connector

* [Reference](https://docs.oracle.com/cd/E11882_01/owb.112/e10582/sap_integrate.htm#WBDOD90777)

#### Any APIs SAP expose?

#### References

* [PS: Introduction to SAP Integration for .NET Developers](https://app.pluralsight.com/library/courses/sap-integration-dotnet-developers/table-of-contents)
* [PS: SAP: Getting Started](https://app.pluralsight.com/library/courses/sap-getting-started/table-of-contents)
* [Oracle SAP Connector](https://docs.oracle.com/cd/E11882_01/owb.112/e10582/sap_integrate.htm)