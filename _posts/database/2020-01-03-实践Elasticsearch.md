---
layout: post
categories: 数据库
tags: Database 读写分离
---

[Elastic英文社区](https://discuss.elastic.co/c/elasticsearch/)

[Elasticsearch中文社区](https://elasticsearch.cn/)

## 名词解释

索引（index），好比MySQL里面的schema

类型（type），好比数据库里面的表，字段组成类似

ID，文档的唯一编号，自定义或由Elasticsearch生成

集群

节点：主节点、数据节点

分片：主分片（前期规划固定，后期不可更改），副本分片（后期可以更改）。可以按照索引维度设置。



*技术上来说，一个主分片最大能够存储 Integer.MAX_VALUE - 128 个文档*

文档

域（field）

分析器

词条（token）

映射

## 安装配置

```shell
$ ./elasticsearch -h

Option                Description                                               
------                -----------                                               
-E <KeyValuePair>     Configure a setting                                       
-V, --version         Prints Elasticsearch version information and exits        
-d, --daemonize       Starts Elasticsearch in the background                    
-h, --help            Show help                                                 
-p, --pidfile <Path>  Creates a pid file in the specified path on start         
-q, --quiet           Turns off standard output/error streams logging in console
-s, --silent          Show minimal output                                       
-v, --verbose         Show verbose output  
```



### single

elasticsearch.yml

```
network.host: 0.0.0.0
discovery.seed_hosts: ["127.0.0.1", "[::1]"]
```



### cluster

```shell
$ ./elasticsearch -Epath.data=data -Epath.logs=log
$ ./elasticsearch -Epath.data=data2 -Epath.logs=log2
$ ./elasticsearch -Epath.data=data3 -Epath.logs=log3
```



```shell
$ curl -X GET "localhost:9200/_cat/nodes?v"
```



```shell
$ curl -X GET "localhost:9200/_cat/health?v&pretty"
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1578366048 03:00:48  elasticsearch green           3         3      0   0    0    0        0             0                  -                100.0%
```

## index

创建索引

```shell
$ curl -Xput "http://192.168.0.22:9200/test_index" -H 'Content-Type: application/json' -d'
{
	"mappings": {
		"properties": {
			"goodId": {
				"type": "long"
			},
			"goodName": {
				"type": "text",
				"analyzer": "ik_max_word"
			}
		}
	}
}
'
```



## data

向索引添加数据，如果索引不存在会自动创建索引（`text类型`的会自动创建`{field}.keyword`）

### single

```shell
$ curl -X PUT "localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "John Doe"
}
'
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}

```



```shell
$ curl -X GET "localhost:9200/_cat/health?v&pretty"
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1578366293 03:04:53  elasticsearch green           3         3      2   1    0    0        0             0                  -                100.0%
```



```shell
$ curl -X GET "localhost:9200/customer/_doc/1?pretty"
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "John Doe"
  }
}
```



### bulk

accounts.json

```json
{"index":{"_id":"1"}}
{"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
{"index":{"_id":"6"}}
{"account_number":6,"balance":5686,"firstname":"Hattie","lastname":"Bond","age":36,"gender":"M","address":"671 Bristol Street","employer":"Netagy","email":"hattiebond@netagy.com","city":"Dante","state":"TN"}
...
```



```shell
$ curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_bulk?pretty&refresh" --data-binary "@accounts.json"
{
  "took" : 8384,
  "errors" : false,
  "items" : [
    {
      "index" : {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "1",
        "_version" : 1,
        "result" : "created",
        "forced_refresh" : true,
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 0,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "index" : {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "6",
        "_version" : 1,
        "result" : "created",
        "forced_refresh" : true,
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 1,
        "_primary_term" : 1,
        "status" : 201
      }
    },
...
```



```shell
$ curl "localhost:9200/_cat/indices?v"
health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   bank     ArXWxoNXQXi-lLEOaHeCDw   1   1       1000            0    857.7kb        443.3kb
green  open   customer 8jczugDEQgyCIp2vYl5Ysg   1   1          1            0        7kb          3.5kb
```



```shell
$ curl -X GET "localhost:9200/_cat/health?v&pretty"
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1578374414 05:20:14  elasticsearch green           3         3      4   2    0    0        0             0                  -                100.0%
```

## search

### match_all

```shell
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
'
{
  "took" : 52,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "0",
        "_score" : null,
        "_source" : {
          "account_number" : 0,
          "balance" : 16623,
          "firstname" : "Bradshaw",
          "lastname" : "Mckenzie",
          "age" : 29,
          "gender" : "F",
          "address" : "244 Columbus Place",
          "employer" : "Euron",
          "email" : "bradshawmckenzie@euron.com",
          "city" : "Hobucken",
          "state" : "CO"
        },
        "sort" : [
          0
        ]
      },
...
```

#### pagination

```shell
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ],
  "from": 10,
  "size": 10
}
'
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1000,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "10",
        "_score" : null,
        "_source" : {
          "account_number" : 10,
          "balance" : 46170,
          "firstname" : "Dominique",
          "lastname" : "Park",
          "age" : 37,
          "gender" : "F",
          "address" : "100 Gatling Place",
          "employer" : "Conjurica",
          "email" : "dominiquepark@conjurica.com",
          "city" : "Omar",
          "state" : "NJ"
        },
        "sort" : [
          10
        ]
      },

```

### match

```shell
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match": { "address": "mill lane" } }
}
'

```



### match_phrase

```shell
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_phrase": { "address": "mill lane" } }
}
'

```

### highlight

```shell
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_phrase": { "address": "mill lane" } },
  "highlight": {
        "fields" : {
            "address" : {}
        }
    }
}
'
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 9.507477,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "136",
        "_score" : 9.507477,
        "_source" : {
          "account_number" : 136,
          "balance" : 45801,
          "firstname" : "Winnie",
          "lastname" : "Holland",
          "age" : 38,
          "gender" : "M",
          "address" : "198 Mill Lane",
          "employer" : "Neteria",
          "email" : "winnieholland@neteria.com",
          "city" : "Urie",
          "state" : "IL"
        },
        "highlight" : {
          "address" : [
            "198 <em>Mill</em> <em>Lane</em>"
          ]
        }
      }
    ]
  }
}
```



### bool

```shell
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
'

```



## aggregation

### term

```
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
'

```

### avg

```
$ curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
'

```

## analyzer

analyzer分index analyzer和search analyzer，当search analyzer未单独设定时使用index analyzer。

[Index and search analysis | Elasticsearch Guide | Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/7.7/analysis-index-search-time.html)

默认analyzer是standard，中文是按汉字一个一个拆成token，比如：“中华人民”拆成“中”，“华”，“人”和”民“四个token，英文句号（.）和下划线（_）不作为拆分符，比如“org.springframework.boot”是一个token，“ik_max_word”是一个token

```
$ curl -XGET "http://192.168.0.22:9200/_analyze" -H 'Content-Type: application/json' -d'{   "text": "中华人民共和国国歌"}'

{
	"tokens": [
		{
			"token": "中",
			"start_offset": 0,
			"end_offset": 1,
			"type": "<IDEOGRAPHIC>",
			"position": 0
		},
		{
			"token": "华",
			"start_offset": 1,
			"end_offset": 2,
			"type": "<IDEOGRAPHIC>",
			"position": 1
		},
		{
			"token": "人",
			"start_offset": 2,
			"end_offset": 3,
			"type": "<IDEOGRAPHIC>",
			"position": 2
		},
		{
			"token": "民",
			"start_offset": 3,
			"end_offset": 4,
			"type": "<IDEOGRAPHIC>",
			"position": 3
		},
		{
			"token": "共",
			"start_offset": 4,
			"end_offset": 5,
			"type": "<IDEOGRAPHIC>",
			"position": 4
		},
		{
			"token": "和",
			"start_offset": 5,
			"end_offset": 6,
			"type": "<IDEOGRAPHIC>",
			"position": 5
		},
		{
			"token": "国",
			"start_offset": 6,
			"end_offset": 7,
			"type": "<IDEOGRAPHIC>",
			"position": 6
		},
		{
			"token": "国",
			"start_offset": 7,
			"end_offset": 8,
			"type": "<IDEOGRAPHIC>",
			"position": 7
		},
		{
			"token": "歌",
			"start_offset": 8,
			"end_offset": 9,
			"type": "<IDEOGRAPHIC>",
			"position": 8
		}
	]
}
```

ik是第三方analyzer

ik_smart

```
$ curl -XGET "http://192.168.0.22:9200/_analyze" -H 'Content-Type: application/json' -d'{   "text": "中华人民共和国国歌",   "analyzer": "ik_smart" }'

{
	"tokens": [
		{
			"token": "中华人民共和国",
			"start_offset": 0,
			"end_offset": 7,
			"type": "CN_WORD",
			"position": 0
		},
		{
			"token": "国歌",
			"start_offset": 7,
			"end_offset": 9,
			"type": "CN_WORD",
			"position": 1
		}
	]
}
```

ik_max_word

```
$ curl -XGET "http://192.168.0.22:9200/_analyze" -H 'Content-Type: application/json' -d'{   "text": "中华人民共和国国歌",   "analyzer": "ik_max_word" }'

{
	"tokens": [
		{
			"token": "中华人民共和国",
			"start_offset": 0,
			"end_offset": 7,
			"type": "CN_WORD",
			"position": 0
		},
		{
			"token": "中华人民",
			"start_offset": 0,
			"end_offset": 4,
			"type": "CN_WORD",
			"position": 1
		},
		{
			"token": "中华",
			"start_offset": 0,
			"end_offset": 2,
			"type": "CN_WORD",
			"position": 2
		},
		{
			"token": "华人",
			"start_offset": 1,
			"end_offset": 3,
			"type": "CN_WORD",
			"position": 3
		},
		{
			"token": "人民共和国",
			"start_offset": 2,
			"end_offset": 7,
			"type": "CN_WORD",
			"position": 4
		},
		{
			"token": "人民",
			"start_offset": 2,
			"end_offset": 4,
			"type": "CN_WORD",
			"position": 5
		},
		{
			"token": "共和国",
			"start_offset": 4,
			"end_offset": 7,
			"type": "CN_WORD",
			"position": 6
		},
		{
			"token": "共和",
			"start_offset": 4,
			"end_offset": 6,
			"type": "CN_WORD",
			"position": 7
		},
		{
			"token": "国",
			"start_offset": 6,
			"end_offset": 7,
			"type": "CN_CHAR",
			"position": 8
		},
		{
			"token": "国歌",
			"start_offset": 7,
			"end_offset": 9,
			"type": "CN_WORD",
			"position": 9
		}
	]
}
```

[Specify an analyzer | Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/7.7/specify-analyzer.html#specify-index-time-default-analyzer)

指定查询的analyzer

```
GET /test_index4/_search
{
  "query": {
    "match": {
      "goodName":{
        "query": "中华人民共和国国歌"
     //          ,"analyzer": "ik_max_word"
    ,"analyzer": "ik_smart"
      }
    }
  }
}
```





## 参考资料

《ElasticSearch技术解析与实战》，2.3.0

《ElasticSearch实战》，1.5

《从Lucene到ElasticSearch：全文检索实战》

《ElasticSearch集成Hadoop最佳实践》，1.7.1

[《ElasticSearch权威指南》](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)，[Elasticsearch: The Definitive Guide](https://www.elastic.co/guide/en/elasticsearch/guide/current/index.html)，2.x，已经停止维护

[Elasticsearch Reference](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)

《深入理解ElasticSearch》，4.0

《大数据搜索与挖掘及可视化管理方案1/2/3/4版》

《ElasticSearch源码解析与优化实战》，6.1.2

《ELK stack权威指南》