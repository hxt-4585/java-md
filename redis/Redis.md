# Redis基础篇

## 附：

redis命令行客户端

```shell
redis-cli -h [ip] -p [port] -a [password]
```



## 一、

## 二、Redis常用命令

### 2.1 Redis通用命令

| 命令         | 描述                      |
| ------------ | ------------------------- |
| KEYs pattern | 查找所有给定模式的key     |
| EXISTs key   | 检查给定的key是否存在     |
| TYPE key     | 返回key所存储的值的类型   |
| TTL key      | 返回给定key的剩余生存时间 |
| DEL key      | 在key存在时删除key        |



### 2.2 String类型

String类型是Redis中最简单的存储类型，可分为三类

​	`string`：普通字符串

​	`int`：整型

​	`float`：浮点型

**不管是哪种格式，底层都是字节数组形式存储，只不过编码方式不同**



#### 2.2.1 String的常用命令

| 命令        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| SET         | 添加或修改已存在的String类型键值对                           |
| GET         | 根据key获取value                                             |
| MSET        | 批量添加，可修改原本存在的键值对                             |
| MGET        | 批量获取                                                     |
| INCR        | 让整型自增1                                                  |
| INCRBY      | 让整型自增指定步长，incrby num 2，让num自增2                 |
| INCRBYFLOAT | 让一个浮点类型的数字自增并指定步长值                         |
| SETNX       | 添加一个String类型的键值对，前提是这个key不存在，否则不执行，可以理解为真正的`新`增 |
| SETEX       | 添加一个String类型的键值对，并指定有效期                     |



#### 2.2.2 Key结构

​	Redis没有类似Mysql中的Table的概念，如何区分不同类型的Key呢？

​	例如，需要存储用户、商品信息到Redis，有一个用户的id是1，有一个商品的id恰好也是1，如果此时使用id作为key，那么就会冲突。

​	我们可以通过给key添加前缀加以区分，不过这个前缀不是随便加的，有一定的规范

```
项目名:类型名:类型:id	
```

| KEY           | VALUE（这里的value为字符串）                |
| ------------- | ------------------------------------------- |
| reggie:user:1 | {“id”:1, “name”: “Jack”, “age”: 21}         |
| reggie:dish:1 | {“id”:1, “name”: “鲟鱼火锅”, “price”: 4999} |

```shell
set reggie:user:1 '{“id”:1, “name”: “Jack”, “age”: 21}'
```



### 2.3 Hash类型

![image-20241101215631644](.\img\image-20241101215631644.png)

| 命令                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| HSET key field value | 添加或者修改hash类型key的field的值，貌似也能批量操作         |
| HGET key field       | 获取一个hash类型key的field的值                               |
| HMSET                | 批量添加多个hash类型key的field的值                           |
| HMGET                | 批量获取多个hash类型key的field的值                           |
| HGETALL              | 获取一个hash类型的key中的所有的field和value                  |
| HKEYS                | 获取一个hash类型的key中的所有的field                         |
| HINCRBY              | 让一个hash类型key的字段值自增并指定步长                      |
| HSETNX               | 添加一个hash类型的key的field值，前提是这个field不存在，否则不执行 |



### 2.4 Lish类型

Redis中的List类型和Java中的LinkList类似，双向链表结构。

| 命令                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| LPUSH key element   | 向列表左侧插入一个或者多个元素                               |
| LPOP key            | 移除并返回列表左侧第一个元素，没有则返回nil                  |
| RPUSH key element   | 向列表右侧插入一个或多个元素                                 |
| RPOP key            | 移除并返回列表右侧的第一个元素                               |
| LRANGE key star end | 返回一段角标范围内的所有元素                                 |
| BLPOP和BRPOP        | 与LPOP和RPOP类似，只不过在没有元素时等待指定时间，而不是直接返回nil |

![image-20241101222132032](.\img\image-20241101222132032.png)

![image-20241101222202664](.\img\Redis.md)



### 2.5 Set类型

支持交集、并集、差集等功能

| 命令                 | 描述                                     |
| -------------------- | ---------------------------------------- |
| SADD key member ...  | 向set中添加一个或多个元素                |
| SREM key member ...  | 删除元素                                 |
| SCARD key            | 返回元素个数                             |
| SISMEMBER key member | 判断一个元素是否在set中                  |
| SMEMBERS key         | 获取set中所有元素                        |
| SINTER key1 key2     | 求key1和key2的交集                       |
| SUNION key1 key2 …   | 求key1与key2的并集                       |
| SDIFF key1 key2      | 求key1和key2的差集，注意顺序不同结果不同 |

![image-20241104123522339](.\img\image-20241104123522339.png)



### 2.6 SortSet类型

SortSet为可排序的set集合，与Java的TreeSet类似，但底层结构相差很大。

SortedSet中的每一个元素都带有一个score属性，可以基于score属性对元素排序，底层的实现是一个跳表（SkipList）加 hash表。

| 命令                      | 描述                                             |
| ------------------------- | ------------------------------------------------ |
| ZADD key score member ... | 添加一个或多个元素到SortSet，若已存在则更新score |
| ZREM key member           | 删除sorted set中的一个指定元素                   |
|                           |                                                  |
|                           |                                                  |
|                           |                                                  |
|                           |                                                  |
|                           |                                                  |
|                           |                                                  |
|                           |                                                  |
|                           |                                                  |