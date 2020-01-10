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

### logstash

### elasticsearch

