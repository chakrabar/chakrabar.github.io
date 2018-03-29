---
layout: post
title: "Elasticsearch"
excerpt: "let's see..."
date: 2018-03-29
tags: [tech, fulltextsearch]
categories: articles
comments: true
share: true
published: false
---

# Elasticsearch

#### What is Elasticsearch?

* A wrapper over Lucene (written in Java), as a distributed system with http interface - that is very fast & scalable
* At heart, the actual work is done by Lucene which provides full-text search
* Works as a JSON-based full-text no-sql DB

Why? Very fast, result ordered by relevance, statistical trends in data
When? Blogs, Logs, unstructured document data (e.g. to provide text suggestion)

#### Architecture

* Data is stored as JSON documents
* Documents are stored as Index (similar to DB in relational DB)
* Indexes are sliced into SHARDs, which are nothing but chunk of data from an index - each shard is one instance of Lucene
* Shards are stored in multiple nodes (Elasticsearch servers), forming an easily scalable cluster

###### So, what is the relation between Lucene & Elasticsearch ?

Lucene is the core & heart of Elasticsearch. It is the engine that provides all the fast searching capabilities. What Elasticsearch does is, makes it a distributed (thus scalable) system. It also provides easy to use REST APIs and lot of very useful tools & plugins, and provides frameworks like ELK stack (Elasticsearch, Logstash & Kibana) etc.

So, if you just want to use the super-fast full-text-search capabilities on a single (not so big) data source, you can just use Lucene. If you want to make it a scalable distributed system, use other cool stuffs (like Kibana), add monitoring & visualization etc., go for Elasticsearch.

#### Installation

* Install on Windows as service
* In `config.yml`, change node and cluster name if required
* Now the APIs are available at `http://localhost:9200`
* Install plugins - Marvel as interactive GUI for monitoring clusters & query data (built on Kibana - data visualization tool)
  * Go to Elasticsearch install directory `C:\elasticsearch\bin` in command prompt
  * Type in `plugin.bat -i elasticsearch/marvel/latest`
  * Once installed, restart Elasticsearch service
  * The cluster dashboard is viewable at http://localhost:9200/_plugin/marvel 
  * The query interface is available at Sense dashboard http://localhost:9200/_plugin/marvel/sense/index.html





#### .NET clients

###### Elasticsearch .NET clients

* Elasticsearch.Net - low level, covering the basic elasticsearch APIs as methods
* NEST - high level wrapper over Elasticsearch.Net that provides more abstraction & strongly-typed DSL that maps 1 to 1 with the Elasticsearch query DSL

###### Lucene .NET clients

* [Lucene.net 3.0.3](https://blogs.apache.org/lucenenet/entry/lucene_net_3_0_3) - stable, for .NET Framework 4.0
* [Lucene.net 4.8.0](https://github.com/apache/lucenenet) - pre-release (as on March 2018), for .NET Core & .NET Framework 4.5

# Read, check & Remove...

* Lucene.net examples
* https://dzone.com/articles/how-add-search-capability-net
* https://www.codeproject.com/Articles/609980/Small-Lucene-NET-Demo-App
* https://www.codeproject.com/Articles/29755/Introducing-Lucene-Net
* [Elasticsearch with .NET](https://www.red-gate.com/simple-talk/dotnet/net-development/how-to-build-a-search-page-with-elasticsearch-and-net/)
* https://app.pluralsight.com/library/courses/elasticsearch-for-dotnet-developers/table-of-contents
* https://app.pluralsight.com/player?course=elasticsearch-for-dotnet-developers&author=james-toto&name=elasticsearch-for-dotnet-developers-m2&clip=1&mode=live

#### References

* [Setting up Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup.html)
* [Setting up Lucene]
* [Kibana](https://www.elastic.co/products/kibana)
* [Marvel](https://www.elastic.co/downloads/marvel), [Why Marvel](https://www.elastic.co/blog/building-marvel)
* [ELK stack](https://www.elastic.co/elk-stack)