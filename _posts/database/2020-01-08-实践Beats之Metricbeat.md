---
layout: post
categories: 数据库
tags: Database ELK Elasticsearch Beats Metricbeat
---

Download

```shell
curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.5.1-linux-x86_64.tar.gz
tar xzvf metricbeat-7.5.1-linux-x86_64.tar.gz
```

Set up the system module

```
./metricbeat modules enable system
```

Set up the initial environment

```
./metricbeat setup -e
```

Start Metricbeat

```
./metricbeat -e
```

visit the [Metricbeat system overview dashboard](http://localhost:5601/app/kibana#/dashboard/Metricbeat-system-overview-ecs)