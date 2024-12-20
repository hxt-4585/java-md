# Mybatis

## 一、Mybatis入门程序

### 1、SqlSession、SqlSessionFactory、SqlSessionFactoryBuilder

​	在 Mybatis 中，负责执行SQL语句的对象叫做SqlSession，是一个Java程序和数据库之间的会话。SqlSession对象获取流程如下：

​	SqlSessionFactoryBuilder→  SqlSessionFactory→  SqlSession

```java
SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
// 下面这种方式移植性太差
// InputStream is = new FileInputStream("E:\\Users\\IdeaProjects\\mybatis\\mybatis-001\\src\\main\\resources\\mybatis-config.xml");


// 这种方式只能从类路径当中获取资源，也就是说mybatis-config.xml文件需要在类路径下。一般情况下与Resources有关，都是从类路径开始寻找
// 类路径指的是编译以后classes目录
// InputStream is = Thread.currentThread().getContextClassLoader().getResourceAsStream("mybatis-config.xml");
InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
// 一般情况下，一个数据库对应一个sqlSessionFactory对象
SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(is);
sqlSession = sqlSessionFactory.openSession();
```



## 二、Mybatis核心配置文件

### 1、多环境

```xml
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN" 
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!-- configuration为根标签，http://mybatis.org/dtd/mybatis-3-config.dtd记录了约束条件 -->
<configuration>
    <!-- 在没有明确指定使用哪个环境时，根据default来选择 -->
    <environments default="powernode">

        <!-- 一般一个数据库对应一个SqlSessionFactory对象 -->
        <!-- 一个环境environment对应一个SqlSessionFactory对象 -->
        <environment id="powernode">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="XXX"/>
                <property name="url" value="XXX"/>
                <property name="username" value="XXX"/>
                <property name="password" value="XXX"/>
            </dataSource>
        </environment>
        
        <environment id="mybatis">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="XXX"/>
                <property name="url" value="XXX"/>
                <property name="username" value="XXX"/>
                <property name="password" value="XXX"/>
            </dataSource>
        </environment>
        
    </environments>
</configuration>
```

```java
// 指定mybatis
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsReader("mybatis-config.xml"), "mybatis");
SqlSession sqlSession = sqlSessionFactory.openSession();
```



### 2、Mybatis事务管理机制

```xml
<transactionManager type="JDBC"/>
```

​	Mybatis事务管理机制有两个类型分别是JDBC和MANAGED。

​	JDBC事务管理器：

​		mybatis框架自己管理事务，自己采用原生的 JDBC 代码去管理事务：

```java
	    conn.setAutoCommit(false); // 开启事务
	    //...业务处理...//
	    conn.commit(); //手动提交事务
```

​		使用JDBC事务管理器的话，就是底层创建事务管理器对象：JdbcTransaction对象。

​		如果编写的代码如下：

```java
		SqlSession sqlsession = sqlSessionFactory.openSession(true);
		// 表示没有开启事务，所以也不会执行conn.setAutoCommit(false);
```

​	MANAGED事务管理器：

​		mybatis不在负责处理事务，由其他容器来处理，如spring。

​		

### 3、数据源

```xml
<dataSource type="POOLED">
    <property name="driver" value="XXX"/>
    <property name="url" value="XXX"/>
    <property name="username" value="XXX"/>
    <property name="password" value="XXX"/>
</dataSource>
<!--
	1、dataSource被称为数据源。
	2、dataSource为程序提供Connection对象，但凡提供Connection对象的都叫做数据源。
	3、数据源实际上是一套规范，在JDK中。
	4、我们自己也可以写一套数据源组件，只要实现javax.sql.DataSource接口。
	5、常见的数据源有druid、c3p0、dbcp。
	6、type属性可以有UNPOOLED、POOLED、JNDI，分别代表不使用连接池、使用mybatis提供的连接池、使用第三方连接池。不使用连接池，每一次java要和数据库建立连接对象，非常消耗资源。
-->
```



## 三、Mybatis底层原理



## 四、Mybatis技巧

### 1、区分${}和#{}

```xml
<select id="selectByCarType" resultType="com.powernode.mybatis.pojo.Car">
    select
        id,car_num as carNum,brand,guide_price as guidePrice,produce_time as produceTime,car_type as carType
    from t_car 
    where car_type = #{carType}
</select>
```

执行上述程序后结果如下：

![image-20240813162050088](.\img\image-20240813162050088.png)

可见在 car_type 后有一个 ? ，这里的 #{} 会对sql语句进行预编译。



```xml
<select id="selectByCarType" resultType="com.powernode.mybatis.pojo.Car">
    select
        id,car_num as carNum,brand,guide_price as guidePrice,produce_time as produceTime,car_type as carType
    from t_car 
    where car_type = ${carType}
</select>
```

执行上述程序后结果如下：

![image-20240813162357293](.\img\image-20240813162357293.png)

发生编译报错，因为 **燃油车** 是一个字符串，在sql语句中应该添加单引号，修改上述xml文件，将不会报错。

```xml
<select id="selectByCarType" resultType="com.powernode.mybatis.pojo.Car">
    select
        id,car_num as carNum,brand,guide_price as guidePrice,produce_time as produceTime,car_type as carType
    from t_car 
    where car_type = '${carType}'
</select>
```



什么情况下必须使用 ${} ：

#### 情况1：

```java
/**
 * 查询所有的Car
 * @param ascOrDesc asc或desc
 * @return
 */
List<Car> selectAll(String ascOrDesc);
```

```xml
<select id="selectAll" resultType="com.powernode.mybatis.pojo.Car">
  select
  id,car_num as carNum,brand,guide_price as guidePrice,produce_time as produceTime,car_type as carType
  from
  t_car
  order by carNum #{key}
</select>
```

![image-20240813163645457](.\img\image-20240813163645457.png)

最终sql语句会是这样：

```sql
select id,car_num as carNum,brand,guide_price as guidePrice,produce_time as produceTime,car_type as carType from t_car order by carNum 'desc'
```

desc是一个关键字，不能带单引号的，所以在进行sql语句关键字拼接的时候，必须使用${}



#### 情况2：拼接表名

​	实际开发中，有的表数据量非常庞大，可能会采用分表方式进行存储，比如每天生成一张表，表的名字与日期挂钩，例如：2022年8月1日生成的表：t_user20220108。2000年1月1日生成的表：t_user20000101。

​	使用#{}会是这样：select * from 't_car'

​	使用${}会是这样：select * from t_car

```xml
<select id="selectAllByTableName" resultType="car">
  select
  id,car_num as carNum,brand,guide_price as guidePrice,produce_time as produceTime,car_type as carType
  from
  ${tableName}
</select>
```



#### 情况3：批量删除

```sql
delete from t_user where id in('1,2,3') # 执行错误：1292 - Truncated incorrect DOUBLE value: '1,2,3'
delete from t_user where id in(1, 2, 3)
```

```xml
<delete id="deleteBatch">
  delete from t_car where id in(${ids})
</delete>
```



#### 情况4：模糊查询

```xml
<!-- 使用${}
select * from t_car where car_type like "%油%" 
-->
<select id="selectLikeByBrand" resultType="Car">
  select
  id,car_num as carNum,brand,guide_price as guidePrice,produce_time as produceTime,car_type as carType
  from
  t_car
  where
  brand like '%${brand}%'
</select>

<!-- 使用#{} 
select * from t_car where car_type like "%"'油'"%"
-->
<select id="selectLikeByBrand" resultType="Car">
  select
  id,car_num as carNum,brand,guide_price as guidePrice,produce_time as produceTime,car_type as carType
  from
  t_car
  where
  brand like concat('%',#{brand},'%')
</select>
```



### 2、typeAliases 名称简写

```xml
<!-- 方式一（不建议）
适用于pojo包中类比较少的情况 
-->
<typeAliases>
  <typeAlias type="com.powernode.mybatis.pojo.Car" alias="Car"/>
</typeAliases>

<!-- 方式二 -->
<typeAliases>
  <package name="com.powernode.mybatis.pojo"/>
</typeAliases>
```



### 3、mappers的配置

```xml
<!-- 方式一：resource
	必须在类路径下
-->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>
</mappers>

<!-- 方式二：url
	采用了绝对路径，mapper文件可以放在任意位置 
-->
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
  <mapper url="file:///var/mappers/BlogMapper.xml"/>
  <mapper url="file:///var/mappers/PostMapper.xml"/>
</mappers>


<!-- 方式三：class
    SQL映射文件和mapper接口放在同一个目录下。
    SQL映射文件的名字也必须和mapper接口名一致。
-->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <mapper class="org.mybatis.builder.BlogMapper"/>
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>

<!-- 如果class较多，可以使用这种package的方式，但前提条件和上一种方式一样。 -->
<mappers>
  <package name="com.powernode.mybatis.mapper"/>
</mappers>
```



### 4、插入数据时获取自动生成的主键

```xml
<!--
    useGeneratedKeys="true"：启用获取数据库生成的主键。
    keyProperty="id"：指定将生成的主键值存储到 Car 对象的 id 属性中。
-->
<insert id="insertUseGeneratedKeys" useGeneratedKeys="true" keyProperty="id">
  insert into t_car(id,car_num,brand,guide_price,produce_time,car_type) values(null,#{carNum},#{brand},#{guidePrice},#{produceTime},#{carType})
</insert>
```

```java
@Test
public void testInsertUseGeneratedKeys(){
    Car car = new Car(null, "10", "比亚迪", 11.11, "2022-11-11", "燃油车");
    CarMapper mapper = (CarMapper) SqlSessionUtil.openSession().getMapper(CarMapper.class);
    int count = mapper.insertUseGeneratedKeys(car);
    System.out.println(count);
    SqlSessionUtil.openSession().commit();
    System.out.println(car.getId());
}
```



#### 5、param参数注解

```java
// 自动生成一个map{"name" : name, "age" : age}
// 如果不加注解，那么map的键为 "arg0"、"arg1"、"param1"、"params"，arg0和param1所对应的值一样，两者可以混合使用
List<Student> selectByNameAndAge(@Param(value="name") String name, @Param("age") int age);
```



## 五、MyBatis查询语句专题

### 1、Map<String,Map>

```java
/**
     * 获取所有的Car，返回一个Map集合。
     * Map集合的key是Car的id。
     * Map集合的value是对应Car。
     * @return
     */
@MapKey("id")
Map<Long,Map<String,Object>> selectAllRetMap();
```

```xml
<select id="selectAllRetMap" resultType="map">
  select id,car_num carNum,brand,guide_price guidePrice,produce_time produceTime,car_type carType from t_car
</select>
```

### 2、resultMap

```xml
<!--
        resultMap:
            id：这个结果映射的标识，作为select标签的resultMap属性的值。
            type：结果集要映射的类。可以使用别名。
-->
<resultMap id="carResultMap" type="car">
  <!--对象的唯一标识，官方解释是：为了提高mybatis的性能。建议写上。-->
  <id property="id" column="id"/>
  <result property="carNum" column="car_num"/>
  <!--当属性名和数据库列名一致时，可以省略。但建议都写上。-->
  <!--javaType用来指定属性类型。jdbcType用来指定列类型。一般可以省略。-->
  <result property="brand" column="brand" javaType="string" jdbcType="VARCHAR"/>
  <result property="guidePrice" column="guide_price"/>
  <result property="produceTime" column="produce_time"/>
  <result property="carType" column="car_type"/>
</resultMap>

<!--resultMap属性的值必须和resultMap标签中id属性值一致。-->
<select id="selectAllByResultMap" resultMap="carResultMap">
  select * from t_car
</select>

<!-- 是否开启驼峰命名
	在config文件中设置。
 	carType  ==>   car_type
	guidePrice  ==>   guide_price
	...
-->

<settings>
  <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```



## 六、动态SQL

### 1、where标签

```xml
<!-- 可以自动去掉前面多余的and，不可以自动去掉后面多余的and -->
<select id="selectByMultiConditionWithWhere" resultMap="carResultMap">
    select * from t_car
    <where>
        <if test="brand != null and brand != ''">
            and brand like "%"#{brand}"%"
        </if>
        <if test="guidePrice != null and guidePrice != ''">
            and guide_price >= #{guidePrice}
        </if>
        <if test="carType != null and carType != ''">
            and car_type = #{carType}
        </if>
    </where>
</select>
```



### 2、trim标签

```xml
<!--
	prefix：在trim内容前面加上前缀（动态），比如where
	suffix：加内容后加上后缀
	prefixOverrides：删除前缀
	suffixOverrides：删除后缀
-->
<select id="selectByMultiConditionWithTrim" resultType="car">
  select * from t_car
  <trim prefix="where" suffixOverrides="and|or">
    <if test="brand != null and brand != ''">
      brand like #{brand}"%" and
    </if>
    <if test="guidePrice != null and guidePrice != ''">
      guide_price >= #{guidePrice} and
    </if>
    <if test="carType != null and carType != ''">
      car_type = #{carType}
    </if>
  </trim>
</select>
```



### 3、set标签

```xml
<!--
    主要使用在update语句当中，用来生成set关键字，同时去掉最后多余的“,”
    比如我们只更新提交的不为空的字段，如果提交的数据是空或者""，那么这个字段我们将不更新
-->
<update id="updateWithSet">
  update t_car
  <set>
    <if test="carNum != null and carNum != ''">car_num = #{carNum},</if>
    <if test="brand != null and brand != ''">brand = #{brand},</if>
    <if test="guidePrice != null and guidePrice != ''">guide_price = #{guidePrice},</if>
    <if test="produceTime != null and produceTime != ''">produce_time = #{produceTime},</if>
    <if test="carType != null and carType != ''">car_type = #{carType},</if>
  </set>
  where id = #{id}
</update>
```



### 4、choose标签

```xml
<!--
	先根据品牌查询，如果没有提供品牌，再根据指导价格查询，如果没有提供指导价格，就根据生产日期查询。    
-->
<select id="selectWithChoose" resultType="car">
  select * from t_car
  <where>
    <choose>
      <when test="brand != null and brand != ''">
        brand like #{brand}"%"
      </when>
      <when test="guidePrice != null and guidePrice != ''">
        guide_price >= #{guidePrice}
      </when>
      <otherwise>
        produce_time >= #{produceTime}
      </otherwise>
    </choose>
  </where>
</select>
```



### 5、foreach标签

```java
/**
* 通过foreach完成批量删除
* @param ids
* @return
*/
int deleteBatchByForeach(@Param("ids") Long[] ids);

/**
* 批量添加，使用foreach标签
* @param cars
* @return
*/
int insertBatchByForeach(@Param("cars") List<Car> cars);

```

```xml
<!--
    collection：集合或数组
    item：集合或数组中的元素
    separator：分隔符
    open：foreach标签中所有内容的开始
    close：foreach标签中所有内容的结束
-->
<!-- 使用in删除 -->
<delete id="deleteBatchByForeach">
  delete from t_car where id in
  <foreach collection="ids" item="id" separator="," open="(" close=")">
    #{id}
  </foreach>
</delete>

<!-- 使用or来删除 -->
<delete id="deleteBatchByForeach2">
  delete from t_car where
  <foreach collection="ids" item="id" separator="or">
    id = #{id}
  </foreach>
</delete>

<insert id="insertBatchByForeach">
  insert into t_car values 
  <foreach collection="cars" item="car" separator=",">
    (null,#{car.carNum},#{car.brand},#{car.guidePrice},#{car.produceTime},#{car.carType})
  </foreach>
</insert>
```



### 6、sql和include标签

```xml
<sql id="carCols">id,car_num carNum,brand,guide_price guidePrice,produce_time produceTime,car_type carType</sql>

<select id="selectAllRetMap" resultType="map">
  select <include refid="carCols"/> from t_car
</select>

<select id="selectAllRetListMap" resultType="map">
  select <include refid="carCols"/> carType from t_car
</select>

<select id="selectByIdRetMap" resultType="map">
  select <include refid="carCols"/> from t_car where id = #{id}
</select>
```



## 七、mybatis高级映射

### 1、多对一

```java
// clazz类
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Clazz {
    private Integer cid;
    private String cname;

}

// student类
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Student {
    private Integer sid;
    private String sname;
    private Clazz clazz;
}

```

```xml
<!-- 方式一
	级联 
-->
<resultMap id="studentResultMap" type="student">
    <id property="sid" column="sid"/>
    <result property="sname" column="sname"/>
    <result property="clazz.cid" column="cid"/>
    <result property="clazz.cname" column="cname"/>
</resultMap>
<select id="selectBySid" resultMap="studentResultMap">
    select s.*, c.* from t_stu s join t_clazz c on s.cid = c.cid where sid = #{sid}
</select>


<!-- 方式二
	 association
-->
<resultMap id="studentResultMap" type="student">
    <id property="sid" column="sid"/>
    <result property="sname" column="sname"/>
    <association property="clazz" javaType="Clazz">
        <id property="cid" column="cid"/>
        <result property="cname" column="cid"/>
    </association>
</resultMap>


<!-- 方式三
	分部查询
-->
<resultMap id="studentResultMap" type="student">
    <id property="sid" column="sid"/>
    <result property="sname" column="sname"/>
    <association property="clazz"
                 select="com.powernode.mybatis.mapper.ClazzMapper.selectByCid"
                 column="cid"/>
</resultMap>
<select id="selectBySid" resultMap="studentResultMap">
    <!-- 查出来的cid放到association中 -->
    select sid,sname, cid from t_stu where sid = #{sid}
</select>
<select id="selectByCid" resultType="Clazz">
    select * from t_clazz where cid = #{cid}
</select>
<!-- 懒加载 
	在 association 标签中将属性设置为 fetchType="lazy"
-->
<!-- 开启全局懒加载（所有带有分布查询的都会开启）
	若某一些分布查询不需要开启只需fetchType="eager" 
-->
<settings>
  <setting name="lazyLoadingEnabled" value="true"/>
</settings>
```



### 2、一对多

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Clazz {
    private Integer cid;
    private String cname;
    private List<Student> students;

}

```



```xml
<!-- 方式一
	collection
-->
<resultMap id="clazzResultMap" type="Clazz">
    <id property="cid" column="cid"/>
    <result property="cname" column="cname"/>
    <collection property="students" ofType="student">
        <id property="sid" column="sid"/>
        <result property="sname" column="sname"/>
    </collection>
</resultMap>
<select id="selectClazzAndStudentsByCid" resultMap="clazzResultMap">
    select * from t_clazz c join t_stu s on c.cid = s.cid where c.cid = #{cid}
</select>

<!-- 分布查询
	延时加载同样适用 
-->
<resultMap id="clazzResultMap" type="Clazz">
    <id property="cid" column="cid"/>
    <result property="cname" column="cname"/>
    <collection property="students"
                select="com.powernode.mybatis.mapper.StudentMapper.selectByCid"
                column="cid"
                fetchType="lazy"
                />
</resultMap>
<select id="selectClazzAndStudentsByCid" resultMap="clazzResultMap">
    select * from t_clazz where cid = #{cid}
</select>
<select id="selectByCid" resultType="Student">
    select * from t_stu where cid = #{cid}
</select>
```

## 八、缓存

一级缓存：将查询的数据存储到SqlSession中。

二级缓存：将查询的数据存储到SqlSessionFactory中。



### （1）、一级缓存

```java
// 只有在同一个会话下才会触发缓存（同一个sqlSession对象），且必须是DQL语句。
@Test
public void testSelectByCache(){
    SqlSession sqlSession = SqlSessionUtil.openSession();
    CarMapper mapper = sqlSession.getMapper(CarMapper.class);
    Map<String, Object> car1 = mapper.selectByIdRetMap(165L);
    System.out.println(car1);

    Map<String, Object> car2 = mapper.selectByIdRetMap(165L);
    System.out.println(car2);
}
```

![image-20240818143718681](.\img\image-20240818143718681.png)

一级缓存失效：

​	1、执行了sqlSession.clearCache()，手动清空缓存。

​	2、两条DQL语句之间执行了 DELETE 和 UPDATE 语句。



### （2）、二级缓存

```java
@AllArgsConstructor
@NoArgsConstructor
@Data
public class Car implements Serializable {
    private Long id;
    private String carNum;
    private String brand;
    private Double guidePrice;
    private String produceTime;
    private String carType;
    // 构造方法
    // set get方法
    // toString方法
}

@Test
public void testSelectById2() throws Exception{
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("mybatis-config.xml"));

    SqlSession sqlSession1 = sqlSessionFactory.openSession();
    CarMapper mapper1 = sqlSession1.getMapper(CarMapper.class);
    Map car1 = mapper1.selectByIdRetMap(165L);
    System.out.println(car1);

    // 关键一步
    sqlSession1.close();

    SqlSession sqlSession2 = sqlSessionFactory.openSession();
    CarMapper mapper2 = sqlSession2.getMapper(CarMapper.class);
    Map car2 = mapper2.selectByIdRetMap(165L);
    System.out.println(car2);
}
```

![image-20240818152544806](.\img\image-20240818152544806.png)



默认情况下，二级缓存是开启的，使用二级缓存的条件：

​	1、<setting name="cacheEnabled" value="true"> 全局性地开启或关闭所有映射器文件中已配置的任何缓存，默认是true，无需设置。

​	2、在需要使用二级缓存的SqlMapper.xml文件中添加：<cache/>

​	3、使用二级缓存的实体类对象必须是可序列化的，也就是必须实现java.io.Serializable接口

​	4、SqlSession对象关闭或提交之后，一级缓存中的数据才会被写入到二级缓存，此时二级缓存才能使用



二级缓存的相关配置：

![image-20240818152738672](.\img\image-20240818152738672.png)

​	1、eviction：指定从缓存中移除某个对象的淘汰算法。默认采用LRU策略。

​		a、LRU：Least Recently Used。最近最少使用。优先淘汰在间隔时间内使用频率最低的对象。(其实还有一种淘汰算法LFU，最不常用。)

​		b、FIFO：First In First Out。一种先进先出的数据缓存器。先进入二级缓存的对象最先被淘汰。

​		c、SOFT：软引用。淘汰软引用指向的对象。具体算法和JVM的垃圾回收算法有关。

​		d、WEAK：弱引用。淘汰弱引用指向的对象。具体算法和JVM的垃圾回收算法有关。

​	2、flushInterval：

​		a、二级缓存的刷新时间间隔。单位毫秒。如果没有设置。就代表不刷新缓存，只要内存足够大，一直会向二级缓存中缓存数据。除非执行了增删改。

​	3、readOnly：

​		a、rue：多条相同的sql语句执行之后返回的对象是共享的同一个。性能好。但是多线程并发可能会存在安全问题。

​		b、多条相同的sql语句执行之后返回的对象是副本，调用了clone方法。性能一般。但安全。

​	4、size：

​		a、设置二级缓存中最多可存储的java对象数量。默认值1024。



### （3）、集成EhCache

​	集成EhCache是为了代替mybatis自带的二级缓存。一级缓存是无法替代的。

```xml
<!--mybatis集成ehcache的组件-->
<dependency>
  <groupId>org.mybatis.caches</groupId>
  <artifactId>mybatis-ehcache</artifactId>
  <version>1.2.2</version>
</dependency>
```

```xml
<!-- 新建echcache.xml文件 -->
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
         updateCheck="false">
    <!--磁盘存储:将缓存中暂时不使用的对象,转移到硬盘,类似于Windows系统的虚拟内存-->
    <diskStore path="e:/ehcache"/>
  
    <!--defaultCache：默认的管理策略-->
    <!--eternal：设定缓存的elements是否永远不过期。如果为true，则缓存的数据始终有效，如果为false那么还要根据timeToIdleSeconds，timeToLiveSeconds判断-->
    <!--maxElementsInMemory：在内存中缓存的element的最大数目-->
    <!--overflowToDisk：如果内存中数据超过内存限制，是否要缓存到磁盘上-->
    <!--diskPersistent：是否在磁盘上持久化。指重启jvm后，数据是否有效。默认为false-->
    <!--timeToIdleSeconds：对象空闲时间(单位：秒)，指对象在多长时间没有被访问就会失效。只对eternal为false的有效。默认值0，表示一直可以访问-->
    <!--timeToLiveSeconds：对象存活时间(单位：秒)，指对象从创建到失效所需要的时间。只对eternal为false的有效。默认值0，表示一直可以访问-->
    <!--memoryStoreEvictionPolicy：缓存的3 种清空策略-->
    <!--FIFO：first in first out (先进先出)-->
    <!--LFU：Less Frequently Used (最少使用).意思是一直以来最少被使用的。缓存的元素有一个hit 属性，hit 值最小的将会被清出缓存-->
    <!--LRU：Least Recently Used(最近最少使用). (ehcache 默认值).缓存的元素有一个时间戳，当缓存容量满了，而又需要腾出地方来缓存新的元素的时候，那么现有缓存元素中时间戳离当前时间最远的元素将被清出缓存-->
    <defaultCache eternal="false" maxElementsInMemory="1000" overflowToDisk="false" diskPersistent="false"
                  timeToIdleSeconds="0" timeToLiveSeconds="600" memoryStoreEvictionPolicy="LRU"/>

</ehcache>
```

```xml
<!-- 修改SqlMapper.xml文件中的<cache/>标签，添加type属性。 -->
<cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
```

```java
@Test
public void testSelectById2() throws Exception{
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("mybatis-config.xml"));

    SqlSession sqlSession1 = sqlSessionFactory.openSession();
    CarMapper mapper1 = sqlSession1.getMapper(CarMapper.class);
    Map car1 = mapper1.selectByIdRetMap(165L);
    System.out.println(car1);

    // 关键一步
    sqlSession1.close();

    SqlSession sqlSession2 = sqlSessionFactory.openSession();
    CarMapper mapper2 = sqlSession2.getMapper(CarMapper.class);
    Map car2 = mapper2.selectByIdRetMap(165L);
    System.out.println(car2);
}
```



## 十、注解式开发

```java
@Insert(value="insert into t_car values(null,#{carNum},#{brand},#{guidePrice},#{produceTime},#{carType})")
int insert(Car car);

@Delete("delete from t_car where id = #{id}")
int deleteById(Long id);

@Update("update t_car set car_num=#{carNum},brand=#{brand},guide_price=#{guidePrice},produce_time=#{produceTime},car_type=#{carType} where id=#{id}")
int update(Car car);

@Select("select * from t_car where id = #{id}")
@Results({
    @Result(column = "id", property = "id", id = true),
    @Result(column = "car_num", property = "carNum"),
    @Result(column = "brand", property = "brand"),
    @Result(column = "guide_price", property = "guidePrice"),
    @Result(column = "produce_time", property = "produceTime"),
    @Result(column = "car_type", property = "carType")
})
Car selectById(Long id);
```

