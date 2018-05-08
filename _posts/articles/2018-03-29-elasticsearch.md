---
layout: post
title: "Elasticsearch"
excerpt: "let's see..."
date: 2018-03-29
tags: [tech, database, nosql, search, textsearch, fulltextsearch, lucene, elasticsearch]
categories: articles
comments: true
share: true
published: true
---

# Elasticsearch

**Note:** This article is currently incomplete & in-progress
{: .notice--danger}

#### What is Elasticsearch?

A distributed schema-less database with very fast full-text search capabilities. [Official documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html).

* A wrapper over Lucene (written in Java), as a distributed system with http interface - that is very fast & (auto) scalable
* At heart, the core work is done by Lucene which provides full-text search
* Works as a JSON-based full-text no-sql DB
* Returns text-search result based on relevance, also creates analytics on data
* Comes with rich set of built-in aggregation and statistics/calculation options

**Why?** Very fast, result ordered by relevance, statistical trends in data
**When?** Any data that is generally big chunk of text - e.g. blogs, logs, unstructured document data (e.g. to sentence completion suggestion)

#### Architecture

* Data is stored as `JSON documents`
* A `Type` is a collection of similar documents (li'l like tables in relation database)
* Types are stored as `Index` (similar to DB in relational DB)
* Indices are sliced into `Shards`, which are nothing but chunk of data from an index - each shard is one instance of `Lucene`
* Shards are stored in multiple `Nodes` (Node = Elasticsearch server), forming an easily scalable `Cluster`
* Elasticsearch can make copies of index shards, which are called `Replicas`

For a vague conceptual analogy

**_Relational:_** _Databases_ >> _Tables_ >> _Rows with Columns_ <br />
**_Elasticsearch:_** _Indices_ >> _Types_ >> _Documents with Properties_
{: .notice--success}

###### So, what is the relation between Lucene & Elasticsearch ?

**_Lucene_** is the core & heart of **_Elasticsearch_**. It is the engine that provides all the fast searching capabilities. What Elasticsearch does is, makes it a distributed (thus scalable) system. It also provides easy to use REST APIs and lot of very useful tools & plugins, and provides frameworks like ELK stack (Elasticsearch, Logstash & Kibana) etc.

So, if you just want to use the super-fast full-text-search capabilities on a single (not so big) data source, you can just use Lucene. If you want to make it a scalable distributed system, use other cool stuffs (like Kibana), add monitoring & visualization etc., go for Elasticsearch.

###### How does it work ?

Doc.1 : "I love cakes" <br />
Doc.2 : "I Write code" <br />
Doc.3 : "Cakes and cookies"

|Term|Document IDs|
|:---|:---|
i|1,2
love|1
cakes|1,3
write|2
code|2
cookies|3

1. Filter out unnecessary stuffs, like HTML
2. Tokenize - split characters by white-space & punctuation to extract "words" or "terms"
3. Remove common _"stop words"_ like "and", "the" etc.

**Analyzers**

They defined how data is tokenized. It comes with bunch of default analyzers, and custom ones can also be added. The default `standard` analyzer includes white-spaces & punctuations, and there is only `whitespace` analyzers etc. The term/token casing also depends on analyzers.

It's important to select the correct analyzer, otherwise search results may not be as expected.

Elasticsearch comes with APIs to test the analyzers with sample text. Specific analyzers can be set in index mappings. Also, some fields can be excluded from analyzing with `not_analyzed` attribute. Examples will follow.

```bash
POST http://localhost:9200/_analyze?analyzer=whitespace
One can use the .ToUpper(target_string) funtion
```

#### Installation

* Install on Windows as service from [here](https://www.elastic.co/guide/en/elasticsearch/reference/current/_installation.html)
* In `config.yml`, change node and cluster name if required
* Now the APIs are available at `http://localhost:9200`
* Install plugins - [Marvel](https://www.elastic.co/downloads/marvel) as interactive GUI for monitoring clusters & query data (built on Kibana - data visualization tool)
  * Go to Elasticsearch install directory `C:\elasticsearch\bin` in command prompt
  * Type in `plugin.bat -i elasticsearch/marvel/latest`
  * Once installed, restart Elasticsearch service
  * The cluster dashboard is viewable at http://localhost:9200/_plugin/marvel 
  * The query interface is available at Sense dashboard http://localhost:9200/_plugin/marvel/sense/index.html
* Also look at plugin [head](https://github.com/mobz/elasticsearch-head) which gives an easy to use GUI

#### Elasticsearch Schemas or Mappings

Elasticsearch comes with default set of data types viz. `integer`, `string` (kind of the default & most used data type, internally `analyzed text`), `date` (can have time and specific format), `numbers` (can be short, byte, double, float etc.), `boolean` & `binary` etc. Using them, we can define schema of a `Type`. To create a schema, `POST` to the index-name path on the base API URI.

```javascript
//index schema creation
POST http://localhost:9200/blog //blog index
{
  "settings": {
    "index": {
	  "number_of_shards": 5 //optional: create 5 shards for this index
	  }
  },
  "mappings": { //schema of index
    "post": { //a type
      "properties": {
        "title": {
          "type": "string",
          "index": "not_analyzed" //optional, do not analyze this field
        },
        "author": {
          "type": "integer"
        },
        "date": {
          "type": "date",
          "format": "YYYY-MM-DD" //optional: data needs to match this format
        },
        "body": { //main search target
          "type": "string", ??
          "analyzer": "whitespace" //optional analyzer setting
        }
      }
    }
  }
}
```

**Note:** The `_id (string)` property is generated by default. They are also populated by Elasticsearch.
{: .notice--info}

**Data storage options**

Many a times Elasticsearch is used as a secondary DB for searching only (on specific fields) rather than a complete data storage solution. In that case, data from another DB (e.g. MongoDB) is used as source and Elasticsearch creates an index, also keeps a copy of the original data, as default behavior. That does not slow down Elasticsearch, but takes more space.

This behavior can be changed, so that copy of all data are not stored in Elasticsearch, only indexed. Following example will disable **_"_source"_** to not keep _"copy of source data"_ and keep only selected fields (title & author). Other fields, e.g. post body will only have a indexed version not a full copy, even if they are passed in insert/POST query.

```javascript
{
  "mappings": {
    "post": {
      "_source": { ??
        "enabled": false //do not copy source data
      },
      "properties": {
        "title": {
          "type": "string",
          "store": true //keep copy of data
        },
        "author": {
          "type": "integer",
          "store": true //keep copy of data
        },
        "date": {
          "type": "date",
          "format": "YYYY-MM-DD" //like below
        },
        "body": {
          "type": "string" //original data not kept
        }
      }
    }
  }
}
```

Another thing to note is, Elasticsearch by default creates a **_"_all"_** field, that is concatenation of all the fields in the type. It if useful to look at all the data together, but it also increases data size. If wanted, it can be disabled the same way as above. `"_all": { "enabled": false }` ??

#### Routing

Based on **_`_routing`_** defined, documents with same value of `route` will be placed in same shard. With this in place, queries can be directed with specific _"routing"_, so that Elasticsearch can directly search in specific shard making it much faster!

```javascript
# generate index schema
POST http://localhost:9200/blog
{
  "mappings": {
    "post": {
      "_routing": {
	      "required": true, //queries must use routing
		    "path": "author" //route by author-value
      },
      "properties": {
        "title": {
          "type": "string"
        },
        "author": {
          "type": "integer" //author is actually author-id here
        },
        "date": {
          "type": "date",
          "format": "YYYY-MM-DD"
        },
        "body": {
          "type": "string"
        }
      }
    }
  }
}

#query | find posts with "concurrency" given author=101 (in same shard)
GET blog/post/_search?routing=101&body:concurrency
```

#### Aliases

An `alias` for a single or set of indices. Very vaguely similar to relational database views, with no WHERE clause. With **_"_aliases"_**, we can search within a set of (generally simlar/related) types.

```javascript
POST _aliases
{
  "actions": {
    "add": {
	    "index": "biologybooks",
	    "alias": "books"
	  }
  }
}

POST _aliases
{
  "actions": {
    "add": {
	    "index": "graphicnovels",
	    "alias": "books"
	  }
  }
}

# query to find ANY book with "brain"
GET books/_search?content:brain ??
```

#### Some simple queries

As Elasticsearch comes with full set of REST APIs, all the data modification & queryoperations can be done with simple http REST calls.

```javascript
# base uri for all queries is http://localhost:9200

# see all indices
GET /_cat/indices

# see schema of an index
GET index_name/_mapping

# see all data
GET index_name/type_name/_search 
# OR
GET index_name/_search ??

# insert a document
POST index_name/type_name
{
  "user_name": "AC",
  "email": "cool@code.com"
}

# read all documents from a type
GET index_name/type_name/_search

# get by _id
GET index_name/type_name/_id_string_value

# insert document with custom numeric id
# no change in schema, the string _id property can take integer too
POST index_name/type_name/id_number
{
  "user_name": "AC with numeric id",
  "email": "cool2@code.com"
}

# retrieve with custom id
GET index_name/type_name/id_number

# delete index
DELETE index-name

# SEARCH LITE
# basic search - blog index, post type with "concurrency" in body field (content)
GET /blog/post/_search?q=body:concurrency
```

#### Elasticsearch Query DSL

Elasticsearch supports a `Query DSL` or _"Domain Specific Language"_ query, where we can provide detailed queries as object load in GET calls. For example, the equivalent of the above _"Search lite"_ query in DSL will have a `match` object with `"body": "concurrency"`. We'll see some sample in a while.

**Relevance**

As told earlier, Elasticsearch returns results of a query as per relevance. That means, when there are multiple documents returned in a query, they are returned by descending order of relevance, i.e. most relevant or closest match to a query is placed at top, followed by second most relevant one and so on.

This _"relevance"_ is represented with a Elasticsearch generated property **_"_score"_**. Higher the `_score`, more relevant the search result is. So, practically the search results are arranged by descending order of `_score`.

```javascript
GET /blog/post/_search
{
  "query": {
    "match": {
      "body": "parallel programming"
    }
  }
} //match will individually match the words "parallel" & "programming"

# Full-text search
GET /blog/post/_search
{
  "query": {
    "match_phrase": {
      "body": "parallel programming"
    }
  }
} //match_phrase will match the whole phrase "parallel programming"

# Search with filters - narrowing down results with logical operators
GET /blog/post/_search
{
  "query": {
    "filtered": {
      "filter": {
        "range": {
          "date": {
            "lt": "2018-01-01"
          }
        }
      }
    },
    "query": {
      "match": {
        "body": "parallel programming"
      }
    }
  } //search maching posts that were posted before 2018
}

# other types of filter
"term" {
  "author": 101
}
```

**Highlight**

Elasticsearch can return results with match words highlighted, as HTML. For that, we need to include `highlight` field with the actual query. Returned HTML will have matches emphasized with `<em>` tag.

```javascript
{
  "query": ...,
  "highlight": {
    "fields": {
      "body": {} //highlight the matches in body field
    }
  }
}
```

#### Aggregation

Elasticsearch comes with a lot of aggregation functionalities including those similar to SQL `GROUP BY`, `MIN`, `MAX` etc., and many statistical calculations like - word frequency, variance, standard deviation, percentiles, histograms and many more! And what's more? It has in-built support for lots of geo location based calculations! See the [documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html) for more details.

Example ??






#### Clients

###### Clients for different languages

* [Official & community clients](https://www.elastic.co/guide/en/elasticsearch/client/index.html)

###### Elasticsearch .NET clients

* Elasticsearch.Net - low level, covering the basic elasticsearch APIs as methods
* NEST - high level wrapper over Elasticsearch.Net that provides more abstraction & strongly-typed DSL that maps 1 to 1 with the Elasticsearch query DSL, officially supported

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

# Questions on mah mindo

1. What about required/mandatory properties of a type ??
2. Is Elasticsearch schema-less ??
3. Unique constraint on one or more properties ??
4. Query, where, and ??
5. Can we create links/relationships between different documents in different types ??
6. How does Elasticsearch/Lucene do full text search ??
7. All Elasticsearch DSL commands ??

X. Creating data model for Elasticsearch (importing from relational)

#### References

* [Setting up Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup.html)
* [Setting up Lucene]
* [Kibana](https://www.elastic.co/products/kibana)
* [Marvel](https://www.elastic.co/downloads/marvel), [Why Marvel](https://www.elastic.co/blog/building-marvel)
* [ELK stack](https://www.elastic.co/elk-stack)
* [Elasticsearch top-down explanation](https://www.elastic.co/blog/found-elasticsearch-top-down)
* [Elasticsearch bottom-up explanation](https://www.elastic.co/blog/found-elasticsearch-from-the-bottom-up)
* [Ref: The Google prototype by Sergey Brin and Lawrence Page](http://infolab.stanford.edu/~backrub/google.html)
* Some very basic explanation (?) of Lucene on [Quora](https://www.quora.com/in/What-is-an-intuitive-description-of-how-Lucene-works) and [SO](https://stackoverflow.com/questions/2705670/how-does-lucene-work)
* [TF-IDF algorithm](https://en.wikipedia.org/wiki/Tf%E2%80%93idf)