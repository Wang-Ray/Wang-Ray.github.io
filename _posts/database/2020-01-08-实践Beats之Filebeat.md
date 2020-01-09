---
layout: post
categories: 数据库
tags: Database Beats Filebeat 
---

[Filebeat Reference](https://www.elastic.co/guide/en/beats/filebeat/current/index.html)

Download

```shell
$ curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.5.1-linux-x86_64.tar.gz
$ tar xzvf filebeat-7.5.1-linux-x86_64.tar.gz
```

Set up the system module

```shell
$ ./filebeat modules enable system
```

Set up the initial environment

```shell
$ ./filebeat setup -e
```

Start Filebeat

```shell
$ ./filebeat -e
```

