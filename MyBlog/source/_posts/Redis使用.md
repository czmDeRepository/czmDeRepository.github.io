---
title: Redis之java简单使用
date: 2020-09-25 22:38:39
tags: 
  - Redis
categories: 数据库
---



## jedis

### 简介：

​	Jedis是Redis官方推荐的Java连接开发工具。要在Java开发中使用好Redis中间件，必须对Jedis熟悉 

### 示例代码

##### maven依赖

```xml
<!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.2.0</version>
</dependency>
```

##### 代码示例（与命令行操作基本一致）

```java
Jedis jedis = new Jedis("127.0.0.1", 6379);
 //验证密码，如果没有设置密码这段代码省略
jedis.auth("password");

System.out.println("清空数据："+jedis.flushDB());

System.out.println("判断某个键是否存在："+jedis.exists("username"));
System.out.println("新增<'username','kuangshen'>的键值对："+jedis.set("username", "kuangshen"));
System.out.println("新增<'password','password'>的键值对："+jedis.set("password", "password"));
System.out.print("系统中所有的键如下：");
Set<String> keys = jedis.keys("*");
System.out.println(keys);
System.out.println("删除键password:"+jedis.del("password"));
System.out.println("判断键password是否存在："+jedis.exists("password"));
System.out.println("查看键username所存储的值的类型："+jedis.type("username"));
System.out.println("随机返回key空间的一个："+jedis.randomKey());
System.out.println("重命名key："+jedis.rename("username","name"));
System.out.println("取出改后的name："+jedis.get("name"));
System.out.println("按索引查询："+jedis.select(0));
System.out.println("删除当前选择数据库中的所有key："+jedis.flushDB());
System.out.println("返回当前数据库中key的数目："+jedis.dbSize());
System.out.println("删除所有数据库中的所有key："+jedis.flushAll());
System.out.println("增加多个键值对："+jedis.mset("key01","value01","key02","value02","key03","value03"));
System.out.println("获取多个键值对："+jedis.mget("key01","key02","key03"));
System.out.println("获取多个键值对："+jedis.mget("key01","key02","key03","key04"));
System.out.println("删除多个键值对："+jedis.del("key01","key02"));
System.out.println("获取多个键值对："+jedis.mget("key01","key02","key03"));

Map<String,String> map = new HashMap<String,String>();
map.put("key1","value1");
map.put("key2","value2");
map.put("key3","value3");
map.put("key4","value4");
//添加名称为hash（key）的hash元素
jedis.hmset("hash",map);
//向名称为hash的hash中添加key为key5，value为value5元素
jedis.hset("hash", "key5", "value5");
System.out.println("散列hash的所有键值对为："+jedis.hgetAll("hash"));//return Map<String,String>
System.out.println("散列hash的所有键为："+jedis.hkeys("hash"));//return Set<String>
System.out.println("散列hash的所有值为："+jedis.hvals("hash"));//return List<String>
System.out.println("将key6保存的值加上一个整数，如果key6不存在则添加key6："+jedis.hincrBy("hash", "key6", 6));
System.out.println("散列hash的所有键值对为："+jedis.hgetAll("hash"));
System.out.println("将key6保存的值加上一个整数，如果key6不存在则添加key6："+jedis.hincrBy("hash", "key6", 3));
System.out.println("散列hash的所有键值对为："+jedis.hgetAll("hash"));
System.out.println("删除一个或者多个键值对："+jedis.hdel("hash", "key2"));
System.out.println("散列hash的所有键值对为："+jedis.hgetAll("hash"));
System.out.println("散列hash中键值对的个数："+jedis.hlen("hash"));
System.out.println("判断hash中是否存在key2："+jedis.hexists("hash","key2"));
System.out.println("判断hash中是否存在key3："+jedis.hexists("hash","key3"));
System.out.println("获取hash中的值："+jedis.hmget("hash","key3"));
System.out.println("获取hash中的值："+jedis.hmget("hash","key3","key4"));

//……其他类型操作类似，可参考redis命令


//redis事务
JSONObject jsonObject = new JSONObject();
jsonObject.put("hello", "world");
jsonObject.put("name", "java");
//开启事务
Transaction multi = jedis.multi();
String result = jsonObject.toJSONString();
try{
    //向redis存入一条数据
    multi.set("json", result);
    //再存入一条数据
    multi.set("json2", result);
    //这里引发了异常，用0作为被除数
    int i = 100/0;
    //如果没有引发异常，执行进入队列的命令
    multi.exec();
}catch(Exception e){
    e.printStackTrace();
    //如果出现异常，回滚
    multi.discard();
}finally{
    System.out.println(jedis.get("json"));
    System.out.println(jedis.get("json2"));
    //最终关闭客户端
    jedis.close();
}
```



## Redis与SpringBoot整合

### maven依赖

```xml
 <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

### yml配置

```yml
spring:
    redis:
      database: 0 #数据库索引，默认0
      host: 127.0.0.1
      port: 6379
      password: 
      jedis:
        pool:
          max-active: 8 #连接池最大连接数 
          max-wait: -1ms #连接池最大阻塞连接时间，负值表示无限制
          max-idle: 5 #连接池中最大空闲连接
          min-idle: 0 #连接池最小空闲连接
      lettuce:
        shutdown-timeout: 100ms #关闭超时时间，默认100ms
```

### 自定义Redistemplate模板

```java
@Configuration
public class RedisConfig {

  @Bean
  public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
      RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
      template.setConnectionFactory(factory);
      Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new 							
          													Jackson2JsonRedisSerializer(Object.class);
      ObjectMapper om = new ObjectMapper();
      om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
      om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
      jackson2JsonRedisSerializer.setObjectMapper(om);
      StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();

      // key采用String的序列化方式
      template.setKeySerializer(stringRedisSerializer);
      // hash的key也采用String的序列化方式
      template.setHashKeySerializer(stringRedisSerializer);
      // value序列化方式采用jackson
      template.setValueSerializer(jackson2JsonRedisSerializer);
      // hash的value序列化方式采用jackson
      template.setHashValueSerializer(jackson2JsonRedisSerializer);
      
      //必须执行这个函数,初始化RedisTemplate
      template.afterPropertiesSet();

      return template;
  }


}
```

