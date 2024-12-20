# SpringCloud



## 附

原文链接：https://blog.csdn.net/qq_64225133/article/details/139359850





## 一、SpringCloud介绍

### 1.1 组件介绍

在2019年之前，使用的大部分技术都是Netflix提供的，但是由于开发SpringCloud的相关技术不挣钱，因此Netflix就暂停开发相关技术了，虽然他提供的那些技术依旧可以使用，但是已经不推荐了。

因此该笔记只学习新的架构，对于老的技术栈，如果老项目中需要用到，请去B站继续学习

![](F:\java\SpringCloud\img\245811292da5633265be7a62a17595f2.png)

**注册与发现**
	Eureka【Netflix最后的火种，不推荐】
	Consul【推荐使用】
	Etcd【可以使用】
	Nacos【推荐使用，发音：呐扣丝，阿里巴巴提供的】
**服务调用和负载均衡**
	Ribbon【Netflix提供的，建议直接弃用】
	OpenFeign
	LoadBalancer
**分布式事务**
	Seata【推荐使用，阿里巴巴的】
	LCN
	Hmily
**服务熔断和降级**
	Hystrix【已经停更了，不推荐】
	Circuit Breaker【这只一套规范，使用的是它的实现类】
	Resilience4J【CircuitBreaker的实现类，可以使用】
	Sentinel【阿里巴巴的，推荐使用】
**服务链路追踪**
	Sleuth+Zipkin【逐渐被替代了，不推荐】
	Micrometer Tracing【推荐使用】
**服务网关**
	Zuul【不推荐使用】
	Gate Way
**分布式配置管理**
	Config+Bus【不推荐了】
	Consul
	Nacos

## 二、单体项目构建

### 2.1 项目构建

新建一个Maven工程，除了`pom.xml`、`.idea`其他的东西都删了

![](F:\java\SpringCloud\img\06bcceaac639a7b7b32d3a1acc9a6898.png)

检查项目的编码格式，统一为UTF-8

![](F:\java\SpringCloud\img\7f2810fbb3d37409a1b75ffd42e51250.png)

检查注解支撑是否打开

![](F:\java\SpringCloud\img\ca5b9dcf38f639c90181ab5002671175.png)

检查java编译版本

![](F:\java\SpringCloud\img\cde43e5cff74c938d51d6ee1f9e8a22c.png)

父工程的pom文件导入依赖，然后刷新

```xml
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <hutool.version>5.8.22</hutool.version>
        <druid.version>1.1.20</druid.version>
        <mybatis.springboot.version>3.0.3</mybatis.springboot.version>
        <mysql.version>8.0.11</mysql.version>
        <swagger3.version>2.2.0</swagger3.version>
        <mapper.version>4.2.3</mapper.version>
        <fastjson2.version>2.0.40</fastjson2.version>
        <persistence-api.version>1.0.2</persistence-api.version>
        <spring.boot.test.version>3.1.5</spring.boot.test.version>
        <spring.boot.version>3.2.0</spring.boot.version>
        <spring.cloud.version>2023.0.0</spring.cloud.version>
        <spring.cloud.alibaba.version>2022.0.0.0-RC2</spring.cloud.alibaba.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <!--springboot 3.2.0-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-parent</artifactId>
                <version>${spring.boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--springcloud 2023.0.0-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--springcloud alibaba 2022.0.0.0-RC2-->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring.cloud.alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--SpringBoot集成mybatis-->
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>${mybatis.springboot.version}</version>
            </dependency>
            <!--Mysql数据库驱动8 -->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql.version}</version>
            </dependency>
            <!--SpringBoot集成druid连接池-->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid-spring-boot-starter</artifactId>
                <version>${druid.version}</version>
            </dependency>
            <!--通用Mapper4之tk.mybatis-->
            <dependency>
                <groupId>tk.mybatis</groupId>
                <artifactId>mapper</artifactId>
                <version>${mapper.version}</version>
            </dependency>
            <!--persistence-->
            <dependency>
                <groupId>javax.persistence</groupId>
                <artifactId>persistence-api</artifactId>
                <version>${persistence-api.version}</version>
            </dependency>
            <!-- fastjson2 -->
            <dependency>
                <groupId>com.alibaba.fastjson2</groupId>
                <artifactId>fastjson2</artifactId>
                <version>${fastjson2.version}</version>
            </dependency>
            <!-- swagger3 调用方式 http://你的主机IP地址:5555/swagger-ui/index.html -->
            <dependency>
                <groupId>org.springdoc</groupId>
                <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
                <version>${swagger3.version}</version>
            </dependency>
            <!--hutool-->
            <dependency>
                <groupId>cn.hutool</groupId>
                <artifactId>hutool-all</artifactId>
                <version>${hutool.version}</version>
            </dependency>
            <!-- spring-boot-starter-test -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <version>${spring.boot.test.version}</version>
                <scope>test</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

```

建库建表，表明`t_pay`

```sql
create database db2024;
use db2024;
DROP TABLE IF EXISTS `t_pay`;
CREATE TABLE `t_pay` (
  `id` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `pay_no` VARCHAR(50) NOT NULL COMMENT '支付流水号',
  `order_no` VARCHAR(50) NOT NULL COMMENT '订单流水号',
  `user_id` INT(10) DEFAULT '1' COMMENT '用户账号ID',
  `amount` DECIMAL(8,2) NOT NULL DEFAULT '9.9' COMMENT '交易金额',
  `deleted` TINYINT(4) UNSIGNED NOT NULL DEFAULT '0' COMMENT '删除标志，默认0不删除，1删除',
  `create_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`)

) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COMMENT='支付交易表';
INSERT INTO t_pay(pay_no,order_no) VALUES('pay17203699','6544bafb424a');
SELECT * FROM t_pay;

```



### 2.2 MyBatis逆向工程

新建模块`mybatis_generator2024`

![image-20241209190601493](F:\java\SpringCloud\img\image-20241209190601493.png)

导入依赖

```xml
<dependencies>
        <!--Mybatis 通用mapper tk单独使用，自己独有+自带版本号-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.13</version>
        </dependency>
        <!-- Mybatis Generator 自己独有+自带版本号-->
        <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-core</artifactId>
            <version>1.4.2</version>
        </dependency>
        <!--通用Mapper-->
        <dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper</artifactId>
        </dependency>
        <!--mysql8.0-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!--persistence-->
        <dependency>
            <groupId>javax.persistence</groupId>
            <artifactId>persistence-api</artifactId>
        </dependency>
        <!--hutool-->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
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
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <build>
        <resources>
            <resource>
                <directory>${basedir}/src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
            <resource>
                <directory>${basedir}/src/main/resources</directory>
            </resource>
        </resources>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.4.2</version>
                <configuration>
                    <configurationFile>${basedir}/src/main/resources/generatorConfig.xml</configurationFile>
                    <overwrite>true</overwrite>
                    <verbose>true</verbose>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>8.0.33</version>
                    </dependency>
                    <dependency>
                        <groupId>tk.mybatis</groupId>
                        <artifactId>mapper</artifactId>
                        <version>4.2.3</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>

```

在子模块的resources下新建文件`config.properties`

```properties
#t_pay表包名
package.name=com.hxt.cloud

# mysql8.0
jdbc.driverClass = com.mysql.cj.jdbc.Driver
jdbc.url= jdbc:mysql://localhost:3306/db2024?characterEncoding=utf8&useSSL=false&serverTimezone=GMT%2B8&rewriteBatchedStatements=true&allowPublicKeyRetrieval=true
jdbc.user = root
jdbc.password =20170440

```

在子模块的resources下新建文件`generatorConfig.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <properties resource="config.properties"/>

    <context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>

        <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
            <property name="mappers" value="tk.mybatis.mapper.common.Mapper"/>
            <property name="caseSensitive" value="true"/>
        </plugin>

        <jdbcConnection driverClass="${jdbc.driverClass}"
                        connectionURL="${jdbc.url}"
                        userId="${jdbc.user}"
                        password="${jdbc.password}">
        </jdbcConnection>

        <javaModelGenerator targetPackage="${package.name}.entities" targetProject="src/main/java"/>

        <sqlMapGenerator targetPackage="${package.name}.mapper" targetProject="src/main/java"/>

        <javaClientGenerator targetPackage="${package.name}.mapper" targetProject="src/main/java" type="XMLMAPPER"/>

        <table tableName="t_pay" domainObjectName="Pay">
            <generatedKey column="id" sqlStatement="JDBC"/>
        </table>
    </context>
</generatorConfiguration>

```

双击运行Maven中的插件

![](F:\java\SpringCloud\img\c22ec5b28bbe81b2d44df53fd42d0b61.png)



### 2.3 编写业务逻辑

创建一个业务逻辑模块`cloud-provider-payment8001`

![](F:\java\SpringCloud\img\image-20241209190601493.png)

导入依赖

```xml
<dependencies>
    <!--SpringBoot通用依赖模块-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <!--SpringBoot集成druid连接池-->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
    </dependency>
    <!-- Swagger3 调用方式 http://你的主机IP地址:5555/swagger-ui/index.html -->
    <dependency>
        <groupId>org.springdoc</groupId>
        <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    </dependency>
    <!--mybatis和springboot整合-->
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
    </dependency>
    <!--Mysql数据库驱动8 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <!--persistence-->
    <dependency>
        <groupId>javax.persistence</groupId>
        <artifactId>persistence-api</artifactId>
    </dependency>
    <!--通用Mapper4-->
    <dependency>
        <groupId>tk.mybatis</groupId>
        <artifactId>mapper</artifactId>
    </dependency>
    <!--hutool-->
    <dependency>
        <groupId>cn.hutool</groupId>
        <artifactId>hutool-all</artifactId>
    </dependency>
    <!-- fastjson2 -->
    <dependency>
        <groupId>com.alibaba.fastjson2</groupId>
        <artifactId>fastjson2</artifactId>
    </dependency>
    <!--lombok-->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.28</version>
        <scope>provided</scope>
    </dependency>
    <!--test-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>3.2.0</version>
        </plugin>
    </plugins>
</build>

```

编写`yml`配置文件

```yml
server:
  port: 8001

# ==========applicationName + druid-mysql8 driver===================
spring:
  application:
    name: cloud-payment-service

  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/db2024?characterEncoding=utf8&useSSL=false&serverTimezone=GMT%2B8&rewriteBatchedStatements=true&allowPublicKeyRetrieval=true
    username: root
    password: 20170440

# ========================mybatis===================
mybatis:
  mapper-locations: classpath:mapper/*.xml
  type-aliases-package: com.atguigu.cloud.entities
  configuration:
    map-underscore-to-camel-case: true
```

创建启动类

```java
// 注意这边@MapperScan注解的包名，使用的是tk.mybatis.spring.annotation.MapperScan; 要和org.mybatis.spring.annotation.MapperScan;区分一下

@SpringBootApplication
@MapperScan("com.hxt.cloud.mapper")
public class Main8001 {
    public static void main(String[] args) {
        SpringApplication.run(Main8001.class, args);
    }
}
```

将模块`mybatis_generator2024`中的`enties`和`mapper`复制到本模块，并删掉原模块中的包

![image-20241209192940538](F:\java\SpringCloud\img\image-20241209192940538.png)

编写数据传输对象

```java
@AllArgsConstructor
@NoArgsConstructor
@Data
public class PayDTO implements Serializable {
    private Integer id;

    private String payNo;

    private String orderNo;

    private Integer userId;

    private BigDecimal amount;
}
```



**编写业务层接口及实现**

```java
public interface PayService {

    public int add(Pay pay);
    public int delete(Integer id);
    public int update(Pay pay);

    public Pay getById(Integer id);
    public List<Pay> getAll();
}

@Service
public class PayServiceImpl implements PayService {
    // 推荐
    @Resource
    private PayMapper payMapper;

    @Override
    public int add(Pay pay) {
        return payMapper.insertSelective(pay);
    }

    @Override
    public int delete(Integer id) {
        return payMapper.deleteByPrimaryKey(id);
    }

    @Override
    public int update(Pay pay) {
        return payMapper.updateByPrimaryKeySelective(pay);
    }

    @Override
    public Pay getById(Integer id) {
        return payMapper.selectByPrimaryKey(id);
    }

    @Override
    public List<Pay> getAll() {
        return payMapper.selectAll();
    }
}

```

### 2.4 整合Swagger3

| 注解       | 标注位置                    | 作用                   |
| ---------- | --------------------------- | ---------------------- |
| @Tag       | Controller类                | 标识Controller作用     |
| @Operation | 方法上                      | 描述方法作用           |
| @Schema    | model层的bean和bean的方法上 | 描述模型作用及每个属性 |

**添加依赖**

```xml
<!-- 父工程 -->
<dependency>
   <groupId>org.springdoc</groupId>
   <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
   <version>${swagger3.version}</version>
</dependency>
```



![image-20241210191751463](F:\java\SpringCloud\img\image-20241210191751463.png)

![image-20241210192110713](F:\java\SpringCloud\img\image-20241210192110713.png)

![image-20241210192147683](F:\java\SpringCloud\img\image-20241210192147683.png)



**使用配置文件**

```java
@Configuration
public class SwaggerConfiguration {
    @Bean
    public GroupedOpenApi PayApi()
    {
        //以/pay开头的请求都是支付模块
        return GroupedOpenApi.builder().group("支付微服务模块").pathsToMatch("/pay/**").build();
    }
    @Bean
    public GroupedOpenApi OtherApi()
    {
        //以/other开头的都是其他模块的请求
        return GroupedOpenApi.builder().group("其它微服务模块").pathsToMatch("/other/**", "/others").build();
    }
    
    @Bean
    public OpenAPI docsOpenApi()
    {
        return new OpenAPI()
                .info(new Info().title("cloud2024")
                        .description("通用设计rest")
                        .version("v1.0"))
                .externalDocs(new ExternalDocumentation()
                        .description("www.atguigu.com")
                        .url("https://yiyan.baidu.com/"));
    }
}

```

访问`localhost:8001/swagger-ui/index.html`

![image-20241210192953756](F:\java\SpringCloud\img\image-20241210192953756.png)



### 2.5 改进

#### 2.5.1 时间格式问题

**方式一**

在实体类的时间属性上加 `@JsonFormat`

```java
@JsonFormat(pattern = "yyyy-MM-dd HH-mm-ss" ,timezone = "GMT+8")
@Column(name = "create_time")
private Date createTime;
```

**方式二**

在配置文件中配置

```
spring:
  jackson:
    date-format: yyyy-MM-dd HH-mm-ss
    time-zone: GMT+8

```



#### 2.5.2 统一返回值

![image-20241210194206864](F:\java\SpringCloud\img\image-20241210194206864.png)

![image-20241210194223966](F:\java\SpringCloud\img\image-20241210194223966.png)

**定义一个枚举类**

```java
@Getter
public enum ReturnCodeEnum {
    //1.举值
    RC999("999", "操作XXX失败"),
    RC200("200", "success"),
    RC201("201", "服务开启降级保护,请稍后再试!"),
    RC202("202", "热点参数限流,请稍后再试!"),
    RC203("203", "系统规则不满足要求,请稍后再试!"),
    RC204("204", "授权规则不通过,请稍后再试!"),
    RC403("403", "无访问权限,请联系管理员授予权限"),
    RC401("401", "匿名用户访问无权限资源时的异常"),
    RC404("404", "404页面找不到的异常"),
    RC500("500", "系统异常，请稍后重试"),
    RC375("375", "数学运算异常，请稍后重试"),
    INVALID_TOKEN("2001", "访问令牌不合法"),
    ACCESS_DENIED("2003", "没有权限访问该资源"),
    CLIENT_AUTHENTICATION_FAILED("1001", "客户端认证失败"),
    USERNAME_OR_PASSWORD_ERROR("1002", "用户名或密码错误"),
    BUSINESS_ERROR("1004", "业务逻辑异常"),
    UNSUPPORTED_GRANT_TYPE("1003", "不支持的认证模式");
    
    // 2.构造
    private final String code; // 自定义状态码，对应前面枚举的第一个参数
    private final String message; // 自定义信息，对应前面枚举的第二个参数
    ReturnCodeEnum(String code, String message) {
        this.code = code;
        this.message = message;
    }
    
    // 3.遍历
    public static ReturnCodeEnum getReturnCodeEnum(String code) {
        for (ReturnCodeEnum e : ReturnCodeEnum.values()) {
            if(e.getCode().equalsIgnoreCase(code)){
                return e;
            }
        }
        return null;
    }
}
```

**定义统一返回类Result**

```java
@Data
@Accessors(chain = true)
public class ResultData<T> {
    private String code;
    private String message;
    private T data;
    private long timestamp;

    public ResultData() {
        this.timestamp = System.currentTimeMillis();
    }

    public static <T> ResultData<T> success(T data) {
        ResultData<T> resultData = new ResultData<>();
        resultData.setCode(ReturnCodeEnum.RC200.getCode());
        resultData.setMessage(ReturnCodeEnum.RC200.getMessage());
        resultData.setData(data);
        return resultData;
    }

    public static <T> ResultData<T> fail(String code, String message) {
        ResultData<T> resultData = new ResultData<>();
        resultData.setCode(code);
        resultData.setMessage(message);
        return resultData;
    }
}
```



#### 2.5.3 异常处理

```

```

