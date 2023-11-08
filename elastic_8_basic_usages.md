# ElasticSearch: Zero to Hero in 12 Commands

It's relatively easy to get started with ElasticSearch. But as our use cases get more specific, we found the documentation lacking. This guided cheatsheet will execute 12 commands: from setting up your ES index to making advanced ES queries to support advanced (but common) use cases.

The 12 commands work when done sequentially. I will explain each of them, but trying them for yourself is still the best.

This post is part of a broader series on ElasticSearch that will be released in the coming weeks:
- The Guided ElasticSearch Cheatsheet you need to Get Started with ES - you are here
- Using DynamoDB + ElasticSearch for prod workloads - coming soon
- And how to create DynamoDB Streams to sync data changes from DynamoDB to ES asynchronously - coming soon

## 0 | Prerequisites

Install Elasticsearch with this official ES Guide. And then, turn on the ES server on localhost:9200

For easier testing, installing an API platform like Postman is a must.

## A | Setup the index

In ElasticSearch, we store our data in indexes (similar to tables in your MySQL database). We populate indexes with documents (similar to rows). We will create and set up your first index in the subsequent commands.

### [1] Verify the ES cluster is accessible

```bash
GET localhost:9200
```

First, make sure your local ES server is online, and you have your Postman open. Create a new GET request headed for localhost:9200. You should see something like this.

### [2] Create an index

```bash
PUT localhost:9200/mynewindex
```

Now, let's create our first index. Indexes store our data. It is equivalent to creating a table in relational databases.

### [3] Create the mapping for the index

The index we just created has no mapping. A mapping is similar to a schema in SQL databases. It dictates the form of the documents that our index will ingest. Once defined, the index will refuse to accept documents that cannot fit into this mapping (i.e, we defined stocks as integer below. If we try to insert a row with stocks="none", the operation will not continue).

One thing you'd notice with ES is that these mappings are permissive by default. If I add a row with a new attribute "perishable" = true, when I push a document to ES, the schema will add that attribute and infer its data type. In this case, it will add a new attribute in the mapping for "perishable" with data type "boolean".

There are options that you can add when you create your index to only allow attributes defined in mapping of your index, nothing more, nothing less.

In this command, we create the mapping for our newly created index.

```bash
PUT localhost:9200/mynewindex/_mapping

{
    "properties": {
        "product_id": {
            "type": "keyword"
        },
        "price": {
            "type": "float"
        },
        "stocks": {
            "type": "integer"
        },
        "published": {
            "type": "boolean"
        },
        "title": {
            "type": "text"
        },
        "sortable_title": {
            "type": "text"
        },
        "tags": {
            "type": "text"
        }
    }
}
```

Most of the data types are straightforward, except for Text and Keyword. This article explains the difference clearly.

But TLDR, Text allows you to query words inside the field (i.e querying "Burger" will show the product "Cheese Burger with Fries"). It does this by treating each word in the text as individual tokens that could be searched: "cheese", "burger", "with", "fries".

On the other hand, Keyword treats the content of the field as one, so if you want to get the cheeseburger with fries, you'd have to query it: "Cheese Burger with Fries". Querying "burger" will return nothing.

### [4] Show the mapping of the index

Let's verify if we have successfully created the mapping for the index by sending a GET request.

```bash
GET localhost:9200/mynewindex
```

## B | Data Operations with our ES Index

With our index already set up, let's add data and chip away at the more exciting bits of ES!

### [5] Create data for the index

For this section, let's send three consecutive post requests with different a request body per request. This adds 3 "rows" inside our Elasticsearch index.

```bash
POST localhost:9200/mynewindex/_doc

{
    "product_id": "123",
    "price": 99.75,
    "stocks": 10,
    "published": true,
    "sortable_title": "Kenny Rogers Chicken Sauce",
    "title": "Kenny Rogers Chicken Sauce",
    "tags": "chicken sauce poultry cooked party"
}

POST localhost:9200/mynewindex/_doc

{
    "product_id": "456",
    "price": 200.75,
    "stocks": 0,
    "published": true,
    "sortable_title": "Best Selling Beer Flavor",
    "title": "Best Selling Beer Flavor",
    "tags": "beer best-seller party"
}

POST localhost:9200/mynewindex/_doc

{
    "product_id": "789",
    "price": 350.5,
    "stocks": 200,
    "published": false,
    "sortable_title": "Female Lotion",
    "title": "Female Lotion",
    "tags": "lotion female"
}
```

### [6] Display all the data

Now, let's see if the three documents we inserted via command #5 got inside our index. This command shows all the documents inside your index:

```bash
POST localhost:9200/mynewindex/_search

{
    "query": {
        "match_all": {}
    }
}
```

It does!

### [7] Exact search with product id

Now, let's start with a simple search. Let's search by product id.

```bash
POST localhost:9200/mynewindex/_search

{
    "query": {
        "term": {
            "product_id": "456"
        }
    }
}
```

In the command above, we are using a "term query" because we are looking for a product with a "product_id" that exactly matches the string "456". The term query works because the data type of "product_id" is "keyword".

### [8] Fuzzy search with titles

Now, onto the more exciting bits.

ES is known for its comprehensive search capability. Let's sample that by creating our first Fuzzy search. Fuzzy searches allow us to search for products by typing just a few words instead of the whole text of the field. Instead of typing the full name of the product name (i.e Incredible Tuna Mayo Jumbo 250), the customer just instead has to search for the part he recalls of the product (i.e Tuna Mayo).

```bash
POST localhost:9200/mynewindex/_search

{
    "query": {
        "match":

 {
            "title": "chicken sauce"
        }
    }
}
```

### [9] Search with multiple criteria

Now, let's search for products that are both published and have stocks. This is a common use case for e-commerce websites.

```bash
POST localhost:9200/mynewindex/_search

{
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "published": true
                    }
                },
                {
                    "range": {
                        "stocks": {
                            "gt": 0
                        }
                    }
                }
            ]
        }
    }
}
```

### [10] Search with sorting

Now, let's search for products that are both published and have stocks. But this time, let's sort them by price in descending order.

```bash
POST localhost:9200/mynewindex/_search

{
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "published": true
                    }
                },
                {
                    "range": {
                        "stocks": {
                            "gt": 0
                        }
                    }
                }
            ]
        }
    },
    "sort": [
        {
            "price": {
                "order": "desc"
            }
        }
    ]
}
```

### [11] Search with pagination

Now, let's search for products that are both published and have stocks. But this time, let's sort them by price in descending order and paginate the results.

```bash
POST localhost:9200/mynewindex/_search

{
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "published": true
                    }
                },
                {
                    "range": {
                        "stocks": {
                            "gt": 0
                        }
                    }
                }
            ]
        }
    },
    "sort": [
        {
            "price": {
                "order": "desc"
            }
        }
    ],
    "from": 0,
    "size": 10
}
```

### [12] Search with aggregations

Lastly, let's search for products that are both published and have stocks. But this time, let's sort them by price in descending order, paginate the results, and aggregate the results by the "tags" field.

```bash
POST localhost:9200/mynewindex/_search

{
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "published": true
                    }
                },
                {
                    "range": {
                        "stocks": {
                            "gt": 0
                        }
                    }
                }
            ]
        }
    },
    "sort": [
        {
            "price": {
                "order": "desc"
            }
        }
    ],
    "from": 0,
    "size": 10,
    "aggs": {
        "tags": {
            "terms": {
                "field": "tags"
            }
        }
    }
}
```
