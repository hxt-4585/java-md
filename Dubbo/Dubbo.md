# Dubbo



## 一、分布式简要说明

老式系统(单一应用架构)就是把一个系统，统一放到一个服务器当中然后每一个服务器上放一个系统，如果说要更新代码的话，每一个服务器上的系统都要重新去部署十分的麻烦。

而分布式系统就是将一个完整的系统拆分成多个不同的服务，然后在将每一个服务单独的放到一个服务器当中。



### 1.1 发展演变

![image-20241116193531655](.\img\image-20241116193531655.png)



### 1.2 ORM架构

**单一应用架构**：一个项目装到一个服务器当中，也可以运行多个服务器每一个服务器当中都装一个项目。
缺点：1.如果要添加某一个功能的话就要把一个项目重新打包，在分别部署到每一个服务器当中去。2.如果后期项目越来越大的话单台服务器跑一个项目压力会很大的。会不利于维护，开发和程序的性能。



### 1.3 MVC

垂直应用架构中被拆成小的服务，且被部署到多个服务器中，但是由于业务逻辑不可能完全独立，各个服务器需要通信

![image-20241116193713611](.\img\image-20241116193713611.png)



### 1.4 分布式服务架构

![image-202411161939072241](.\img\image-20241116193907224.png)

### 1.5 RPC简介

**分布式应用架构(远程过程调用)**：当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。

什么叫RPC

**RPC [ Remote Procedure Call]**：是指远程过程调用，是一种进程问通信方式，他是一种技术的思想，而不是规范。它允许程序调用另一个地址空间(通常是共享网络的另一台机器上)的过程或函数，而不用程序员显式编码这个远程调用的细节。即程序员无论是调用本地的还是远程的函数，本质上编写的调用代码基本相同。
**RPC**——远程过程调用，它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。也就是说两台服务器A，B，一个应用部署在A服务器上，想要调用B服务器上应用提供的方法，由于不在一个内存空间，不能直接调用，需要通过网络来表达调用的语义和传达调用的数据。



**RPC工作原理**

![image-20241116194736541](.\img\image-20241116194736541.png)

1、Client像调用本地服务似的调用远程服务；
2、Client stub接收到调用后，将方法、参数序列化
3、客户端通过sockets将消息发送到服务端
4、Server stub 收到消息后进行解码（将消息对象反序列化）
5、Server stub 根据解码结果调用本地的服务
6、本地服务执行(对于服务端来说是本地执行)并将结果返回给Server stub
7、Server stub将返回结果打包成消息（将结果消息对象序列化）
8、服务端通过sockets将消息发送到客户端
9、Client stub接收到结果消息，并进行解码（将结果消息发序列化）
10、客户端得到最终结果



**RPC 调用分以下两种：**
**同步调用**：客户方等待调用执行完成并返回结果。
**异步调用**：客户方调用后不用等待执行结果返回，但依然可以通过回调通知等方式获取返回结果。若客户方不关心调用返回结果，则变成单向异步调用，单向调用不用返回结果。



### 1.6 SOA

**流动计算架构**：在分布式应用架构的基础上增加了一个**调度、治理中心**基于访问压力实时管理集群容量、提高集群的利用率，用于提高机器利用率的 资源调度和治理中心(SOA) 是关键 **(不浪费计算机资源)**

![image-20241116193931451](.\img\image-20241116193931451.png)



## 二、Dubbo

### 2.1 核心概念

Dubbo 是一款高性能、轻量级的Java RPC框架，它提供了三大核心能力：面向接口的远程方法调用，智能容错和负载均衡，服务自动注册和发现。分布式系统是将一个系统拆分为多个不同的服务



### 2.2 特性

![image-20241116195218757](.\img\image-20241116195218757.png)



### 2.3 设计架构

![image-20241116195240599](.\img\image-20241116195240599.png)

**服务提供者（Provider）**：暴露服务的服务提供方，服务提供者在启动时，向注册中心注册自己提供的服务。
**服务消费者（Consumer）**: 调用远程服务的服务消费方，服务消费者在启动时，向注册中心订阅自己所需的服务，服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
**注册中心（Registry）**：注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
**监控中心（Monitor）**：服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

## 三、环境搭建

### 3.1 zookeeper注册中心

搭建略

### 3.2 监控中心



### 3.3 简易控制中心



## 四、Dubbo项目环境搭建

### 4.1 提供者

```xml
<!-- 依赖 -->
<dependencies>
    <dependency>
        <groupId>com.hxt</groupId>
        <artifactId>gmall-interface</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>

    <!--dubbo-->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>dubbo</artifactId>
        <version>2.6.2</version>
    </dependency>
    <!--注册中心是 zookeeper，引入zookeeper客户端-->
    <dependency>
        <groupId>org.apache.curator</groupId>
        <artifactId>curator-framework</artifactId>
        <version>2.12.0</version>
    </dependency>
</dependencies>
```



```java
public class UserServiceImpl implements UserService {
    @Override
    public List<UserAddress> getUserAddressList(String userId) {
        System.out.println("UserServiceImpl.....old...");
        // TODO Auto-generated method stub
        UserAddress address1 = new UserAddress(1, "北京市昌平区宏福科技园综合楼3层", "1", "李老师", "010-56253825", "Y");
        UserAddress address2 = new UserAddress(2, "深圳市宝安区西部硅谷大厦B座3层（深圳分校）", "1", "王老师", "010-56253825", "N");
		/*try {
			Thread.sleep(4000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}*/
        return Arrays.asList(address1,address2);
    }
}


public class MainApplication {
    public static void main(String[] args) throws IOException {
        ClassPathXmlApplicationContext ioc = new ClassPathXmlApplicationContext("provider.xml");
        ioc.start();

        System.in.read();
    }
}

```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd
		http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
    <!--1、指定当前服务/应用的名字(同样的服务名字相同，不要和别的服务同名)-->
    <dubbo:application name="user-service-provider"></dubbo:application>
    <!--2、指定注册中心的位置-->
    <!--<dubbo:registry address="zookeeper://127.0.0.1:2181"></dubbo:registry>-->
    <dubbo:registry protocol="zookeeper" address="127.0.0.1:2181"></dubbo:registry>
    <!--3、指定通信规则（通信协议? 服务端口）-->
    <dubbo:protocol name="dubbo" port="20880"></dubbo:protocol>
    <!--4、暴露服务 让别人调用 ref指向服务的真正实现对象-->
    <dubbo:service interface="com.hxt.gmall.service.UserService" ref="userServiceImpl"></dubbo:service>


    <dubbo:monitor protocol="registry"></dubbo:monitor>
    <!--服务的实现-->
    <bean id="userServiceImpl" class="com.hxt.gmall.service.impl.UserServiceImpl"></bean>
</beans>
```



### 4.2 消费者

```xml
<!-- 依赖 -->
<dependencies>
    <dependency>
        <groupId>com.hxt</groupId>
        <artifactId>gmall-interface</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>

    <!--dubbo-->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>dubbo</artifactId>
        <version>2.6.2</version>
    </dependency>
    <!--注册中心是 zookeeper，引入zookeeper客户端-->
    <dependency>
        <groupId>org.apache.curator</groupId>
        <artifactId>curator-framework</artifactId>
        <version>2.12.0</version>
    </dependency>
</dependencies>
```

```java
@Service
public class OrderServiceImpl implements OrderService {

	@Autowired
	UserService userService;
	@Override
	public List<UserAddress> initOrder(String userId) {
		// TODO Auto-generated method stub
		System.out.println("用户id："+userId);
		//1、查询用户的收货地址
		List<UserAddress> addressList = userService.getUserAddressList(userId);
		for (UserAddress userAddress : addressList) {
			System.out.println(userAddress.getUserAddress());
		}
		return addressList;
	}

}


public class MainApplication {
    public static void main(String[] args) throws IOException {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("consumer.xml");
        OrderService orderService = applicationContext.getBean(OrderService.class);

        //调用方法查询出数据
        orderService.initOrder("1");
        System.out.println("调用完成...");
        System.in.read();
    }
}

```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd
		http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd
		http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
    <!--包扫描-->
    <context:component-scan base-package="com.hxt.gmall.service.impl"/>

    <!--指定当前服务/应用的名字(同样的服务名字相同，不要和别的服务同名)-->
    <dubbo:application name="order-service-consumer"></dubbo:application>
    <!--指定注册中心的位置-->
    <dubbo:registry address="zookeeper://127.0.0.1:2181"></dubbo:registry>

    <!--调用远程暴露的服务，生成远程服务代理-->
    <dubbo:reference interface="com.hxt.gmall.service.UserService" id="userService"></dubbo:reference>

    <!-- 两者都可以，registry表示从注册中心发现监控中心地址，否则直连监控中心 -->
    <dubbo:monitor protocol="registry"></dubbo:monitor>
<!--    <dubbo:monitor address="127.0.0.1:7070"></dubbo:monitor>-->
</beans>
```

### 4.3 测试

![image-20241120180342836](.\img\image-20241120180342836.png)

![image-20241120180416059](.\img\image-20241120180416059.png)







## 五、Dubbo整合SpringBoot

### 5.1 提供者

```java
// 这里的service并非spring中的注解，是为了暴露服务，这边用@component代替
// 再未整合的项目中，provider.xml文件里暴露服务的操作为    <dubbo:reference interface="com.hxt.gmall.service.UserService" id="userService"></dubbo:reference>
@Service
@Component
public class UserServiceImpl implements UserService {
    @Override
    public List<UserAddress> getUserAddressList(String userId) {
        System.out.println("UserServiceImpl.....old...");
        // TODO Auto-generated method stub
        UserAddress address1 = new UserAddress(1, "北京市昌平区宏福科技园综合楼3层", "1", "李老师", "010-56253825", "Y");
        UserAddress address2 = new UserAddress(2, "深圳市宝安区西部硅谷大厦B座3层（深圳分校）", "1", "王老师", "010-56253825", "N");
		/*try {
			Thread.sleep(4000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}*/
        return Arrays.asList(address1,address2);
    }
}


@EnableDubbo// 开启基于注解的dubbo功能
@SpringBootApplication
public class BootUserServiceProviderApplication {

	public static void main(String[] args) {
		SpringApplication.run(BootUserServiceProviderApplication.class, args);
	}
}

```

```properties
dubbo.application.name=user-service-provider
dubbo.registry.address=127.0.0.1:2181
dubbo.registry.protocol=zookeeper

dubbo.protocol.name=dubbo
dubbo.protocol.port=20881

dubbo.monitor.protocol=registry
```



### 5.2 消费者

```java
@Service // 此处为spring注解
public class OrderServiceImpl implements OrderService {
	// 注意这个注解
	@Reference
	UserService userService;
	@Override
	public List<UserAddress> initOrder(String userId) {
		// TODO Auto-generated method stub
		System.out.println("用户id："+userId);
		//1、查询用户的收货地址
		List<UserAddress> addressList = userService.getUserAddressList(userId);
//		for (UserAddress userAddress : addressList) {
//			System.out.println(userAddress.getUserAddress());
//		}
		return addressList;
	}

}

@Controller
public class OrderController {

    @Autowired
    private OrderService orderService;

    @ResponseBody
    @GetMapping("/initOrder")
    public List<UserAddress> initOrder(@RequestParam("uid")String userId){
        return orderService.initOrder(userId);
    }
}

@EnableDubbo
@SpringBootApplication
public class BootOrderServiceConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(BootOrderServiceConsumerApplication.class, args);
    }

}
```

```properties
server.port=8082

dubbo.application.name=boot-order-service-consumer
dubbo.registry.address=zookeeper://127.0.0.1:2181
dubbo.monitor.protocol=registry
```

### 5.3 测试

![image-20241120181109047](.\img\image-20241120181109047.png)

![image-20241120181203763](.\img\image-20241120181203763.png)

## 六、Dubbo配置

### 6.1 配置原则

![image-20241120173715967](.\img\image-20241120173715967.png)

**JVM 启动** -D 参数优先，这样可以使用户在部署和启动时进行参数重写，比如在启动时需改变协议的端口。
**XML 次之**，如果在 XML 中有配置，则 dubbo.properties 中的相应配置项无效。
**Properties 最后**，相当于缺省值，只有 XML 没有配置时，dubbo.properties 的相应配置项才会生效，通常用于共享公共配置，比如应用名。

![image-20241120203151016](.\img\image-20241120203151016.png)



### 6.2 启动时检查

Dubbo 缺省会在启动时检查依赖的服务是否可用，不可用时会抛出异常，阻止 Spring 初始化完成，以便上线时，能及早发现问题，默认 check=“true”。如，当`boot-user-service-provider`未启动时，先去启动 `boot-order-service-consumer` 会发生报错

可以通过 check=“false” 关闭检查

```properties
server.port=8082

dubbo.application.name=boot-order-service-consumer
dubbo.registry.address=zookeeper://127.0.0.1:2181
dubbo.monitor.protocol=registry

dubbo.consumer.check=false
```

![image-20241120203625945](.\img\image-20241120203625945.png)



### 6.3 全局超时配置

#### 6.3.1 消费者超时配置

```xml
<!-- 优先级最高，指定方法超时 -->
<dubbo:reference interface="com.hxt.gmall.service.UserService" id="userService" >
    <dubbo:method name="getUserAddressList" timeout="1000"/>
</dubbo:reference>

<!-- consumer.xml: 限制超时时间，默认为1000ms -->
<dubbo:reference interface="com.hxt.gmall.service.UserService" id="userService" timeout="3000"></dubbo:reference>

<!-- 也可以全局配置 -->
<dubbo:consumer check="false" timeout="3000"></dubbo:consumer>

```

```java
public class UserServiceImpl implements UserService {
    @Override
    public List<UserAddress> getUserAddressList(String userId) {
        System.out.println("UserServiceImpl.....old...");
        // TODO Auto-generated method stub
        UserAddress address1 = new UserAddress(1, "北京市昌平区宏福科技园综合楼3层", "1", "李老师", "010-56253825", "Y");
        UserAddress address2 = new UserAddress(2, "深圳市宝安区西部硅谷大厦B座3层（深圳分校）", "1", "王老师", "010-56253825", "N");
		try {
            // 设定睡眠时间大于限定时间
			Thread.sleep(4000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

        return Arrays.asList(address1,address2);
    }
}

```

消费方报错

![image-20241120204224439](C:\Users\hxt\AppData\Roaming\Typora\typora-user-images\image-20241120204224439.png)







#### 6.3.2 提供者超时配置

```java
<dubbo:service interface="com.hxt.gmall.service.UserService" ref="userServiceImpl" >
    <dubbo:method name="getUserAddressList" timeout="1000"/>
</dubbo:service>

<dubbo:service interface="com.hxt.gmall.service.UserService" ref="userServiceImpl" timeout="3000"></dubbo:service>
    
<dubbo:provider timeout="5000"></dubbo:provider>

```

级别一样的同时，消费者优先。



#### 6.3.3 总结

配置的覆盖规则：

1. 方法级配置别优于接口级别，即小Scope优先
2. Consumer端配置 优于 Provider配置 优于 全局配置，
3. 最后是Dubbo Hard Code的配置值（见配置文档）

![image-20241120204801087](.\img\image-20241120204801087.png)

### 6.4 重试次数

```xml
<dubbo:service interface="com.hxt.gmall.service.UserService" ref="userServiceImpl" >
	<dubbo:method name="getUserAddressList" timeout="1000" retries="3"/>
</dubbo:service>
```

当服务提供方有三个

![image-20241120210529730](.\img\image-20241120210529730.png)

![image-20241120210708751](.\img\image-20241120210708751.png)

总共访问四次（3次重复），是轮询访问



**幂等操作**：查询、删除、修改。意味着重复多次操作不会有变化，用同一条sql反复删除和修改数据库不会在发生变化

**非幂等操作**：新增。重复新增的话，数据库的内容会不断增多



### 6.5 多版本

当一个接口实现，出现不兼容升级时，可以使用版本号进行过度，版本号不同的服务互相间不引用。

```xml
<!-- 提供者 -->
<dubbo:service interface="com.hxt.gmall.service.UserService" ref="userServiceImpl01" version="1.0.0">
    <dubbo:method name="getUserAddressList" retries="3"/>
</dubbo:service>

<dubbo:service interface="com.hxt.gmall.service.UserService" ref="userServiceImpl02" version="2.0.0">
    <dubbo:method name="getUserAddressList" retries="3"/>
</dubbo:service>

<!--服务的实现-->
<bean id="userServiceImpl01" class="com.hxt.gmall.service.impl.UserServiceImpl"></bean>
<bean id="userServiceImpl02" class="com.hxt.gmall.service.impl.UserServiceImpl02"></bean>

```

```xml
<!-- 消费者 -->
<!--调用远程暴露的服务，生成远程服务代理-->
<dubbo:reference interface="com.hxt.gmall.service.UserService" id="userService" version="2.0.0">
    <dubbo:method name="getUserAddressList"/>
</dubbo:reference>
```



### 6.6 本地存根

远程服务后，客户端通常只剩下接口，而实现全在服务端，但提供方有时候想在客户端也执行部分逻辑，比如：做ThreadLocal缓存，提前验证参数，调用失败后伪造容错数据等等，此时就需要在API中带上 `Stub` ，客户端生成 `Proxy` 实例，会把 `Proxy` 通过构造函数传给 `Stub` ，然后把 `Stub` 暴露给用户， `Stub` 可以决定要不要去调 `Proxy`

![image-20241121202803905](.\img\image-20241121202803905.png)

```java
// 放在了gmall-interface包中
public class UserServiceStub implements UserService {

    private final UserService userService;

    public UserServiceStub(UserService userService) {
        this.userService = userService;
    }

    @Override
    public List<UserAddress> getUserAddressList(String userId) {
        System.out.println("UserServiceStub....");
        if (!StringUtils.isEmpty(userId)){
            return userService.getUserAddressList(userId);
        }
        return null;
    }
}

```

```xml
<!-- 消费者端 -->
<dubbo:reference interface="com.hxt.gmall.service.UserService" id="userService" version="2.0.0" stub="com.hxt.gmall.service.impl.UserServiceStub">
    <dubbo:method name="getUserAddressList"/>
</dubbo:reference>
```



### 6.7 springboot与dubbo整合

在配置文件中添加

```properties
dubbo.scan.base-packages=com.hxt.gmall
```

则不需要在添加注解 `@EnableDubbo`

#### 6.7.1 方式一

在 `application.properties` 配置属性，使用 `@Service` 暴露服务和使用 `@Reference` 使用服务，两个注解可以添加各种属性，如 `@Service(timeout = 5000)`

#### 6.7.2 方式二

保留 `dubbo.xml` 配置文件

```java
@ImportResource(locations = "classpath:provider.xml")
@SpringBootApplication
public class BootUserServiceProviderApplication {

    public static void main(String[] args) {
       SpringApplication.run(BootUserServiceProviderApplication.class, args);
    }
}
```

需要把 `@Service` 注解掉，因为在`provider.xml`中已经暴露了

#### 6.7.3 方式三

使用注解API方式，将每一个组件手动配置到容器中

```java
@Configuration
public class MyDubboConfig {

    @Bean
    public ApplicationConfig applicationConfig(){
        ApplicationConfig applicationConfig = new ApplicationConfig();
        applicationConfig.setName("boot-user-service-provider");
        return applicationConfig;
    }

//    <dubbo:registry protocol="zookeeper" address="127.0.0.1:2181"></dubbo:registry>
    @Bean
    public RegistryConfig registryConfig(){
        RegistryConfig registryConfig = new RegistryConfig();
        registryConfig.setAddress("127.0.0.1:2181");
        registryConfig.setProtocol("zookeeper");
        return registryConfig;
    }

//    <dubbo:protocol name="dubbo" port="20882"></dubbo:protocol>
    @Bean
    public ProtocolConfig protocolConfig(){
        ProtocolConfig protocolConfig = new ProtocolConfig();
        protocolConfig.setName("dubbo");
        protocolConfig.setPort(20880);
        return protocolConfig;
    }

//    <dubbo:service interface="com.hxt.gmall.service.UserService" ref="userServiceImpl"></dubbo:service>
    @Bean
    public ServiceConfig<UserService> userServiceConfig(UserService userService){
        ServiceConfig<UserService> serviceConfig = new ServiceConfig<>();
        serviceConfig.setInterface(UserService.class);
        // 已经注入了在UserServiceImpl.java中
        serviceConfig.setRef(userService);
//        serviceConfig.setVersion("1.0.0");

        // 配置每一个method信息
        MethodConfig methodConfig = new MethodConfig();
        methodConfig.setName("getUserAddressList");
//        methodConfig.setTimeout(1000);

        // 将method关联到ServiceConfig中
        ArrayList<MethodConfig> methods = new ArrayList<>();
        methods.add(methodConfig);
        serviceConfig.setMethods(methods);

        // ProviderConfig
        ProviderConfig providerConfig = new ProviderConfig();
        providerConfig.setTimeout(5000);
        serviceConfig.setProvider(providerConfig);

        // MonitorConfig
        MonitorConfig monitorConfig = new MonitorConfig();
        monitorConfig.setProtocol("zookeeper");
        serviceConfig.setMonitor(monitorConfig);

        return serviceConfig;
    }
}
```

开启 `@EnableDubbo` 和 `@Service` 暴露服务的注解



## 七、高可用

### 7.1 zookeeper宕机与dubbo直连

![image-20241121213259456](.\img\image-20241121213259456.png)

此时一切正常



关闭zookeeper

![image-20241121213612825](.\img\image-20241121213612825.png)

但是消费者仍旧能访问

![image-20241121213707738](.\img\image-20241121213707738.png)



原因：

健壮性↓

​		监控中心宕掉不影响使用，只是丢失部分采样数据
​		数据库宕掉后，注册中心仍能通过缓存提供服务列表查询，但不能注册新服务
​		注册中心对等集群，任意一台宕掉后，将自动切换到另一台
​		注册中心全部宕掉后，服务提供者和服务消费者仍能通过本地缓存通讯
​		服务提供者无状态，任意一台宕掉后，不影响使用
​		服务提供者全部宕掉后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复
​		高可用：通过设计，减少系统不能提供服务的时间；



![image-20241121214154760](.\img\image-20241121214154760.png)

```java
// 采用直连的方式
@Reference(url = "127.0.0.1:20880")
UserService userService;
```



### 7.2 集群下dubbo负载均衡配置

**如何配置**

![image-20241121215910593](.\img\image-20241121215910593.png)



![image-20241121215927577](.\img\image-20241121215927577.png)

#### 7.2.1 RoundRobin LoadBalance

有顺序的，且默认使用

![image-20241121214753853](.\img\image-20241121214753853.png)

![image-20241121215806951](.\img\image-20241121215806951.png)

![image-20241121215744092](.\img\image-20241121215744092.png)



#### 7.2.2 LeastActive LoadBalance

![image-20241121214915850](.\img\image-20241121214915850.png)





#### 7.2.3 ConsistentHash LoadBalance

![image-20241121215008011](.\img\image-20241121215008011.png)





### 7.3 服务降级

**当服务器压力剧增的情况下，根据实际业务情况及流量，对一些服务和页面有策略的不处理或换种简单的方式处理，从而释放服务器资源以保证核心交易正常运作或高效运作。**
可以通过服务降级功能临时屏蔽某个出错的非关键服务，并定义降级后的返回策略。
向注册中心写入动态配置覆盖规则：

![image-20241122214210309](.\img\image-20241122214210309.png)

点击屏蔽

**![image-20241122214247005](.\img\image-20241122214247005.png)**

点击容错，可以设置超时

![image-20241122214415807](.\img\image-20241122214415807.png)





### 7.4 集群容错

**Failover Cluster**

```
失败自动切换，当出现失败，重试其它服务器。通常用于读操作，但重试会带来更长延迟。可通过 retries=“2” 来设置重试次数(不含第一次)。
重试次数配置如下：
```

```
<dubbo:service retries=“2” />
或
<dubbo:reference retries=“2” />
或
dubbo:reference
<dubbo:method name=“findFoo” retries=“2” />
</dubbo:reference>
```

**Failfast Cluster**

```
快速失败，只发起一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增记录。
```

**Failsafe Cluster**

```
失败安全，出现异常时，直接忽略。通常用于写入审计日志等操作。
```

**Failback Cluster**

```
失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作。
```

**Forking Cluster**

```
并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多服务资源。可通过 forks=“2” 来设置最大并行数。
```

**Broadcast Cluster**

```
广播调用所有提供者，逐个调用，任意一台报错则报错 [2]。通常用于通知所有提供者更新缓存或日志等本地资源信息。
```



## 八、Dubbo原理

### 8.1 RPC&Netty调用

#### 8.1.1 RPC原理

![image-20241122215016190](.\img\image-20241122215016190.png)

```
一次完整的RPC调用流程（同步调用，异步另说）如下：
1）服务消费方（client）调用以本地调用方式调用服务；
2）client stub接收到调用后负责将方法、参数等组装成能够进行网络传输的消息体
3）client stub找到服务地址，并将消息发送到服务端
4）server stub收到消息后进行解码
5）server stub根据解码结果调用本地服务
6）本地服务执行并将结果返回给server stub
7）server stub将返回结果打包成消息并发送至消费方
8）client stub接收到消息，并进行解码
9）服务消费方得到最终结果
```



#### 8.1.2 Netty通信原理



### 8.2 Dubbo框架设计

![](.\img\482d9fb120fa48c780937bda6155642a.png)

```
上层依赖下层

1）config配置层：对外接口配置，以ServiceConfig、ReferenceConfig为中心，可以直接初始化配置类，也可以通过spring解析配置生成配置类
2）proxy服务代理层：服务接口透明代理，生成服务的客户端 Stub 和服务器端 Skeleton, 以 ServiceProxy 为中心，扩展接口为 ProxyFactory
3）registry 注册中心层：封装服务地址的注册与发现，以服务 URL 为中心，扩展接口为 RegistryFactory, Registry, RegistryService
4）cluster 路由层：封装多个提供者的路由及负载均衡，并桥接注册中心，以 Invoker 为中心，扩展接口为 Cluster, Directory, Router, LoadBalance
5）monitor 监控层：RPC 调用次数和调用时间监控，以 Statistics 为中心，扩展接口为 MonitorFactory, Monitor, MonitorService
6）protocol 远程调用层：封装 RPC 调用，以 Invocation, Result 为中心，扩展接口为 Protocol, Invoker, Exporter
7）exchange 信息交换层：封装请求响应模式，同步转异步，以 Request, Response 为中心，扩展接口为 Exchanger, ExchangeChannel, ExchangeClient, ExchangeServer
8）transport 网络传输层：抽象 mina 和 netty 为统一接口，以 Message 为中心，扩展接口为 Channel, Transporter, Client, Server, Codec
9）serialize 数据序列化层：可复用的一些工具，扩展接口为 Serialization, ObjectInput, ObjectOutput, ThreadPool
```

### 8.3 启动解析、加载配置信息