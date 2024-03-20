#  Elastic Intro
Basic doc about how to create a simple Elastic with docker


## Run
```bash 
docker-compose up -d
```

Elastic can be like a RESTful Api(GET, PUT, POST, DELETE)

## Check if its running
```bash
 curl -XGET 127.0.0.1:9200 
 ```
 Result similar to
 ```json
 {
  "name" : "38bb8f67ed9d",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "3q1siBHrTIuiK0uSYhsuXA",
  "version" : {
    "number" : "7.16.2",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "2b937c44140b6559905130a8650c64dbd0879cfb",
    "build_date" : "2021-12-18T19:42:46.604893745Z",
    "build_snapshot" : false,
    "lucene_version" : "8.10.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
 ```




## Create index(like a table)
```bash
    curl -XPUT 127.0.0.1:9200/movies 

 ```
Mapping - definition of schema(kind of schema registry)

Field types (**string, byte, integer, long, float, double, boolean, date**)
example:
```json
    "properties":{
        "user_id":{
            "type": "long"
        }
    }
```

Field index - Can be only text
example:
```json
    "properties":{
        "genre":{
            "index": "not_analyzed"
        }
    }
```

Field analyzer - Define filters, tokenizer, token filter
example:
```json
    "properties":{
        "descriptioin":{
            "analyzer": "english"
        }
    }
```

## Database of movies

Insert new movie
```json
   Method: PUT 
   Host + index - 127.0.0.1:9200/movies
   Body: 
   {
    "mappings":{
        "properties":{
            "year":{
                "type":"date"
            }
        }
    }
   }
 ```

Get mapping movies
```json
   Method: GET 
   Host + index - 127.0.0.1:9200/movies/_mapping
   
   Result:
   {
    "movies": {
        "mappings": {
            "properties": {
                "genre": {
                    "type": "keyword"
                },
                "id": {
                    "type": "integer"
                },
                "title": {
                    "type": "text",
                    "analyzer": "english"
                },
                "year": {
                    "type": "date"
                }
            }
        }
    }
}
```
 


Inset one movie
```json
   Method: POST 
   Host + index : 127.0.0.1:9200/movies/_doc/109487
   Body:
    {
        "genre": ["Sci-Fi","IMAX"],
        "title": "Interstellar",
        "year": 2014
    } 

    result: 

    {
    "_index": "movies",
    "_type": "_doc",
    "_id": "109487",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 5,
    "_primary_term": 1
}
```


List movies 
```json
   Method: POST 
   Host + index : 127.0.0.1:9200/movies/_search
   Body:
    {
        "genre": ["Sci-Fi","IMAX"],
        "title": "Interstellar",
        "year": 2014
    } 

    result: 

    {
    "took": 1002,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1,
            "relation": "eq"
        },
        "max_score": 1.0,
        "hits": [
            {
                "_index": "movies",
                "_type": "_doc",
                "_id": "109487",
                "_score": 1.0,
                "_source": {
                    "id": "109487",
                    "title": "Interstellar",
                    "year": 2014,
                    "genre": [
                        "Sci-Fi",
                        "IMAX"
                    ]
                }
            } 
        ]
    }
}
```


Inset multiple movies 
```json
   Method: POST 
   Host + index : 127.0.0.1:9200/movies/_bulk
   Body:

    { "create" : { "_index" : "movies", "_id" : "135569" } }
    { "id": "135569", "title" : "Star Trek Beyond", "year":2016 , "genre":["Action", "Adventure", "Sci-Fi"] }
    { "create" : { "_index" : "movies", "_id" : "122886" } }
    { "id": "122886", "title" : "Star Wars: Episode VII - The Force Awakens", "year":2015 , "genre":["Action", "Adventure", "Fantasy", "Sci-Fi", "IMAX"] }
    { "create" : { "_index" : "movies", "_id" : "58559" } }
    { "id": "58559", "title" : "Dark Knight, The", "year":2008 , "genre":["Action", "Crime", "Drama", "IMAX"] }
    { "create" : { "_index" : "movies", "_id" : "1924" } }
    { "id": "1924", "title" : "Plan 9 from Outer Space", "year":1959 , "genre":["Horror", "Sci-Fi"] }

```


## Query Line Search(QLS)

```json
    find all movies with title star

   [GET] 127.0.0.1:9200/movies/_search?q=title:star

    Result:
    {
    "took": 5,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 2,
            "relation": "eq"
        },
        "max_score": 1.0296195,
        "hits": [
            {
                "_index": "movies",
                "_type": "_doc",
                "_id": "135569",
                "_score": 1.0296195,
                "_source": {
                    "id": "135569",
                    "title": "Star Trek Beyond",
                    "year": 2016,
                    "genre": [
                        "Action",
                        "Adventure",
                        "Sci-Fi"
                    ]
                }
            },
            {
                "_index": "movies",
                "_type": "_doc",
                "_id": "122886",
                "_score": 0.73069775,
                "_source": {
                    "id": "122886",
                    "title": "Star Wars: Episode VII - The Force Awakens",
                    "year": 2015,
                    "genre": [
                        "Action",
                        "Adventure",
                        "Fantasy",
                        "Sci-Fi",
                        "IMAX"
                    ]
                }
            }
        ]
    }
}

```



```json
    Other query
    
    127.0.0.1:9200/movies/_search?q=+year:>2015+title:star

    Result:
    {
    "took": 5,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 2,
            "relation": "eq"
        },
        "max_score": 2.0296195,
        "hits": [
            {
                "_index": "movies",
                "_type": "_doc",
                "_id": "135569",
                "_score": 2.0296195,
                "_source": {
                    "id": "135569",
                    "title": "Star Trek Beyond",
                    "year": 2016,
                    "genre": [
                        "Action",
                        "Adventure",
                        "Sci-Fi"
                    ]
                }
            },
            {
                "_index": "movies",
                "_type": "_doc",
                "_id": "122886",
                "_score": 0.73069775,
                "_source": {
                    "id": "122886",
                    "title": "Star Wars: Episode VII - The Force Awakens",
                    "year": 2015,
                    "genre": [
                        "Action",
                        "Adventure",
                        "Fantasy",
                        "Sci-Fi",
                        "IMAX"
                    ]
                }
            }
        ]
    }
}
```

> :warning: Can be **danger** using query with url, because:
> * Security expose
> * Onde especial caracter(+,-...) can break query
>  * Hard to debug



## Json Search

While filters are essential for narrowing down search results, caching can significantly improve performance

```json
    127.0.0.1:9200/movies/_search

    {
        "query":{
            "bool":{
                "must": { "term": { "title": "trek"}},
                "filter": { "range": { "year": {"gte": 2010}}},
            }
        }
    }

    Result:

    {
    "took": 15,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1,
            "relation": "eq"
        },
        "max_score": 1.540445,
        "hits": [
            {
                "_index": "movies",
                "_type": "_doc",
                "_id": "135569",
                "_score": 1.540445,
                "_source": {
                    "id": "135569",
                    "title": "Star Trek Beyond",
                    "year": 2016,
                    "genre": [
                        "Action",
                        "Adventure",
                        "Sci-Fi"
                    ]
                }
            }
        ]
    }
}
```



**Types of filters:**

* term - query by exact value
* terms - match any value of list
* range - find date intervals(gt, gte, lt, lte)
* exists - find in doc where field existis
* missing - find in doc where field is missing
* bool - conbine(must, must_not, should)

**Type of queries**

* match_all - return all documents
* match - analyses result, like full text search
* multi_match
* bool - result ranked by relevance


**Slop** - 
```json
 {"title": "quick brown fox"}

 "query":{
    "match_phrase":{
        "title":{"query":"quick fox", "slop":1}
    }
 }
Tolerance to return a valid result
slop 1 - search by **quick brown fox**, will find one result
```

**Proximity queries**
```json
 {"title": "The terminal will display the response from Elasticsearch in a JSON format (made human-readable by the pretty parameter). The response will contain information about matching documents, including their scores and source data (depending on your index mapping)"}

 "query":{
    "match_phrase":{
        "title":{"query":"terminal scores", "slop":100}
    }
 }

 Return one result
 


```
**the closer the words to the query, the higher the score**