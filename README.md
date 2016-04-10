#### udelasticinaction
######1
######install head and marvel
go to bin folder
```
plugin -i mobz/elasticsearch-head
plugin -i elasticsearch/marvel/latest
```
#####2
######create a document in elasticsearch
open this link:
```
http://localhost:9200/_plugin/marvel/sense/index.html
```
PUT format: /_index/_type/_id.  
Other metadata: _version, created

insert some docs: (the id part can be omitted, it will generate 24-char length id
```
PUT /elastic_coursea/book/1
{
  "title":"Luck",
  "author":"king",
  "score":3.8,
  "synopsis":"xxxxxx",
  "genre":["Fiction","classics","Humor"]
}

PUT /elastic_coursea/book/2
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
localhost:9200/elastic_coursea/book/2 (get)
```
create doc if index not exist,for exp
```
PUT /elastic_coursea/book/2/_create
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
GET /elastic_coursea/book/2?version=1   //may return error is updated
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
