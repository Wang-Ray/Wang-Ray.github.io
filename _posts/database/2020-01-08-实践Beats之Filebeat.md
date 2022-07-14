---
layout: post
categories: 数据库
tags: Database Beats Filebeat 
---

[Filebeat Reference](https://www.elastic.co/guide/en/beats/filebeat/current/index.html)

## install

Download

```shell
$ curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.5.1-linux-x86_64.tar.gz
$ tar xzvf filebeat-7.5.1-linux-x86_64.tar.gz
```

【可选】根据需要开启相应的module，比如system、mysql等

```shell
$ ./filebeat modules enable system
```

初始化

```shell
$ ./filebeat setup -e
```

启动

```shell
$ ./filebeat -e
```



start-filebeat.sh

```shell
#!/bin/bash

nohup ./filebeat -e -c filebeat.yml -d "publish" >/dev/null 2>&1 &
```



## input

[Configure inputs](https://www.elastic.co/guide/en/beats/filebeat/7.5/configuration-filebeat-options.html)

### 多行

```yaml
multiline:
  # 不以"yyyy-MM-dd"这种日期开始的行与前一行合并
  pattern: ^\d{4}-\d{1,2}-\d{1,2}
  negate: true
  match: after
```

## modules

[filebeat-modules](https://www.elastic.co/guide/en/beats/filebeat/7.5/filebeat-modules.html)

开启指定module，实际是修改`modules.d`下的对应文件

```shell
$ ./filebeat modules enable mysql
```

查询开启或关闭状态的module

```
$ ./filebeat modules list
```



## output

```
{
          "@version" : "1",
          "log" : {
            "file" : {
              "path" : "/app/third-party-service/log/zhkj-webapi.log"
            },
            "offset" : 351812
          },
          "message" : """[2021-09-03 12:18:51.723] [http-nio-8080-exec-1] [INFO] [] [c.z.k.a.m.i.service.CabinetAsynNotify] - 通知转发: {"actionType":2,"code":200,"deviceId":"868089050237340","gridNo":"02&01","reqNo":"200000000561320210519"}""",
          "ecs" : {
            "version" : "1.5.0"
          },
          "host" : {
            "containerized" : false,
            "os" : {
              "platform" : "centos",
              "version" : "7 (Core)",
              "kernel" : "3.10.0-957.el7.x86_64",
              "codename" : "Core",
              "name" : "CentOS Linux",
              "family" : "redhat"
            },
            "architecture" : "x86_64",
            "mac" : [
              "52:54:00:34:d7:d4"
            ],
            "name" : "dev-1-202",
            "ip" : [
              "192.168.1.202",
              "fe80::39ca:ea1d:2de2:c04",
              "fe80::c31:cdf:e1b9:c9fe",
              "fe80::99df:f4da:4cfa:c141"
            ],
            "hostname" : "dev-1-202",
            "id" : "4019616339ee41f2a235a10f31397ac7"
          },
          "@timestamp" : "2021-09-03T04:18:52.572Z",
          "agent" : {
            "type" : "filebeat",
            "version" : "7.7.1",
            "ephemeral_id" : "b1d610c2-188f-4515-b03e-e836bc78b157",
            "hostname" : "dev-1-202",
            "id" : "4c3f3598-cb25-4098-979d-b1dbfeb54e54"
          },
          "input" : {
            "type" : "log"
          },
          "tags" : [
            "beats_input_codec_plain_applied"
          ]
        },
        "sort" : [
          1630642732572
        ]
      }
```



### logstash

### elasticsearch

