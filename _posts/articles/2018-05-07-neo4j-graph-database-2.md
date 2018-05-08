---
layout: post
title: "Introduction to Graph Database with Neo4j - Part II"
excerpt: "Working with neo4j Graph database & Cypher query"
date: 2018-05-07
tags: [tech, database, neo4j, graph, graphdatabase, cypher, query]
categories: articles
image:
  feature: posts/neo4j/graph_1.png
  credit: neo4j
  creditlink: https://neo4j.com/blog/other-graph-database-technologies/
comments: true
share: true
published: false
---

# neo4j

**Note:** This article is currently incomplete & in-progress
{: .notice--danger}

#### What is a "Cypher"?

Cypher - the neo4j query language

`Cypher` is a query language (not SQ) designed specifically for `neo4j` graph database, that is used to both manipulate & query data. It is generally easier to learn compared to SQL, also easier to read & write. Learn more about Cypher [here](https://neo4j.com/developer/cypher-query-language/). Some of the main features of Cypher are

* Queries follow the pattern - `MATCH <PATTERN> RETURN`, where MATCH & RETURN are keywords
* All the main query logic are in the pattern, where a pattern represents what data we are interested in
* It is a _declarative query language_. It specify _"what data to fetch"_ rather than _"how to fetch the data"_, unlike SQL
* It also uses bunch of SQL-like constructs e.g. WHERE, OREDR BY, MAX, MIN, EXISTS, .... etc.
* It supports many graph-specific constructs (which are not available by default in other database query languages) like - get _shortest distance_ between nodes, find all relationships between a pair of nodes, n-th level relationships etc.

Patterns are like ASCII-art_, i.e. representation of the actual graph with ASCII characters. 

MATCH (node:Label) RETURN node.property

MATCH (node1:Label1)-[:RELATION_NAME]->(node2:Label2)
WHERE node1.propertyA = {value}
RETURN node2.propertyA, node2.propertyB

**Note** Type of relationship is optional in a query. The relationship name is preceeded by a colon [:FREIND_OF] or [rel:FRIEND_OF]

#### Questions TBA

* How to store images or binary data
* How to create new database
* What about schema