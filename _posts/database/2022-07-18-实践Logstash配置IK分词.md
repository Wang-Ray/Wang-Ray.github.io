---
layout: post
categories: 数据库
tags: Database Logstash elastic elk IK
---



logstash-ik.json

```json
  {
    "order" : 99,
    "version" : 1,
    "index_patterns" : [
      "*"
    ],
    "settings" : {
      "index" : {
        "number_of_shards" : "1",
        "refresh_interval" : "5s"
      }
    },
    "mappings" : {
      "dynamic_templates" : [
        {
          "message_field" : {
            "path_match" : "message",
            "mapping" : {
              "norms" : false,
              "type" : "text"
            },
            "match_mapping_type" : "string"
          }
        },
        {
          "string_fields" : {
            "mapping" : {
              "norms" : false,
              "type" : "text",
              "analyzer": "ik_max_word",
              "search_analyzer": "ik_smart",
              "fields" : {
                "keyword" : {
                  "ignore_above" : 256,
                  "type" : "keyword"
                }
              }
            },
            "match_mapping_type" : "string",
            "match" : "*"
          }
        }
      ],
      "properties" : {
        "@timestamp" : {
          "type" : "date"
        },
        "geoip" : {
          "dynamic" : true,
          "properties" : {
            "ip" : {
              "type" : "ip"
            },
            "latitude" : {
              "type" : "half_float"
            },
            "location" : {
              "type" : "geo_point"
            },
            "longitude" : {
              "type" : "half_float"
            }
          }
        },
        "@version" : {
          "type" : "keyword"
        }
      }
    },
    "aliases" : { }
  }
```



```
output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "dev-log-%{+YYYY.MM}"
    # 定义模板名称
    template_name => "myik"
    # 模板所在位置
    template => "/app/logstash-7.7.1/sync/logstash-ik.json"
    # 重写模板
    template_overwrite => true
    # 默认为 true，false 关闭 logstash 自动管理模板的功能，如果自定义模板，则设置为 false
    #manage_template => false
    #user => "elastic"
    #password => "changeme"
  }
}
```

