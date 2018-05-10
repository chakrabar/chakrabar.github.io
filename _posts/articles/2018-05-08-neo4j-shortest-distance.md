---
layout: post
title: "Calculating shortest distance in graph with Neo4j"
excerpt: "Common use cases and neo4j graph database for production"
date: 2018-05-08
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

```fsharp
//csv with headers
LOAD CSV WITH HEADERS FROM "file:///cities.csv"
AS line
CREATE (:City { id: toInteger(line.Id), name: line.City })

//csv without headers
LOAD CSV FROM "file:///paths.csv"
AS line
MATCH (c1:City { id: toInteger(line[1]) }),
  (c2:City { id: toInteger(line[2]) })
CREATE path = (c1) -[:LINK {name:'NH-' + line[0], distance: toInteger(line[3]) }]-> (c2)
```

Here is the actual graph creates

![Image](/images/posts/neo4j/cities.png)

Queries

```fsharp
//in-built shortest-distance = minimum hops
MATCH (c1:City {name: "Amritsar"}),
  (c2:City {name: "Hyderabad"}),
  path = shortestpath((c1)-[:LINK*]-(c2))
RETURN path
```

It returns nodes _"Amritsar --> Bengaluru --> Huderabad"_, with a total distance of 810km.

For a real world _shortest path_ we'll need some calculations.

```fsharp
//actual shortest-distance
MATCH path = (c1:City {name: "Amritsar"}) -[road:LINK*]- (c2:City {name: "Hyderabad"})
WITH path, REDUCE (sum = 0, x IN road | sum + x.distance) AS dist
WITH path, MIN(dist) AS d //not required
ORDER BY d
LIMIT 1
RETURN REDUCE (p = head(nodes(path)).name, n IN tail(nodes(path)) | p + '->' + n.name) AS Route,
  REDUCE (p = head(relationships(path)).name, n IN tail(relationships(path)) | p + ' > ' + n.name) AS Map,
  d + '0 km' AS Distance
```

It produces the following results:
**Route:** Amritsar->Lucknow->Kolkata->Bhubaneswar->Hyderabad
**Map:** NH-1 > NH-4 > NH-9 > NH-10
**Distance:** 560km
{: .notice--success}