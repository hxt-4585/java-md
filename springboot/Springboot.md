# SpringBoot

## 一、SpringBoot3-核心特性

### （一）SpringBoot3-快速入门

#### 1、特性

<ul>
    <li>快速创建独立Spring应用</li>
    <li>直接嵌入Tomcat、Jetty或Undertow（无需部署 war 包）【Servlet 容器】。</li>
    <li><b>重点：</b>提供可选择的starter，简化应用整合</li>
    <li><b>重点：</b>按需自动配置Spring及第三方库</li>
    <li> 提供生产级特性：如监控指标、健康检查、外部化配置等。</li>
    <li>无代码生成、无 xml。</li>
 </ul>



#### 2、快速体验

创建maven项目

```xml
<!-- 所有 Spring Boot 项目都必须继承自 spring-boot-starter-parent -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.1.3</version>
</parent>

```




导入场景

```xml
<dependencies>
    <!-- Web 开发的场景启动器 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>

```



主程序

```java
package com.myxh.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @author MYXH
 * @date 2023/9/11
 * @description 启动 SpringBoot 项目的主入口程序
 */
// 这是一个 SpringBoot 应用
@SpringBootApplication
public class MainApplication
{
    public static void main(String[] args)
    {
       SpringApplication.run(MainApplication.class, args);
    }
}

```



业务

```java
package com.myxh.springboot.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author MYXH
 * @date 2023/9/11
 */
@RestController
public class HelloController
{
    @GetMapping("/hello")
    public String hello()
    {
        return "Hello, Spring Boot 3!";
    }
}

```



打包，在maven中添加

```mxl
<build>
    <plugins>
        <!-- SpringBoot 应用打包插件-->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>

```

在maven中运行 clean→package 后，项目 `target` 文件中会出现 `.jar` 文件，在该文件的当前目录下进入终端，输入 `java -jar XXX.jar` 即可启动项目。在上述 `.jar` 文件中添加 `.properties` 或 `.yml` 文件，修改配置并重新项目即可，无需重新打成 jar 包。



#### 3、应用分析

##### 3.1 依赖管理机制

为什么导入 **start-web** 所有相关依赖都导入进来？

​	开发什么场景，导入什么场景启动器；maven依赖传递原则。A→B→C：A就拥有B和C；导入场景启动器，会自动将所有核心依赖全部导入进来



为什么版本号不用写？

​	每一个boot项目都有一个父项目 `spring-boot-starter-parent` ，**parent** 的父项目是`spring-boot-dependencies`



自定义版本号

​	利用maven就近原则，直接在当前项目 `properties `标签中声明父项目用的版本属性的key

```xml
<properties>
    <mysql.version>8.0.31</mysql.version>
</properties>
```



##### 3.2 自动配置机制

**自动配置** `Tomcat`、`SpringMVC` 等

​	以前：DispacherServlet、ViewResolver等

​	现在：自动配置好这些组件

```java
//java10： 局部变量类型的自动推断
var ioc = SpringApplication.run(Springboot002Application.class, args);

//1、获取容器中所有组件的名字
String[] names = ioc.getBeanDefinitionNames();
//2、挨个遍历：
// dispatcherServlet、beanNameViewResolver、characterEncodingFilter、multipartResolver
// SpringBoot把以前配置的核心组件现在都给我们自动配置好了。
for (String name : names) {
    System.out.println(name);
}
```



**默认的包扫描规则**

​	`@SpringBootApplication` 标注的类就是主程序类

​	springboot只会扫描主程序所在的包以及其下面的子包（默认的扫描规则）

​	自定义扫描规则

​		Ⅰ`@SpringBootApplication(scanBasePackages = "com.atguigu")`

​		Ⅱ`@ComponentScan("com.atguigu")` 直接指定扫描的路径



**配置默认值**

​	配置文件 `application.properties` / `application.yml` ，是和某几个类对象（**配置属性类**）的值进行一一绑定。

```java
// 服务器相关
public class ServerProperties {
	private Integer port;
    private InetAddress address;
    @NestedConfigurationProperty
    private final ErrorProperties error = new ErrorProperties();
    private ForwardHeadersStrategy forwardHeadersStrategy;
    private String serverHeader;
    private DataSize maxHttpRequestHeaderSize = DataSize.ofKilobytes(8L);
    private Shutdown shutdown;
    @NestedConfigurationProperty
    private Ssl ssl;
    @NestedConfigurationProperty
    private final Compression compression;
    @NestedConfigurationProperty
    private final Http2 http2;
    private final Servlet servlet;
    private final Reactive reactive;
    private final Tomcat tomcat;
    private final Jetty jetty;
    private final Netty netty;
    private final Undertow undertow;
    
	// ...
}

// 文件相关
public class MultipartProperties {
    private boolean enabled = true;
    private String location;
    private DataSize maxFileSize = DataSize.ofMegabytes(1L);
    private DataSize maxRequestSize = DataSize.ofMegabytes(10L);
    private DataSize fileSizeThreshold = DataSize.ofBytes(0L);
    private boolean resolveLazily = false;
	// ...
}

// ...
```



**按需加载自动配置**

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-thymeleaf</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
    <!-- ... -->
</dependencies>

```

上述场景中，除了会导入相关功能依赖，还会导入 `spring-boot-starter` ，是所有 `starter`  的  `starter` 。

 `spring-boot-starter` 导入了一个包 `spring-boot-autoconfigure`。包里面是各种场景的 `AutoConfiguration` 自动配置类 

虽然全场景的自动配置都在 `spring-boot-autoconfigure`这个包，但是不是全都开启的。

​	导入哪个场景就会开启哪个自动配置



**深入理解自动配置原理**

1、导入 `start-web`：导入了web开发场景

- 1、场景启动器导入了相关场景的所有依赖：`starter-json`、`starter-tomcat`、`springmvc`
- 2、每个场景启动器都引入了一个`spring-boot-starter`，核心场景启动器。
- 3、**核心场景启动器**引入了`spring-boot-autoconfigure`包。
- 4、`spring-boot-autoConfiguration` 里面囊括了所有场景的所有配置。
- 5、只要这个包下的所有类都能生效，那么相当于SpringBoot官方写好的整合功能就生效了。
- 6、springBoot默认却扫描不到 `spring-boot-autoconfigure`下写好的所有**配置类**。（这些**配置类**给我们做了整合操作），**默认只扫描主程序所在的包**。



2、主程序：`@SpringBootApplication`

- 1、`@SpringBootApplication`由三个注解组成`@SpringBootConfiguration`、`@EnableAutoConfiguratio`、`@ComponentScan`

- 3、`@EnableAutoConfiguration`：SpringBoot **开启自动配置的核心**。

- - 1、是由 `@Import(AutoConfigurationImportSelector.class)` 提供功能：批量给容器中导入组件。
  - 2、springBoot启动会默认加载 142 （看版本）配置类
  - 3、这**142个配置类**来自于 `spring-boot-autoconfigure` 下  `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 文件指定的

- 4、按需生效：

- - 并不是这`142`个自动配置类都能生效
  - 每一个自动配置类，都有条件注解`@ConditionalOnxxx`，只有条件成立，才能生效 



3、`xxxxAutoConfiguration`**自动配置类**

- **1、给容器中使用@Bean 放一堆组件。**
- 2、每个**自动配置类**都可能有这个注解`@EnableConfigurationProperties(ServerProperties.class)`，用来把配置文件中配的指定前缀的属性值封装到 `xxxProperties`**属性类**中
- 3、以Tomcat为例：把服务器的所有配置都是以`server`开头的。配置都封装到了属性类中。
- 4、给**容器**中放的所有**组件**的一些**核心参数**，都来自于`xxxProperties`**。**`**xxxProperties**`**都是和配置文件绑定。**
- **只需要改配置文件的值，核心组件的底层参数都能修改**

```java
@ConfigurationProperties(
    prefix = "server",
    ignoreUnknownFields = true
)
public class ServerProperties {
    private Integer port;
    private InetAddress address;
    @NestedConfigurationProperty
    private final ErrorProperties error = new ErrorProperties();
    private ForwardHeadersStrategy forwardHeadersStrategy;
    private String serverHeader;
    //...
}
```



#### 4、核心技能

##### 4.1 常用注解

###### 4.1.1 组件注册

**@Configuration、@SpringBootConfiguration（springboot自己的配置类，和Configuration没啥区别）**

**@Bean、@Scope**

**@Controller、 @Service、@Repository、@Component**

**@Import**

**@ComponentScan**



@import

```java
// 第三方库，无法在.class文件上添加注解，所以可以采用import
@SpringBootApplication
@Import({FastsqlException.class, User.class})
public class SpringBootConfiguration {
}

/* 
    打印结果
    com.alibaba.druid.FastsqlException
    com.hxt.pojo.User
*/
```



@Scope

```java
// singleton：默认作用域，每个Spring IoC容器中，一个bean定义对应一个对象实例。
// prototype：每次请求都会创建一个新的bean实例，每个实例都有自己的生命周期。
// session：在一个HTTP Session中，一个bean定义对应一个实例。
// application：在一个Web应用程序中，一个bean定义对应一个实例，所有HTTP请求共享同一个实例。
// websocket：在一个WebSocket生命周期内，一个bean定义对应一个实例。
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

@Component
@Scope("prototype") // 定义bean为prototype作用域
public class MyBean {
    private String name;

    public MyBean(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```



###### 4.1.2 条件注解

如果注解指定的条件成立，则触发指定行为

`@ConditionalOnXXX` 

`@ConditionalOnClass` ：如果类路径中存在这个**类**，则触发指定行为

`@ConditionalOnMissingClass` ：如果类路径中不存在这个**类**，则触发指定行为

`@ConditionalOnBean `：如果容器中存在这个**Bean（组件）**，则触发指定行为

`@ConditionalOnMissingBean` ：如果容器中不存在这个**Bean（组件）**，则触发指定行为





###### 4.1.3 属性绑定

`@ConfigurationProperties` ： 声明组件的属性和配置文件哪些前缀开始项进行绑定

```properties
student.age=20
student.name=张三
```

```java
import lombok.Getter;
import lombok.Setter;
import lombok.ToString;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Getter
@Setter
@ToString
@Component
// @Target({ElementType.TYPE, ElementType.METHOD})既可以在类上也可以在方法上
@ConfigurationProperties(prefix = "student")
public class Student {
    private String name;
    private Integer age;
}


@ConfigurationProperties(prefix = "student")
@Bean
public Student student(){
    return new Student();
}
```

注意properties文件的编码

![image-20241013164400291](.\img\image-20241013164400291.png)



`@EnableConfigurationProperties`（@Bean + @ConfigurationProperties） ：快速注册注解

​	SpringBoot默认只扫描自己主程序所在的包。如果导入第三方包，即使组件上标注了 @Component、@ConfigurationProperties 注解，也没用。因为组件都扫描不进来，此时使用这个注解就可以快速进行属性绑定并把组件注册进容器。





##### 4.2 YAML配置文件

###### 	4.2.1 基本语法

- **大小写敏感**
- 使用**缩进表示层级关系，k: v，使用空格分割k,v**
- 缩进时不允许使用Tab键，只允许**使用空格**。换行
- 缩进的空格数目不重要，只要**相同层级**的元素**左侧对齐**即可
- **# 表示注释**，从这个字符一直到行尾，都会被解析器忽略。



支持的写法：

- **对象**：**键值对**的集合，如：映射（map）/ 哈希（hash） / 字典（dictionary）
- **数组**：一组按次序排列的值，如：序列（sequence） / 列表（list）
- **纯量**：单个的、不可再分的值，如：字符串、数字、bool、日期



###### 4.2.2 示例

```java
@Component
@ConfigurationProperties(prefix = "person") //和配置文件person前缀的所有配置进行绑定
@Data //自动生成JavaBean属性的getter/setter
//@NoArgsConstructor //自动生成无参构造器
//@AllArgsConstructor //自动生成全参构造器
public class Person {
    private String name;
    private Integer age;
    private Date birthDay;
    private Boolean like;
    private Child child; //嵌套对象
    private List<Dog> dogs; //数组（里面是对象）
    private Map<String,Cat> cats; //表示Map
}

@Data
public class Dog {
    private String name;
    private Integer age;
}

@Data
public class Child {
    private String name;
    private Integer age;
    private Date birthDay;
    private List<String> text; //数组
}

@Data
public class Cat {
    private String name;
    private Integer age;
}
```



**properties**

```properties
person.name=张三
person.age=18
person.birthDay=2010/10/12 12:12:12
person.like=true
person.child.name=李四
person.child.age=12
person.child.birthDay=2018/10/12
person.child.text[0]=abc
person.child.text[1]=def
person.dogs[0].name=小黑
person.dogs[0].age=3
person.dogs[1].name=小白
person.dogs[1].age=2
person.cats.c1.name=小蓝
person.cats.c1.age=3
person.cats.c2.name=小灰
person.cats.c2.age=2
```

**yaml**

```yaml
person:
  name: 张三
  age: 18
  birthDay: 2010/10/10 12:12:12
  like: true
  child:
    name: 李四
    age: 20
    birthDay: 2018/10/10
    # text: ["abc","def"]
    text:
      - abc
      - def
  dogs:
    - name: 小黑
      age: 3
    - name: 小白
      age: 2
  cats:
    c1:
      name: 小蓝
      age: 3
    c2: {name: 小绿,age: 2} #对象也可用{}表示
```



###### 4.2.3 细节

- birthDay（驼峰命名）推荐写为 birth-day

- **文本**：

- - **单引号**不会转义【\n 则为普通字符串显示】
  - **双引号**会转义【\n会显示为**换行符**】

![image-20241016225355654](.\img\image-20241016225355654.png)

![image-20241016225326117](.\img\image-20241016225326117.png)

- **大文本**

- - `|`开头，大文本写在下层，**保留文本格式**，**换行符正确显示**

- ![image-20241016225624333](.\img\image-20241016225624333.png)

- ![image-20241016225648993](.\img\image-20241016225648993.png)

- - `>`开头，大文本写在下层，折叠换行符，将换行符换成空格

![image-20241016230102430](.\img\image-20241016230102430.png)

![image-20241016230122411](.\img\image-20241016230122411.png)



- **多文档合并**

- - 使用`---`可以把多个yaml文档合并在一个文档中，每个文档区依然认为内容独立

```yaml
---
student:
  age: 10
  name: zhangsan

server:
  port: 9002

---
person:
  name: '张三 \n 单引号'
  age: 30
  birthDay: 2001/1/1 12:12:12
  like: true
  child:
    name: "李四 \n 双引号"
    age: 20
    birthday: 2018/10/10 12:12:12
    text:
      - abc
      - def
      - |
        dogs:
          - name: 小黑
            age: 3
          - name: 小白
            age: 2
      - >
        ahsdjak
        ask的la

  dogs:
    - name: 小黑
      age: 3
    - name: 小白
      age: 2
  cats:
    c1:
      name: 小蓝
      age: 3
    c2: {name: 小绿,age: 2}

```

![image-20241016230430993](.\img\image-20241016230430993.png)

##### 4.3 日志配置

![image-20241017200625058](.\img\image-20241017200625058.png)

门面类似于 `JDBC`，提供一个接口。



###### 4.3.1 简介

Spring 使用 `commons-logging` 作为内部日志，但底层日志实现是开放的。可对接其他日志框架。支持 `jul`，`log4j2`，`logback（默认）`

SpringBoot 是如何将日志配置好的

1、每个`starter`场景，都会导入一个核心场景`spring-boot-starter`

2、核心场景引入了日志的所用功能 `spring-boot-starter-logging`

3、默认使用 `logback + slf4j` 组合作为默认底层日志

4、日志是系统一启动就要用的，`xxxAutoConfiguration`是系统启动好了以后放好的组件

5、日志是利用**监听器机制**配置好的。`ApplicationListener`。

6、日志所有的配置都可以通过修改配置文件实现。以`logging`开始的所有配置。



###### 4.3.2 日志格式

```
2024-10-17T20:11:46.030+08:00  INFO 1364 --- [           main] com.hxt.Springboot002Application         : Starting Springboot002Application using Java 17.0.12 with PID 1364 (E:\AppData\IdeaProjects\springboot\springboot-002\target\classes started by hxt in E:\AppData\IdeaProjects\springboot)

2024-10-17T20:11:46.032+08:00  INFO 1364 --- [           main] com.hxt.Springboot002Application         : No active profile set, falling back to 1 default profile: "default"

```

默认输出格式：

- 时间和日期：毫秒级精度
- 日志级别：`ERROR`、`WARN`、`INFO`、`DEBUG`、`TRACE`

- 进程ID
- ---： 消息分割符

- 线程名：使用 [] 包含
- Logger 名：通常是产生日志的**类名**

- 消息： 日志记录的内容



默认输出格式值：`%clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd'T'HH:mm:ss.SSSXXX}}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}`

可在 `yml` 文件中修改

```yml
logging:
  pattern:
    console: '%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%thread] %logger{15} ===> %msg%n'
```



一个类可以有一个日志记录器

```java
@RestController
public class HelloController {
    Logger logger = LoggerFactory.getLogger(getClass());
    @GetMapping("/hello")
    public String hello(){
        logger.info("hahaha");
        return "hello";
    }
}

// 或者用注解
@Slf4j
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello(){
        log.info("hahaha");
        return "hello";
    }
}
```



###### 4.3.3 日志级别

- 由低到高：`ALL,TRACE, DEBUG, INFO, WARN, ERROR,FATAL,OFF`；

- - **只会打印指定级别及以上级别的日志**
  - ALL：打印所有日志
  - TRACE：追踪框架详细流程日志，一般不使用
  - DEBUG：开发调试细节日志
  - INFO：关键、感兴趣信息日志
  - WARN：警告但不是错误的信息日志，比如：版本过时
  - ERROR：业务错误日志，比如出现各种异常
  - FATAL：致命错误日志，比如jvm系统崩溃
  - OFF：关闭所有日志记录

- 不指定级别的所有类，都使用root指定的级别作为默认级别（INFO）

- 







### （二）SpringBoot3-Web开发

#### 1、Web场景

##### 1.1 自动配置

整合web场景

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

引入 `autoconfigure` 功能

`@EnableAutoConfiguration`注解使用`@Import(AutoConfigurationImportSelector.class)`批量导入组件

加载 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 文件中配置的所有组件

所有与web相关自动配置类如下

```
org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration
org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration
====以下是响应式web场景和现在的没关系======
org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration
org.springframework.boot.autoconfigure.web.reactive.ReactiveMultipartAutoConfiguration
org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactoryAutoConfiguration
org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration
org.springframework.boot.autoconfigure.web.reactive.WebSessionIdResolverAutoConfiguration
org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoConfiguration
org.springframework.boot.autoconfigure.web.reactive.function.client.ClientHttpConnectorAutoConfiguration
org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration
================以上没关系=================
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration
org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration
org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration
org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration
org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration
```



绑定了配置文件的一堆配置项

- 1、SpringMVC的所有配置 `spring.mvc`
- 2、Web场景通用配置 `spring.web`
- 3、文件上传配置 `spring.servlet.multipart`
- 4、服务器的配置 `server`: 比如：编码方式



##### 1.2 默认效果

默认配置：

1、包含了 `ContentNegotiatingViewResolver `和` BeanNameViewResolver `组件， **方便视图解析**

2、**默认的静态资源处理机制**：静态资源放在 `static` 文件夹

3、**自动注册了**   `Converter,GenericConverter,Formatter`组件。适配常见的 **数据类型转换**和**格式化需求**

4、支持 **HttpMessageConverters**，可以**方便返回**json等**数据类型**

5、**注册** MessageCodesResolver，方便**国际化**及错误消息处理

6、**支持 静态** index.html

7、**自动使用**ConfigurableWebBindingInitializer，实现消息处理、数据绑定、类型转化、数据校验等功能



#### 2、静态资源

##### 2.1 WebMvcAutoConfiguration原理

生效条件

```java
@AutoConfiguration(
    after = {DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class, ValidationAutoConfiguration.class}
)
@ConditionalOnWebApplication(
    type = Type.SERVLET
)
@ConditionalOnClass({Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class})
@ConditionalOnMissingBean({WebMvcConfigurationSupport.class})
@AutoConfigureOrder(-2147483638)
@ImportRuntimeHints({WebResourcesRuntimeHints.class})
```



效果

1、放了两个`Filter`：

​	a、`HiddenHttpMethodFilter`：页面表单提交rest请求（GET，POST，PUT，DELETE）

​	b、`FormContentFilter`：表单内容Filter，GET、POST请求可以携带数据，PUT、DELETE的请求体数据会被忽略

2、给容器中放了`WebMvcConfigurer`组件，给SpringMvc添加各种定制功能。



##### 2.2  默认规则

###### 2.2.1 静态资源映射

静态资源的映射规则在`WebMvcAutoConfiguration`中进行了定义

1、`/webjars/**`的所有路径  资源都在 `classpath:/META-INF/resources/webjars/`

![image-20241101205053967](.\img\image-20241101205053967.png)

<img src=".\img\image-20241101205125366.png" alt="image-20241101205125366" style="zoom: 50%;" />

2、`/**` 的所有路径 资源都在 `classpath:/META-INF/resources/、classpath:/resources/、classpath:/static/、classpath:/public/`

![image-20241101204902797](.\img\image-20241101204902797.png)

![image-20241101204943918](.\img\image-20241101204943918.png)



###### 2.2.2 静态资源缓存

所有静态资源都定义了缓存规则。【浏览器访问过一次，就会缓存一段时间】，但此功能参数无默认值

1. period： 缓存间隔。 默认 0S；

2. cacheControl：缓存控制。 默认无；

3. useLastModified：是否使用lastModified头。 默认 false；



###### 2.2.3 欢迎页

欢迎页规则在 `WebMvcAutoConfiguration` 中进行了定义：

​	1、在 **静态资源** 目录下找 `index.html`

​	2、没有则在 `templates` 下找 `index` 模板页



###### 2.2.4 Favicon

![image-20241104183321667](.\img\image-20241104183321667.png)

​	标签页上的图标

​	在静态资源目录下找 `favicon.ico` ，只需把图标名字改掉就行



###### 2.2.5 缓存机制测试

自定义静态资源路径、自定义缓存规则



**配置文件方式**

`spring.mvc`： 静态资源访问前缀路径

`spring.web`：

- 静态资源目录
- 静态资源缓存策略

```properties
#自定义静态资源文件夹位置
spring.web.resources.static-locations=classpath:/a/,classpath:/b/,classpath:/static/

#2、 spring.mvc
## 2.1. 自定义webjars路径前缀
spring.mvc.webjars-path-pattern=/wj/**
## 2.2. 静态资源访问路径前缀
## http://localhost:8080/static/5.jpg
spring.mvc.static-path-pattern=/static/**
```

PS：为什么 `META-INF/resources/` 目录下的静态资源仍就能访问到  

![image-20241104192304268](F:\java\springboot\img\image-20241104192304268.png)



**代码方式**

```java
// 第一种配置方式
// @EnableWebMvc 禁用boot默认配置
@Configuration
public class MyConfig implements WebMvcConfigurer {
    @Override
    // http://localhost:8080/static/5.jpg 访问a目录下的内容，但boot默认配置的静态文件目录无需加/static
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        WebMvcConfigurer.super.addResourceHandlers(registry);

        registry.addResourceHandler("/static/**")
                .addResourceLocations("classpath:/a/","classpath:/b/")
                .setCacheControl(CacheControl.maxAge(1180, TimeUnit.SECONDS));    }
}

// 第二种配置方式
@Configuration //这是一个配置类,给容器中放一个 WebMvcConfigurer 组件，就能自定义底层
public class MyConfig  /*implements WebMvcConfigurer*/ {
    @Bean
    public WebMvcConfigurer webMvcConfigurer(){
        return new WebMvcConfigurer() {
            @Override
            public void addResourceHandlers(ResourceHandlerRegistry registry) {
                registry.addResourceHandler("/static/**")
                        .addResourceLocations("classpath:/a/", "classpath:/b/")
                        .setCacheControl(CacheControl.maxAge(1180, TimeUnit.SECONDS));
            }
        };
    }

}
```

**为什么容器中放一个 webMvcConfigurer 就能生效？ ** 

![image-20241104194137501](F:\java\springboot\img\image-20241104194137501.png)

```java
// 类DelegatingWebMvcConfiguration
@Autowired(
	required = false
)
public void setConfigurers(List<WebMvcConfigurer> configurers) {
    if (!CollectionUtils.isEmpty(configurers)) {
    	this.configurers.addWebMvcConfigurers(configurers);
    }

}
```



#### 3、路径匹配

**Spring5.3** 之后加入了更多的请求路径匹配的实现策略；

以前只支持 AntPathMatcher 策略, 现在提供了 **PathPatternParser** （默认使用）策略。并且可以让我们指定到底使用那种策略。



##### 3.1 Ant风格路径用法

Ant 风格的规则：

- *：表示**任意数量**的字符。
- ?：表示任意**一个字符**。
- **：表示**任意数量的目录**。
- {}：表示一个命名的模式**占位符**。
- []：表示**字符集合**，例如[a-z]表示小写字母。

例如：

- *.html 匹配任意名称，扩展名为.html的文件。
- /folder1/*/*.java 匹配在folder1目录下的任意两级目录下的.java文件。
- /folder2/**/*.jsp 匹配在folder2目录下任意目录深度的.jsp文件。
- /{type}/{id}.html 匹配任意文件名为{id}.html，在任意命名的{type}目录下的文件。



##### 3.2 模式切换

如果路径中有 **，`如 /a*/**/b?/{p1:[a-f]+}`，PathPatternParser不支持，需要切换成AntPathMatcher

```properties
spring.mvc.pathmatch.matching-strategy=ant_path_matcher
```



#### 4、内容协商

一套系统适配多端数据返回

![image-20241104195947786](.\img\image-20241104195947786.png)



##### 4.1 多端内容适配

###### 4.1.1 默认规则

SpringBoot 多端内容分配

​	1、基于 **请求头** 内容协商（默认）

​			a、客户端向服务端发送请求，携带HTTP标准的 **Accept请求头**

​			b、服务器根据客户端期望的数据类型进行动态返回

​	2、基于 **请求参数** 的内容协商（需要手动开启）

​			a、发生请求 `GET   /XXX/XXX?fomat=json`

​			b、匹配到 `@GetMapping("/XXX/XXX")`

​			c、根据参数协商，优先返回 `json` 类型数据 【需要开启参数匹配设置】

​			d、发生请求 `GET /XXX/XXX?fomat=xml` 优先返回 `xml` 数据



###### 4.1.2 效果演示

导入xml相关依赖

```xml
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

标注注解

```java
@JacksonXmlRootElement
@Data
@AllArgsConstructor
@NoArgsConstructor
@ToString
public class Person {
    private int id;
    private String name;
    private int age;
}

```

似乎不需要加注解@JacksonXmlRootElement，只需引入依赖即可

![image-20241105191335913](.\img\image-20241105191335913.png)

开启基于请求参数的内容协商

```properties
# 开启基于请求参数的内容协商功能。 默认参数名：format。 默认此功能不开启
spring.mvc.contentnegotiation.favor-parameter=true
# 指定内容协商时使用的参数名。默认是 format，原本是format=xml现在是type=xml
spring.mvc.contentnegotiation.parameter-name=type
```

![image-20241105191739034](.\img\123123132.png)



###### 4.1.3 协商配置规则与支持类型

```properties
#使用参数进行内容协商
spring.mvc.contentnegotiation.favor-parameter=true  
#自定义参数名，默认为format
spring.mvc.contentnegotiation.parameter-name=myparam 
```



##### 4.2 自定义内容返回







##### 4.3 内容协商原理

`HttpMessageConverter` 怎么工作？何时工作？

定制 `HttpMessageConverter` 来实现多端内容协商

编写 `WebMvcConfigurer` 提供的 `configureMessageConverters`

###### 4.3.1 @ResponseBody由HTTPMessageConverter处理

标注了`@ResponseBody`的返回值 将会由支持它的 `HttpMessageConverter`写给浏览器

1、如果controller方法标注了`@ResponseBody` 注解

​	a、请求进来先来到`DispatcherServlet`的`doDispatch()`进行处理

​	b、找到一个 `HandlerAdapter `适配器。利用适配器执行目标方法

​	c、





###### 4.3.2 WebMvcAutoConfiguration`提供几种默认`HttpMessageConverters



#### 5、模板引擎



#### 6、国际化

国际化的自动配置参照 `MessageSourceAutoConfiguration`



#### 7、错误处理

##### 7.1 默认机制

错误处理的自动配置在 `ErrorMvcAutoConfiguration` ，两大核心机制

​	1、SpringBoot会 **自适应** 处理错误，**响应页面** 或 **JSON数据**

​	2、SpringMVC的错误处理机制依然保留，MVC处理不了，才会交给boot进行处理

![image-20241107224119449](.\img\image-20241107224119449.png)



```java
@GetMapping("/hello")
public Person hello() {
    Person person = new Person();
    person.setId(1);
    person.setName("张三");
    person.setAge(18);

    int i = 10/0;
    return person;
}

@ExceptionHandler(Exception.class)
public String exception(Exception e) {
    return e.getMessage();
}
```

![image-20241107230547291](.\img\image-20241104183312367.png)

![image-20241107230439650](.\img\image-20241104183321a.png)

```java
@Controller
// ${server.error.path:${error.path:/error}}先从前面的路径server.error.path寻找，如果没有配置，则采用error.path:/error
@RequestMapping({"${server.error.path:${error.path:/error}}"})
public class BasicErrorController extends AbstractErrorController {
    @RequestMapping(
        produces = {"text/html"}
    )//返回HTML
    public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
        HttpStatus status = this.getStatus(request);
        Map<String, Object> model = Collections.unmodifiableMap(this.getErrorAttributes(request, this.getErrorAttributeOptions(request, MediaType.TEXT_HTML)));
        response.setStatus(status.value());
        ModelAndView modelAndView = this.resolveErrorView(request, response, status, model);
        return modelAndView != null ? modelAndView : new ModelAndView("error", model);
    }

    @RequestMapping//返回 ResponseEntity, JSON
    public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
        HttpStatus status = this.getStatus(request);
        if (status == HttpStatus.NO_CONTENT) {
            return new ResponseEntity(status);
        } else {
            Map<String, Object> body = this.getErrorAttributes(request, this.getErrorAttributeOptions(request, MediaType.ALL));
            return new ResponseEntity(body, status);
        }
    }
//...
}
```



错误页面是如何解析到的

```java
ModelAndView modelAndView = this.resolveErrorView(request, response, status, model);
// 如果解析不到错误页面的地址，则使用error
return modelAndView != null ? modelAndView : new ModelAndView("error", model);
```





##### 7.2 自定义错误响应



##### 7.3 最佳实战





#### 8、嵌入式容器





### （三）SpringBoot3-数据访问

#### 1、整合SSM项目

SpringBoot 整合 `Spring`、`SpringMVC`、`MyBatis` 进行**数据访问场景**开发

```xml
<!-- 引入mybatis依赖和sql驱动 -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter-test</artifactId>
    <version>3.0.3</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>
```

```
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.type=com.zaxxer.hikari.HikariDataSource
spring.datasource.url=jdbc:mysql://localhost:3306/springboot
spring.datasource.username=root
spring.datasource.password=20170440

# 告诉springboot mybatis的配置文件位置
mybatis.mapper-locations=classpath:/mapper/*.xml
# 开启驼峰命名
mybatis.configuration.map-underscore-to-camel-case=true
```

```java
public interface UserMapper {
    User selectUserById(@Param("id") int id);
}

@RestController
public class UserController {

    @Autowired
    private UserMapper userMapper;

    @GetMapping("/user/{id}")
    public User getUser(@PathVariable("id") Integer id){
        User user = userMapper.selectUserById(id);
        return user;
    }
}

// @MapperScan指定接口目录，因为是接口，无法在接口上添加@Component注解
@MapperScan(basePackages = "com.hxt.mapper")
@SpringBootApplication
public class Springboot005Application {

    public static void main(String[] args) {
        SpringApplication.run(Springboot005Application.class, args);
    }

}
```



#### 2、自动配置原理



#### 3、快速定位生效的配置



#### 4、整合其他数据源



### （四）SpringBoot3-基础特性

#### 1、SpringApplication

##### 1.1 自定义banner

下图为springboot默认的banner，版本为3.3.5

![image-20241112185857350](F:\java\springboot\img\image-20241112185857350.png)

1. 类路径添加banner.txt或设置spring.banner.location就可以定制 banner
2. 推荐网站：[Spring Boot banner 在线生成工具，制作下载英文 banner.txt，修改替换 banner.txt 文字实现自定义，个性化启动 banner-bootschool.net](https://www.bootschool.net/ascii)



```properties
spring.banner.location=classpath:banner.txt
# 关闭banner
spring.main.banner-mode=off
```





##### 1.2 自定义SpringApplication

在SpringApplication类中

```java
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
    return (new SpringApplication(primarySources)).run(args);
}
```

所以有以下三种写法

```java
// 三种写法等价
@SpringBootApplication
public class Springboot006Application {

    public static void main(String[] args) {
        // 1. SpringApplication: Boot应用的核心API
//        SpringApplication.run(Springboot006Application.class, args);

//        // 自定义SpringApplication的底层设置
//        SpringApplication springApplication = new SpringApplication(Springboot006Application.class);
//
//        // 调整SpringApplication的参数
//        // springApplication.setDefaultProperties();
//
//        //这个不优先，配置文件优先，由于配置文件是off，所以不会打印banner
//        springApplication.setBannerMode(Banner.Mode.CONSOLE);
//
//        // 配置文件优先于程序
//
//        springApplication.run(args);

        new SpringApplicationBuilder()
                .main(Springboot006Application.class)
                .sources(Springboot006Application.class)
                .bannerMode(Banner.Mode.CONSOLE)
                .run(args);
    }

}

```



#### 2、Profiles

环境隔离能力；快速切换开发、测试、生产环境

步骤：

1. **标识环境**：指定哪些组件、配置在哪个环境生效
2. **切换环境**：这个环境对应的所有组件和配置就应该生效



可以标注于任意环境



##### 2.1 使用

```
1、区分出几个不同的环境：dev（开发环境）、test（测试环境）、prod（生产环境）。环境名字可以换
2、指定每个组件在哪个环境下生效：default环境：默认环境
3、默认只有激活指定的环境，这些组件才会生效
4、如果激活某个环境，如dev，那么Tearcher类上如果标注了@Profile({"default"})，则不会被放到容器中，如果没有注解，则会出现在容器中
5、命令行激活--spring.profiles.active=???
```



```
@Profile({"dev"})
@Component
@Data
public class Cat {
    private String name;
    private Integer age;
}


@Profile({"test"})
@Component
@Data
public class Dog {
    private String name;
    private Integer age;
}


@Profile({"prod"})
@Component
@Data
@ToString
public class Student {
    private String name;
    private Integer age;
}

// @Profile({"default"})标记不标记都可以
@Profile({"default"})
@Component
@Data
public class Teacher {
    private String name;
    private Integer age;
}
```

```properties
# 可以激活多个环境spring.profiles.active=dev,test
spring.profiles.active=dev
# 将默认环境修改为default
spring.profiles.default=test
# 包含指定环境，不管你激活哪个环境，这个都要有
spring.profiles.include=dev,test
```

最佳实战：

- **生效的环境** = **激活的环境/默认环境**  + **包含的环境**
- 项目里面这么用

- - 基础的配置`mybatis`、`log`、`xxx`：写到**包含环境中**
  - 需要动态切换变化的 `db`、`redis`：写到**激活的环境中**



##### 2.2 Profile分组

```properties
spring.profiles.active=haha
spring.profiles.group.haha=prod,test
spring.profiles.group.hehe=dev
```

```properties
# 另一种写法
spring.profiles.group.prod[0]=db
spring.profiles.group.prod[1]=mq
```





##### 2.3 Profile配置文件

- `application-{profile}.properties`可以作为**指定环境的配置文件**。
- 激活这个环境，**配置**就会生效。最终生效的所有**配置**是

- - `application.properties`：主配置文件，任意时候都生效
  - `application-{profile}.properties`：指定环境配置文件，**激活指定环境生效**

**profile优先级 > application** 



![image-20241112223935965](F:\java\springboot\img\image-20241112223935965.png)

![image-20241112224022914](F:\java\springboot\img\image-20241112224022914.png)

项目的端口为9000



#### 3、外部化配置

**场景**：线上应用如何**快速修改配置**，并应**用最新配置**？

- SpringBoot 使用  **配置优先级** + **外部配置**  简化配置更新、简化运维。
- 只需要给`jar`应用所在的文件夹放一个`application.properties`最新配置文件，重启项目就能自动应用最新配置



##### 3.1 配置优先级

SpringBoot 允许配置外部化，以便可以在不同的环境中使用相同的应用程序代码

我们可以使用各种 **外部配置源** ，包括 `Java Properties文件`、`YAML文件`、`环境变量`、`命令行参数`

@Value可以获取值，也可以用@ConfigurationProperties将所有属性绑定到java object中

**以下是 SpringBoot 属性源加载顺序。**后面的会覆盖前面的值。由低到高，高优先级配置覆盖低优先级

1. **默认属性**（通过`SpringApplication.setDefaultProperties`指定的）

   ```java
   ConfigurableApplicationContext context = new SpringApplicationBuilder()
       .main(Springboot006Application.class)
       .sources(Springboot006Application.class)
       .bannerMode(Banner.Mode.CONSOLE)
       // properties修改
       .properties("server.port=8088", "spring.profiles.active=dev")
       .run(args);
   ```

   

2. @PropertySource指定加载的配置（需要写在@Configuration类上才可生效）

3. **配置文件**（application.properties/yml等）

4. RandomValuePropertySource支持的random.*配置（如：@Value("${random.int}")）

5. OS 环境变量

6. Java 系统属性（System.getProperties()）

7. JNDI 属性（来自java:comp/env）

8. ServletContext 初始化参数

9. ServletConfig 初始化参数

10. SPRING_APPLICATION_JSON属性（内置在环境变量或系统属性中的 JSON）

11. **命令行参数**

12. 测试属性。(@SpringBootTest进行测试时指定的属性)

13. 测试类@TestPropertySource注解

14. Devtools 设置的全局属性。($HOME/.config/spring-boot)

结论：配置可以写到很多位置，常见的优先级顺序：

- `命令行`> `配置文件`> `springapplication配置`



![image-20241113192017749](F:\java\springboot\img\image-20241113192017749.png)

**配置文件优先级**如下：(**后面覆盖前面**)

1. **jar 包内**的application.properties/yml
2. **jar 包内**的application-{profile}.properties/yml
3. **jar 包外**的application.properties/yml
4. **jar 包外**的application-{profile}.properties/yml

**建议**：**用一种格式的配置文件**。`如果.properties和.yml同时存在,则.properties优先`

**结论：**包外 > 包内；同级情况：profile配置 > application配置

![image-20241113191915010](F:\java\springboot\img\image-20241113191915010.png)



![image-20241113191839726](F:\java\springboot\img\image-20241113191839726.png)





##### 3.2 外部配置

SpringBoot 应用启动时会自动寻找application.properties和application.yaml位置，进行加载。顺序如下：（**后面覆盖前面**）

​	1.类路径: 内部

1. 1. 类根路径
   2. 类下/config包

​	2.当前路径（项目所在的位置）

1. 1. 当前路径
   2. 当前下/config子目录
   3. /config目录的直接子目录（优先级最高）

2. 

3. config包内 `application-dev.properties` 端口被设为 9098

![image-20241113192223651](F:\java\springboot\img\image-20241113192223651.png)

![image-20241113192427517](F:\java\springboot\img\image-20241113192427517.png)





最终效果：优先级由高到低，前面覆盖后面

- 命令行 > 包外config直接子目录 > 包外config目录 > 包外根目录 > 包内目录
- 同级比较： 

- - profile配置 > 默认配置
  - properties配置 > yaml配置

![image-20241113192527599](F:\java\springboot\img\image-20241113192527599.png)

规律：最外层的最优先。

- 命令行 > 所有
- 包外 > 包内
- config目录 > 根目录
- profile > application 

配置不同就都生效（互补），配置相同高优先级覆盖低优先级



##### 3.3 导入配置

使用**spring.config.import**可以导入额外配置

```properties
spring.config.import=my.properties
my.property=value
```

无论以上写法的先后顺序，**my.properties** 的值总是 **优先于 **直接在文件中编写的 **my.property**。



##### 3.4 属性占位符

配置文件中可以使用 ${name:default}形式取出之前配置过的值

```properties
app.name=MyApp
# 如果username不存在，那么用:后面的值填充
app.description=${app.name} is a Spring Boot application written by ${username:Unknown}
```

@Value

![image-20241113193158073](F:\java\springboot\img\image-20241113193158073.png)



#### 4、单元测试

##### 4.1 整合

`spring-boot-test `提供核心测试能力，`spring-boot-test-autoconfigure` 提供测试的一些自动配置。

我们只需要导入`spring-boot-starter-test `即可整合测试

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

spring-boot-starter-test 默认提供了以下库供我们测试使用

- [JUnit 5](https://junit.org/junit5/)
- [Spring Test](https://docs.spring.io/spring-framework/docs/6.0.4/reference/html/testing.html#integration-testing)
- [AssertJ](https://assertj.github.io/doc/)
- [Hamcrest](https://github.com/hamcrest/JavaHamcrest)
- [Mockito](https://site.mockito.org/)
- [JSONassert](https://github.com/skyscreamer/JSONassert)
- [JsonPath](https://github.com/jayway/JsonPath)



##### 4.2 测试

###### 4.2.1 注解

JUnit5的注解与JUnit4的注解有所变化

https://junit.org/junit5/docs/current/user-guide/#writing-tests-annotations

- **@Test :**表示方法是测试方法。但是与JUnit4的@Test不同，他的职责非常单一不能声明任何属性，拓展的测试将会由Jupiter提供额外测试
- **@ParameterizedTest :**表示方法是参数化测试，下方会有详细介绍
- **@RepeatedTest :**表示方法可重复执行，下方会有详细介绍
- **@DisplayName :**为测试类或者测试方法设置展示名称![image-20241113194843932](F:\java\springboot\img\image-20241113194843932.png)
- **@BeforeEach :**表示在每个单元测试之前执行
- **@AfterEach :**表示在每个单元测试之后执行
- **@BeforeAll :**表示在所有单元测试之前执行
- **@AfterAll :**表示在所有单元测试之后执行
- **@Tag :**表示单元测试类别，类似于JUnit4中的@Categories
- **@Disabled :**表示测试类或测试方法不执行，类似于JUnit4中的@Ignore
- **@Timeout :**表示测试方法运行如果超过了指定时间将会返回错误
- **@ExtendWith :**为测试类或测试方法提供扩展类引用



###### 4.2.2 断言

| 方法              | 说明                                 |
| ----------------- | ------------------------------------ |
| assertEquals      | 判断两个对象或两个原始类型是否相等   |
| assertNotEquals   | 判断两个对象或两个原始类型是否不相等 |
| assertSame        | 判断两个对象引用是否指向同一个对象   |
| assertNotSame     | 判断两个对象引用是否指向不同的对象   |
| assertTrue        | 判断给定的布尔值是否为 true          |
| assertFalse       | 判断给定的布尔值是否为 false         |
| assertNull        | 判断给定的对象引用是否为 null        |
| assertNotNull     | 判断给定的对象引用是否不为 null      |
| assertArrayEquals | 数组断言                             |
| assertAll         | 组合断言                             |
| assertThrows      | 异常断言                             |
| assertTimeout     | 超时断言                             |
| fail              | 快速失败                             |

###### 4.2.3 嵌套测试

```java
@DisplayName("A stack")
class TestingAStackDemo {

    Stack<Object> stack;

    @Test
    @DisplayName("is instantiated with new Stack()")
    void isInstantiatedWithNew() {
        new Stack<>();
    }

    @Nested
    @DisplayName("when new")
    class WhenNew {

        @BeforeEach
        void createNewStack() {
            stack = new Stack<>();
        }

        @Test
        @DisplayName("is empty")
        void isEmpty() {
            assertTrue(stack.isEmpty());
        }

        @Test
        @DisplayName("throws EmptyStackException when popped")
        void throwsExceptionWhenPopped() {
            assertThrows(EmptyStackException.class, stack::pop);
        }

        @Test
        @DisplayName("throws EmptyStackException when peeked")
        void throwsExceptionWhenPeeked() {
            assertThrows(EmptyStackException.class, stack::peek);
        }

        @Nested
        @DisplayName("after pushing an element")
        class AfterPushing {

            String anElement = "an element";

            @BeforeEach
            void pushAnElement() {
                stack.push(anElement);
            }

            @Test
            @DisplayName("it is no longer empty")
            void isNotEmpty() {
                assertFalse(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when popped and is empty")
            void returnElementWhenPopped() {
                assertEquals(anElement, stack.pop());
                assertTrue(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when peeked but remains not empty")
            void returnElementWhenPeeked() {
                assertEquals(anElement, stack.peek());
                assertFalse(stack.isEmpty());
            }
        }
    }
}
```



###### 4.2.4 参数化测试







### （五）SpringBoot3-核心原理

#### 1、事件和监听器

##### 1.1 生命周期监听

###### 1.1.1 监听器

1. 自定义`SpringApplicationRunListener`来**监听事件**；

1. 1. 编写`SpringApplicationRunListener` **实现类**
   2. 在 `META-INF/spring.factories` 中配置 `org.springframework.boot.SpringApplicationRunListener=自己的Listener`，还可以指定一个**有参构造器**，接受两个参数`(SpringApplication application, String[] args)`
   3. springboot 在`spring-boot.jar`中配置了默认的 Listener，如下

```properties
org.springframework.boot.SpringApplicationRunListener=org.springframework.boot.context.event.EventPublishingRunListener
```

```java
/**
 * Listener先要从 META-INF/spring.factories
 *
 * 1、引导：利用 BootstrapContext 引导整个项目启动
 *      starting：               应用开始，SpringApplication的run方法一调用，只要有了 BootstrapContext 就执行
 *      environmentPrepared：    环境准备好（把启动参数等绑定到环境变量中），但是ioc还没有创建
 * 2、启动
 *      contextPrepared：        ioc容器创建并准备好，但是sources（主配置类）没有加载。并关闭引导上下文；组件都没创建
 *      contextLoaded：          ioc容器加载。主配置类加载进去。但是ioc容器还没刷新（bean没创建）
 *      ===============截至以前，ioc容器还没有bean
 *      started：                ioc容器刷新（所有bean创建），但是runner没调用
 *      ready：                  ioc容器刷新（所有bean创建），所有runner调用
 * 3、运行
 */
public class AppListener implements SpringApplicationRunListener {
    @Override
    public void starting(ConfigurableBootstrapContext bootstrapContext) {
        System.out.println("==========starting===========正在启动====");
    }

    @Override
    public void environmentPrepared(ConfigurableBootstrapContext bootstrapContext, ConfigurableEnvironment environment) {
        System.out.println("==========environmentPrepared===========环境准备完成====");    }

    @Override
    public void contextPrepared(ConfigurableApplicationContext context) {
        System.out.println("==========contextPrepared===========上下文准备完成====");    }

    @Override
    public void contextLoaded(ConfigurableApplicationContext context) {
        System.out.println("==========contextLoaded===========IOC容器加载完成====");

}

    @Override
    public void started(ConfigurableApplicationContext context, Duration timeTaken) {
        System.out.println("==========started===========启动完成====");
    }

    @Override
    public void ready(ConfigurableApplicationContext context, Duration timeTaken) {
        System.out.println("==========ready===========准备就绪====");
    }

    @Override
    public void failed(ConfigurableApplicationContext context, Throwable exception) {
        System.out.println("==========failed===========启动失败====");
    }
}

```



###### 1.1.2 生命周期全流程

![image-20241114205444660](F:\java\springboot\img\image-20241114205444660.png)



##### 1.2 事件触发时机









#### 2、自动配置原理

