---
layout: post
title: "Basics of CI/CD and creating CD pipelines in GoCD"
excerpt: "let's see..."
date: 2018-02-20
tags: [tech, ci, cd, gocd, build, pipelines]
categories: articles
comments: true
share: true
published: false
---

# Continuous Integration

Integration of source control, automated build, unit test etc. (maybe deployment)

**Continuous Deployment:** automatic deployment (continuously) from CI process. The deployment is ideally to production, but may be to test or staging as well. May NOT involve functional tests, performance tests etc., but should do if the deployment is to production.

**Continuous Delivery:** always ready for production deployment. There is always a stable, tested, deployable build ready. So, there is always proof that the latest version works as per expectations. - SO, continuous delivery is the process of making sure, every little change that is made to the code, is production ready. It is little broader than continuous development, and it has the choice of actually deploying or not on each commit.

For these stuffs to work:

1. Everything needs to be automate-able. So, apart from source code, the application configuration, database scripts, deployment scripts etc. all needs to be go source control and be machine readable. 
2. The development team must understand the essential operational process and know the operational consequences of each change in the application. So it needs high level of collaboration between dev and ops (DevOps fans would look happy now).

The whole essence of things here are:

1. Lowering the risk of the deployment/release, where many large entities are involved.
  1. Ideally, there should always be an options to find and fix issues on production in case that happens. 
  2. As a minimum, there should always be option to rollback to previous stable version as and when necessary.
2. Real progress in terms of developed, tested, deployable functionalities
3. Building the right thing. From continuous delivery, one can get quick and frequent user feedback and fix stuffs quickly when that goes away from expectation.

**DevOps** is a culture. It is how the different parts of the team interact & collaborate. Generally, there is no sense in having a "DevOps team"!

Usability testing: Users approve that they feel the app is easy to use.
User acceptance testing: Users can complete transactions using the app in near-production setup.

Every process must be automated and repeatable (e.g. provisioning a new hardware)
Keep logic out of CI/CD tools - use external scripts if required (and version control that)

### GoCD server

**Server:** Is the main system that coordinates all the work and provides the user interface. They provide agents the work to do, and does not do any user-work by themselves.

**Agent:** A system that actually does the user created works. Like, building, running unit tests etc. To enable running any command from GoCD, the corresponding client/application needs to be installed on the GoCD Agents.

**Material:** A material is something that can trigger a pipeline. Most common form of material are source control systems (like Git, SVN etc.). A pipeline can have one or more materials, even another pipeline or a stage in a pipeline as material.

**A pipeline has stages. Stages have jobs. Jobs have tasks.**

**Task:** Runs serially, within a job

* Smallest piece of a work, like a single command (e.g. Build)
* It's not necessary for that command to be simple, that one command can be something like a NAnt batch
* GoCD supports `Ant`, `NAnt`, `Rake` tasks by default. More are available as plugins. If not plugin, virtually anything can be run as "custom command"
* Task is not a specific element within GoCD system, it can by any type of command

**Job:** A collection of tasks

* Tasks in a job run sequentially. If one fails, job fails
* A job is run on one agent, so all tasks in a job run in same agent. Different jobs in same stage may run on different agents
* Different jobs in same stage always run in parallel, in any possible order (should be independent of each other). If not required, create one job per stage

**Stage:** collection of build job [the green/yellow/red bar at bottom of pipeline]

* They (different stages) run in sequence, either in same agent or different
* Stages can be like - build, unit test, publish report, pack, deploy etc.

**Pipeline:** Workflow for a build process, can run single or multiple commands - like a job in Jenkins [big white square cards on UI]

* Pipelines **can run in parallel**, even in different systems
* Ideally, you'll have one pipeline for each project
* There can be one or more pipelines in an *Environment**



	
Environment: development, staging etc.
Agents: users for an environment



Value Stream: The UI representation of the whole work-flow (multiple interacting pipelines, with fan-in and fan-out etc.)

Pipeline: name, group
Group: group of pipelines (?), e.g Mobile app pipelines for different runtime

#### GoCD

* [Installation](https://docs.gocd.org/current/installation/installing_go_server.html)
* [Bare basic setup](https://www.gocd.org/getting-started/part-1/)
* [Detailed documentation](https://docs.gocd.org/current/)
* [GoCD code](https://github.com/gocd/gocd/)