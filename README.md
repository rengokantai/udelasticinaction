#### udelasticinaction
#####2
######install head and marvel
go to bin folder
```
plugin -i mobz/elasticsearch-head
plugin -i elasticsearch/marvel/latest
```
#####3
######create a document in elasticsearch
open this link:
```
http://localhost:9200/_plugin/marvel/sense/index.html
```
PUT format: /_index/_type/_id.  
Other metadata: _version, created

insert some docs: (the id part can be omitted, it will generate 24-char length id
```
PUT /elastic_course/book/1
{
  "title":"Luck",
  "author":"king",
  "score":3.8,
  "synopsis":"xxxxxx",
  "genre":["Fiction","classics","Humor"]
}

PUT /elastic_course/book/2
{
  "title":"Haha",
  "author":"all",
  "score":3.1,
  "synopsis":"yyyyy",
  "genre":["Fiction","Art"]
}
```
search all:
```
GET elastic_course/book/_search
```
######gettting from elasticsearch
get all fields
```
GET elastic_course/book/1
```
get specific fields
```
GET elastic_course/book/1?_source=title,author
```

multiple get:  
first->naive
```
GET /_mget
{
  "docs":[
    {
      "_index":"elastic_course",
      "_type":"book",
      "_id":1
    },
    {
        "_index":"elastic_course",
      "_type":"book",
      "_id":2
    }
  ]
}
```
improved:
```
GET /elastic_course/book/_mget
{
  "docs":[
    {
      "_id":1
      },    {
      "_id":2
      },    {
      "_id":3
      }]
}
```
Or even shorter
```
GET /elastic_course/book/_mget
{
  "ids":[1,2,3]
}
```
######Check if exists
For example, using postman
```
localhost:9200/elastic_course/book/2 (get)
```
create doc if index not exist,for exp
```
PUT /elastic_course/book/2/_create
````
######Delete document in es
basic:
```
DELETE elastic_course/book/2
```
first, search
```
GET elatic_course/book/_search
{
  "query":{
    "match":{
      "genre":"Classics"
    }
  }
```
turn this to delete query:
```
DELETE elatic_course/book/_query
{
  "query":{
    "match":{
      "genre":"Classics"
    }
  }
```
######concurrency in es
version number: index=1 update=2 delete=3
```
GET /elastic_course/book/2?version=1   //may return error is updated
```
######update documents in es
full update(or partial): delete document->index new document (document are immutable),same as new update
partial update is completed in a shard,faster
```
POST /elastic_course/book/1/_update
{
  "doc":{
    "synopsis":"kkkk"
  }
}
```
```
POST /elastic_course/book/1/_update
{
  "doc":{
    "genre":["Fiction","Science"]
  }
}
```
######bulk requests in es
syntax:{action:{metadata}} {request body}  
action: create index delete update  
For request body:
- create, index: full doc  
- update: partial doc  
- delete: none  
a example:
```
POST /_bulk
{"create":{"_index":"elastic_course","_type":"book","_id":1}}
    {"title":"msms","author":"lack"}
    {"update":{"_index":"elastic_course","_type":"book","_id":1}}
  {"doc":{"genre":["Fiction","Science"]}}
      {"update":{"_index":"elastic_course","_type":"book","_id":1 }}
  {"doc":{"score":4.1}}
  ```
  or simpler:
  ```
POST /elastic_course/book/_bulk
{"create":{"_id":1}}
    {"title":"msms","author":"lack"}
    {"update":{"_id":1}}
  {"doc":{"genre":["Fiction","Science"]}}
      {"update":{"_id":1 }}
  {"doc":{"score":4.1}}
  ```
  test
  ```
  GET /elastic_course/book/1/
  ```
  choose buffersize: normal 1k-5k. short doc: higher buffer
  #####4
  ######mapping in es
  basic:
  ```
  GET /elastic_course/book/_mapping
  ```
  if dont specify a mapping,es will guess from data.  
  simple field types(byte short integer long- float double boolean[true] string date[12-12-1999])
  
  ######buildin analyzer types
  standard,simple,whitespace,stop,keyword,pattern,language,snowball  
  Example:
  ```
    GET /_analyze?analyzer=standard    // or simple
{
We want to do test
}
```
######custom mappings for simple field types
index: analyzed(full text), not_analyzed(exact value),no(skip this field)  
for string, 'analyzed' is available. For others:no(wont be used in search),not_analyzed(exact value)  
combine index and analyzer:  
Ex:
```
"index":"analyzed",
"analyzer":"english"
```
we can update mapping for a new field and index  
we cannot update mappng for fields of already indexed documents  
Ex
```
PUT elastic_course
{
  "mappings": {
    "book":{
      "properties": {
        "author":{
          "type":"string",
        "index":"not_analyzed"
          
        },
        "genre":{
          "type":"string",
        "index":"not_analyzed"},
        "score":{
          "type": "double"
        },
          "synopsis":{
          "type": "string",
          "index": "analyzed",
          "analyzer": "english"
        },
        "title":{
          "type": "string"
        }
      }
    }
  }
}
```
search by query
```
GET /elastic_course/book/_search?q=author:Tim+Dave
```
######Custom mappings 
```
PUT elastic_coursea
{
  "mappings": {
    "book":{
      "properties": {
        "author":{
          "type":"object",
          "properties": {
            "birth_place":{"type":"string","index":"not_analyzed"},
            "birth_date":{"type":"date","index":"not_analyzed"},
            "death_place":{"type":"date","index":"not_analyzed"},
            "gender":{"type":"string","index":"not_analyzed"},
            "bio":{"type":"string","index":"analyzed","analyzer": "english"}
          }
        },
        "genre":{
          "type":"string",
        "index":"not_analyzed"},
        "score":{
          "type": "double"
        },
          "synopsis":{
          "type": "string",
          "index": "analyzed",
          "analyzer": "english"
        },
        "title":{
          "type": "string"
        }
      }
    }
  }
}
```
insert
```
PUT /elastic_coursea/book/8
{
  "title":"Haha",
  "author":{
     "birth_place":"US",
     "birth_date":"1934-01-04",
            "death_place":"1934-05-04",
            "gender":"male",
            "bio":"xxx"
  },
  
  "score":3.1,
  "synopsis":"yyyyy",
  "genre":["Fiction","Art"]
}
```
test
```
GET /elastic_coursea/book/_search?q=author.birth_place:US
```
And this will not return anything(date must be full match)
```
GET /elastic_coursea/book/_search?q=author.birth_date:1934
```
#####5
######search with query string
using double quote
```
GET /elastic_course/book/_search?q=author:"Tim Dave"
```
Advanced: + is like AND , - is like AND NOT.  
We have to url encode the query
```
GET elastic_course/book/_search?q=%2Bauthor%3A%22King%22%2Bgenre%3A%22Fiction%22 (must contain space between query)
  // +author:"King"  =>encode = %2Bauthor%3A%22King%22
  //+genre:"Fiction"=>  %2Bgenre%3A%22Fiction%22
```
Greater than
```
GET elastic_course/book/_search?q=score:>4        //score>4
```
A Or B 
```
//this query search record either with score>4 and contains kkkk or yyyy in synopsis
+score:>4 +synopsis:(kkkk yyyyy)  //must contain a space between queries.
+score:>4+synopsis:(kkkk yyyyy) //incorrect
GET elastic_course/book/_search?q=%2Bscore%3A%3E4%20%2Bsynopsis%3A(kkkk%20yyyyy)  //actual
```
other query: ~,[2 TO 4]..
######Searching with query DSL
most basic
```
GET _search{}
GET elastic_course/_search{}
GET elastic_course/book/_search{}
```
search all items of multiple types
```
GET elastic_course/book,movie/_search{}
```
regex
```
GET elastic*/book,movie/_search{}
```
match_all: when you need all docs
```
GET elastic_course/book/_search
{
  "query": {
    "match_all": {
      
    }
  }
}
```
match: one field. and multi_match ( same query on several fields)
```
GET elastic_course/book/_search
{
  "query": {
    "match": {
      "synopsis": "kkkk"
    }
  }
}
```
```
GET elastic_course/book/_search
{
  "query": {
    "multi_match": {
      "query": "kkkk",
      "fields": ["title","synopsis"]
    }
  }
}
```
######filters
must:given query must match  
must_not:must not match  
should: at least one of them must match  
```
GET elastic_course/book/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {
          "genre": "Fiction"
        }}
      ],
      "must_not": [
        {"match": {
          "author": "King"
        }}
      ]
    }
  }
}
```

```
GET elastic_course/book/_search
{  
   "query":{  
      "bool":{  
         "must":[  
            {  
               "range":{  
                  "score":{  
                     "gte":3,
                     "lte":100
                  }
               }
            }
         ],
         "should":[  
            {  
               "match":{  
                  "synopsis":"kkkk"
               }
            },
            {  
               "match":{  
                  "author":"all"
               }
            }
         ],
         "minimum_number_should_match":1   //(at least)1 of 2  should criteria must match
      }
   }
}
```
Nested filter
```
GET elastic_course/book/_search
{  
   "query":{  
      "bool":{  
         "should":[  
            {  
               "match":{  
                  "genre":"Fiction"
               }
            },
            {  
               "bool":{  
                  "must":[  
                     {  
                        "match":{  
                           "author":"all"
                        }
                     }
                  ],
                  "must_not":[  
                     {  
                        "match":{  
                           "title":"xxxxxx"
                        }
                     }
                  ]
               }
            }
         ],
         "minimum_number_should_match":1
      }
   }
}
```
######Queries and filters combined
general syntax:
```
{"query":{
"filtered":{
"query":{},
"filter":{}
}}}
```
Ex
```
GET elastic_course/book/_search
{  
   "query":{  
      "filtered":{  
         "query":{  
            "match":{  
               "synopsis":"kkkk"
            }
         },
         "filter":{  
            "range":{  
               "score":{  
                  "gte":3,
                  "lte":20
               }
            }
         }
      }
   }
}
```
######validating queries
send the query and get back the result and see if there is an error,or use validate endpoint  
```
GET elastic_coursea/book/_validate/query
{
  "query":{
    "match":{ 
      "genre":"Fiction"
    }
  }
}
```
analyze an error
```
GET elastic_course/book/_validate/query?explain
{
  "query":{
    "xxx":{ //incorrect
      "genre":"Fiction"
    }
  }
}
```
######pagination
you have to set 2 parameters.  
size(limit number of  documents default=10) from(number of  documents to be skipped default=0)  
Ex:
```
//from =starting point  size=each page show numbers
GET elastic_course/book/_search?from=0&size=1
```
#####6
######sorting on exactvalue fields
Ex1:single field
```
GET elastic_course/book/_search
{
  "filter":{
    "term":{
      "genre":"Science Fiction"
    }
  },
  
  "sort": "score"
}
```
Ex2 multiple field
```
GET elastic_course/book/_search
{
  "filter":{
    "term":{
      "genre":"Science Fiction"
    }
  },
  
  "sort": [
    {
      "score": {
        "order": "desc"
      }
    },
    {
      "author": {
        "order": "desc"
      }
    }
  ]
}
```
######sorting on fulltext
```
DELETE elastic_course
```
And review chapter 4 we have
```
PUT elastic_course
{
  "mappings": {
    "book":{
      "properties": {
        "author":{
          "type":"string",
        "index":"not_analyzed"
          
        },
        "genre":{
          "type":"string",
        "index":"not_analyzed"},
        "score":{
          "type": "double"
        },
          "synopsis":{
          "type": "string",
          "index": "analyzed",
          "analyzer": "english"
        },
        "title":{
          "type": "string"
        }
      }
    }
  }
}
```
Add filed in synopsis:
```
PUT elastic_coursea
{
  "mappings": {
    "book":{
      "properties": {
        "author":{
          "type":"string",
        "index":"not_analyzed"
          
        },
        "genre":{
          "type":"string",
        "index":"not_analyzed"},
        "score":{
          "type": "double"
        },
          "synopsis":{
          "type": "string",
          "analyzer": "english"
          , "fields": {
            "not_analyzed":{
              "type": "string",
              "index": "not_analyzed"   //Must memorize
            }
          }
        },
        "title":{
          "type": "string"
        }
      }
    }
  }
}
```
Then sort base on `not_analyzed`
```
GET elastic_coursea/book/_search
{
"query": {
  "match_all": {}
},
"sort": [
  {
    "synopsis.not_analyzed": {
      "order": "asc"
    }
  }
]
}
```
######Relevance in documents
Note, higher the _score, higher relevance of the document.  
First try
```
GET elastic_coursea/book/_search?explain
{
"query": {
  "match": {"synopsis":"novel"}
}
}
```
and we can see the feedback
```
"details": [
  {
   "value": 0.30685282,
    "description": "fieldWeight in 0, product of:",
      "details": [
          {
             "value": 1,
             "description": "tf(freq=1.0), with freq of:",
             "details": [
                {
                   "value": 1,  //how many matches
                   "description": "termFreq=1.0"
                }
             ]
          },
          {
             "value": 0.30685282,
             "description": "idf(docFreq=1, maxDocs=1)"
          },
          {
             "value": 1,       //higher value means % of words in the paragraph are matched
             "description": "fieldNorm(doc=0)"
          }
       ]
    }
  ]
```
