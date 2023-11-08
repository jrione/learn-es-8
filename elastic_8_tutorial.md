## Chapter 1: Installing and Understanding Elasticsearch

### Installing Elasticsearch using Docker-Compose

In today's fast-paced development environment, setting up software should be quick and hassle-free. One of the most efficient ways to install Elasticsearch is by using Docker-Compose. Docker-Compose allows you to define and run multi-container Docker applications, ensuring that Elasticsearch and its dependencies are correctly configured in an isolated environment. This method is especially beneficial for developers who frequently set up and tear down instances for testing.

**Installation Steps**:
1. First, ensure you have both Docker and Docker-Compose installed on your machine.
2. Create a `docker-compose.yml` file with the necessary Elasticsearch configuration.
3. Run the command:
```bash
docker-compose up -d
```
This command will start Elasticsearch in the background, and you're ready to dive in!

### Elasticsearch: A Brief Overview

Elasticsearch is not just a search engine; it's a powerful distributed system that allows you to store, search, and analyze vast amounts of data in near real-time. Built on top of the Apache Lucene library, Elasticsearch provides a RESTful interface, making it easy to perform CRUD operations and advanced searches.

### Introduction to HTTP and RESTful API’s

The beauty of Elasticsearch lies in its simplicity. It communicates using HTTP, the same protocol that powers the web. This makes it incredibly versatile and easy to integrate with other systems. Furthermore, Elasticsearch uses RESTful APIs, which means you can interact with your data using standard HTTP methods like GET, POST, PUT, and DELETE. Each piece of data, or document, in Elasticsearch is treated as a resource, making data manipulation intuitive.

### Delving into Elasticsearch's Logical Concepts

Before you start indexing and searching data, it's crucial to grasp some of Elasticsearch's fundamental logical concepts:

- **Node**: Think of a node as a single server or instance of Elasticsearch. It stores data and participates in the cluster's indexing and search capabilities.
- **Cluster**: A cluster is a collection of nodes that work together, sharing the workload and ensuring data redundancy.
- **Index**: An index is akin to a database in traditional relational databases. It's where you store related documents.
- **Shard**: As your data grows, managing it can become challenging. That's where sharding comes into play. An index can be split into multiple shards, allowing Elasticsearch to distribute data and ensure scalability.
- **Replica**: Data loss is a nightmare. To prevent this, Elasticsearch uses replicas. A replica is a copy of the primary shard, ensuring high availability and data redundancy.

### The Magic of TF/IDF

When it comes to search, relevance is king. Elasticsearch employs a statistical measure called Term Frequency / Inverse Document Frequency (TF/IDF) to determine the relevance of a word in a document. In simple terms:

- **Term Frequency (TF)**: Represents how often a term appears in a document.
- **Inverse Document Frequency (IDF)**: Calculates the importance of a term relative to the entire collection of documents.

By combining these two metrics, Elasticsearch can rank documents, ensuring that the most relevant results are returned first.

### Using Elasticsearch

With Elasticsearch up and running, it's time to dive in! One of the most common tools to interact with Elasticsearch is `curl`, a command-line utility for making HTTP requests. To check the health of your Elasticsearch cluster, simply run:

```bash
curl -X GET "localhost:9200/_cat/health?v"
```

### What's Brewing in Elasticsearch 8?

While the exact differences between Elasticsearch 7 and 8 would require a detailed comparison, here are some general changes:

- Performance Improvements
- Enhanced Security Features
- Improved Observability
- New APIs and Query Features
- Deprecation and Removal of older features or APIs

### How Elasticsearch Scales: A Deep Dive

Elasticsearch's ability to scale is one of its most powerful features. As data grows, the need for a system that can handle increased load without compromising on performance becomes paramount. Elasticsearch achieves this through a combination of horizontal scaling, sharding, and replication.

#### Horizontal Scaling

Unlike vertical scaling, where you increase the resources (CPU, memory, storage) of an existing server, horizontal scaling involves adding more machines to your setup. In the context of Elasticsearch:

- **Node**: Every instance of Elasticsearch is called a node. When data or load increases, you can simply add more nodes to the cluster.
  
- **Cluster**: A cluster is a collection of nodes that work together, sharing the workload and ensuring data redundancy.

The beauty of horizontal scaling in Elasticsearch is its dynamic nature. You can add or remove nodes on the fly without any downtime.

#### Sharding

As mentioned earlier, an index can be thought of as a database, holding a collection of documents. But what happens when an index becomes too large for a node to handle? This is where sharding comes into play.

- **Primary Shards**: When you create an index, you define the number of primary shards. Each shard is a self-contained index, and Elasticsearch distributes these shards across the nodes in the cluster. This means that as you add more nodes, Elasticsearch can redistribute these shards to ensure an even distribution.

- **Shard Splitting**: In earlier versions of Elasticsearch, once you set the number of primary shards, it was fixed. However, recent versions allow you to split primary shards, giving you more flexibility as your data grows.

#### Replication

While sharding helps with scalability, replication ensures high availability and fault tolerance.

- **Replica Shards**: For every primary shard, you can have one or more

 replica shards. A replica is a copy of the primary shard and serves two main purposes:
  1. **Fault Tolerance**: If a node fails, having replicas ensures that no data is lost.
  2. **Increased Query Capacity**: Both primary and replica shards can serve read requests (like search queries). By having replicas, you can spread the search load, resulting in faster search speeds.

It's worth noting that while primary and replica shards can serve read requests, write operations (like indexing) are only performed on the primary shard. Once the primary shard has been updated, the changes are then replicated to the replica shards.

#### Rebalancing and Recovery

Elasticsearch continuously monitors the health of the nodes in the cluster. If a node fails or becomes unresponsive:

- **Rebalancing**: Elasticsearch will redistribute the shards to ensure an even distribution across the remaining nodes.
  
- **Recovery**: The failed node's data is not lost. Elasticsearch will use the replica shards from other nodes to recover the primary shards of the failed node.

In conclusion, Elasticsearch's ability to scale is a combination of its distributed nature, sharding strategy, and replication mechanism. Whether you're dealing with gigabytes or petabytes of data, Elasticsearch is designed to handle it efficiently.


## Chapter 2: Mapping and Indexing Data

### Connecting to your Cluster

Before performing any operations, you need to connect to your Elasticsearch cluster. The default endpoint for a locally running Elasticsearch instance is `http://localhost:9200`. You can check the health of your cluster using:

```bash
curl -X GET "localhost:9200/_cluster/health"
```

### Introducing the MovieLens Data Set

The MovieLens dataset is a collection of movie ratings data used for personalizing movie recommendations. It's maintained by the GroupLens research group at the University of Minnesota. The dataset contains ratings, movie metadata (genres and year), and demographic data about the users.

**Downloading the Dataset**:
You can download the MovieLens dataset from the [GroupLens website](https://grouplens.org/datasets/movielens/). For this tutorial, we'll use the "small" dataset, which contains 100,000 ratings and 3,600 tag applications applied to 9,000 movies by 600 users.

```bash
curl -u elastic:changeme -H "Content-Type: application/json" -XPUT localhost:9200/movies -d '
{
"mappings": {
"properties" : {
"year" : {"type": "date"}
}
}
}'
```

List indices:
```bash
curl -u elastic:changeme -H "Content-Type: application/json" localhost:9200/_cat/indices?v
```


### Analyzers in Elasticsearch

Analyzers are at the heart of text analysis in Elasticsearch. They process a text field during indexing and querying to make it searchable. An analyzer consists of a tokenizer and optional token filters.

**Types of Analyzers**:
1. **Standard Analyzer**: Splits text by whitespace and punctuation, and then lowercases it.
2. **Simple Analyzer**: Splits text by whitespace, then lowercases it.
3. **Whitespace Analyzer**: Splits text by whitespace.
4. **Stop Analyzer**: Like the simple analyzer but also removes stop words.
5. **Keyword Analyzer**: Treats the entire input as a single token.
6. **Pattern Analyzer**: Uses a regular expression to split the text into terms.

**Setting an Analyzer for a Field**:
To set the `standard` analyzer for a movie title:

```bash
curl -X PUT "localhost:9200/movies" -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "standard"
      }
    }
  }
}'
```

### Import a Single Movie via JSON / REST

To add a single movie to our index:

```bash
curl -X POST "localhost:9200/movies/_doc/" -H 'Content-Type: application/json' -d'
{
  "title": "Inception",
  "genre": "Sci-Fi"
}'
```

### Insert Many Movies at Once with the Bulk API

The Bulk API allows you to perform multiple indexing operations in a single request:

```bash
curl -X POST "localhost:9200/movies/_bulk" -H 'Content-Type: application/json' -d'
{ "index": {} }
{ "title": "Interstellar", "genre": "Sci-Fi" }
{ "index": {} }
{ "title": "The Dark Knight", "genre": "Action" }
'
```

Index stats:
```bash
curl -u elastic:changeme -H "Content-Type: application/json" localhost:9200/movies/_stats?pretty
```

Show all docs:
```bash
curl -u elastic:changeme -H "Content-Type: application/json" -XPOST localhost:9200/movies/_search?pretty
```


### Updating Data in Elasticsearch

To update an existing movie's data:

```bash
curl -X POST "localhost:9200/movies/_update/YYh1Y4sBsBiMUTdvbugY" -H 'Content-Type: application/json' -d'
{
  "doc": {
    "genre": "Adventure"
  }
}'
```

Replace `MOVIE_ID` with the actual ID of the movie you want to update.

### Deleting Data in Elasticsearch

To delete a movie:

```bash
curl -X DELETE "localhost:9200/movies/_doc/MOVIE_ID"
```

### Dealing with Concurrency

Elasticsearch handles concurrent requests using versioning. Each document has a version number, which is incremented on every update. If two updates are attempted simultaneously, the one with the older version number will be rejected. This ensures that changes are applied in the correct order and that no updates are lost.


### Data Modeling and Parent/Child Relationships

Elasticsearch provides a way to model parent-child relationships within a single index. This is useful when you have one-to-many relationships, like movies and ratings.

#### Setting up the Parent-Child Relationship

To set up a parent-child relationship between movies and ratings:

```bash
curl -X PUT "localhost:9200/movies" -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "properties": {
      "relation": {
        "type": "join",
        "relations": {
          "movie": "rating"
        }
      }
    }
  }
}'
```

#### Indexing Parent and Child Documents

To index a movie (parent):

```bash
curl -X POST "localhost:9200/movies/_doc/1" -H 'Content-Type: application/json' -d'
{
  "title": "Inception",
  "relation": {
    "name": "movie"
  }
}'
```

To index a rating (child) for the movie:

```bash
curl -X POST "localhost:9200/movies/_doc/2?routing=1" -H 'Content-Type: application/json' -d'
{
  "stars": 5,
  "relation": {
    "name": "rating",
    "parent": "1"
  }
}'
```

Note: The `routing` parameter is crucial when indexing child documents. It ensures that the child and parent documents are co-located on the same shard.

#### Updating Child Documents

To update a rating:

```bash
curl -X POST "localhost:9200/movies/_update/2?routing=1" -H 'Content-Type: application/json' -d'
{
  "doc": {
    "stars": 4
  }
}'
```

#### Deletion Mechanism

In Elasticsearch, there isn't a direct CASCADE delete mechanism like in RDBMS. If you delete a parent document, its child documents remain in the index but become orphaned. If you need to delete both parent and child documents, you must delete them individually.

#### Retrieving Parent Document with Related Child Documents

To retrieve a movie and its associated ratings:

```bash
curl -X GET "localhost:9200/movies/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "has_child": {
      "type": "rating",
      "query": {
        "match_all": {}
      },
      "inner_hits": {}
    }
  }
}'
```

The `inner_hits` section will contain the child documents (ratings) related to the parent document (movie).

---

This provides a more detailed overview of parent-child relationships in Elasticsearch, complete with `curl` examples. If there are any other sections or details you'd like to add or modify, please let me know!

### Flattened Datatype

The `flattened` datatype is designed for object fields that contain arbitrary keys. It maps an entire object as a single field and allows for simple searches over its contents.

**Example**:
Suppose you have movie attributes that vary significantly between movies:

```bash
curl -X PUT "localhost:9200/movies" -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "properties": {
      "attributes": {
        "type": "flattened"
      }
    }
  }
}'
```

To index a movie with attributes:

```bash
curl -X POST "localhost:9200/movies/_doc/3" -H 'Content-Type: application/json' -d'
{
  "title": "Inception",
  "attributes": {
    "release_date": "2010-07-16",
    "director": "Christopher Nolan",
    "box_office": "829.9 million USD"
  }
}'
```

You can then search for movies based on these attributes, even if the keys are not explicitly defined in the mapping.


## Chapter 3: Searching with Elasticsearch

### “Query Lite” Interface

Elasticsearch provides a simple search interface called "Query Lite." It allows for quick searches without the need for complex query DSL.

**Example**:
Search for the term "Inception" in all fields:

```bash
curl -X GET "localhost:9200/movies/_search?q=Inception"
```
```bash
curl -u elastic:changeme -H "Content-Type: application/json" -XPOST 'localhost:9200/movies/_search?q=title:star&pretty'
```

### JSON Search In-Depth

For more complex queries, you can use the JSON-based query DSL.

**Example**:
Search for movies with the title "Inception":

```bash
curl -X GET "localhost:9200/movies/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "title": "Inception"
    }
  }
}'
```

```bash
curl -u elastic:changeme -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
	"query": {
		"match": {
			"title": "star"
		}
	}
}'
```

### Phrase Matching

Phrase matching allows you to search for exact phrases.

**Example**:
Search for the exact phrase "Dark Knight":

```bash
curl -X GET "localhost:9200/movies/_search"  -u elastic:changeme -H "Content-Type: application/json" -d'
{
  "query": {
    "match_phrase": {
      "title": "Dark Knight"
    }
  }
}'
```

### Pagination

You can paginate search results using the `from` and `size` parameters.

**Example**:
Retrieve results 10-20:

```bash
curl -u elastic:changeme -X GET "localhost:9200/movies/_search" -H 'Content-Type: application/json' -d'
{
  "from": 10,
  "size": 10,
  "query": {
    "match_all": {}
  }
}'
```

### Sorting

Results can be sorted based on specific fields.

**Example**:

Sort movies by year:
```bash
curl -u elastic:changeme -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?sort=year&pretty'
```
```bash
curl -u elastic:changeme -X GET "localhost:9200/movies/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "sort": [
    {
      "year": {
        "order": "desc"
      }
    }
  ],
  "query": {
    "match_all": {}
  }
}'
```

Sort movies by title in ascending order:

```bash
curl -u elastic:changeme -X GET "localhost:9200/movies/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "sort": [
    {
      "title": {
        "order": "asc"
      }
    }
  ],
  "query": {
    "match_all": {}
  }
}'
```

Will throw error. you have to reindex!:

1. create new index
```bash
curl -u elastic:changeme -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies_new/ -d '
{
	"mappings": {
		"properties" : {
			"title": {
				"type" : "text",
				"fields": {
					"raw": {
						"type": "keyword"
					}
				}
			}
		}
	}
}'
```
2. Reindex the data from the old index to the new index:
```bash
curl -u elastic:changeme -H "Content-Type: application/json" -XPOST 127.0.0.1:9200/_reindex -d '
{
  "source": {
    "index": "movies"
  },
  "dest": {
    "index": "movies_new"
  }
}'
```
3. (Optional) Delete the old index and rename the new index:
```bash
curl -u elastic:changeme -H "Content-Type: application/json" -XDELETE 127.0.0.1:9200/movies
```

### More with Filters

Filters allow you to refine search results without affecting the score.

**Example**:
Find Sci-Fi movies released after 2010:

```bash
curl -X GET "localhost:9200/movies/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "genre": "Sci-Fi"
        }
      },
      "filter": {
        "range": {
          "release_date": {
            "gte": "2011-01-01"
          }
        }
      }
    }
  }
}'
```

```bash
curl -u elastic:changeme -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_search?pretty -d'
{
	"query":{
		"bool": {
			"must": {"term": {"title": "trek"}},
			"filter": {"range": {"year": {"gte": 2010}}}
		}
	}
}'
```

**filters** ask a yes/no question of your data. 
**queries** return data in terms of relevance.

Some types of filters:

**term**: filter by exact values
`{"term": {"year": 2014}}`
**terms**: match if any exact values in a list match
`{"terms": {"genre": ["Sci-Fi", "Adventure"] } }`
**range**: Find numbers or dates in a given range (gt, gte, lt, lte)
`{"range": {"year": {"gte": 2010}}}`
**exists**: Find documents where a field exists
`{"exists": {"field": "tags"}}`
**missing**: Find documents where a field is missing
`{"missing": {"field": "tags"}}`
**bool**: Combine filters with Boolean logic (must, must_not, should)


Some types of queries:

**match_all**: returns all documents and is the default. Normally used with a filter.
`{"match_all": {}}`
**match**: searches analyzed results, such as full text search.
`{"match": {"title": "star"}}`
**multi_match**: run the same query on multiple fields.
`{"multi_match": {"query": "star", "fields": ["title", "synopsis" ] } }`
**match_phrase**: must find all terms, in the right order.
```bash
curl -u elastic:changeme -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
	"query": {
		"match_phrase": {
			"title": "star wars"
		}
	}
}'
```
**slop**: order matters, but you’re OK with some words being in between the terms
```bash
curl -u elastic:changeme -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_search?pretty -d '
{
	"query": {
		"match_phrase": {
			"title": {"query": "star beyond", "slop": 1}
		}
	}
}'
```

**bool**: Works like a bool filter, but results are scored by relevance.


```bash
curl -u elastic:changeme -H "Content-Type: application/json" -XGET 127.0.0.1:9200/movies/_search?pretty -d'
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "genre": "Sci-Fi"
        }
      },
      "must_not": {
        "match": {
          "title": "trek"
        }
      },
      "filter": {
        "range": {
          "year": {
            "gte": 2010,
            "lt": 2015
          }
        }
      }
    }
  }
}'
```

### Fuzzy Queries

Fuzzy queries allow for search results that are approximately, but not exactly, like the search term. A way to account for typos and misspellings.

the **levenshtein edit distance** accounts for:

- substitutions of characters (interstellar -> intersteller)
- insertions of characters (interstellar -> insterstellar)
- deletion of characters (interstellar -> interstelar)

all of the above have an edit distance of 1.

**Example**:
Search for titles similar to "Incepton" (with a typo):

```bash
curl -u elastic:changeme -H "Content-Type: application/json" -X GET "localhost:9200/movies/_search?pretty" -d'
{
  "query": {
    "fuzzy": {
      "title": {
        "value": "interstelar",
        "fuzziness": 1
      }
    }
  }
}'
```

fuzziness: AUTO

- 0 for 1-2 character strings
- 1 for 3-5 character strings
- 2 for anything else



### Partial Matching

You can use the `prefix` query for partial matching.

**Example**:
Find movies that start with "Inter":

```bash
curl -X GET "localhost:9200/movies/_search?pretty" -u elastic:changeme -H "Content-Type: application/json" -d'
{
  "query": {
    "prefix": {
      "title": "Inter"
    }
  }
}'
```

You can also use the `wildcard` query for partial matching.
```bash
curl -u elastic:changeme -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
  "query": {
    "wildcard": {
      "title": "in*"
    }
  }
}'
```

### Query-time Search As You Type

This feature allows for real-time feedback as users type their queries.

**Example**:
Search as you type for "star trek":

```bash
curl -u elastic:changeme -H "Content-Type: application/json" -XGET '127.0.0.1:9200/movies/_search?pretty' -d '
{
  "query": {
    "match_phrase_prefix": {
      "title": {
        "query": "star trek",
        "slop": 10
      }
    }
  }
}'
```

### N-Grams

N-Grams are substrings of a given string. They can be used to improve partial matching.

**Example**:
To configure an N-Gram tokenizer:

```bash
curl -u elastic:changeme -X PUT "localhost:9200/movies" -H 'Content-Type: application/json' -d'
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "ngram_tokenizer": {
          "type": "nGram",
          "min_gram": 2,
          "max_gram": 3,
          "token_chars": ["letter", "digit"]
        }
      }
    }
  }
}'
```

### Search-As-You-Type Field Type

This field type optimizes fields for search-as-you-type functionality.

**Example**:

To define a search-as-you-type field:
```bash
curl -u elastic:changeme -XPUT 'localhost:9200/movies/_mapping' -H 'Content-Type: application/json' -d'
{
  "properties": {
    "title_ac": {
      "type": "search_as_you_type"
    }
  }
}'
```
Fill the new field:
```bash
curl -u elastic:changeme -XPOST 'localhost:9200/movies/_update_by_query' -H 'Content-Type: application/json' -d'
{
  "script": {
    "source": "ctx._source.title_ac = ctx._source.title"
  }
}'

```

Test autocomplete
```bash
curl -u elastic:changeme -XGET 'localhost:9200/movies/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "multi_match": {
      "query": "Int",
      "type": "bool_prefix",
      "fields": [
        "title_ac",
        "title_ac._2gram",
        "title_ac._3gram",
        "title_ac._index_prefix"
      ]
    }
  }
}'

```

