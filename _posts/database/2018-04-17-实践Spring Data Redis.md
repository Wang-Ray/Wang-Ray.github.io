---
layout: post
title: "实践Spring Data Redis"
date: 2018-04-17 14:05:00 +0800
categories: 数据库
tags: redis nosql sping-data-redis
---



## RedisConnection&RedisConnectionFactory

首推Redis客户端为[Lettuce](https://lettuce.io/)（`spring-boot-starter-data-redis默认已集成`），Jedis也支持（需要自行添加）。

Lettuce：

```xml
<dependency>
	<groupId>io.lettuce</groupId>
	<artifactId>lettuce-core</artifactId>
</dependency>
```

Jedis：

```xml
<dependency>
	<groupId>redis.clients</groupId>
	<artifactId>jedis</artifactId>
</dependency>
```

RedisConnection代表Redis客户端与Redis服务端的连接，一般通过RedisConnectionFactory（LettuceConnectionFactory或JedisConnectionFactory）创建，创建RedisConnectionFactory需要客户端配置（包括连接池（基于`commons-pool2`））和单机、哨兵或集群配置。

### 客户端配置

Lettuce

![LettuceClientConfiguration](/images/LettuceClientConfiguration.png)

Jedis

![JedisClientConfiguration](/images/JedisClientConfiguration.png)



### 连接池

Lettuce之前使用的LettucePool不再支持，已被LettucePoolingClientConfiguration替代

Jedis使用JedisPoolConfig

都是基于commons-pool2实现

单机

RedisStandaloneConfiguration

哨兵

RedisSentinelConfiguration

集群

RedisClusterConfiguration

### 样例

Lettuce

```java
    @Bean
    public RedisConnectionFactory lettuceConnectionFactory() {
        // 单机
        RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration();
        redisStandaloneConfiguration.setHostName("localhost");
        redisStandaloneConfiguration.setPort(6379);

        // 连接池
        GenericObjectPoolConfig poolConfig = new GenericObjectPoolConfig();
        poolConfig.setMinIdle(1);
        poolConfig.setMaxIdle(5);

        // 客户端配置
        LettuceClientConfiguration lettuceClientConfiguration = LettucePoolingClientConfiguration.builder().poolConfig(poolConfig).build();

        return new LettuceConnectionFactory(redisStandaloneConfiguration, lettuceClientConfiguration);
    }
```

Jedis

```java
    @Bean
    public RedisConnectionFactory jedisConnectionFactory() {
        RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration();
        redisStandaloneConfiguration.setHostName("localhost");
        redisStandaloneConfiguration.setPort(6379);
        JedisConnectionFactory jedisConnectionFactory = new JedisConnectionFactory(redisStandaloneConfiguration);
        return jedisConnectionFactory;
    }
```

## RedisTemplate

操作Redis的入口，提供了丰富的操作集，相对于`RedisConnection`提供较低级别的操作，`RedisTemplate`提供了较高级别的抽象。

同时根据Redis的键类型或值类型，提供了多种抽象的操作视图，包括丰富且泛型化的操作集。

| Interface              | Description                              |
| ---------------------- | ---------------------------------------- |
| *Key Type Operations*  |                                          |
| GeoOperations          | Redis geospatial operations like `GEOADD`, `GEORADIUS`,…) |
| HashOperations         | Redis hash operations                    |
| HyperLogLogOperations  | Redis HyperLogLog operations like (`PFADD`, `PFCOUNT`,…) |
| ListOperations         | Redis list operations                    |
| SetOperations          | Redis set operations                     |
| ValueOperations        | Redis string (or value) operations       |
| ZSetOperations         | Redis zset (or sorted set) operations    |
| *Key Bound Operations* |                                          |
| BoundGeoOperations     | Redis key bound geospatial operations.   |
| BoundHashOperations    | Redis hash key bound operations          |
| BoundKeyOperations     | Redis key bound operations               |
| BoundListOperations    | Redis list key bound operations          |
| BoundSetOperations     | Redis set key bound operations           |
| BoundValueOperations   | Redis string (or value) key bound operations |
| BoundZSetOperations    | Redis zset (or sorted set) key bound operations |

### 样例

```java
	@Bean
    RedisTemplate<String, String> redisTemplate() {
        RedisTemplate redisTemplate = new RedisTemplate();
        redisTemplate.setConnectionFactory(jedisConnectionFactory());
        return redisTemplate;
    }

	@Bean
    StringRedisTemplate stringRedisTemplate() {
        StringRedisTemplate stringRedisTemplate = new StringRedisTemplate();
        stringRedisTemplate.setConnectionFactory(jedisConnectionFactory());
        return stringRedisTemplate;
    }
```



```java
    // inject the actual template
    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    // inject the template as valueOperations
    @Resource(name = "redisTemplate")
    private ValueOperations<String, String> valueOps;

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Test
    public void contextLoads() {
    }

    @Test
    public void testRedisTemplate() {
        redisTemplate.opsForValue().set("key1", "value1");
        Assert.assertEquals("value1", redisTemplate.opsForValue().get("key1"));
    }

    @Test
    public void testValueOperations() {
        valueOps.set("key2", "value2");
        Assert.assertEquals("value2", valueOps.get("key2"));
    }

    @Test
    public void testStringRedisTemplate() {
        stringRedisTemplate.opsForValue().set("key1", "value1");
        Assert.assertEquals("value1", stringRedisTemplate.opsForValue().get("key1"));
    }
```

## Serializers

## Redis Message/PubSub

## Redis Transactions

## Reactive Redis Support

只有Lettuce支持Reactive，Jedis不支持

#### ReactiveRedisConnectionFactory

#### ReactiveRedisTemplate

| Interface                     | Description                              |
| ----------------------------- | ---------------------------------------- |
| *Key Type Operations*         |                                          |
| ReactiveGeoOperations         | Redis geospatial operations like `GEOADD`, `GEORADIUS`,…) |
| ReactiveHashOperations        | Redis hash operations                    |
| ReactiveHyperLogLogOperations | Redis HyperLogLog operations like (`PFADD`, `PFCOUNT`,…) |
| ReactiveListOperations        | Redis list operations                    |
| ReactiveSetOperations         | Redis set operations                     |
| ReactiveValueOperations       | Redis string (or value) operations       |
| ReactiveZSetOperations        | Redis zset (or sorted set) operations    |



```java
    @Bean
    public ReactiveRedisConnectionFactory lettuceConnectionFactory() {
        // 单机
        RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration();
        redisStandaloneConfiguration.setHostName("localhost");
        redisStandaloneConfiguration.setPort(6379);

        // 连接池
        GenericObjectPoolConfig poolConfig = new GenericObjectPoolConfig();
        poolConfig.setMinIdle(1);
        poolConfig.setMaxIdle(5);

        // 客户端配置
        LettuceClientConfiguration lettuceClientConfiguration = LettucePoolingClientConfiguration.builder().poolConfig(poolConfig).build();

        return new LettuceConnectionFactory(redisStandaloneConfiguration, lettuceClientConfiguration);
    }    

	@Bean
    ReactiveRedisTemplate<String, String> reactiveRedisTemplate() {
        return new ReactiveRedisTemplate<>(lettuceConnectionFactory(), RedisSerializationContext.string());
    }
```



```java
   	@Autowired
    private ReactiveRedisTemplate<String, String> reactiveRedisTemplate;

	@Test
    public void testReactiveRedisTemplate() {
        StepVerifier.create(reactiveRedisTemplate.opsForValue().set("key3", "value3")).expectNext(true);
        StepVerifier.create(reactiveRedisTemplate.opsForValue().get("key3")).expectNext("value3");
    }
```



## Redis Repositories

![spring-data-RedisRepository](/images/spring-data-RedisRepository.png)

### Reactive Repositories

![spring-data-ReactiveRedisRepository](/images/spring-data-ReactiveRedisRepository.png)

## QA

### spring boot project

```java
// 报错，没有注入
@Autowired
private ReactiveRedisTemplate<String, String> reactiveRedisTemplate;

// OK
@Autowired
private ReactiveRedisTemplate<Object, Object> reactiveRedisTemplate;

// OK
@Autowired
private RedisTemplate<String, String> redisTemplate;

// OK
@Autowired
private RedisTemplate<Object, Object> redisTemplate;
```



## 参考

[Redis命令参考](http://redisdoc.com/)