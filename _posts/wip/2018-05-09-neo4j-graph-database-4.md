---
layout: post
title: "Some common scenarios and considerations - Neo4j Part IV"
excerpt: "Common use cases and neo4j graph database for production"
date: 2018-05-09
tags: [tech, database, neo4j, graph, graphdatabase, cypher, query]
categories: articles
image:
  feature: posts/neo4j/graph_1.jpg
  credit: neo4j
  creditlink: https://neo4j.com/blog/other-graph-database-technologies/
comments: true
share: true
published: false
---

**Note:** This article is currently incomplete & in-progress. It'll be updated soon.
{: .notice--danger}

#### Importing RDBMS data

```fs
//csv with headers
LOAD CSV WITH HEADERS FROM
"file:///cars.csv"
AS csvLine
CREATE (:Car{id:toInteger(csvLine.Id), make:csvLine.Make, model:csvLine.Model})

//csv without headers
LOAD CSV FROM "https://www.quackit.com/neo4j/tutorial/genres.csv"
AS line
CREATE (:Genre { GenreId: line[0], Name: line[1]})
```

#### Some common questions

* Data types for properties - neo4j supports a set of primitive types for properties - boolean, byte, short, int, long, float, double, char, string and their arrays, like int[], char[] etc. Null is not allowed.
* Can it store images or binary data - yes, it can be stored as `boolean[]` data. But for large data, it is suggested to store them separately and just have the path in neo4j nodes, as that'll just make the data heavy without contributing anything to what neo4j does well.
* Type & range constraints on properties - nope
* How to create new database - the community edition supports only one database per instance. That means, to have a new database, you'll have to create a new instance with new port. Or, to create a new database in the same instance (for each database, it creates a separate directory inside _neo4j_root/data/databases_)
  * Stop the service
  * Go to file _neo4j_root/conf/neo4j.conf_
  * Uncomment & update value of _active_database_, e.g. `dbms.active_database=new.db`
  * Start the service
* What about schema - really, there is no concept of schema in neo4j. However, with use of `index`, `unique constraint`, `existance constraint` & `node key`, it can create [optional, partical schema](https://neo4j.com/docs/developer-manual/current/cypher/schema/).
* Query execution plans - Prepend the Cypher statement with `EXPLAIN` to see estimated execution plan without executing the statement. Use `PROFILE` to run query and see plan. See [query tuning](https://neo4j.com/docs/developer-manual/current/cypher/query-tuning/) for more details.

#### Data modelling

https://neo4j.com/blog/data-modeling-basics/
https://neo4j.com/blog/data-modeling-pitfalls/

Graph database book, chapter 3 - 5

#### Graph algorithms - naah

* Shortest path
* DFS
* BFS

Learning neo4j book, chapter 6

#### Sample weird queries

Vague example of CASE and UNWIND

```fsharp
MATCH (:Person {name: 'Robin Williams'})-[]-(x)
WITH collect(CASE WHEN exists(x.name) THEN x.name ELSE x.title END) AS Names
UNWIND Names as name //from list back to individuals
WITH DISTINCT name AS dn
RETURN 'Name - ' + dn
```

BAsic recommendation system

```fsharp
MATCH (p1:Person) -[:BOUGHT]-> (prod1:Product) <-[:BOUGHT]- (p2:Person) -[:BOUGHT]-> (prod2:Product),
  (p1) -[:FRIEND_OF*..2]- (p2),
  (prod1) -[:MADE_BY]-> (br:Brand) <-[:MADE_BY]- (prod2)
WHERE count(prod1) > 3 //some pre-decided threashold
AND NOT (p1 -[:BROUGHT]-> prod2)
RETURN p1.name AS TargetCustomer, p2.name AS ReferenceCustomer, br.name AS Brand,
  prod2.name AS Recommendedproduct, count(prod1) AS CommonProducts
ORDER BY CommonProducts DESC
```

WITH DISTINCT 1 AS ResetCardinality

#### Neo4j for production

* Installation - based on the type of data and scale, neo4j can be installed with different startegies.
  * Embedded - inside an application's process
  * On-premise - a full server/cluster installation of neo4j
  * Cloud - AWS or other cloud platforms with suitable VMs. Or on [Docker](https://neo4j.com/developer/docker/)

* Scaling neo4j - it can be scaled vertically & horizontally. The suggested [model](https://neo4j.com/blog/graphs-to-production-at-scale/) is to have traditional master-slave clustering with load balancer(s). One master handles all writes and as many slaves can be used for read, as all will have eventual data consistency (with a few milliseconds of polling lag) as same data is replicated across all instances. With this configuration, vertical scaling mostly helps with write speed whereas horizontal scaling helps in read redundancy. Beyond some threasholds, the enterprise edition will be required for scaling. Static sharding strategy can also be used, but sharding does not seem to be great with the possibility of all data being connected. So again, for data-size scaling, vertical scaling is the way to go.

* Backups & reporting - backups can be done online (without bringing down the DB) to local and remote instances. It supports full as well as incremental backups. For reporting, ideally a separate instance is maintained so that reporting can run without compromising production capacity.

#### Alternative query language - `Gremlin`

As an alternative to `Cypher`, there is **`Gremlin`** (Apache TinkerPop project) which is a general purpose query language for graph databases. It's actually a path-oriented data-flow DSL which can express graph traversal & mutation operations.

![image-right](/images/posts/neo4j/gremlins.png){: .pull-right}
Cypher is a descriptive query language, Cypher queries describe what data to fetch rather than how to get them. As the actual traversal logic is not described, the Cypher engine tries to provide most optimized traversal that it can. This might get limited and degrade in performance in some scenarios where there is a need to traverse huge graphs. On the other hand Gremlin provides more low-level access to the graph APIs, and one can device specific traversal algorithms with Gremlin, which is not possible with Cypher. Some references for Gremlin below.

* [Apache TinkerPop documentation](http://tinkerpop.apache.org/docs/current/reference/)
* [TinkerPop Gremlin 3 codebase](https://github.com/apache/tinkerpop)
* [Release-wise downloads](http://tinkerpop.apache.org/downloads.html)
* [Gremlin console](http://tinkerpop.apache.org/docs/current/tutorials/the-gremlin-console/)
* [Documentation reference](http://docs.janusgraph.org/latest/gremlin.html)

#### More

* [Extending neo4j](https://neo4j.com/docs/java-reference/current/#server-extending)
* [Sample projects](https://neo4j.com/developer/graphgist/)

#### C# client

* [codeproject](https://www.codeproject.com/Articles/1066378/Introduction-to-Graph-Databases-using-Neo-J-and-it)
* [docs](https://github.com/Readify/Neo4jClient/wiki)
* [examples](https://github.com/Readify/Neo4jClient/wiki/cypher-examples)
