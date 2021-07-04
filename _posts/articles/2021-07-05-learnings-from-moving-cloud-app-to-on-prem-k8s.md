---
layout: post
title: "Moving a cloud-native appliction to on-prem & the pains"
excerpt: "Learnings from trying to move a cloud-native enterprise application to on-prem prod deplyment, and the pains!"
date: 2021-07-05
tags: [dev, programming, onprem, deployment, cicd, devops, cloud, k8s]
categories: articles
comments: true
share: true
published: false
---


Issues during initial release

Incorrect format of RSA key pair breaking application auth

 

the-databaseDB connection failure because of auth mechanism mismatch

application client code has used socketio-the-database-adapter which is used to store socket.io session on the-database. It uses an old library mubsub that uses outdated the-database driver 2.x.x. This used SCRAM-SHA-1 as auth mechanism

But the-databaseDB on platform CLuster is modern, 4.x.x, which uses SCRAM-SHA-256 by default. So auth fails on platform. This works fine on the-database Atlas though.

The latest k8s enforces SHA-1 for security reasons. Atlas still supports SHA-1 for customers with older client. To fix this, the-database + platform did some fixes to allow SCRAM-SHA-1 on the platform k8s cluster

 

 

 

Moving app across accounts fail because of 401 error in Orchestrator API

 

Wait for the-databasedb to be ready before getting application up

 

 

High resource usage on platform Node making application crash

 

Current plan: [1] For develop, use higher tier VM or do not enable all application [2] Update document for customer

Clean up (hard coded the-database connection string)

 

More issues around business-application : mainly [1] failing to start [2] intermittent crash (last week of June - 1st week of July, 2021)

 

 

 

 

 

Probing, Investigation & issues:

Some possible causes of the problem & scenarios are discussed below:

Disk space issue - sum of logical PVC allocations > Actual disk available. As per PVC allocation, Longhorn needs at least 500GB per machine

Child process of main application Node processes crashing because of SIGKILL signal. platform: SIGTERM and SIGKILLS might be due to you livenessProbes, if the livenessProbe condition failed, SIGTERM will get generated first and after a certain duration, container will receive SIGKILL and pod gets terminated

There was an initial thought that application was crashing because of either [1] k8s killing it because of bad health, because some issue in livenss probe [2] the-database instance having some problem and making application nodejs server crash, as logged "connection dropped"

[1] probes were removed, still it crashed [2] the-database connection was moved to atlas, still it crashed [3] connection drop was possibly because of node server losing connection to the-database because it got crashed itself!

Investigated root cause:

application failed to start issue: application takes around 220-250 seconds to boot but initialDelaySeconds was set to only 30 seconds. To change it for readiness & liveness probes

application crashing intermittently: application have set the limits to 2GB (server & ws-server) and application isn't getting loaded under 2GB and is getting OOMKilled after some time. To change memory limit as well
