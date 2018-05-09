---
layout: post
title: "Introduction to Graph Database with Neo4j - Part II"
excerpt: "Working with neo4j Graph database & Cypher query"
date: 2018-05-07
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

# neo4j

**Note:** This article is currently incomplete & in-progress
{: .notice--danger}

#### What is a "Cypher"?

Cypher - the neo4j query language

`Cypher` is a query language (not SQ) designed specifically for `neo4j` graph database, that is used to both manipulate & query data. It is generally easier to learn compared to SQL, also easier to read & write. Learn more about Cypher [here](https://neo4j.com/developer/cypher-query-language/). Some of the main features of Cypher are

* Queries follow the pattern - `MATCH <PATTERN> RETURN`, where MATCH & RETURN are keywords
* All the main query logic are in the pattern, where a pattern represents what data we are interested in
* It is a _declarative query language_. It specify _"what data to fetch"_ rather than _"how to fetch the data"_, unlike SQL
* It also uses bunch of SQL-like constructs e.g. WHERE, OREDR BY, MAX, MIN, EXISTS, IN, UNION, STARTS WITH etc.
* It supports many graph-specific constructs (which are not available by default in other database query languages) like - get _shortest distance_ between nodes, find all relationships between a pair of nodes, n-th level relationships etc.

Patterns are like ASCII-art_, i.e. representation of the actual graph with ASCII characters. 

MATCH (node:Label) RETURN node.property

MATCH (node1:Label1)-[:RELATION_NAME]->(node2:Label2)
WHERE node1.propertyA = {value}
RETURN node2.propertyA, node2.propertyB

**Note** Type of relationship is optional in a query. The relationship name is preceeded by a colon [:FREIND_OF] or [rel:FRIEND_OF]

* functions
* path
* range constraint
* type constraint
* modelling

#### Data creation

```fsharp
//create 1, 2 nodes. // for comment, end ; is optional
CREATE (n);
CREATE (n),(m)
//create a node with a label & property
CREATE (:Author{name:'Herge'})
//create a node with label, properties and return the created node
CREATE (b:Book{name:'Tintin in Tibet', languages:['English', 'Hindi', 'French', 'Turkish'], isbn:'aa001', year:1960, isPdf:true})
RETURN b
//create two nodes WITH a relationship
CREATE path = (:Book{name:'Chacha Chowdhry 4', languages:['Hindi', 'Bengali', 'Marathi'], isbn:'aa002', year:1998, isPdf:false}) -[:IS_WRITTEN_BY]-> (:Author{name:'Pran'})
RETURN path
//create a relationship between two existing nodes
MATCH (a:Author {name: 'Herge'}), (b:Book {name: 'Tintin in Tibet'})
CREATE (b) -[:IS_WRITTEN_BY]-> (a)
//create a node with two labels
Create (tintin:Character:Fictional {name: 'Tintin'}) RETURN tintin
//create a relationship between two existing nodes, with a property
MATCH (b:Book), (c:Character)
WHERE b.name = 'Tintin in Tibet'
AND c.name = 'Tintin'
CREATE path = (b) -[:HAS_CHARACTER {type:'Main'}]-> (c)
RETURN path
```

#### Querying

Basic queries to fetch data

```fsharp
//get all nodes
MATCH (n) RETURN n;
//the database creates an interal <id> which is incremental integer starting at 0. It is unique across all nodes and cannot be customized
//....??
MATCH (n)
WHERE id(n) <= 100
RETURN id(n) AS Id, n.name AS Name
ORDER BY id(n)
//get all nodes with Book label, upto 10 items
MATCH (n:Book) RETURN n LIMIT 10
//get all nodes, that HAS isbn property AND is not empty
MATCH (n) WHERE n.isbn <> '' RETURN n
//get all nodes that does NOT have isbn property
MATCH (n) WHERE NOT exists(n.isbn) RETURN n
```

For the rest of the quesries, we'll be using the sample Movies database [available on neo4j site](https://neo4j.com/developer/movie-database/). This also comes with the default installation. Follow the instructions from the initial screen in the web interface. Jump into code -> Write code -> Movie Graph -> Create a graph

![Image](/images/posts/neo4j/movie.png)

```fsharp
//get uppercase names of all person that acted in movies, sort, skip first 10 and show 5 (pagination)
MATCH (p:Person) -[:ACTED_IN]- (Movie)
RETURN toUpper(p.name) AS Actor
ORDER BY p.name
SKIP 10 LIMIT 5

//get all movies and the crew (actors, directors, producers)
MATCH (p:Person) -[:ACTED_IN|:DIRECTED|:PRODUCED]-> (m:Movie)
Return m.title AS Movie, collect(p.name) AS Crew

//Return all person names, movie titles that acted in same movies with Tom Hanks
MATCH (Person{name:'Tom Hanks'})-[:ACTED_IN]->(m:Movie)<-[:ACTED_IN]-(p:Person)
RETURN p.name, m.title

//Return co-actors, no. of actors acted in same movies with Tom Cruise, grouped (implicit) by movie
MATCH
(Person{name:'Tom Cruise'})-[:ACTED_IN]->(m:Movie)<-[:ACTED_IN]-(p:Person)
RETURN m.title AS Movie, collect(p.name) AS CoActors, count(p) + 1 AS TotalCast

//UNION - get directors of all movies by Charlize Theron/Demi Moore. Union All to include duplicates
MATCH (Person {name:'Charlize Theron'}) -[:ACTED_IN]-> (m:Movie) <-[:DIRECTED]- (p:Person)
Return m.title AS Movie, p.name AS Director, 'Charlize Theron' AS Actress
UNION
MATCH (Person {name:'Demi Moore'}) -[:ACTED_IN]-> (m:Movie) <-[:DIRECTED]- (p:Person)
Return m.title AS Movie, p.name AS Director, 'Demi Moore' AS Actress

//Get all nodes that are directly related (in any direction) to Robin Williams
MATCH (Person{name:'Robin Williams'})-[]-(x) RETURN x

//Get all nodes that are related (in any direction) to Al Pacino, within 5 levels/hops
MATCH (:Person{name:'Al Pacino'})-[*1..5]-(x) RETURN x
```

When finding relationships with multiple hops, neo4j avoids linking back to same node. So that it is not like _(node1) -[:some_relation]- (node2) -[:some_relation]- (node1)_. Same nodes are also not included in result set, but can be added explicitly. Following is a sample query and the result set.

```fsharp
MATCH (p:Person{name:'Al Pacino'})-[*1..3]-(x) RETURN x, p
```

![Image](/images/posts/neo4j/al-pacino.png)

#### Data definition/schema

```fsharp
//create an index - does NOT make the property required/unique
CREATE INDEX ON :Book(name)
//DROP INDEX ON :Book(name) ...??

//show all indexes & constraints across database
:schema
//index hint, generally neo4j will figure out anyway
MATCH (b:Book)
USING INDEX b:Book(name)
WHERE b.name STARTS WITH 'Tintin'
RETURN b
//create unique constraint
CREATE CONSTRAINT ON (b:Book)
ASSERT b.isbn IS UNIQUE
//show all constraints
CALL db.constraints

//following 2 features are available *ONLY ON Enterprise Edition*
//existence constraint - property must exist
CREATE CONSTRAINT ON (c:Character) ASSERT exists(c.Name)
//NODE KEY - must exist & be unique
//can also be combination of multiple properties (n.prop1, .prop2)
CREATE CONSTRAINT ON (n:Character)
ASSERT (n.name) IS NODE KEY
```

#### Deleting data

```fsharp
//DELETE ALL NODES, if there are no relationships
MATCH (n) DELETE n
//DELETE ALL NODES & RELATIONSHIPS, even if there are relationships
MATCH (n) DETACH DELETE n
//match and delete a node
MATCH (a:Author {name: 'Pran'}) DETACH DELETE a
//delete all characters, use DETACH to remove relationships too
MATCH (x:Character) DELETE x
//delete all characters without name
MATCH (c:Character)
WHERE NOT exists(c.name)
DELETE c
//delete all relationships of a node. DELETE c,r to delete node too
MATCH (c:Character {name: 'Tintin'}) -[r]- ()
DELETE r
//delete a whole path
MATCH path = (:Author) <-[]- (b:Book {isbn: 'aa002'})
DELETE path
```

#### Updating data

```fsharp

```

#### References

* [Basic query video tutorials](https://neo4j.com/blog/neo4j-video-tutorials/)
* [Cypher documentation](https://neo4j.com/docs/developer-manual/current/cypher/)
* [Cypher syntaxes](https://neo4j.com/docs/developer-manual/current/cypher/syntax/)
* [Cypher reference sheet](https://neo4j.com/docs/cypher-refcard/current/)