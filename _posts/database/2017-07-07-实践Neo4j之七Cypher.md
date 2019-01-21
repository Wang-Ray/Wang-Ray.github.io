---
layout: post
title: "实践Neo4j之七Cypher"
date: 2017-07-07 09:00:00 +0800
categories: 数据库
tags: database nosql neo4j
---

## Cypher

```cypher
基本格式：
()-[]-()
([identifier][:label][{map-construct}])-[[identifier]:relationShipType[|relationShipType][*n..m]]-([identifier][:label])
(): 代表node(s)
[]: 代表relationShip
-,->,<-: direction
identifier：标识符或变量，可选，如果不指定，则表示anonymous，当指定时，()可省略
label: node的lable，可选，支持多个，例如：(a:User:Admin)
*n..m: 设定关系层次，可选，默认*1，表示一层关系，例如：[:KNOWS*2..5]、[:KNOWS*0..1]、[:KNOWS*..5]、[:KNOWS*]
{map-construct}: 属性过滤，例如：(a {name: 'Andres', sport: 'Brazilian Ju-Jitsu'})
```

### node(s) lookup by ID

```cypher
start user=node(1) return user;
start user=node(1,3) return user;
```

### node(s) lookup using an index

```cypher
//users is index name
start john=node:users(name = "John Johnson") return john
//native Luence query
start john=node:users("name:John Johnson") return john
//native Luence query
start john=node:users("name:John* AND yearOfBirth<1980") return john

//automatice indexing
start john=node:node_auto_index(name = "John Johnson") return john

//schema indexing, USER is a label
match (john:USER) where john.name='John Johnson' return john
```

### getting the results

return **nodes**, **properties**, **relationship**, **path** and **paging**.

```cypher
MATCH (s1:Station)-[r:connected]-(s2:Station)
WHERE s1.name="Euston"
RETURN DISTINCT r.line
```

```java
ExecutionEngine engine = new ExecutionEngine(graphDb);
String cql = "start user=node(1)" + " match (user)-[:HAS_SEEN]->(movie)" + " return movie;";
ExecutionResult result = engine.execute(cql);
for(Map<String,Object> row : result){
	System.out.println("Row:" + row);
}
```

### create

```cypher
//create node
create newuser{
	name: 'Grace Spencer',
	yearOfBirth: 1982,
	email: 'grace@mycompany.com'
}
return newuser;
```



```cypher
start john = node:users(name = "John Johnson"), grace = node(10)
//create relationship
create john-[r:IS_FRIEND_OF]->grace
return r;
```

### delete

```cypher
start grace = node(10)
delete grace
```

### update

```cypher
start john=node:users(name = "John Johnson")
set john.yearOfBirth = 1973
```

### aggregation
聚合

### function
函数

### piping using the with clause
管道

### cypher compatibility
cypher兼容性


