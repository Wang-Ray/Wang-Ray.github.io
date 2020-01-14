---
layout: post
categories: 数据库
tags: Database 读写分离
---

[Elastic英文社区](https://discuss.elastic.co/c/elasticsearch/)

[Elasticsearch中文社区](https://elasticsearch.cn/)



索引（index），好比MySQL里面的schema

类型（type），好比数据库里面的表，字段组成类似

ID，文档的唯一编号，自定义或由Elasticsearch生成

集群

节点：主节点、数据节点

分片：主分片（前期规划固定，后期不可更改），副本分片（后期可以更改）。可以按照索引维度设置。



*技术上来说，一个主分片最大能够存储 Integer.MAX_VALUE - 128 个文档*

## cluster

```shell
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