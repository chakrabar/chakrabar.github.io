---
layout: post
title: "Practical graph algorithms with grapgh DB - Neo4j Part III"
excerpt: "Common use cases and neo4j graph database for production"
date: 2018-05-08
tags: [tech, database, neo4j, graph, graphdatabase, cypher, query]
categories: articles
image:
  feature: posts/neo4j/graph_header.jpg
comments: true
share: true
published: true
---

**Note:** This article is currently incomplete & in-progress. It'll be updated soon.
{: .notice--danger}

https://neo4j.com/blog/graph-search-algorithm-basics/

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

You can download the sample [cities.csv](/images/posts/neo4j/cities.csv) and [paths.csv](/images/posts/neo4j/paths.csv) files. Here is the actual graph creates

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
MATCH (source:City {name: "Amritsar"}), (destination:City {name: "Hyderabad"}),
  path = (source) -[road:LINK*]- (destination)
WITH path, REDUCE (sum = 0, r IN road | sum + r.distance) AS dist
ORDER BY dist
LIMIT 1
RETURN path

MATCH path = (:City {name: "Amritsar"}) -[road:LINK*]- (:City {name: "Hyderabad"})
WITH path, REDUCE (sum = 0, r IN road | sum + r.distance) AS dist
ORDER BY dist
LIMIT 1
RETURN REDUCE (cities = head(nodes(path)).name, n IN tail(nodes(path)) | cities + '->' + n.name) AS Route,
  REDUCE (rd = head(relationships(path)).name, r IN tail(relationships(path)) | rd + ' > ' + r.name) AS Roads,
  dist + ' km' AS Distance
```

It produces the following results:
<br />**Route:** Amritsar->Lucknow->Kolkata->Bhubaneswar->Hyderabad
<br />**Roads:** NH-1 > NH-4 > NH-9 > NH-10
<br />**Distance:** 560 km
{: .notice--success}

lol - with road blocks

```fsharp
MATCH () -[r:LINK]-> ()
WHERE r.name IN ['NH-3', 'NH-4']
SET r.isBlocked = true

MATCH path = (:City {name: "Amritsar"}) -[road:LINK*]- (:City {name: "Hyderabad"})
WHERE ALL (r IN road WHERE NOT exists(r.isBlocked)) //all roads are non-blocked
WITH path, REDUCE (sum = 0, r IN road | sum + r.distance) AS dist
ORDER BY dist
LIMIT 1
RETURN REDUCE (cities = head(nodes(path)).name, n IN tail(nodes(path)) | cities + '->' + n.name) AS Route,
  REDUCE (rd = head(relationships(path)).name, r IN tail(relationships(path)) | rd + ' > ' + r.name) AS Roads,
  dist + ' km' AS Distance
```

fastest route

```fsharp
//roads with average speed
LOAD CSV FROM "file:///paths2.csv"
AS line
MATCH (c1:City { id: toInteger(line[1]) }),
  (c2:City { id: toInteger(line[2]) })
CREATE path = (c1) -[:LINK {avgSpeed: toFloat(line[4]), name:'NH-' + line[0], distance: toInteger(line[3]) }]-> (c2)

//fastest route
MATCH path = (:City {name: "Amritsar"}) -[road:LINK*]- (:City {name: "Hyderabad"})
WHERE ALL (r IN road WHERE NOT EXISTS (r.isBlocked))
WITH path, REDUCE (sum = 0, r IN road | sum + (r.distance / r.avgSpeed)) AS time
ORDER BY time
LIMIT 1
RETURN REDUCE (cities = head(nodes(path)).name, n IN tail(nodes(path)) | cities + '->' + n.name) AS Route,
  REDUCE (rd = head(relationships(path)).name, r IN tail(relationships(path)) | rd + ' > ' + r.name) AS Roads,
  toString((round(time * 100))/100) + ' hr' AS TotalTime
```

It produces the following results:
<br />**Route:** Amritsar->Surat->Delhi->Bhubaneswar->Hyderabad
<br />**Roads:** NH-2 > NH-6 > NH-8 > NH-10
<br />**Distance:** 600 km
{: .notice--success}