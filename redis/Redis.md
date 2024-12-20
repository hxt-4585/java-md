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

![image-20241111231324067](.\img\image-202411112312423430.png)

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

| 命令                         | 描述                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| ZADD key score member        | 添加一个或多个元素到sorted set ，如果已经存在则更新其score值 |
| ZREM key member              | 删除sorted set中的一个指定元素                               |
| ZSCORE key member            | 获取sorted set中的指定元素的score值                          |
| ZRANK key member             | 获取sorted set 中的指定元素的排名                            |
| ZCARD key                    | 获取sorted set中的元素个数                                   |
| ZCOUNT key min max           | 统计score值在给定范围内的所有元素的个数                      |
| ZINCRBY key increment member | 让sorted set中的指定元素的分数自增，步长为指定的increment值  |
| ZRANGE key min max           | 按照score排序后，获取指定排名范围内的元素                    |
| ZRANGEBYSCORE key min max    | 按照score排序后，获取指定score范围内的元素                   |
| ZDIFF、ZINTER、ZUNION        | 求差集、交集、并集                                           |



## 三、Redis的Java客户端

### 3.1 Jedis客户端

#### 3.1.1 快速入门

1、导入Jedis的maven坐标

```xml
<!--jedis-->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.7.0</version>
</dependency>
<!--单元测试-->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.7.0</version>
    <scope>test</scope>
</dependency>
```

2、建立连接

```java
Jedis jedis = null;

@BeforeEach
void setUp(){
    jedis = new Jedis("192.168.116.128", 6379);
    jedis.auth("20170440");
    jedis.select(0);
}

@Test
void testString(){
    jedis.set("name", "Kyle");
    String name = jedis.get("name");
    System.out.println(name);
}

@Test
void testHash(){
    jedis.hset("reggie:user:1","name","Jack");
    jedis.hset("reggie:user:2","name","Rose");
    jedis.hset("reggie:user:1","age","21");
    jedis.hset("reggie:user:2","age","18");
    Map<String, String> map = jedis.hgetAll("reggie:user:1");
    System.out.println(map);
}

@AfterEach
void tearDown(){
    if (jedis != null){
    	jedis.close();
    }
}
```



#### 3.1.2 连接池

`Jedis`本身是不安全的，并且频繁的创建和销毁线程会有性能损耗，因此推荐使用`Jedis连接池`

```java
public class JedisConnectionFactory {
    private  static JedisPool jedisPool;

    static {
        JedisPoolConfig poolConfig = new JedisPoolConfig();
        // 最大连接
        poolConfig.setMaxTotal(8);
        // 最大空闲连接
        // 在连接池中没有被当前的客户端线程使用，处于等待状态的连接
        poolConfig.setMaxIdle(8);
        // 最小空闲连接
        poolConfig.setMinIdle(0);
        // 设置最长等待时间
        poolConfig.setMaxWaitMillis(1000);

        // 创建连接池对象，参数：连接池配置、ip、端口、超时时间、密码
        jedisPool = new JedisPool(poolConfig, "192.168.116.128", 6379, 1000, "20170440");

    }

    public static Jedis getJedis(){
        return jedisPool.getResource();
    }
}


@BeforeEach
void setUp(){
    jedis = JedisConnectionFactory.getJedis();
    jedis.select(0);
}
```

### 3.2 SpringDataRedis客户端

SpringData 是 Spring 中的数据操作模块，包括最各种数据集的集成

- 提供了对不同Redis客户端的整合（Lettuce和Jedis）
- 提供了RedisTemplate统一API来操作Redis
- 支持Redis的发布订阅模型
- 支持Redis哨兵和Redis集群
- 支持基于Lettuce的响应式编程
- 支持基于JDK、JSON、字符串、Spring对象的数据序列化及反序列化
- 支持基于Redis的JDKCollection实现



SpringDataRedis中提供了RedisTemplate工具类，其中封装了对redis的各种操作

| API                          | 返回类型值      | 说明                |
| ---------------------------- | --------------- | ------------------- |
| redisTemplates.opsForValue() | ValueOperations | 操作String类型数据  |
| redisTemplate.opsForHash()   | HashOperations  | 操作Hash类型数据    |
| redisTemplate.opsForList()   | ListOperations  | 操作List类型数据    |
| redisTemplate.opsForSet()    | SetOperations   | 操作Set类型数据     |
| redisTemplate.opsForzSet()   | ZSetOperations  | 操作SortedSet类型数 |



#### 3.2.1 快速入门

```xml
<!--redis依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<!--common-pool-->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
<!--Jackson依赖-->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
<!--lombok-->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```



```yml
spring:
  data:
    redis:
      host: 192.168.116.128
      port: 6379
      password: 20170440
      lettuce:
        pool:
          max-active: 8
          max-idle: 8
          min-idle: 0
          max-wait: 100ms
```

```java
@Autowired
private RedisTemplate redisTemplate;

@Test
void testSpringDataRedisString(){
    redisTemplate.opsForValue().set("name", "李四");
    Object name = redisTemplate.opsForValue().get("name");
    System.out.println(name);
}
```

#### 3.2.2 自定义序列化



RedisTemplates可以接受 **任意Object** 作为值写入Redis

只不过再写入前会 **把Object序列化为字节形式** ，默认采用 **JDK序列化**，如：

```
\xAC\xED\x00\x05t\x00\x06\xE5\xBC\xA0\xE4\xB8\x89
```

缺点：

​		可读性差

​		内存占用大

采用JDK序列化：

![image-20241105132756326](.\img\image-20241105132756326.png)

第九行中的键值对才记录了 “李四”



源码分析：

```java
// defaultSerializer为默认序列化器
@Nullable
private RedisSerializer<?> defaultSerializer;
@Nullable
private ClassLoader classLoader;
@Nullable
private RedisSerializer keySerializer = null;
@Nullable
private RedisSerializer valueSerializer = null;
@Nullable
private RedisSerializer hashKeySerializer = null;
@Nullable
private RedisSerializer hashValueSerializer = null;
private RedisSerializer<String> stringSerializer = RedisSerializer.string();

//...
// 如果defaultSerializer为空，就会使用JDK序列化器
public void afterPropertiesSet() {
    super.afterPropertiesSet();
    if (this.defaultSerializer == null) {
        this.defaultSerializer = new JdkSerializationRedisSerializer(this.classLoader != null ? this.classLoader : this.getClass().getClassLoader());
    }
    //...
}
```





我们可以自定义RedisTemplate的序列化方式

```java
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        // 创建RedisTemplate对象
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // 设置连接工厂
        template.setConnectionFactory(redisConnectionFactory);
        // 创建JSON序列化工具
        GenericJackson2JsonRedisSerializer jackson2JsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
        // 设置key序列化
        template.setKeySerializer(RedisSerializer.string());
        template.setHashKeySerializer(RedisSerializer.string());
        // 设置value的序列化
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.setHashValueSerializer(jackson2JsonRedisSerializer);

        return template;
    }
```



```java
@Autowired
private RedisTemplate<String, Object> redisTemplate;

@Test
void testSpringDataRedisString(){
    redisTemplate.opsForValue().set("name", "王五");
    Object name = redisTemplate.opsForValue().get("name");
    System.out.println(name);

    User user = new User(10, "wangwu");
    redisTemplate.opsForValue().set("user", user);
    user = (User) redisTemplate.opsForValue().get("user");
    System.out.println(user);
}
/*
{
  "@class": "com.hxt.pojo.User",
  "id": 10,
  "name": "wangwu"
}

还写入了"@class": "com.hxt.pojo.User",用于反序列化

*/
```



#### 3.2.3 StringRedisTemplate

![image-20241105230513431](.\img\image-20241105230513431.png)

**夹带私活："@class": "com.hxt.pojo.User"**。为了节省内存空间，我们会统一使用String序列化器，要求只能存储String类型的 **key** 和 **value**。

```java
// StringRedisTemplate的序列化器均为RedisSerializer源码
// StringRedisTemplate的序列化器均为RedisSerializer.string()
public class StringRedisTemplate extends RedisTemplate<String, String> {
    public StringRedisTemplate() {
        this.setKeySerializer(RedisSerializer.string());
        this.setValueSerializer(RedisSerializer.string());
        this.setHashKeySerializer(RedisSerializer.string());
        this.setHashValueSerializer(RedisSerializer.string());
    }
    // ...
}
```



```java
@Autowired
private StringRedisTemplate stringRedisTemplate;

@Test
public void testStringRedisTemplate() {
    stringRedisTemplate.opsForValue().set("name", "zhangsan");
    String name = stringRedisTemplate.opsForValue().get("name");
    System.out.println(name);
}

private static final ObjectMapper objectMapper = new ObjectMapper();

@Test
public void testSaveUser() throws JsonProcessingException {
    User user = new User(10, "hxt");
    // 手动序列化
    String json = objectMapper.writeValueAsString(user);
    // 写入数据
    stringRedisTemplate.opsForValue().set("user", json);
    String userString = stringRedisTemplate.opsForValue().get("user");
    // 手动反序列化
    User readValue = objectMapper.readValue(userString, User.class);
    System.out.println(readValue);
}
```





# Redis实战篇



## 一、短信登陆

### 1.1 导入项目

#### 1.1.1 导入SQL

| 表               | 说明                      |
| ---------------- | ------------------------- |
| tb_user          | 用户表                    |
| tb_user_info     | 用户详情表                |
| tb_shop          | 商户信息表                |
| tb_shop_type     | 商户类型表                |
| tb_blog          | 用户日记表（达人探店日记) |
| tb_follow        | 用户关注表                |
| tb_voucher       | 优惠券表                  |
| tb_voucher_order | 优惠券的订单表            |



#### 1.1.2 导入后端项目

```
项目名称 hm-dianping
```



#### 1.1.3 导入前端项目

```
项目名称 nginx-1.18.0
```



### 1.2 基于Session实现登录流

A、发送验证码

​		用户在提交手机号后，会校验手机号是否合法，如果不合法，则要求用户重新输入手机号；

​		如果手机号合法，后台产生对应的验证码，同时将验证码保存，然后通过短信的方式发送给用户。

![image-20241111231147748](.\img\image-20241111231147748.png)



B、短信验证码登录、注册

​		用户将验证码和手机号进行输入，后台从session中拿到当前验证码，然后和用户输入的验证码进行校验，如果不一致，则无法通过校验，如果一致，则后台根据手机号查询用户，如果用户不存在，则为用户创建账号信息，保存到数据库，无论是否存在，都会将用户信息保存到session中，方便后续获得当前登录信息

![image-20241111231243860](.\img\image-20241111231243860.png)



C、校验登录状态

​		用户在请求的时候，会从cookie中携带JsessionId到后台，后台通过JsessionId从session中拿到用户信息，如果没有session信息，则进行拦截，如果有session信息，则将用户信息保存到threadLocal中，并放行

![image-20241111231440429](.\img\image-20241111231440429.png)



### 1.3 实现发送短信验证码功能

```java
@Override
public Result sendCode(String phone, HttpSession session) {
    // 校验手机号
    if (RegexUtils.isPhoneInvalid(phone)) {
        return Result.fail("手机号格式错误");
    }

    // 符合
    String code = RandomUtil.randomNumbers(6);
    // 保存验证码到session
    session.setAttribute("code", code);

    // 发送验证码
    log.debug("成功" + code);

    return Result.ok();
}

@Override
public Result login(LoginFormDTO loginForm, HttpSession session) {
    // 校验手机号
    String phone = loginForm.getPhone();
    if (RegexUtils.isPhoneInvalid(phone)) {
        return Result.fail("手机号格式错误");
    }
    // 校验验证码
    String cacheCode = (String) session.getAttribute("code");
    String code = loginForm.getCode();

    // 不一致，报错
    if (cacheCode == null || !cacheCode.equals(code)) {
        return Result.fail("验证码错误");
    }
    // 一致，根据手机查询用户
    User user = query().eq("phone", phone).one();
    // 判断用户是否存在
    if(user == null){
        // 不存在
        user = createUserWithPhone(phone);
    }
    // 保存用户信息到session
    session.setAttribute("user", user);
    return Result.ok();
}

private User createUserWithPhone(String phone) {
    User user = new User();
    user.setPhone(phone);
    user.setNickName(USER_NICK_NAME_PREFIX + RandomUtil.randomString(10));
    save(user);
    return user;

}
```



### 1.4 实现登录拦截功能

编写拦截器

```java
public class LoginInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        HttpSession session = request.getSession();
        UserDTO user = (UserDTO) session.getAttribute("user");

        if (user == null) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            return false;
        }

        UserHolder.saveUser(user);

        return true;

    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        HandlerInterceptor.super.postHandle(request, response, handler, modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        HandlerInterceptor.super.afterCompletion(request, response, handler, ex);
    }
}
```



配置拦截器路径

```java
@Configuration
public class MyConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())
                .excludePathPatterns(
                        "/shop/**",
                        "/voucher/**",
                        "/shop-type/**",
                        "/upload/**",
                        "/blog/hot",
                        "/user/code",
                        "/user/login"
                );
    }
}
```



### 1.5 session共享问题

- 每个tomcat中都有一份属于自己的session,假设用户第一次访问第一台tomcat，并且把自己的信息存放到第一台服务器的session中，但是第二次这个用户访问到了第二台tomcat，那么在第二台服务器上，肯定没有第一台服务器存放的session，所以此时 整个登录拦截功能就会出现问题，我们能如何解决这个问题呢？早期的方案是session拷贝，就是说虽然每个tomcat上都有不同的session，但是每当任意一台服务器的session修改时，都会同步给其他的Tomcat服务器的session，这样的话，就可以实现session的共享了

- 但是这种方案具有两个大问题
  1. 每台服务器中都有完整的一份session数据，服务器压力过大。
  2. session拷贝数据时，可能会出现延迟
- 所以我们后面都是基于Redis来完成，我们把session换成Redis，Redis数据本身就是共享的，就可以避免session共享的问题了

![image-20241114220545279](.\img\image-20241114220545279.png)

### 1.6 Redis代替Session流程

![image-20241115222254123](.\img\image-20241115222254123.png)

```java
// 注入StringRedisTemplate
@Configuration
public class MyConfig implements WebMvcConfigurer {
	// 引入redis依赖，在yml中配置好redis就可以注入
    @Resource
    private StringRedisTemplate stringRedisTemplate;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor(stringRedisTemplate))
                .excludePathPatterns(
                        "/shop/**",
                        "/voucher/**",
                        "/shop-type/**",
                        "/upload/**",
                        "/blog/hot",
                        "/user/code",
                        "/user/login"
                );
    }
}

// 拦截器修改
public class LoginInterceptor implements HandlerInterceptor {

    private StringRedisTemplate stringRedisTemplate;

    public LoginInterceptor(StringRedisTemplate stringRedisTemplate) {
        this.stringRedisTemplate = stringRedisTemplate;
    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 获取请求头中的token
//        HttpSession session = request.getSession();
        String token = request.getHeader("authorization");
        if (StrUtil.isBlankIfStr(token)){
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            return false;
        }
        // 基于token获得redis中的用户
//        UserDTO user = (UserDTO) session.getAttribute("user");
        Map<Object, Object> userMap = stringRedisTemplate.opsForHash().entries(RedisConstants.LOGIN_USER_KEY + token);
        if (userMap.isEmpty()) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            return false;
        }
        // 将查询到的hash转为userDTO对象
        UserDTO userDTO = BeanUtil.fillBeanWithMap(userMap, new UserDTO(), false);

        // 保存用户信息到threadLocal
        UserHolder.saveUser(userDTO);
        // 刷新token的有效期
        stringRedisTemplate.expire(RedisConstants.LOGIN_USER_KEY + token, 30, TimeUnit.MINUTES);
        return true;

    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        HandlerInterceptor.super.postHandle(request, response, handler, modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        HandlerInterceptor.super.afterCompletion(request, response, handler, ex);
    }
}


// 业务代码修改
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements IUserService {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Override
    public Result sendCode(String phone, HttpSession session) {
        // 校验手机号
        if (RegexUtils.isPhoneInvalid(phone)) {
            return Result.fail("手机号格式错误");
        }

        // 符合
        String code = RandomUtil.randomNumbers(6);
        // 保存验证码到redis
        stringRedisTemplate.opsForValue().set(LOGIN_CODE_KEY + phone, code, LOGIN_CODE_TTL, TimeUnit.MINUTES);

        // 发送验证码
        log.debug("成功" + code);

        return Result.ok();
    }

    @Override
    public Result login(LoginFormDTO loginForm, HttpSession session) {
        // 校验手机号
        String phone = loginForm.getPhone();
        if (RegexUtils.isPhoneInvalid(phone)) {
            return Result.fail("手机号格式错误");
        }
        // 校验验证码
        String cacheCode = stringRedisTemplate.opsForValue().get(LOGIN_CODE_KEY + phone);
        String code = loginForm.getCode();

        // 不一致，报错
        if (cacheCode == null || !cacheCode.equals(code)) {
            return Result.fail("验证码错误");
        }
        // 一致，根据手机查询用户
        User user = query().eq("phone", phone).one();
        // 判断用户是否存在
        if(user == null){
            // 不存在
            user = createUserWithPhone(phone);
        }
        // 保存用户信息到redis
        // 随机生成token，作为登录令牌
        String token = UUID.randomUUID().toString(true);
        // 将user转换为hash存储
        UserDTO userDTO = BeanUtil.copyProperties(user, UserDTO.class);
        Map<String, Object> userMap = BeanUtil.beanToMap(userDTO, new HashMap<>(),
                CopyOptions.create().
                setIgnoreNullValue(true).
                setFieldValueEditor((fieldName, fieldValue) -> fieldValue.toString()));
        stringRedisTemplate.opsForHash().putAll(LOGIN_USER_KEY  + token, userMap);
        stringRedisTemplate.expire(LOGIN_USER_KEY + token, LOGIN_USER_TTL, TimeUnit.MINUTES);
        return Result.ok(token);
    }


    private User createUserWithPhone(String phone) {
        User user = new User();
        user.setPhone(phone);
        user.setNickName(USER_NICK_NAME_PREFIX + RandomUtil.randomString(10));
        save(user);
        return user;

    }
}
```



### 1.7 优化

当用户登录后，一直访问那些不被拦截器拦截的接口，那么在redis中所设置的时限不会被刷新

![image-20241117164624618](.\img\image-20241117164624618.png)

```java
// 拦截器LoginInterceptor
public class LoginInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //判断用户是否存在
        if (UserHolder.getUser()==null){
            //不存在则拦截
            response.setStatus(401);
            return false;
        }
        //存在则放行

        return true;

    }
    // ...
}

// 拦截器RefreshTokenInterceptor
public class RefreshTokenInterceptor implements HandlerInterceptor {

    private StringRedisTemplate stringRedisTemplate;

    public RefreshTokenInterceptor(StringRedisTemplate stringRedisTemplate) {
        this.stringRedisTemplate = stringRedisTemplate;
    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        String token = request.getHeader("authorization");
        if (StrUtil.isBlank(token)) {
            return true;
        }

        Map<Object, Object> userMap = stringRedisTemplate.opsForHash().entries(RedisConstants.LOGIN_USER_KEY + token);
        if (!userMap.isEmpty()) {
            UserDTO userDTO = BeanUtil.fillBeanWithMap(userMap, new UserDTO(), false);
            //6. 将用户信息保存到ThreadLocal
            UserHolder.saveUser(userDTO);
            stringRedisTemplate.expire(RedisConstants.LOGIN_USER_KEY + token, 30, TimeUnit.MINUTES);
        }

        return true;
    }
}

// Config文件
package com.hmdp.config;

import com.hmdp.utils.LoginInterceptor;
import com.hmdp.utils.RefreshTokenInterceptor;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import javax.annotation.Resource;

@Configuration
public class MyConfig implements WebMvcConfigurer {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())
                .excludePathPatterns(
                        "/shop/**",
                        "/voucher/**",
                        "/shop-type/**",
                        "/upload/**",
                        "/blog/hot",
                        "/user/code",
                        "/user/login"
                ).order(1);
        // 设置优先级，越小越高
        registry.addInterceptor(new RefreshTokenInterceptor(stringRedisTemplate)).order(0);
    }
}

```



## 二、商户查询的缓存

### 2.1 什么是缓存

![image-20241117171453613](.\img\image-20241117171453613.png)

#### 2.1.1 为什么使用

- 言简意赅：速度快，好用

- 缓存数据存储于代码中，而代码运行在内存中，内存的读写性能远高于磁盘，缓存可以大大降低用户访问并发量带来的服务器读写压力

- 实际开发中，企业的数据量，少则几十万，多则几千万，这么大的数据量，如果没有缓存来作为`避震器`系统是几乎撑不住的，所以企业会大量运用缓存技术

- 但是缓存也会增加代码复杂度和运营成本

- ```
  缓存的作用
  ```

  1. 降低后端负载
  2. 提高读写效率，降低响应时间

- ```
  缓存的成本
  ```

  1. 数据一致性成本
  2. 代码维护成本
  3. 运维成本（一般采用服务器集群，需要多加机器，机器就是钱）



#### 2.1.2 如何使用

- 实际开发中，会构筑多级缓存来时系统运行速度进一步提升，例如：本地缓存与Redis中的缓存并发使用
- `浏览器缓存：`主要是存在于浏览器端的缓存
- `应用层缓存：`可以分为toncat本地缓存，例如之前提到的map或者是使用Redis作为缓存
- `数据库缓存：`在数据库中有一片空间是buffer pool，增改查数据都会先加载到mysql的缓存中
- `CPU缓存：`当代计算机最大的问题就是CPU性能提升了，但是内存读写速度没有跟上，所以为了适应当下的情况，增加了CPU的L1，L2，L3级的缓存



### 2.2 添加商户缓存

![image-20241118190323245](.\img\image-20241118190323245.png)





### 2.3 缓存更新策略

![image-20241118194747544](.\img\image-20241118194747544.png)

在实际业务中，主动更新策略中第一个策略用得比较多

![image-20241118195628725](.\img\image-20241118195628725.png)



#### 2.3.1 Cache Aside Pattern

![image-20241118200013420](.\img\image-20241118200013420.png)

**先删缓存，再操作数据库**

**情况1**

正常情况

![image-20241118201932998](.\img\image-20241118201932998.png)

**情况2**

不一致

![image-20241118202022242](.\img\image-20241118202022242.png)



**先操作数据库，再删除缓存**

**情况3**

正常情况

![image-20241118202256655](.\img\image-20241118202256655.png)

**情况4**

不一致。不论删除缓存还是写入缓存，速度都是微秒级别的，线程1开始写缓存到写完，线程2是几乎不可能完成两项工作，因此这种情况发生的概率很低（相较于情况2）。所以**先操作数据库，再删除缓存**策略更优

![image-20241118202432200](.\img\image-20241118202432200.png)



![image-20241118203029070](.\img\image-20241118203029070.png)



### 2.4 缓存与数据库双写一致

**给查询店铺的缓存添加超时剔除和主动更新策略**

```java

// 并在public Result queryById(Long id)方法中添加时限
stringRedisTemplate.opsForValue().set(CACHE_SHOP_KEY + id, JSONUtil.toJsonStr(shop), CACHE_SHOP_TTL, TimeUnit.MINUTES);



@Override
@Transactional
public Result update(Shop shop) {
    if (shop.getId() == null) {
        return Result.fail("FAIL");
    }
    boolean flag = updateById(shop);

    stringRedisTemplate.delete(CACHE_SHOP_KEY + shop.getId());
    return Result.ok();
}
```

经过修改后redis中的数据被删除

![image-20241118205831883](.\img\image-20241118205754653.png)



### 2.5 缓存穿透

#### 2.5.1 什么是缓存穿透

​		缓存穿透是指客户端请求的数据在缓存中和数据库中都不存在，这样缓存永远都不会生效（只有数据库查到了，才会让redis缓存，但现在的问题是查不到），会频繁的去访问数据库。



#### 2.5.2 解决方法

##### 2.5.2.1 缓存空对象

- 优点：实现简单，维护方便
- 缺点：额外的内存消耗，可能造成短期的不一致（第一次访问为空，但第一次后，数据库中有了这条数据，因此会不一致，可以控制TTL时间）

![image-20241118210404399](.\img\image-20241118210404399.png)



##### 2.5.2.2 布隆过滤

- 优点：内存占用少，没有多余的key
- 缺点：实现复杂，可能存在误判

![image-20241118210533025](.\img\image-20241118210533025.png)

#### 2.5.3 代码实现

##### 2.5.3.1 缓存空对象

![image-20241119183452855](.\img\image-20241119183452855.png)



```java
@Override
public Result queryById(Long id) {

    String shopJson = stringRedisTemplate.opsForValue().get(CACHE_SHOP_KEY+ id);

    if (StrUtil.isNotBlank(shopJson)) {
        Shop shop = JSONUtil.toBean(shopJson, Shop.class);
        return Result.ok(shop);
    }

    if (shopJson != null) {
        return Result.fail("店铺信息不存在");
    }

    Shop shop = getById(id);
    if (shop != null) {
        stringRedisTemplate.opsForValue().set(CACHE_SHOP_KEY + id, JSONUtil.toJsonStr(shop), CACHE_SHOP_TTL, TimeUnit.MINUTES);
        return Result.ok(shop);
    }else {
        stringRedisTemplate.opsForValue().set(CACHE_SHOP_KEY + id, "", CACHE_NULL_TTL, TimeUnit.MINUTES);
        return Result.fail("店铺不存在");
    }
}
```

第二次访问 `http://localhost:8081/shop/0` 时，不会访问数据库，redis中存入空字符串

![image-20241119185217299](.\img\image-20241119185217299.png)

![image-20241119185333456](.\img\image-20241119185333456.png)

##### 2.5.3.2 布隆过滤

上网去找

#### 2.5.4 总结

不一定总是靠上述两种方案

![image-20241119185704905](.\img\image-202411012156323644.png)



### 2.6 缓存雪崩

- 缓存雪崩是指在同一时间段，大量缓存的key同时失效，或者Redis服务宕机，导致大量请求到达数据库，带来巨大压力

- 解决方案
  - 给不同的Key的TTL添加随机值，让其在不同时间段分批失效
  - 利用Redis集群提高服务的可用性（使用一个或者多个哨兵(`Sentinel`)实例组成的系统，对redis节点进行监控，在主节点出现故障的情况下，能将从节点中的一个升级为主节点，进行故障转义，保证系统的可用性。 ）
  - 给缓存业务添加降级限流策略
  - 给业务添加多级缓存（浏览器访问静态资源时，优先读取浏览器本地缓存；访问非静态资源（ajax查询数据）时，访问服务端；请求到达Nginx后，优先读取Nginx本地缓存；如果Nginx本地缓存未命中，则去直接查询Redis（不经过Tomcat）；如果Redis查询未命中，则查询Tomcat；请求进入Tomcat后，优先查询JVM进程缓存；如果JVM进程缓存未命中，则查询数据库）



### 2.7 缓存击穿

- 缓存击穿也叫热点Key问题，就是一个被`高并发访问`并且`缓存重建业务较复杂`的key突然失效了，那么无数请求访问就会在瞬间给数据库带来巨大的冲击
- 举个不太恰当的例子：一件秒杀中的商品的key突然失效了，大家都在疯狂抢购，那么这个瞬间就会有无数的请求访问去直接抵达数据库，从而造成缓存击穿

- 常见的解决方案有两种
  
  **互斥锁**
  
  **逻辑过期：**我们之前是TTL设置在redis的value中，注意：这个过期时间并不会直接作用于Redis，而是我们后续通过逻辑去处理。假设线程1去查询缓存，然后从value中判断当前数据已经过期了，此时线程1去获得互斥锁，那么其他线程会进行阻塞，获得了锁的进程他会开启一个新线程去进行之前的重建缓存数据的逻辑，直到新开的线程完成者逻辑之后，才会释放锁，而线程1直接进行返回，假设现在线程3过来访问，由于线程2拿着锁，所以线程3无法获得锁，线程3也直接返回数据（但只能返回旧数据，牺牲了数据一致性，换取性能上的提高），只有等待线程2重建缓存数据之后，其他线程才能返回正确的数据



| 解决方案 | 优点                                   | 缺点                                    |
| -------- | -------------------------------------- | --------------------------------------- |
| 互斥锁   | 没有额外的内存消耗 保证一致性 实现简单 | 线程需要等待，性能受影响 可能有死锁风险 |
| 逻辑过期 | 线程无需等待，性能较好                 | 不保证一致性 有额外内存消耗 实现复杂    |



#### 2.7.1 互斥锁

```java
private boolean tryLock(String key){
    Boolean flag = stringRedisTemplate.opsForValue().setIfAbsent(key, "1", 10, TimeUnit.SECONDS);
    return BooleanUtil.isTrue(flag);
}

private void unlock(String key){
    stringRedisTemplate.delete(key);
}

public Shop queryWithPassThrough(Long id){
    String shopJson = stringRedisTemplate.opsForValue().get(CACHE_SHOP_KEY+ id);

    if (StrUtil.isNotBlank(shopJson)) {
        Shop shop = JSONUtil.toBean(shopJson, Shop.class);
        return shop;
    }

    if (shopJson != null) {
        return null;
    }

    Shop shop = getById(id);
    if (shop != null) {
        stringRedisTemplate.opsForValue().set(CACHE_SHOP_KEY + id, JSONUtil.toJsonStr(shop), CACHE_SHOP_TTL, TimeUnit.MINUTES);
        return shop;
    }else {
        stringRedisTemplate.opsForValue().set(CACHE_SHOP_KEY + id, "", CACHE_NULL_TTL, TimeUnit.MINUTES);
        return null;
    }
}

public Shop queryWithMutex(Long id) {
    String shopJson = stringRedisTemplate.opsForValue().get(CACHE_SHOP_KEY+ id);

    if (StrUtil.isNotBlank(shopJson)) {
        Shop shop = JSONUtil.toBean(shopJson, Shop.class);
        return shop;
    }

    if (shopJson != null) {
        return null;
    }

    // 实现缓存重建
    // 获取互斥锁
    String lockKey = "lock:shop:" + id;
    //        while (tryLock(lockKey)){
    //            Thread.sleep(50);
    //        }
    try {
        if (!tryLock(lockKey)) {
            Thread.sleep(50);
            return queryWithMutex(id);
        }
        //

        Shop shop = getById(id);
        if (shop != null) {
            stringRedisTemplate.opsForValue().set(CACHE_SHOP_KEY + id, JSONUtil.toJsonStr(shop), CACHE_SHOP_TTL, TimeUnit.MINUTES);
            return shop;
        }else {
            stringRedisTemplate.opsForValue().set(CACHE_SHOP_KEY + id, "", CACHE_NULL_TTL, TimeUnit.MINUTES);
            return null;
        }
    }catch (InterruptedException e){
        throw new RuntimeException(e);
    }finally {
        // 释放互斥锁
        unlock(lockKey);
    }

}

@Override
public Result queryById(Long id) {
    // 缓存穿透
    //        Shop shop = queryWithPassThrough(id);

    Shop shop = queryWithMutex(id);
    if (shop == null) {
        return Result.fail("店铺不存在");
    }
    // 缓存击穿
    return Result.ok(shop);
}
```





#### 2.7.2 逻辑过期

![image-20241120220750212](.\img\image-20241120220750212.png)

```java
private static final ExecutorService CACHE_REBUILD_EXECUTOR = Executors.newFixedThreadPool(10);


public void saveShopToRedis(Long id, Long expireSeconds) throws InterruptedException {
    // 查询商铺信息
    Shop shop = getById(id);
    Thread.sleep(200);
    // 封装逻辑过期时间
    RedisData redisData = new RedisData();
    redisData.setData(shop);
    redisData.setExpireTime(LocalDateTime.now().plusSeconds(expireSeconds));
    // 写入redis
    stringRedisTemplate.opsForValue().set(CACHE_SHOP_KEY + id, JSONUtil.toJsonStr(redisData));
}

public Shop queryWithLogicalExpire(Long id) {
    String shopJson = stringRedisTemplate.opsForValue().get(CACHE_SHOP_KEY+ id);

    if (StrUtil.isBlank(shopJson)) {
        return null;
    }

    // 命中，需要反序列化
    RedisData redisData = JSONUtil.toBean(shopJson, RedisData.class);
    JSONObject data = (JSONObject) redisData.getData();
    Shop shop = JSONUtil.toBean(data, Shop.class);
    LocalDateTime expireTime = redisData.getExpireTime();

    // 判断是否过期
    if (expireTime.isAfter(LocalDateTime.now())) {
        // 未过期
        return shop;
    }

    // 已过期
    String lockKey = LOCK_SHOP_KEY + id;
    boolean lock = tryLock(lockKey);
    if (lock) {
        // 开启线程池去做
        CACHE_REBUILD_EXECUTOR.submit(()->{
            try {
                // 重建
                this.saveShopToRedis(id, 20L);

            }catch (Exception e){
                // 释放锁
                unlock(lockKey);
            }
        });
    }

    return shop;
}
```

### 2.8 缓存工具封装

![image-20241120221204230](.\img\image-20241120221204230.png)

```java
@Slf4j
@Component
public class CacheClient {

    private final StringRedisTemplate stringRedisTemplate;

    private static final ExecutorService CACHE_REBUILD_EXECUTOR = Executors.newFixedThreadPool(10);

    private boolean tryLock(String key){
        Boolean flag = stringRedisTemplate.opsForValue().setIfAbsent(key, "1", 10, TimeUnit.SECONDS);
        return BooleanUtil.isTrue(flag);
    }

    private void unlock(String key){
        stringRedisTemplate.delete(key);
    }


    public CacheClient(StringRedisTemplate stringRedisTemplate) {
        this.stringRedisTemplate = stringRedisTemplate;
    }

    public void set(String key, Object value, Long time, TimeUnit unit) {
        stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(value), time, unit);
    }

    public void setWithLogicalExpire(String key, Object value, Long time, TimeUnit unit) {
        RedisData redisData = new RedisData();
        redisData.setData(value);
        redisData.setExpireTime(LocalDateTime.now().plusSeconds(unit.toSeconds(time)));
        stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(redisData));
    }

    // 前缀keyPrefix，如CACHE_SHOP_KEY，由于ID的类型不确定
    public <R, ID> R queryWithPassThrough(String keyPrefix, ID id, Class<R> type, Function<ID, R> dbFallback, Long time, TimeUnit unit){
        String key = keyPrefix + id;

        String json = stringRedisTemplate.opsForValue().get(key);

        if (StrUtil.isNotBlank(json)) {
            return JSONUtil.toBean(json, type);
        }

        if (json != null) {
            return null;
        }

//        Shop shop = getById(id);
        R r = dbFallback.apply(id);

        if (r != null) {
            this.set(key, r, time, unit);
            return r;
        }else {
            stringRedisTemplate.opsForValue().set(key, "", CACHE_NULL_TTL, TimeUnit.MINUTES);
            return null;
        }
    }

    public <R, ID> R queryWithLogicalExpire(String keyPrefix, ID id, Class<R> type, Function<ID, R> dbFallback, Long time, TimeUnit unit, String lockKey) {
        String key = keyPrefix + id;

        String json = stringRedisTemplate.opsForValue().get(key);

        if (StrUtil.isBlank(json)) {
            return null;
        }

        // 命中，需要反序列化
        RedisData redisData = JSONUtil.toBean(json, RedisData.class);
        JSONObject data = (JSONObject) redisData.getData();
        R r = JSONUtil.toBean(data, type);
        LocalDateTime expireTime = redisData.getExpireTime();

        // 判断是否过期
        if (expireTime.isAfter(LocalDateTime.now())) {
            // 未过期
            return r;
        }

        // 已过期
        boolean lock = tryLock(lockKey + id);
        if (lock) {
            // 开启线程池去做
            CACHE_REBUILD_EXECUTOR.submit(()->{
                try {
                    // 重建
                    R r1 = dbFallback.apply(id);
                    this.setWithLogicalExpire(key, r1, time, unit);

                }catch (Exception e){
                    // 释放锁
                    unlock(lockKey);
                }
            });
        }

        return r;
    }
}

```



## 三、优惠券秒杀

### 3.1 全局唯一ID

- 在各类购物App中，都会遇到商家发放的优惠券

- 当用户抢购商品时，生成的订单会保存到

  ```
  tb_voucher_order
  ```

  表中，而订单表如果使用数据库自增ID就会存在一些问题

  1. id规律性太明显
  2. 受单表数据量的限制

- 如果我们的订单id有太明显的规律，那么对于用户或者竞争对手，就很容易猜测出我们的一些敏感信息，例如商城一天之内能卖出多少单，这明显不合适

- 随着我们商城的规模越来越大，MySQL的单表容量不宜超过500W，数据量过大之后，我们就要进行拆库拆表，拆分表了之后，他们从逻辑上讲，是同一张表，所以他们的id不能重复，于是乎我们就要保证id的唯一性

- 那么这就引出我们的

  ```
  全局ID生成器
  ```

  了

  - 全局ID生成器是一种在分布式系统下用来生成全局唯一ID的工具，一般要满足一下特性
    - 唯一性
    - 高可用
    - 高性能
    - 递增性
    - 安全性

- 为了增加ID的安全性，我们可以不直接使用Redis自增的数值，而是拼接一些其他信息

```java
@Component
public class RedisIdWorker {

    private static final long BEGIN_TIMESTAMP = 1640995200L;
    private static final int COUNT_BITS = 32;

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    public long nextId(String keyPrefix) {
        // 生成时间戳
        LocalDateTime now = LocalDateTime.now();
        long nowSecond = now.toEpochSecond(ZoneOffset.UTC);
        long timestamp = nowSecond - BEGIN_TIMESTAMP;
        // 生成序列号
        String date = now.format(DateTimeFormatter.ofPattern("yyyy:MM:dd"));
        Long count = stringRedisTemplate.opsForValue().increment("icr:" + keyPrefix + ":" + date);

        // 拼接
        return timestamp << COUNT_BITS | count;
    }

    public static void main(String[] args) {
        LocalDateTime time = LocalDateTime.of(2022, 1, 1, 0, 0, 0);
        System.out.println(time.toEpochSecond(ZoneOffset.UTC));
    }
}

```

![image-20241122211601722](.\img\image-20241122211601722.png)



### 3.2 添加优惠券

![image-20241125180159053](.\img\image-20241125180159053.png)

![image-20241125184703140](.\img\image-20241125184703140.png)

```java
@PostMapping("seckill")
public Result addSeckillVoucher(@RequestBody Voucher voucher) {
    voucherService.addSeckillVoucher(voucher);
    return Result.ok(voucher.getId());
}
```



### 3.3 实现秒杀下单

![image-20241125185514518](.\img\image-20241125185514518.png)

```java
@Service
public class VoucherOrderServiceImpl extends ServiceImpl<VoucherOrderMapper, VoucherOrder> implements IVoucherOrderService {

    @Resource
    private ISeckillVoucherService seckillVoucherService;
    @Autowired
    private RedisIdWorker redisIdWorker;

    @Transactional
    @Override
    public Result seckillVoucher(Long voucherId) {
        SeckillVoucher voucher = seckillVoucherService.getById(voucherId);

        if (voucher.getBeginTime().isAfter(LocalDateTime.now())) {
            return Result.fail("秒杀尚未开始");
        }

        if (voucher.getEndTime().isBefore(LocalDateTime.now())) {
            return Result.fail("秒杀已经结束");
        }

        if (voucher.getStock() < 1) {
            return Result.fail("库存不足");
        }

        boolean success = seckillVoucherService.update()
                .setSql("stock = stock - 1")
                .eq("voucher_id", voucherId).update();

        if (!success) {
            return Result.fail("库存不足");
        }

        VoucherOrder voucherOrder = new VoucherOrder();
        // 订单id
        long orderId = redisIdWorker.nextId("order");
        voucherOrder.setId(orderId);
        // 用户id
        Long id = UserHolder.getUser().getId();
        voucherOrder.setUserId(id);
        // 代金券id
        voucherOrder.setVoucherId(voucherId);
        save(voucherOrder);
        return Result.ok(orderId);
    }
}

```



### 3.4 超卖问题

并发量大时，会发生超卖

![image-20241125193805938](.\img\image-20241125193805938.png)

**原因**

![image-20241125194052471](.\img\image-20241125194052471.png)

#### 3.4.1 悲观锁

![image-20241125194338064](.\img\image-20241125194338064.png)

#### 3.4.2 乐观锁

![image-20241125194357461](.\img\image-20241125194357461.png)

**乐观锁的关键：** 判断之前查询到的数据是否被修改过



##### 3.4.2.1 版本号法

![image-20241125194705791](.\img\image-20241125194705791.png)



##### 3.4.2.2 CAS法

用库存代替版本

![image-20241125194829804](.\img\image-20241125194829804.png)

```java
boolean success = seckillVoucherService.update()
    .setSql("stock = stock - 1") // set stock = stock - 1
    .eq("voucher_id", voucherId).eq("stock", voucher.getStock()) // where id = ? and stock = ?
    .update();
```

上述代码并不能完全解决问题，1000个线程请求100个优惠券会发生优惠券没有抢完的现象

```java
// 解决方案
boolean success = seckillVoucherService.update()
    .setSql("stock = stock - 1") // set stock = stock - 1
    .eq("voucher_id", voucherId).gt("stock", 0) // where id = ? and stock > 0
    .update();
```

#### 3.4.3 总结

![image-20241125200707607](.\img\image-20241125200707607.png)



### 3.5 一人一单（重点）

![image-20241125200911187](.\img\image-20241125200911187.png)

```java
public Result seckillVoucher(Long voucherId) {
    SeckillVoucher voucher = seckillVoucherService.getById(voucherId);

    if (voucher.getBeginTime().isAfter(LocalDateTime.now())) {
        return Result.fail("秒杀尚未开始");
    }

    if (voucher.getEndTime().isBefore(LocalDateTime.now())) {
        return Result.fail("秒杀已经结束");
    }

    if (voucher.getStock() < 1) {
        return Result.fail("库存不足");
    }

    Long id = UserHolder.getUser().getId();
    int count = query().eq("user_id", id).eq("voucher_id", voucherId).count();
    if (count > 0) {
        return Result.fail("用户已经买过一次");
    }

    boolean success = seckillVoucherService.update()
        .setSql("stock = stock - 1") // set stock = stock - 1
        .eq("voucher_id", voucherId).gt("stock", 0) // where id = ? and stock = ?
        .update();

    if (!success) {
        return Result.fail("库存不足");
    }

    VoucherOrder voucherOrder = new VoucherOrder();
    // 订单id
    long orderId = redisIdWorker.nextId("order");
    voucherOrder.setId(orderId);
    // 用户id
    voucherOrder.setUserId(id);
    // 代金券id
    voucherOrder.setVoucherId(voucherId);
    save(voucherOrder);
    return Result.ok(orderId);
}
```

上述方案仍旧有问题

执行200个并发线程后出现以下问题，该问题与超卖问题相同，但是超卖问题可以使用乐观锁，**因为是修改数据**，一人一单问题是**新增数据**，要使用悲观锁

![image-20241125204004240](.\img\image-20241125204004240.png)

```java
@Service
public class VoucherOrderServiceImpl extends ServiceImpl<VoucherOrderMapper, VoucherOrder> implements IVoucherOrderService {

    @Resource
    private ISeckillVoucherService seckillVoucherService;
    @Autowired
    private RedisIdWorker redisIdWorker;

    @Override
    public Result seckillVoucher(Long voucherId) {
        SeckillVoucher voucher = seckillVoucherService.getById(voucherId);

        if (voucher.getBeginTime().isAfter(LocalDateTime.now())) {
            return Result.fail("秒杀尚未开始");
        }

        if (voucher.getEndTime().isBefore(LocalDateTime.now())) {
            return Result.fail("秒杀已经结束");
        }

        if (voucher.getStock() < 1) {
            return Result.fail("库存不足");
        }

        Long id = UserHolder.getUser().getId();
        synchronized(id.toString().intern()){
            IVoucherOrderService proxy = (IVoucherOrderService) AopContext.currentProxy();
            return proxy.createVoucherOrder(voucherId);
        }

    }

    @Transactional
    public Result createVoucherOrder(Long voucherId) {
        Long id = UserHolder.getUser().getId();

        // 不加intern()即使id一样，地址可能不同，不会去常量池寻找

        int count = query().eq("user_id", id).eq("voucher_id", voucherId).count();
        if (count > 0) {
            return Result.fail("用户已经买过一次");
        }

        boolean success = seckillVoucherService.update()
                .setSql("stock = stock - 1") // set stock = stock - 1
                .eq("voucher_id", voucherId).gt("stock", 0) // where id = ? and stock = ?
                .update();

        if (!success) {
            return Result.fail("库存不足");
        }

        VoucherOrder voucherOrder = new VoucherOrder();
        // 订单id
        long orderId = redisIdWorker.nextId("order");
        voucherOrder.setId(orderId);
        // 用户id
        voucherOrder.setUserId(id);
        // 代金券id
        voucherOrder.setVoucherId(voucherId);
        save(voucherOrder);
        return Result.ok(orderId);
    }
}

```

### 3.6 集群下并发问题

![image-20241125211552171](.\img\image-20241125211552171.png)

- 通过加锁可以解决在单机情况下的一人一单安全问题，但是在集群模式下就不行了
  1. 我们将服务启动两份，端口分别为8081和8082
  2. 然后修改nginx的config目录下的nginx.conf文件，配置反向代理和负载均衡（默认轮询就行）
- 具体操作，我们使用`POSTMAN`发送两次请求，header携带同一用户的token，尝试用同一账号抢两张优惠券，发现是可行的。
- 失败原因分析：由于我们部署了多个Tomcat，每个Tomcat都有一个属于自己的jvm，那么假设在服务器A的Tomcat内部，有两个线程，即线程1和线程2，这两个线程使用的是同一份代码，那么他们的锁对象是同一个，是可以实现互斥的。但是如果在Tomcat的内部，又有两个线程，但是他们的锁对象虽然写的和服务器A一样，但是锁对象却不是同一个，所以线程3和线程4可以实现互斥，但是却无法和线程1和线程2互斥
  [![img](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7)](https://pic1.imgdb.cn/item/635a5e3e16f2c2beb1289579.jpg)
- 这就是集群环境下，syn锁失效的原因，在这种情况下，我们需要使用分布式锁来解决这个问题，让锁不存在于每个jvm的内部，而是让所有jvm公用外部的一把锁（Redis）
