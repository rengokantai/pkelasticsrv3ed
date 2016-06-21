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

avoid auto_index (we can still create index directly) I tried only on master node, but should set in all nodes
```
action.auto_create_index: false     //disable all index
action.auto_create_index: +logs*,-*   //allow index name starting from logs, deny from -
```
assign shards and replicas
```
curl -XPUT http://localhost:9200/blog/ -d '{ "settings" : { "number_of_shards" : 1, "number_of_replicas" : 2 } }'
```


#####9
######The gateway and recovery modules
```
gateway.recover_after_nodes: 6
gateway.recover_after_time: 5m
gateway.expected_nodes: 10
```
######Elasticsearch caches
```
indices.memory.index_buffer_size
indices.memory.max_index_buffer_size
indices.memory.min_index_buffer_size
indices.memory.min_shard_index_buffer_size
```

#####10
######Elasticsearch caches
fielddata cache
```
indices.fielddata.cache.size:  20%
```
shared request cache (caches the results of the queries on each shard)
```
curl -XPUT 'localhost:9200/new_library' -d '{ "settings": { "index.requests.cache.enable": true } }'
```
#####Chapter 11. Scaling by Example
######
Avoiding swapping.Es and Jvm based applications, don't like to be swapped(these applications work best if the operating system doesn't put the memory that they use in the swap space)  
(To access the swapped memory, the operating system will have to read it from the disk, which is slow and which would affect the performance in a very bad way.)
```
bootstrap.mlockall: true
```
another step: edit /etc/sysctl.conf
```
0 (or 1 for emergency)
```
or in fstab, delete line with swap,then
```
swapoff -a
```

file descriptors
```
vim /etc/security/limits.conf
```
```
elasticsearch soft nofile 65536
elasticsearch hard nofile 65536
```
then some linux system
```
vim /etc/pam.d/login
```
edit
```
session required pam_limits.so
```
another way
```
elasticsearch -Des.max-open-files=true
```

virtual memory
```
sysctl -w vm.max_map_count=262144
```
Thread pools  
generic,index,search,suggest,bulk,get ,percolate  
change settings: transient
```
curl -XPUT 'localhost:9200/_cluster/settings' -d '{ "transient" : { "threadpool.index.size" : 100, "threadpool.index.queue_size" : 500 } }'
```
######Horizontal expansion
add a single replica
```
curl -XPUT 'localhost:9200/library/_settings' -d '{ "index" : { "number_of_replicas" : 1 } }'
```
note the difference:
```
curl -XPUT http://localhost:9200/blog/ -d '{ "settings" : { "number_of_shards" : 1, "number_of_replicas" : 2 } }'
```
automatic create replicas  
create
```
curl -XPOST 'localhost:9200/shops/' -d '{ "settings" : { "index" : { "auto_expand_replicas" : "0-all" } } }'
```
update
```
curl -XPUT 'localhost:9200/shops/_settings' -d '{ "index" : { "auto_expand_replicas" : "0-all" } }'
```
create master eligible nodes
```
node.master true
node.data: false
http.enabled: false
```
######Monitoring
install hq
```
/usr/share/elasticsearch/bin/plugin install royrusso/elasticsearch-HQ
```
open browser
```
http://54.210.57.2:9200/_plugin/hq/
```
marvel and kibana
```
/usr/share/elasticsearch/bin/plugin install license
/usr/share/elasticsearch/bin/plugin install marvel-agent
bin/kibana plugin --install elasticsearch/marvel/latest
```
