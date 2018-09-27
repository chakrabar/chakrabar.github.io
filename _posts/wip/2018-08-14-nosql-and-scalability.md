---
layout: post
title: "NoSQL databases, partitioning & distributed systems"
excerpt: "To bo broken down into multiple pieces - [1] NoSQL [2] Distributed systems [3] CouchDB"
date: 2018-08-14
tags: [database, sql, nosql, coucdb, distributedsystems, partitioning]
categories: articles
comments: true
share: true
published: true
---

# NoSQL databases

...[from](https://www.linkedin.com/learning/learning-nosql-databases)

What is a No-SQL database?

* Any type of database that does NOT use SQL as the query language.
* Almost all NoSQL databases do NOT store data in table structures as in relational databases.
* Many of them store data directly or in some variant of JSON format. But not necessarily true for all nosql databases.
* Generally they are very flexible in format. Need not have a pre-defined structure beforehand and can be defined as part of data insertion. Generally they don't even enforce them at field levels, meaning that, similar elements can have different type of data in same field, and that can change over time too.

## Some common type of NoSQL databases

1. **Document store** : Objects are stored as documents, which are structured as JSON or XML.
   1. Usually organized into _collections_ and _databases_. Collections contain similar documents, but they need not have same fields.
   2. Generally each document has a unique key
   3. It supports querying the document by fields
2. **Key-value store** : Databases are collections of key-value pairs (like dictionaries in programming)
   1. Each record has a key and a value attached to it
   2. Drawback - the value is opaque to the system, cannot query by fields
   3. Some of them let's you have more than one key, bypassing part of the above problem
   4. Frequently used for caching
3. **Big table/wide column** : Big Table name comes from Google's implementation named Big Table. Designed for large number of columns.
   1. Tabular but _"column oriented"_
   2. Row has columns, but each row can have different set of columns!
   3. Generally structure is somewhat like this, row: [{name: "Name", value: "Arghya"}, {name: "skills", value: ["Database", "Design", "Architecture"]}]
   4. Rows can be versioned
4. **Graph database** : Data are stored as actual graphs with nodes & connections
   1. Data are represented and stored as a set of connected nodes
   2. Nodes have fields in form of key-value pairs, stores the actual data
   3. Nodes are connected to each other with directed relationships
   4. Optionally, relationships can also have fields or properties
   5. Very efficient in hopping through nodes and finding out arbitrary relationships
5. **Object databases** : Programming objects are directly stored as-is. They can be connected through pointers

## Some benefits

* <u>Schema-less</u>, so no worry to update structures and update existing data if data needs change
* Can be used as <u>caching layer</u>, as (in most cases) it can quickly fetch all related data without complex queries
* Many of them can store <u>large binary data</u> directly, without need for disk storage for files
* Some NoSQL databases have features that allows to directly serve full web pages (including js & css) directly from database

## Denormalized data in NoSQL

Most of the NoSQL databases store data in a denormalized manner. That means, same data gets duplicated across many records.

For example, let's say we have a document store, where we have collections (similar type of documents) - employee, department, office etc. Now, the employee document has manager information, who is also an employee, and has a department as well. Now, in relational databases, the employee record would have FK relationships to the department table, and a link back to same employee table for the manager. But actual data is not duplicated.

In document store, the employee records would have some details of the manager (e.g. name, email) and the department (e.g. name) in the employee record itself. So, these kind of data gets duplicated across many records. This helps keep all data related to employee in one place, and makes the read very fast.

But, **_isn't denormalized data bad??_**

In traditional relational world, it is considered bad, mainly because of two reasons

1. Increased storage requirement
2. Difficulty in updating data

Now, if we look in terms of NoSQL databases

1. The first one is not an issue. In today's world, storage is plenty. So having few hundred KBs of redundant data per record is not much of a problem, since it aids to the much faster read, which is much more important than storage space in most cases.
2. Data update or sync is an issue, and is not handled inherently by the NoSQL databases. Some of the common approaches are
   1. Still keep a master copy of all those repeated data (e.g. a department document collection), and make changes to that in case anything needs update. The same data (e.g. department name in employee document) in other documents could be refreshed periodically (e.g. nightly background jobs) from that master data. This retains the high performance at a loss of consistency.
   2. Keep both the collections (employee & department), but do not consider anyone master as such. When any update happens, update in all the places. This makes the system consistent at a cost of write efficiency.
   3. Do not store real data, store references to them (e.g. employee record having department document id). This makes them much closer to SQL databases. This preserves the high write speed as well as consistency, but loses on read speed as you need to query related documents to get complete data.

## SQL vs NoSQL databases

1. SQL databases (or, more correctly, relational databases) use SQL (Structured Query Language) to query data. NoSQL databases don't. Some of them have their own Query Language, while others depend on application logic to query.
2. Relational databases lacks in terms of performance & scalability when compared with NoSQL databases. NoSQL databases generally provide much faster read-write and scales naturally. But on the other hand, relational databases provide lot more functionalities (data structures, joins, SPs, views, functions, transactions, triggers, DB level bulk updates etc.) that NoSQL counterparts.
3. NoSQL is flat, there is generally no relation between two pieces of data. So, there is no join. If a join-like functionality is required, that is done through application code.
4. SQL supports better (ACID type) transactions. Again, for NoSQL, that needs to be handled through application logic.
5. NoSQL databases so much better in terms of scalability. Mainly since data pieces are not related, they can be easily distributed across servers.
6. SQL databases are NOT good at partitioning. Sharding can be done, but generally needs lot of custom work. Also, the schema needs to be created and maintained in each server node. Re-sharding is very difficult. Things like identity columns (auto incremented integer key values) are difficult to maintain. This is rarely used in NoSQL, which prefers GUID like keys.
7. SQL does better in terms of security (generally) and encryption. So for things like confidential data, health records etc. relational databases are preferred over NoSQL (they generally store data as-is).
8. Data is de-normalized in NoSQL databases. Space is not considered one of the issues.
9. NoSQL provides no and/or very flexible schema. So schema updates are totally hassle-free.
10. SQL is CA (Consistent & Available). NoSQL are partition-tolerant. Sharding comes naturally.
11. Many of the NoSQL databases directly support caching. Since all data are in one place (document, key-value), it naturally works as a cache.
12. Generally reads are much faster in NoSQL because of the lack of complex queries and joins. Also they support distributed querying naturally, as different pieces of data can be fetched independently of each other in a distributed manner.

# Distributed (database) systems

Why distribute? Means, why use more than one machine?

1. The data storage needs might exceed one machine
2. Redundancy for reliability, if one machine goes down there are alternatives
3. To handle heavy load (too many requests), put multiple copies behind a load-balancer

## Distributed system fallacies

It is a collection of assumptions that programmers new to distributed systems generally assume, and thus can lead to unexpected failure of the system. It was originally authored by Peter Deutsch of Sun Microsystem in 1994, and later appended by James Gosling in 1997.

Things have changed in last 20 years, and some of the issues are not much a big concern these days. With modern systems and networks, massive cloud systems, abundant storage, improved bandwidth, new tools & technology, some of them do not hold good anymore (e.g. bandwidth, latency etc.). But still almost all of them should be taken into consideration while designing a distributed system.

1. The network is reliable - broken wire to wireless connections are real
2. Latency is zero - even with super-fast connections, the latency (time to move an information from one place in network to other), is still not zero. And depends on the type of network, physical distance, network hops etc.
3. Bandwidth is infinite - we have huge bandwidth (the amount of data that can be transferred in unit time), but still they are not infinite. Also not all systems are on GBPS connections, consider mobile devices, cars, wearables etc.
4. The network is secure - nothing is ever 100% secure, but well designed system can be very secure
5. Topology doesn't change - with every addition or removal of systems in a network, the topology changes. But well designed system can withstand those changes. Systems like large clouds, are inherently immune to these issues
6. There is one administrator - there are generally more than one admin, and they all can change settings or configs
7. Transport cost is zero - transport takes bandwidth and electricity, which in turn costs money. Even in some cloud systems charge for moving data in and out of the cloud
8. The network is homogeneous - there are so many different types of devices, platforms and networks. System designed aiming at one, may not work for others

## Consistency: ACID vs BASE

<u>**ACID**</u> is an acronym that defined the desired behaviors of a reliable database system. It is a set of properties that is desirable from a database transaction.

* **Atomic**: Either all or none of the changes happen. There is no partial transactions, either all the changes happen or the whole thing is rolled back to initial state.
* **Consistent**: The system moves from one consistent state to another, as a result of the transaction. The transactions always results in a valid state of the system. It also means, all reads get the latest updated data from the most recent transactions OR in other words, readers should not get different conflicting states of data.
* **Isolated**: Multiple transactions happening concurrently do not interfere with each other. The result of concurrent transactions must be same if they occurred sequentially, and one transaction must not be affected by another transaction's intermediate state or it being rolled back. This is generally achieved through different levels of locking, sometime making one transaction to wait for another one to complete.
* **Durable**: Once a transaction is complete, the results stay even in case of system failures or restarts.

Almost all relational databases and centralized (or single) database systems exhibit these properties which makes them pretty reliable systems. On the other hand, it's difficult to almost impossible to achieve all these in a <u>distributed database systems</u>. The main reasons being, it takes some time to have data updated across different nodes over network, some parts of the system might become unavailable temporarily or permanently because of various reasons.

In distributed databases systems, trading some consistency for availability can help a lot on increased scalability. Many a times distributed systems are built around a bunch of concepts, which loosely forms the acronym <u>**BASE**</u> (though it does convey the ideas, it's little made as the opposite of ACID). The idea is to have consistency over time in an optimistic way, while have a continuous acceptable changing state.

* **BA: Basically Available** - means all read requests get a response, but that might not be the _"most recent"_ state
* **S(S): Soft State** - the changes in the system flows through the network, gradually updating the records. So, the state of data can change even without an external cause. This means the data is always in a soft state in continuous transaction scenarios.
* **E(C): Eventual Consistency** - all the nodes/servers will have the most recent data given there are no more updates being made externally

It basically says the system will have correct state eventually rather that immediately after each transaction. For example, it might not be a good solution to a scenario like bank transfer (balance has to be matched immediately so that next transaction can work properly), but it does fine in may scenarios where immediate update is not a necessity. e.g. bank account statement which might be updated at the end of the day, or call details from one's mobile phone.

## Database partitioning

When the data size grows very large, it needs to be broken down in pieces or partitions to make them more manageable in terms of storing and retrieving data. This process of breaking data store into pieces is known as partitioning.

* Partitioning splits data into multiple servers (i.e. nodes in a cluster)
* **Partitioning helps in**
  * Managing storage space
  * Performance or read & write
  * Keeping the DB functional even if a part becomes too busy or unavailable
* If the database is small, it might be good idea not to partition, as that increases complexity
* Most of the NoSQL databases are built with partitioning built into them
  
### Types of partitioning

SQL and NoSQL databases, can be partitioned mainly in two ways - **vertical & horizontal**

* **Horizontal partitioning** - or, more commonly known as **<u>Sharding</u>**. In SQL databases, different (set of) rows are kept in different nodes. NoSQL databases like key-value store or document store uses this strategy
* **Vertical partitioning** - in SQL databases, different columns or set of columns are stored in different nodes. NoSQL database like wide column stores use this strategy

**Note**: In relational database, one additional issue with sharding is, the schema needs to be maintained on each server nodes.

### Common partitioning strategies

1. **Memory cache:** Part of data (most frequently used) is kept in memory. A common strategy (e.g. Memcached key-value store) is to distribute that in-memory load over a number of machines on a network, So all of them have all the data, and keeps a part of it in-memory cache. When call to a node fails (cache-miss), the whole system rehashes the keys to distribute in-memory load among available nodes. If the node comes back, it is done again. The total set of nodes is tracked with configuration.
2. **Clustering:** Generally has same data on multiple servers behind load balancers, with active replication/mirroring. Helps in reliability and load-balancing. Generally used when the database itself doesn't support partitioning natively.
3. **Separating read-write:** One or small set of dedicated servers are used for write. Works in master-slave mode where master handles write, slaves serve read requests. Those data are replicated to all other nodes that can serve any read requests. Replication can be synchronous or asynchronous, while synchronous ones gives safer/faster consistency they add write lags. In case of master failure, common strategy is to elect node with most recent data as master.
4. **Sharding:** The whole data is chunked into multiple servers who read-write it's own portion of data. Further replication can be added for reliability. There has to be a dedicated component with a mapping-service, that maps incoming request to correct node. Based on implementation, the mapping can be static or dynamic. While some RDBMS supports sharding, sharding doesn't support joins across nodes, which might not work well in RDBMS. In cases like that, data has to be fetched and joined in-memory.

Generally, the data to node mapping is done by a method like **_simple hashing_**, so that it's easy to calculate/find which data resides where (_All these methods do NOT support dynamic scaling_)

* One way can be ranges, like by starting character of a field as A-L, M-Q, R-Z
* Another way can be categories, like in a book database they can be partitioned as fiction, biography, science etc.
* Or by _simple hashing_ like in general hash tables : **hash a key field, then index_of_node = hash % number_of_nodes**

**Note:** Simple hashing works for systems like _memory caching_, where implicit re-hashing is possible as all nodes have all data. But it does not work for persistent storage systems, because the whole dataset has to be re-shuffled as a result of re-hashing. They need some dynamic method like _consistent hashing_.
{: .notice--info}

## Consistent hashing

Consistent hashing is a strategy that is applied to load balancing, distributed caching & other use cases where it can map entities (data, request etc.) to a group of servers, where the group can change dynamically with minimal cost. Also, this need not have dedicated servers for the mapping! It is comparatively a newer concept, first introduced in 1997.

Let's say we have `N` servers. We have some data that we have to distribute among them. The strategy is easier to visualize with a ring, around which the servers are placed. To place them, we use a hashing algorithm and let's say it has hashing range from `R`. The circle represents R points from `0 to R-1`, around it. Now, to place a server, we simply hash it's id (or name or IP or some unique key), and place it at position `hash(id) % R` on the circle.

Now, to place a data, we hash it's key with the same hash function (or a different function, still works) and again place it on `hash_1(key) % R` position on the circle. This way we place all the server and data around the circle.

Now to determine which data would be served by which server: we simply walk around the circle in clockwise direction from the data, and the first server we encounter is our destination.

![Image](/images/posts/misc/consistent-hashing.png)

In the above figure blue ones are data and red ones are nodes. So (left) data 2 is stored in Node-B, 3 in C and 1 & 4 in A.

If the node configuration changes (e.g. Node C left and D was added on the right), they continue in the same manner. Now, data 1 is stored in A, 2 in B, 3 & 4 in D. So, practically, 3 & 4 needs to be copied to new server D. Rest remains the same. _In big enough cluster, the percentage movement is minimal_.

Practically, a request for a data can arrive at any node, and knowing the current set of servers in the cluster (and the requested data key), that node can re-direct the request to the correct server.

### Necessary improvements

Some of the possible problems with this approach include

1. If the hashing function is not that good, the servers/data might get unevenly distributed putting more load on some of the servers
2. While the configuration changes (servers leaving or joining the cluster), only the neighbouring servers need to shuffle data, which again creates uneven load

The simple answer to this problem is **replicas**. Each new server should come with replicas, where the replica can be a physical server or a **_virtual node_**. What is a _virtual replica_? Simply means, use multiple hashing functions rather than a single one to distribute the nodes. So, each server gets placed X times on the circle, when there are X hashing functions used. Now, the data shuffling load is much more balanced across the cluster. If the correct number of hashing functions are used, the load can get distributed perfectly.

### Node exits

When a node leaves the system, there are few problems

1. The data from the node is lost
2. If the node leaves abruptly (e.g. a system crash), how do others get to know that

For the first prpblem, the common solution approach is to have data replicas. That means data from each node needs to be replicated on multiple _physical servers_. So, with a _replication factor_ of r = 3 (r &lt; n), each data is stored in next 3 (r) physical nodes. So, if one server goes down, there are still r - 1 copies of data available.

When a node leaves the cluster, it's possible that it did not notify others. So, how does the system get to knoe that? This is generally achieved through the _**"Gossip protocol"**_. In this scheme, all the nodes keep talking to other nodes, at least the few neighbours if not all. So, when ever a neighbor stops responding, they get to know. They now re-distribute the share of that node (it's own data and the replicas it was responsible for).

...arghya

## CAP Theorem

The basic set of desirable characteristics of any <u>distributed system</u> (including database).

* <u>**Consistency**</u> - Every read gets the most recent & updated data, irrespective of where they read it from
* <u>**Availability**</u> - Every request gets a (non-error) response. 100% availability means there's no time when a request is not served
* <u>**Partition-tolerance**</u> - Requests still receive (non-error) response, even when some parts of the system is not able to function properly, or are in the process of being moved in or out of the distributed system. That can also mean pure partitions like node A & B can communicate to each other, same with nodes C & D. But A, B cannot communicate to C, D. But ideally the whole system should still work as expected.

The original author of the theorem **[Eric Brewer](https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed)** stated in 1999 that, in a distributed system, it's impossible to to perfect or improve all three of the above at the same time. So, the designers needs to decide which 2 of the 3 are more important and design accordingly. So, you can have either of

1. AP or available, partition tolerant system
2. CP or consistent, partition tolerant system
3. CA or consistent, available system

#### Types of distributed database systems

What is provided by a database system depends on the specific database.

* **Relational or SQL databases generally are CA**. They provide high consistency and availability. But to partition them properly, generally custom code and/or infrastructure are required.
* **NoSQL databases are almost all partition-tolerant**. Most of them has availability over consistency, they follow AP.
* **Many NoSQL databases provide <u>eventual consistency</u>**. That means, all nodes will provide the correct or most recent data eventually over time.
* For example, **MongoDB is CP**. It writes to a single master, which internally figures out on which node to actually write the data..... 
* **Cassandra is AP**. More nodes can be added without hurting the availability.
* **Redis is CP**, like MongoDB. It uses master slave replication. Redis is a key value store where value can be string, list, sets (non repeating list) and hashes (further nested key-value pairs)
* Redis is not completely opaque, and provides some basic schema/data structures to work on. On the other hand, **Memcached** does not provide such data structures and you work with the top level key-value data.

**Note**: Unlike CouchDB, MongoDB & Redis do not work on HTTP. They have dedicated clients written for different tech-stacks. Through them, the application directly talks to the database driver.

#### Case study: Why is MongoDB generally CP

### Further reading

* [ACID](https://en.wikipedia.org/wiki/ACID_(computer_science)), [BASE](https://queue.acm.org/detail.cfm?id=1394128), and some comparison [here](https://www.johndcook.com/blog/2009/07/06/brewer-cap-theorem-base/), [here](https://mongodbforabsolutebeginners.blogspot.com/2016/06/acid-and-cap-theroems.html) et al.
* [Eventual Consistency](https://en.wikipedia.org/wiki/Eventual_consistency), [Linearizability](https://en.wikipedia.org/wiki/Linearizability)
* [Please stop calling databases CP or AP](https://martin.kleppmann.com/2015/05/11/please-stop-calling-databases-cp-or-ap.html)
* [Spanner, TrueTime and the CAP Theorem](https://research.google.com/pubs/pub45855.html?hl=pl)
* [CouchDB - Introduction to Replication](http://docs.couchdb.org/en/master/replication/intro.html)
* [Consistent hashing](http://michaelnielsen.org/blog/consistent-hashing/)
* [Data versioning to compliment consistency(ETags)](https://developers.google.com/gdata/docs/2.0/reference?csw=1#ResourceVersioning)