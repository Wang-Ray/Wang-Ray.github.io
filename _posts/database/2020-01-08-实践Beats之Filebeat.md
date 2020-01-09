---
layout: post
categories: 数据库
tags: Database 读写分离
---

Download

```shell
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.5.1-linux-x86_64.tar.gz
tar xzvf filebeat-7.5.1-linux-x86_64.tar.gz
```

Set up the system/nginx/mysql module

```
./filebeat modules enable system nginx mysql
```

Set up the initial environment

```
./filebeat setup -e
```

Start Filebeat

```
./filebeat -e
```

visit the [Metricbeat system overview dashboard](http://localhost:5601/app/kibana#/dashboard/Metricbeat-system-overview-ecs)