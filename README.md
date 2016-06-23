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




#####chapter3
######Querying Elasticsearch
create a file name map.json
```
{
  "book" : {
    "properties" : {
      "author" : {
        "type" : "string"
      },
      "characters" : {
        "type" : "string"
      },
      "copies" : {
        "type" : "long",
        "ignore_malformed" : false
      },
      "otitle" : {
        "type" : "string"
      },
      "tags" : {
        "type" : "string",
        "index" : "not_analyzed"
      },
      "title" : {
        "type" : "string"
      },
      "year" : {
        "type" : "long",
        "ignore_malformed" : false,
        "index" : "analyzed"
      },
      "available" : {
        "type" : "boolean"
      }
    }
  }
}
```
create an index   note the curl syntax from json file, use @
```
curl -XPOST 'localhost:9200/library'
curl -XPUT 'localhost:9200/library/book/_mapping' -d @mapping.json   
```

insert data, using -s param, line by line. two line per group
```
{ "index": {"_index": "library", "_type": "book", "_id": "1"}}
{ "title": "All Quiet on the Western Front","otitle": "Im Westen nichts Neues","author": "Erich Maria Remarque","year": 1929,"characters": ["Paul Bäumer", "Albert Kropp", "Haie Westhus", "Fredrich Müller", "Stanislaus Katczinsky", "Tjaden"],"tags": ["novel"],"copies": 1, "available": true, "section" : 3}
{ "index": {"_index": "library", "_type": "book", "_id": "2"}}
{ "title": "Catch-22","author": "Joseph Heller","year": 1961,"characters": ["John Yossarian", "Captain Aardvark", "Chaplain Tappman", "Colonel Cathcart", "Doctor Daneeka"],"tags": ["novel"],"copies": 6, "available" : false, "section" : 1}
{ "index": {"_index": "library", "_type": "book", "_id": "3"}}
{ "title": "The Complete Sherlock Holmes","author": "Arthur Conan Doyle","year": 1936,"characters": ["Sherlock Holmes","Dr. Watson", "G. Lestrade"],"tags": [],"copies": 0, "available" : false, "section" : 12}
```
import
```
curl -s -XPOST 'localhost:9200/_bulk' --data-binary @documents.json
```

simple search query (note the syntax: query->query_string->query)
```
curl -XGET 'localhost:9200/library/book/_search?pretty' -d '{ "query" : { "query_string" : { "query" : "title:crime" } } }'
```
from and size (equiv to skip and limit)
```
curl -XGET 'localhost:9200/library/book/_search?pretty' -d '{"from":10,"size":5, "query" : { "query_string" : { "query" : "title:crime" } } }'
```
return versioning number
```
curl -XGET 'localhost:9200/library/book/_search?pretty' -d '{ "version" : true, "query" : { "query_string" : { "query" : "title:crime" } } }'
```
min_score( I think might be deprecated in future)
```
curl -XGET 'localhost:9200/library/book/_search?pretty' -d '{ "min_score" : 0.75, "query" : { "query_string" : { "query" : "title:crime" } } }'
```
choose return fields
```
curl -XGET 'localhost:9200/library/book/_search?pretty' -d '{ "fields" : [ "title", "year" ], "query" : { "query_string" : { "query" : "title:crime" } } }'
```

[Difference between _source and fields](https://discuss.elastic.co/t/difference-between--source-and-fields-projections/22927)
filter source
```
curl -XGET 'localhost:9200/library/book/_search?pretty' -d '{ "_source" : false, "query" : { "query_string" : { "query" : "title:crime" } } }'
curl -XGET 'localhost:9200/library/book/_search?pretty' -d '{ "_source" : ["title", "otitle"], "query" : { "query_string" : { "query" : "title:crime" } } }'
```
include and exclude some part
```
curl -XGET 'localhost:9200/library/book/_search?pretty' -d '{ "_source" : { "include" : [ "t*"], "exclude" : ["title"] }, "query" : { "query_string" : { "query" : "title:crime" } } }'
```
######Understanding the querying process
search_type:
```
query_then_fetch
dfs_query_then_fetch
```
search_type exp:
```
curl -XGET 'localhost:9200/library/book/_search?pretty&search_type=query_then_fetch' -d '{ "query" : { "term" : { "title" : "crime" } } }'
```
preference:  
_primary,_primary_first,_replica,_replica_first,_local,only_node:id,....
```
curl -XGET 'localhost:9200/library/_search?pretty&preference=_local' -d '{ "query" : { "term" : { "title" : "crime" } } }'
```

search shard
```
curl -XGET 'localhost:9200/library/_search_shards?pretty' -d '{"query":"match_all":{}}'
```



#####Chapter 6
######Scripting capabilities
```
_doc = org.elasticsearch.search.loolup.LeafDocLookup                                        
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

######Controlling cluster rebalancing
```
cluster.routing.allocation.allow_rebalance
```
(to be conti)

######CAT api
```
curl -XGET 'localhost:9200/_cat/health'
curl -XGET 'localhost:9200/_cat/health?v'          //verbose output
```
-h: specify return fields
```
curl -XGET 'localhost:9200/_cat/nodes?v&h=name,node.role,load,uptime'
```
recovery status
```
curl -XGET 'localhost:9200/_cat/recovery/blog?v&h=index,shard,time,type,stage,files_percent' 
```
######Warmers (deprecated in es 2.3)
######index aliasing
create and delete index alias
```
curl -XPOST 'http://localhost:9200/_aliases' -d '
{
    "actions" : [
        { "remove" : { "index" : "test1", "alias" : "alias1" } },
        { "add" : { "index" : "test1", "alias" : "alias2" } }
    ]
}'
```
index alias patterns
```
curl -XPOST 'http://localhost:9200/_aliases' -d '
{
    "actions" : [
        { "add" : { "index" : "test*", "alias" : "all_test_indices" } }
    ]
}'
```
same as
```
curl -XPUT 'localhost:9200/test*/_alias/all_test_indices'
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

