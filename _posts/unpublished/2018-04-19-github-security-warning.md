---
layout: post
title: "We found a potential security vulnerability in one of your dependencies."
excerpt: "Desktop apps with HTML, CSS & JavaScript"
date: 2018-04-19
tags: [tech, apps, desktop, electron, html, css, javascript]
categories: articles
published: false
---

# A dependency defined in ./Gemfile.lock has known security vulnerabilities and should be updated.

Notice

```
Known vulnerability found
CVE-2017-18258
Moderate severity
The xz_head function in xzlib.c in libxml2 before 2.9.6 allows remote attackers to cause a denial of service (memory ...

Gemfile.lock update suggested:
nokogiri ~> 1.8.2
Always verify the validity and compatibility of suggestions with your codebase.
```

![Image](/images/posts/security/Warning.png)
![Image](/images/posts/security/Warning-2.png)
![Image](/images/posts/security/Warning-3.png)

[Details](https://nvd.nist.gov/vuln/detail/CVE-2017-18258)

## Currently in my `Gemfile.lock`

`nokogiri (>= 1.4)` line 79
`nokogiri (1.8.1-x64-mingw32)` line 199

## Updated to

`nokogiri (>= 1.4)` line 79
`nokogiri (1.8.2-x64-mingw32)` line 199

> Seems to be working fine! Will check back later.