---
layout: post
title: "How to trigger a Jenkins job remotely"
excerpt: "Triggering a remote Jenkins build job from command-line and build-script"
date: 2018-04-19
tags: [jenkins, ci, cd, cicd, automation, api, build]
categories: articles
image:
  feature: posts/misc/jenkins-2.png
comments: true
share: true
---

[Jenkins](https://jenkins.io/) is one of the oldest & most popular CI (Continuous Integration) servers, and many teams still configure and run their daily and ad-hoc build jobs with Jenkins.

The build workflows, or simply jobs, are set of runnable commands that generally executes a complete CI-build. To run the jobs, one can login to the Jenkins web interface, and click on _"build"_ button to execute a specific job.

#### Remotely triggering Jenkins jobs

But, many a times we want to trigger the build from outside, without manually interacting with the web interface. The most common use cases are triggering a build from a scheduler or another automated job. Here in this post we'll see how we can trigger a job on a remote Jenkins server.

To trigger a build, we simply need to `http POST` to the specific job API. POST to the `build` API for simple builds, and to `buildWithParameters` for the jobs that need some parameters. See the details in [Jenkins wiki](https://wiki.jenkins.io/display/JENKINS/Parameterized+Build).

```bash
# for simple builds
POST http://jenkins_server.com:port/job/job-name/build
# for builds with parameter
POST http://jenkins_server.com:port/job/job-name/buildWithParameters
```

Now, if you have authentication enabled, you need to authenticate your remote POST request. There are several options

* Either pass username/password for basic authentication
* And send a _"job-authentication-token"_ as uri parameter
* BUT, if you have _"Matrix based authentication"_ enabled, then _"job-authentication-token"_ does NOT work. You'll need to use the _"user-api-token"_ for authentication

Job-authentication-token is an authorization token per job. For that, first you have to enable remote trigger of jobs from _"Jenkins >> Project >> Job >> Configure >> Trigger builds remotely"_. Then set a _"job-authentication-token"_  here.

![Image](/images/posts/misc/jenkins-job-token.png)

This does not work if you have matrix based authentication enabled. Check at _"Jenkins >> Manage Jenkins >> Configure Global Security"_.

In that case you need to use an user-api-token. It is an authentication token for the user. With this token, the user can remotely access the APIs and execute tasks for which he/she has authorization. This can be found at _"User (right-top) >> Configure >> Show API Token"_.

![Image](/images/posts/misc/jenkins-api-key.png)

#### Triggering Jenkins jobs from command-line

Let's see how the above three options look in practice. Here we are using [curl](https://curl.haxx.se/) command-line tool for the http requests. See this for [quick reference of curl post](https://gist.github.com/subfuzion/08c5d85437d5d4f00e58).

```bash
# basic curl command syntax for POST
$ curl -d "data-as-query-parameters" -i -X POST http://username:password@site-uri/path
# Option 1 : with basic authentication & job-token
$ curl -d "token=job-token&anotherparam=value" -i -X POST http://username:Passw0rd@myjenkins:8080/job/my-job/buildWithParameters
# Option 2 : with api-token
$ curl -d "param=value" -i -X POST http://username:abcdef123456@myjenkins:8080/job/my-job/buildWithParameters
```

**Note:** Once you have triggered a build remotely in Jenkins, you'd probably want to know the ID of the build that was triggered. Unfortunately, that is not very straight-forward, and flaky. Jenkins returns a 'Location' header with a queue-ID. Then you [have to poll](https://stackoverflow.com/questions/24507262/retrieve-id-of-remotely-triggered-jenkins-job) the API with the queue-ID to see generated job-ID (RunID). Now, that doesn't work in all scenarios, as vaguely discussed [here](https://issues.jenkins-ci.org/browse/JENKINS-30317) and [here](https://issues.jenkins-ci.org/browse/JENKINS-31039).
{: .notice--warning}

#### Triggering Jenkins job from NAnt build script

In real-life scenarios, you'll probably want to trigger the job from a build system. Here, we'll see how to do that from a `NAnt` script. Basically, we'll execute the same `curl` commands through NAnt `exec` command.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<project name="RunTestsOnJenkins" default="run-jenkins">
  <!--details of JENKINS JOB, skip if not required-->
  <property name="jenkins.user" value="username" />
  <property name="jenkins.apikey" value="abcdefghij1234567890"/>
  <property name="jenkins.uri" value="http://172.0.0.1:1234" />
  <property name="jenkins.jobname" value="FunctionalTestSuite"/>
  <property name="jenkins.parameters" value="testenv=DEV"/>

  <!-- Trigger a JENKINS JOB, e.g. a functional test suite -->
  <target name="run-jenkins">
    <property name="jenkins.api" value="http://${jenkins.user}:${jenkins.apikey}@${jenkins.uri}/job/${jenkins.jobname}/buildWithParameters" />
    <exec program="${curl.path}" verbose="true">
      <arg line="-d ${jenkins.parameters}" />
      <arg line="-i -X POST ${jenkins.api}" />
    </exec>
  </target>
</project>
```

If you are not familiar with NAnt, read this post - **[Automating tasks with NAnt](/articles/automating-with-nant/)**.