#### pkelasticsrv3ed
#####Chapter 1. Getting Started with Elasticsearch Cluster
post test
```
curl -XPUT 'http://localhost:9200/blog/article/1' -d '{"title": "New version", "content": "2.2", "priority": 10, "tags": ["announce", "elasticsearch", "release"] }'
```
get
```
curl -XGET 'localhost:9200/blog/article/1?pretty'
```
update
```
curl -XPOST 'http://xx.xx.xx:9200/blog/article/1/_update' -d '{
 "doc" : {
  "content" : "2.3"
 }
}'
```
or use content.(report error,)
```
"type": "script_exception",
"reason": "scripts of type [inline], operation [update] and lang [groovy] are disabled"
```
solution, in data node:elasticsearch.yml
```
script.inline: on 
script.indexed: on
```
test
```
curl -XPOST 'http://localhost:9200/blog/article/2/_update' -d '{ 
 "script" : "ctx._source.priority += 1",
 "upsert" : {
  "title" : "Empty document",
  "priority" : 0,
  "tags" : ["empty"]
 }
}'
```
update using ctx
```
curl -XPOST 'http://localhost:9200/blog/article/2/_update' -d '{ 
 "script" : "ctx._source.priority += 1"
}'
```
using upsert
```
curl -XPOST 'http://localhost:9200/blog/article/1/_update' -d '{ "doc" : { "count" : 1 }, "doc_as_upsert" : true }'
```
delete
```
curl -XDELETE 'localhost:9200/blog/article/1'
curl -XDELETE 'localhost:9200/blog
```
#####Searching with the URI request query
know the difference
```
curl -XGET 'localhost:9200/books/  //show mappings,..etc
curl -XGET 'localhost:9200/books/_search
curl -XGET 'localhost:9200/books/es/_search
```
run queries from multiple indices
```
curl -XGET 'localhost:9200/books,blog/_search // curl -XGET 'localhost:9200/_all/_search
```
handling non existing index
```
curl -XGET 'localhost:9200/books,notexistingindex/_search?pretty&ignore_unavailable=true'
```
first time search
```
curl -XGET 'localhost:9200/books/_search?q=title:elasticsearch'
```
analyze
```
curl -XGET 'localhost:9200/books/_analyze?pretty&field=title' -d 'Elasticsearch Server'
```


#####9
######The gateway and recovery modules
```
gateway.recover_after_nodes: 6
gateway.recover_after_time: 5m
gateway.expected_nodes: 10
```
