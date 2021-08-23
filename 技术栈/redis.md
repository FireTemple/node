## 启动redis

服务端

```bash
redis-server #不带配置
# 需要进入 redis的文件夹
redis-server redis-config #带配置启动
```

客户端 （必须在服务端启动之后再启动客户端）

```bash
redis-cli -a -password
# 进入后密码
auth bohan
```

查询redis进程（用来确认redis是否开启）

```bash
ps -ef | grep -i redis 
kill -9 PID # 杀死进程
```

## 关闭redis

```bash
shutdown
```



1. 没有设置密码

   ```bash
   redis-cli shut down
   ```

2. 设置了密码

   ```bash
   redis-cli -a password
   shutdown
   ```

   

## 常用命令

### 关于key

```redis
keys *
get {key} // 获取值
keys ? // 模糊查找
del {key}
exists {key}
ttl {key}  //查看过期时间 -1代表永久有效 -2代表无效 0代表
expire {key} {second}
pexpire {key} {millssecond} //毫秒
persist {key} // 设置成永久有效
rename {key} {newname}
setnx // 如果不存在则设置
incr {key}
incr by {key} {num}
decr {key}
decr by {key} {num}
```

### 关于库

```mak
select {index} 
```



### 关于对象（hash）

```makefile
hset {object} {属性} {value}
hmset {object} {属性1} {value1} {属性2} {value2}
hdel {object}
hgetall {object}
```



## 1. NoSQL4大分类

* KV : **redis**
* 文档型数据库（bson格式和json一样）： **MongoDB**
* 列存储数据库
* 图关系数据库：用来存储关系，比如微信好友关系

## 2. Redis 简介

Remote Dictinary Server

## 3. 性能测试

```bash
# 测试：100个并发
# 需要再 redis目录下运行
redis-benchmark -h localhost -p 6379 -c 100 -n 100000
```

其他测试方法网上找--菜鸟教程

## 4. 基础知识

### 4.1 16个数据库

```bash
# 选择数据库
select {index}
# 当前数据库大小
DBSIZE
# 清空当前数据库
flushdb
# 清空所有数据库
flushall
```

### 4.2 单线程

* redis是单线程，基于内存所以是单线程。

## 4.3 五大数据类型

### 4.3.1 String

```shell
append {key} {"string"} # 追加长度
strlen {key} # 获取长度
```

```shell
# 自增自减
incr {key}
incr by {key} {num} # 设置步长
decr {key}
decr by {key} {num}
```

```shell
getrange {key} {start} {end} # substring
setrange {key} {index} {newString} # 替换
```

```shell
expire {key} {second}
pexpire {key} {millssecond} # 毫秒
persist {key} # 设置成永久有效
```

```shell
mset {key1} {v1} {k2} {v2} # 批量设置 要么一起成功要么一起失败 原子性
msetnx {key1} {v1} {k2} {v2} # 不存在才创建
getset {key1} {newVallue} # 先获取再更新
```



### 4.3.2 list

注意点：

* lpush是往头部插入
* 一般都分L 和 R 下面的不写全

```shell
lpush {listname} {value} # 创建也是从这类开始 添加一个数到list中
lrange {listname} {start} {end} # subList 查询通过这个查询
rpush {listname} {value} # 插入到后端
lpop {listname} # 移除第一个元素（左边）
lrem {listname} {count} {value} # 移除list中的某个值几次
lrange {listname} {start} {end} # 截取
rpoplpush {listname1} {listname2} # 第一个list的末尾移动到第二个的头
lest {listname} {index} {newValue} # update
linsert {listname} {before or after} {value} {new value} # 在某个值前后插入新值
```

#### 4.3.2.1 注意点

可以当做栈或者队列用都可以 因为可以从左右添加删除

### 4.3.3 set

```shell
sadd {setname} {value} # 创建或者添加
smembers {setname} # 查询
sismember {setname} {queryvalue} # 查询某个值是否存在
scard {setname} # 查询长度
srem {setnamer} {number} # 删除
srandmember {setname} {count} # 随机抽取X个元素
spop {setname} {count} # 随机删除X个
smove {setname1} {setname2} {value in set1} # 移动
```

#### 4.3.3.1 逻辑

```shell
sdiff {setname1} {setname2} # 差集
sinter {setname1} {setname2} # 交集
sunion {setname1} {setname2} # 并集 
```



### 4.3.4 Hash

* 类似 K-MAP类型 多了一层
* 和string没有什么区别

```shell
hset {HashName} {field1} {value} # hmset 和 h get 略
hdel {hashName} {fieldName} # 删除
hsetnx {hashName} {fieldName} {value} # 存在则XXX 略
```



### 4.3.5 Zset（Sorted list）

```shell
zadd {setName} {score} {value}
zrange {setName} {start} {end}
```

```shell
zrangebyscore salary -inf +inf # 后面的是范围 还可以添加 withscore
```

## 5. geospatial 地理位置

```shell
geoadd {地区} {经纬度} {名字}
geopos {地区} {名字} 
```

获取

```shell
geodist china:city beijing shanghai km # exmaple 计算距离
georadius china:city 100 30 2000 km # 根据经纬度查询 X 范围内的 withdist withcoord count
zrange {地区} {start} {end}
```

## 6. Hyperloglog

用作基数统计 占用内存固定 一般用来计算访问量（重复不算）

* **有非常小的错误率！**

```shell
pfadd {key} {value1, value2 ...} # 添加
pfcount {key}	# 计算基数
pfmerge {newkey }{key1} {key2} # 合并
```



## 7. bitmap

做简单统计,跟多的是思想 命令简单

* 活跃用户
* 签到

```shell
setbit {key} {offset} {value}
setbit sign 1 0 # 可以表示这种周1没有签到
```

## 8. 事务

* **redis 单挑命令是原子性的 但是事务不保证原子性**

### 8.1 本质

一组命令的集合 类似队列 按准许执行，不会被打断。

### 8.2 顺序

* 开启事务
* 命令入队
* 执行事务

```shell
127.0.0.1:6379[3]> multi   # 开启
OK
127.0.0.1:6379[3]> set k1 v1
QUEUED
127.0.0.1:6379[3]> set k2 v2
QUEUED
127.0.0.1:6379[3]> get k2
QUEUED
127.0.0.1:6379[3]> set k3 v3
QUEUED
127.0.0.1:6379[3]> exec  # 执行
1) OK
2) OK
3) "v2"
4) OK
127.0.0.1:6379[3]>
```

放弃事务

```shell
discard # 随时写随时放弃
```

### 8.3 错误

* **编译出错** 所有的命令**都不会被执行**，出错就可以退出啦
* **运行异常** 其他命令可以执行，会抛出异常。**谁错谁不执行**



## 9. 悲观锁和乐观锁

### 9.1 悲观锁

无论做什么都加锁，影响性能。

### 9.2 乐观锁

什么时候都不出问题，所以不会上锁。**更新的时候判断是否有人修改数据**

```shell
watch {key}
# 然后再开启事务 如果其他线程修改了key的值 则会操作失败 多线程处理。
unwatch {key}
```

### 9.2.1 操作

* 如果发现事务执行失败就先解锁 
* 获取最新的值然再次监视
* 然后开启最新

## 10.  Jedis

### 10.1 导包

```xml
<dependencies>
            <dependency>
                <groupId>redis.clients</groupId>
                <artifactId>jedis</artifactId>
                <version>3.2.0</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>fastjson</artifactId>
                <version>1.2.62</version>
            </dependency>
        </dependencies>
```

### 10.2 获取

```java
//1. new jedis 对象即可
Jedis jedis = new Jedis("127.0.0.1",6379);
jedis.auth("bohan");
// 所有的命令就是我们之前学习的所有指令
jedis.close()
```

### 10.3 事务

#### 10.3.1 例子

```java
package com.bohan;

import com.alibaba.fastjson.JSONObject;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.Transaction;

public class TestPing {

    public static void main(String[] args) {
        //1. new jedis 对象即可
        Jedis jedis = new Jedis("127.0.0.1",6379);
        jedis.auth("bohan");
        // 所有的民领就是我们之前学习的所有指令

        JSONObject jsonObject = new JSONObject();
        jsonObject.put("hello","world");
        jsonObject.put("name","bohan");
        jedis.flushDB();
        Transaction multi = jedis.multi();
        String result = jsonObject.toJSONString();
        try {
            multi.set("user1",result);
            multi.set("user2",result);
//            int i = 1 / 0;
            // 报错回滚
            multi.exec();
        } catch (Exception e) {
            // 出错放弃
            multi.discard();
            e.printStackTrace();
        } finally {
            System.out.println(jedis.get("user1"));
            System.out.println(jedis.get("user2"));
            jedis.close();
        }


    }
}
```

## 11 springboot整合redis

* Jedis:采用的直连，多线程不安全。 使用redis pool连接池 BIO
* 2.0后 **转用lettuce properties里面要配置lettuce而不是jedis**

### 11.1 导包

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

### 11.2 测试

略比较简单

### 11.3 序列化问题

如果不配置会出现很多未知错误 会在springboot（狂神）讲解

#### 11.3.1 用json

* 无序列化接口仍然可以通过

  ```java
      @Test
      public void test(){
          try {
              User user = new User("bohan", 23);
              // 一般用json
              String jsonUser = new ObjectMapper().writeValueAsString(user);
              redisTemplate.opsForValue().set("user",jsonUser);
              System.out.println(redisTemplate.opsForValue().get("user"));
  
  
          } catch (JsonProcessingException e) {
              e.printStackTrace();
          }
      }
  ```



#### 11.3.2 直接传对象

* 没有序列化接口报错

  ```java
      @Test
      public void test1(){
              // 会报错
              User user = new User("bohan", 23);
              redisTemplate.opsForValue().set("user",user);
              System.out.println(redisTemplate.opsForValue().get("user"));
  
      }
  ```

  ![image-20200422223616464](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200422223616464.png)

* 添加序列化接口 正常。

#### 11.3.3 自己设置序列化

整理好的配置类

```java
package com.bohan.config;


import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.sun.org.apache.bcel.internal.generic.NEW;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.net.UnknownHostException;

@Configuration
public class RedisConfig {

    // 固定模板 可以直接使用

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory){
        // 自己开发一般都用<String, Object>
        RedisTemplate<String, Object> template = new RedisTemplate();
        // 连接
        template.setConnectionFactory(redisConnectionFactory);

        // json 序列化配置
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        // String 的序列化
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();

        //key 采用String的序列化方式
        template.setKeySerializer(stringRedisSerializer);
        //Hash的key也采用string的序列化
        template.setHashKeySerializer(stringRedisSerializer);
        //value序列化采用jackson
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        template.setValueSerializer(jackson2JsonRedisSerializer);

        template.afterPropertiesSet();
        return template;
    }
}

```

### 11.4 工具类

用工具类整理 省的要一直写try catch和 opsForValue 基本都是调用template。

**网上自己找**很多



## 12. Redis.conf

## 13. 持久化

### 13.1 RDB

**固定**的时间保存一下内存里面的集体快照到磁盘（snapshot）.

创建子进程进行保存，所以效率很高。**缺点**是最后一次持久化的数据可能丢失，**默认就是RDB**，一般不用做修改。

保存文件是 **dump.rdb**

* 优点：适合大规模的数据恢复
* 缺点：需要一定的时间间隔，有可能数据会丢失

### 13.2 AOF

固定的时间把直接的操作写到日志里面，需要再redis.confi里面去进行配置

如果生成的日志 appendonly.aof 被破坏（比如瞎鸡儿写） 可以使用下面进行修复

```bash
redis-check-aof --fix appendonly.aof 
```

* 优点：每次同步，文件完整整。每秒都同步最多丢失一秒的数据，从不同步效率最高。
* 缺点：文件太大，修复也麻烦，效率太低 所以默认是**RDB**。

## 14 发布订阅

pub/sub **消息通信模式**

![image-20200423095553783](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200423095553783.png)

#### 14.1 操作

```bash
subscribe {channel} # 订阅
unsubscribe 
publish {channel} {content} # 发布
```

会及时收到 下图是订阅者的视角

![image-20200423100212961](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200423100212961.png)

剩余查文档 用啥查啥

#### 14.2 原理

底层维护的是一个字典，每个channel就是一个key。 订阅了的就可以访问 redis相当于一个中间层 做为发布者和订阅者的中介

比较复杂一些的就会用专业的消息中间件 MQ

## 15 redis主从复制

* **单向** 只能从主到从
* 一个主可以由多个从，一个从只有一个主
* **主机可以写，从机只能读**。
* 主机所有信息都会被**自动**保存到从机。 
* **一旦连接**就会主机就会**全部复制**一份给从机

### 15.1 优点

* 数据冗余，热备份
* 服务冗余相当于备份
* 负载均衡
* 高可用（集群）

### 15.2 实际开发

* 一般都是至少三台
* 一般都是必须使用

### 15.3 环境配置

* **只需配置从不需配置主**

```bash
info replication # 查看当前库的信息
slaveof 127.0.0.1 6379 # 认老大（绑定主机）
```

创建多个启动配置文件  redis-conf

* 端口
* PID
* log名字
* dump.rdb 名字

**真正的配置都是在配置文件里面配置的**

配置文件里面寻找 replication

### 15.4 注意点

* 如果使用命令行配置的从机 如果断开后则会变回主机。只要变回从机就会立马从主机中获取值。

### 15.5 策略

如果主机挂了从机需要选择一个当主机

* 手动 

  ```bash
  slaveof no one # 直接接管这个系统 主机重新连接也没用 主机变成独立系统
  ```

  

## 16 哨兵模式

自动选举老大模式，避免手动的费劲。

![image-20200423104652769](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200423104652769.png)

### 16.1 哨兵

* 负责监控其他的服务器是否运行正常类似打卡。
* 主机回来也是自动作为从机。

### 16.2 哨兵集群

哨兵也可能挂了，所有一般用哨兵监控哨兵。

![image-20200423104830480](/Users/bohanxiao/Library/Application Support/typora-user-images/image-20200423104830480.png)

### 16.3 原理

情景：主机挂了

哨兵1先检测到进入**主观下线** ，此时发起请求让其他的哨兵进行检测，如果达到一定数目的哨兵检测到确实主机宕机，则进行投票。投票结果会由一个哨兵发起进行 **failover（故障转移）**。切换成功后，会发布**订阅模式**，让每个哨兵把自己监控从服务器实现切换主机，这个过程是**客观下线**。



### 16.4 测试

情景：一主两仆

* 配置哨兵配置文件 sentinel.conf

  ```bash
  sentinel monitor myredis 127.0.0.1 6379 1
  # sentinel monitor {被监控的名称} host port 1代表发起投票由这个发起
  ```

* 启动哨兵

  ```bash
  redis-sentinel sentinel.conf
  ```



### 16.4 总结

优点：比较明显主从模式++

缺点：配置繁杂 不好扩展，因为很多哨兵都是配置好的 需要重新添加配置在配置文件里面。



## 17. 缓存穿透和学崩

### 17.1 缓存穿透

**查不到**

查询数据，缓存和数据库都没有。相当于大量请求直接攻击了数据库。比如**秒杀活动**

#### 17.1.1 布隆过滤器

#### 17.1.2 空值过滤器

### 17.2 缓存击穿

**查的太多**

所有人请求一个，如果缓存刷新或者进行存储的时候有真空期，那么很有可能会在这期间数据库被大量查询 从而宕机。

#### 17.2.1 设置热点数据永不过期

#### 17.2.2 加互斥锁

保证之后一个线程可以过去，其余的等待

### 17.3 缓存雪崩

某一时间 缓存全部过期。导致直接缓存层失效。

情景：双11一般会停掉一些服务来保证主业务的正常。

**方案：**

redis 高可用 多设计几台搭建集群

限流降级 ：通过锁或者队列来控制请求同时访问数量

数据预热：把可能的数据先保存到缓存中，避免一瞬间的冲击过大。

## 18. 和Springboot搭配使用

### 18.1 配置

依赖

```xml
<!-- redis -->
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!-- spring2.X集成redis所需common-pool2-->
<dependency>
<groupId>org.apache.commons</groupId>
<artifactId>commons-pool2</artifactId>
<version>2.6.0</version>
</dependency>

```

配置类

* **keyGenerator** 用于自动生成key
* **RedisTemplate** 固定写法
* **cacheManager** 基本也是固定写法

```java
package com.atguigu.yygh.common.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.lang.reflect.Method;
import java.time.Duration;

/**
 * @ClassName RedisConfig
 * @Description TODO
 * @Author bohanxiao
 * @Data 6/11/21 14:35
 * @Version 1.0
 **/
@Configuration
@EnableCaching

public class RedisConfig {

    @Bean
    public KeyGenerator keyGenerator() {
        return new KeyGenerator() {
            @Override
            public Object generate(Object target, Method method, Object... params) {
                StringBuilder sb = new StringBuilder();
                sb.append(target.getClass().getName());
                sb.append(method.getName());
                for (Object obj : params) {
                    sb.append(obj.toString());
                }
                return sb.toString();
            }
        };
    }

    /**
     * 设置RedisTemplate规则
     * @param redisConnectionFactory
     * @return
     */
    @Bean
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<Object, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);

//解决查询缓存转换异常的问题
        ObjectMapper om = new ObjectMapper();
// 指定要序列化的域，field,get和set,以及修饰符范围，ANY是都有包括private和public
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
// 指定序列化输入的类型，类必须是非final修饰的，final修饰的类，比如String,Integer等会跑出异常
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);

//序列号key value
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);

        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }

    /**
     * 设置CacheManager缓存规则
     * @param factory
     * @return
     */
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory) {
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);

//解决查询缓存转换异常的问题
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);

// 配置序列化（解决乱码的问题）,过期时间600秒
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofSeconds(600))
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer))
                .disableCachingNullValues();

        RedisCacheManager cacheManager = RedisCacheManager.builder(factory)
                .cacheDefaults(config)
                .build();
        return cacheManager;
    }
}


```

通过注解使用

* Cacheable 用于方法上，调用的时候自动缓存
* CacheEvict 用于方法上，调用的时候自动清除缓存

```java
@Cacheable(value = "dict",keyGenerator = "keyGenerator")
@Override
public List<Dict> findByParentId(Long parentId) {
       List<Dict> dictList = dictMapper.selectList(new QueryWrapper<Dict>().eq("parent_id", parentId));
       dictList.stream().forEach(dict -> {
boolean isHasChildren = this.isHasChildren(dict.getId());
           dict.setHasChildren(isHasChildren);
       });
return dictList;
}
/**
 * 导入
 * allEntries = true: 方法调用后清空所有缓存
 * @param file
*/
@CacheEvict(value = "dict", allEntries=true)
@Override
public void importData(MultipartFile file) {
   ExcelHelper fileHelper = new ExcelHelper(DictEeVo.class);
   List<DictEeVo> dictVoList = fileHelper.importExcel(file);
if(!CollectionUtils.isEmpty(dictVoList)) {
dictMapper.insertBatch(dictVoList);
   }
}

```

