# spring

## 一、spring启示录

### （1）控制反转

​	**IOC（Inversion of Control）**，控制反转的核心是：**将对象的创建权交出去，将对象和对象之间关系的管理权交出去，由第三方容器来负责创建与维护**。

​	反转：

​		1、不在程序中采用硬编码的方式来new对象。

​		2、不在程序中采用硬编码的方式来维护对象。



### （2）spring概述

![image-20240820153600456](.\img\image-20240820153600456.png)

​	1、Spring Core

​	这是Spring框架最基础的部分，它提供了依赖注入（DependencyInjection）特征来实现容器对Bean的管理。核心容器的主要组件是 BeanFactory，BeanFactory是工厂模式的一个实现，是任何Spring应用的核心。它使用IoC将应用配置和依赖从实际的应用代码中分离出来。

​	2、Spring Context

​	如果说核心模块中的BeanFactory使Spring成为容器的话，那么上下文模块就是Spring成为框架的原因。

​	这个模块扩展了BeanFactory，增加了对国际化（I18N）消息、事件传播、验证的支持。另外提供了许多企业服务，例如电子邮件、JNDI访问、EJB集成、远程以及时序调度（scheduling）服务。也包括了对模版框架例如Velocity和FreeMarker集成的支持

​	3、Spring AOP

​	Spring在它的AOP模块中提供了对面向切面编程的丰富支持，Spring AOP 模块为基于 Spring 的应用程序中的对象提供了事务管理服务。通过使用 Spring AOP，不用依赖组件，就可以将声明性事务管理集成到应用程序中，可以自定义拦截器、切点、日志等操作。

​	4、Spring DAO

​	提供了一个JDBC的抽象层和异常层次结构，消除了烦琐的JDBC编码和数据库厂商特有的错误代码解析，用于简化JDBC。

​	5、Spring ORM

​	Spring提供了ORM模块。Spring并不试图实现它自己的ORM解决方案，而是为几种流行的ORM框架提供了集成方案，包括Hibernate、JDO和iBATIS SQL映射，这些都遵从 Spring 的通用事务和 DAO 异常层次结构。

​	6、Spring Web MVC

​	Spring为构建Web应用提供了一个功能全面的MVC框架。虽然Spring可以很容易地与其它MVC框架集成，例如Struts，但Spring的MVC框架使用IoC对控制逻辑和业务对象提供了完全的分离。

​	7、Spring WebFlux

​	Spring Framework 中包含的原始 Web 框架 Spring Web MVC 是专门为 Servlet API 和 Servlet 容器构建的。反应式堆栈 Web 框架 Spring WebFlux 是在 5.0 版的后期添加的。它是完全非阻塞的，支持反应式流(Reactive Stream)背压，并在Netty，Undertow和Servlet 3.1+容器等服务器上运行。

​	8、Spring Web

​	Web 上下文模块建立在应用程序上下文模块之上，为基于 Web 的应用程序提供了上下文，提供了Spring和其它Web框架的集成，比如Struts、WebWork。还提供了一些面向服务支持，例如：实现文件上传的multipart请求

### 	（3）特点

非侵入式：

![image-20240820154813268](.\img\image-20240820154813268.png)



## 二、spring入门程序

```xml
<!-- 
不同对象id不可相同
创建的对象存储在一个map集合里，并以id为关键字 
-->

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="userBean" class="com.powernode.spring6.test.User"/>
</beans>
```

```java
// spring是通过调用类的无参构造方法来创建对象，如果创建对象中仅包含有参构造，那么会创建失败
// 在spring配置文件中配置的bean可以任意类，只要这个类不是抽象的，并且提供了无参数构造方法。

public class User {
    public User(){}
    public User(String name){
        System.out.println(name);
    }
}

// getBean得到的对象是Object，如果不想向下转型可以采用以下方法
// User user = applicationContext.getBean("userBean", User.class);
@Test
public void testBean(){
    // BeanFactory是ApplicationContext父接口
    // BeanFactory beanFactory = new ClassPathXmlApplicationContext("spring.xml");

    
    // ClassPathXmlApplicationContext是从类路径下读取文件
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");
    // 若不想从类路径读取
    // ApplicationContext applicationContext = new FileSystemXmlApplicationContext("d:/spring6.xml");

    Object userBean = applicationContext.getBean("userBean");
    System.out.println(userBean);
}
```



日志文件的启用

```xml
<!--log4j2的依赖-->
<dependency>
  <groupId>org.apache.logging.log4j</groupId>
  <artifactId>log4j-core</artifactId>
  <version>2.19.0</version>
</dependency>
<dependency>
  <groupId>org.apache.logging.log4j</groupId>
  <artifactId>log4j-slf4j2-impl</artifactId>
  <version>2.19.0</version>
</dependency>
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 类的根路径下提供log4j2.xml配置文件（文件名固定为：log4j2.xml，文件必须放到类根路径下。） -->
<configuration>

    <loggers>
        <!--
            level指定日志级别，从低到高的优先级：
                ALL < TRACE < DEBUG < INFO < WARN < ERROR < FATAL < OFF
        -->
        <root level="DEBUG">
            <appender-ref ref="spring6log"/>
        </root>
    </loggers>

    <appenders>
        <!--输出日志信息到控制台-->
        <console name="spring6log" target="SYSTEM_OUT">
            <!--控制日志输出的格式-->
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss SSS} [%t] %-3level %logger{1024} - %msg%n"/>
        </console>
    </appenders>

</configuration>
```



## 三、spring对IOC的实现

### （1）set注入

```java
public class UserDao {
    public void insert(){
        System.out.println("正在保存用户数据。");
    }
}

public class UserServiceImpl implements UserService {

    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public void save() {
        userDao.insert();
    }
}


@Test
public void testSetDIP(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    UserService userService = applicationContext.getBean("userService", UserService.class);
    userService.save();
}
```

```xml
<bean id="userDao" class="com.powernode.spring6.dao.UserDao"/>
<bean id="userService" class="com.powernode.spring6.service.impl.UserServiceImpl">
    <!-- name是属性名，即set方法后面首字母小写的单词（setUserDao）
 		ref是要注入的bean对象的id
	-->
    <property name="userDao" ref="userDao"/>
</bean>

<!-- 实现原理：
 	通过property标签获取到属性名：userDao
	通过属性名推断出set方法名：setUserDao
	通过反射机制调用setUserDao()方法给属性赋值
-->
```



### （2）构造注入

```java
public class UserServiceImpl implements UserService {

    private UserDao userDao;

    public UserServiceImpl(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public void save() {
        userDao.insert();
    }
}

```

```xml
<!-- 方式一 -->
<bean id="userDao" class="com.powernode.spring6.dao.UserDao"/>
<bean id="userService" class="com.powernode.spring6.service.impl.UserServiceImpl">
    <!-- index代表给第几个参数赋值 -->
    <constructor-arg index="0" ref="userDao"/>
</bean>

<!-- 方式二 -->
<bean id="userDao" class="com.powernode.spring6.dao.UserDao"/>
<bean id="vipDao" class="com.powernode.spring6.dao.VipDao"/>

<bean id="userService" class="com.powernode.spring6.service.impl.UserServiceImpl">
<!-- 调换顺序也可，spring通过判断参数类型进行注入
	如果两个类型相同，那么按照顺序进行注入 
-->
    <constructor-arg ref="userDao"/>
    <constructor-arg ref="vipDao"/>
</bean>

<!-- 方式三 -->
<bean id="userDao" class="com.powernode.spring6.dao.UserDao"/>
<bean id="vipDao" class="com.powernode.spring6.dao.VipDao"/>

<bean id="userService" class="com.powernode.spring6.service.impl.UserServiceImpl">
    <constructor-arg name="userDao" ref="userDao"/>
    <constructor-arg name="vipDao" ref="vipDao"/>
</bean>

```



### （3）set注入专题

```xml
<!-- 外部注入 -->
<bean id="userDaoBean" class="com.powernode.spring6.dao.UserDao"/>
<bean id="userServiceBean" class="com.powernode.spring6.service.UserService">
    <property name="userDao" ref="userDaoBean"/>
</bean>




<!-- 内部注入 -->
<bean id="userServiceBean" class="com.powernode.spring6.service.UserService">
    <property name="userDao">
        <bean class="com.powernode.spring6.dao.UserDao"/>
    </property>
</bean>




<!-- 简单类型注入(byte，short，int，long，char），float，double，boolean...)
	上述这些类型无法通过创建对象，进行注入 
-->
<bean id="userBean" class="com.powernode.spring6.beans.User">
    <!--如果像这种int类型的属性，我们称为简单类型，这种简单类型在注入的时候要使用value属性，不能使用ref-->
    <!--<property name="age" value="20"/>-->
    <property name="age">
        <value>20</value>
    </property>
</bean>
<!--
@Getter
@Setter
@ToString
public class SimpleType {
    private byte b;
    private short s;
    private int i;
    private long l;
    private float f;
    private double d;
    private boolean flag;
    private char c;

    private Byte b1;
    private Short s1;
    private Integer i1;
    private Long l1;
    private Float f1;
    private Double d1;
    private Boolean flag1;
    private Character c1;

    private String str;

    private Date date;

    private Season season;

    private URI uri;

    private URL url;

    private LocalDate localDate;

    private Locale locale;

    private Class clazz;
}

enum Season {
    SPRING, SUMMER, AUTUMN, WINTER
}

-->
<bean id="simpleType" class="com.powernode.spring6.test.SimpleType">
    <property name="b" value="1"/>
    <property name="s" value="1"/>
    <property name="i" value="1"/>
    <property name="l" value="1"/>
    <property name="f" value="1"/>
    <property name="d" value="1"/>
    <property name="flag" value="false"/>

    <property name="c" value="a"/>
    <property name="b1" value="2"/>
    <property name="s1" value="2"/>
    <property name="i1" value="2"/>
    <property name="l1" value="2"/>
    <property name="f1" value="2"/>
    <property name="d1" value="2"/>
    <property name="flag1" value="true"/>
    <property name="c1" value="a"/>

    <property name="str" value="zhangsan"/>
    <!--注意：value后面的日期字符串格式不能随便写，必须是Date对象toString()方法执行的结果。-->
    <!--如果想使用其他格式的日期字符串，就需要进行特殊处理了。具体怎么处理，可以看后面的课程！！！！-->
    <property name="date" value="Fri Sep 30 15:26:38 CST 2022"/>
    <property name="season" value="WINTER"/>
    <property name="uri" value="/save.do"/>
    <!--spring6之后，会自动检查url是否有效，如果无效会报错。-->
    <property name="url" value="http://www.baidu.com"/>
    <property name="localDate" value="EPOCH"/>
    <!--java.util.Locale 主要在软件的本地化时使用。它本身没有什么功能，更多的是作为一个参数辅助其他方法完成输出的本地化。-->
    <property name="locale" value="CHINESE"/>
    <property name="clazz" value="java.lang.String"/>
</bean>




<!-- 级联赋值 -->
<bean id="clazzBean" class="com.powernode.spring6.beans.Clazz"/>

<bean id="student" class="com.powernode.spring6.beans.Student">
    <property name="name" value="张三"/>

    <!--要点1：以下两行配置的顺序不能颠倒-->
    <property name="clazz" ref="clazzBean"/>
    <!--要点2：clazz属性必须有getter方法-->
    <property name="clazz.name" value="高三一班"/>
</bean>



<!-- 注入数组 -->
<bean id="person" class="com.powernode.spring6.beans.Person">
    <property name="favariteFoods">
        <array>
            <value>鸡排</value>
            <value>汉堡</value>
            <value>鹅肝</value>
        </array>
    </property>
</bean>
<!-- 数组是非简单类型的时候 -->
<bean id="goods1" class="com.powernode.spring6.beans.Goods">
    <property name="name" value="西瓜"/>
</bean>

<bean id="goods2" class="com.powernode.spring6.beans.Goods">
    <property name="name" value="苹果"/>
</bean>

<bean id="order" class="com.powernode.spring6.beans.Order">
    <property name="goods">
        <array>
            <!--这里使用ref标签即可-->
            <ref bean="goods1"/>
            <ref bean="goods2"/>
        </array>
    </property>
</bean>



<!-- 注入列表
	列表中为非简单类型是同样要使用ref标签 
-->
<bean id="peopleBean" class="com.powernode.spring6.beans.People">
    <property name="names">
        <list>
            <value>铁锤</value>
            <value>张三</value>
            <value>张三</value>
            <value>张三</value>
            <value>狼</value>
        </list>
    </property>
</bean>


<!-- set类型注入 -->
<bean id="peopleBean" class="com.powernode.spring6.beans.People">
    <property name="phones">
        <set>
            <!--非简单类型可以使用ref，简单类型使用value-->
            <value>110</value>
            <value>110</value>
            <value>120</value>
            <value>120</value>
            <value>119</value>
            <value>119</value>
        </set>
    </property>
</bean>




<!-- map类型注入 -->
<bean id="peopleBean" class="com.powernode.spring6.beans.People">
    <property name="addrs">
        <map>
            <!--如果key不是简单类型，使用 key-ref 属性-->
            <!--如果value不是简单类型，使用 value-ref 属性-->
            <entry key="1" value="北京大兴区"/>
            <entry key="2" value="上海浦东区"/>
            <entry key="3" value="深圳宝安区"/>
        </map>
    </property>
</bean>



<!-- properties注入
 
private Properties properties;

public void setProperties(Properties properties) {
    this.properties = properties;
}
-->
<bean id="peopleBean" class="com.powernode.spring6.beans.People">
    <property name="properties">
        <props>
            <prop key="driver">com.mysql.cj.jdbc.Driver</prop>
            <prop key="url">jdbc:mysql://localhost:3306/spring</prop>
            <prop key="username">root</prop>
            <prop key="password">123456</prop>
        </props>
    </property>
</bean>




<!-- 注入空字符串和null -->
<!-- 注入空字符串 -->
<bean id="vipBean" class="com.powernode.spring6.beans.Vip">
    <!--空串的第一种方式-->
    <!--<property name="email" value=""/>-->
    <!--空串的第二种方式-->
    <property name="email">
        <value/>
    </property>
</bean>

<!-- 注入null -->
<!-- 方式一
	不给属性赋值 
-->
<bean id="vipBean" class="com.powernode.spring6.beans.Vip" />
<!-- 方式二
	null标签 
-->
<bean id="vipBean" class="com.powernode.spring6.beans.Vip">
    <property name="email">
        <null/>
    </property>
</bean>



<!-- 特殊字符 -->
<!-- 方式一
	用转义字符来代替 
    >	&gt;
    <	&lt;
    '	&apos;
    "	&quot;
    &	&amp;
-->
<!-- 方式二
	使用CDATA方式，放入![CDATA[]]中
-->
<bean id="mathBean" class="com.powernode.spring6.beans.Math">
    <property name="result">
        <!--只能使用value标签-->
        <value><![CDATA[2 < 3]]></value>
    </property>
</bean>
```



### （4）p命名空间注入

```xml
<!--
	在头部信息中加入xmlns:p="http://www.springframework.org/schema/p"
	基于setter方法，需要提供setter方法
-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="customerBean" class="com.powernode.spring6.beans.Customer" p:name="zhangsan" p:age="20"/>

</beans>

```





### （5）c命名空间注入

```xml
<!--
	在头部信息中加入xmlns:c="http://www.springframework.org/schema/c"
	需要提供构造方法
-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:c="http://www.springframework.org/schema/c"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--<bean id="myTimeBean" class="com.powernode.spring6.beans.MyTime" c:year="1970" c:month="1" c:day="1"/>-->

    <bean id="myTimeBean" class="com.powernode.spring6.beans.MyTime" c:_0="2008" c:_1="8" c:_2="8"/>

</beans>

```



### （6）util命名空间









## 四、bean的作用域

### （1）单例模式（默认）

```xml
<bean id="customer" class="com.powernode.spring.pojo.Customer" scope="singleton">
</bean>
```

```java
// bean对象会在new完applicationContext对象后创建
@Test
public void testScope(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-scope.xml");
    Customer customer1 = applicationContext.getBean("customer", Customer.class);
    Customer customer2 = applicationContext.getBean("customer", Customer.class);
    System.out.println(customer1);
    System.out.println(customer2);
}
```

![image-20240822165721242](.\img\image-20240822165721242.png)

### （2）多例模式

```xml
<bean id="customer" class="com.powernode.spring.pojo.Customer" scope="prototype">
</bean>
```

```java
// bean对象会在调用applicationContext.getBean后创建
@Test
public void testScope(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-scope.xml");
    Customer customer1 = applicationContext.getBean("customer", Customer.class);
    System.out.println(customer1);
    Customer customer2 = applicationContext.getBean("customer", Customer.class);
    System.out.println(customer2);
}
```

![image-20240822165951825](.\img\image-20240822165951825.png)



## 五、GoF之工厂模式

### （1）工厂模式的三种形态

​	1、简单工厂模式（Simple Factory）：不属于23种设计模式之一。又称为静态工厂方法模式。简单工厂模式是工厂方法模式的一种特殊实现。

​	2、工厂方法模式（Factory Method）：23种设计模式之一。

​	3、抽象工厂模式（Abstract Factory）：23种设计模式之一。



### （2）简单工厂模式

​	简单工厂模式的角色：

​		1、抽象产品 角色

​		2、具体产品 角色

​		3、工厂类 角色



​	抽象产品角色：

```java
public abstract class Weapon {
    public abstract void attack();
}

public class Dagger extends Weapon{
    @Override
    public void attack() {
        System.out.println("dagger");
    }
}

public class Fighter extends Weapon{
    @Override
    public void attack() {
        System.out.println("fighter");
    }
}

public class Tank extends Weapon{
    @Override
    public void attack() {
        System.out.println("tank");
    }
}

public class WeaponFactory{
    public static Weapon get(String weaponType){
        if (weaponType == null || weaponType.trim().length() == 0){
            return null;
        }
        Weapon weapon = null;
        if ("TANK".equals(weaponType)) {
            weapon = new Tank();
        } else if ("FIGHTER".equals(weaponType)) {
            weapon = new Fighter();
        } else if ("DAGGER".equals(weaponType)) {
            weapon = new Dagger();
        } else {
            throw new RuntimeException("不支持该武器！");
        }
        return weapon;
    }
}

public static void main(String[] args) {
    Weapon weapon1 = WeaponFactory.get("TANK");
    weapon1.attack();

    Weapon weapon2 = WeaponFactory.get("FIGHTER");
    weapon2.attack();

    Weapon weapon3 = WeaponFactory.get("DAGGER");
    weapon3.attack();
}
```

​	优点：

​	客户端不需要关心对象的创建细节，需要哪个，只需向工厂索要。

​	缺点：

​	1、工厂中集中了所有产品的创造逻辑，形成了一个无所不知的全能类，一旦出现问题，整个系统就会出现瘫痪。

​	2、不符合OCP开闭原则，进行扩展时需要修改工厂类。



### （3）工厂方法模式

工厂方法模式既保留了简单工厂模式的优点，同时又解决了简单工厂模式的缺点。

工厂方法模式的角色包括：

- **抽象工厂角色**
- **具体工厂角色**
- 抽象产品角色
- 具体产品角色

```java
public abstract class Weapon {
    public abstract void attack();
}
// ---------------------------------------//
public class Dagger extends Weapon{
    @Override
    public void attack() {
        System.out.println("dagger");
    }
}
// .....
// ----------------------------------------//
public class DaggerFactory implements WeaponFactory{
    @Override
    public Weapon get() {
        return new Fighter();
    }
}
// ......
// ---------------------------------------//
public interface WeaponFactory {
    Weapon get();
}

// ---------------------------------------//

public static void main(String[] args) {
    WeaponFactory factory = new DaggerFactory();
    Weapon weapon = factory.get();
    weapon.attack();
}
```





## 六、Bean的实例化方式

### （1）构造方法实例化

​	略

### （2）通过简单工厂模式实例化

```java
public class VipFactory {
    public static Customer get(){
        return new Customer();
    }
}

public class Customer {
    public Customer() {
        System.out.println("构造方法执行");
    }
}
```

```xml
<bean id="customer" class="com.powernode.spring.bean.VipFactory" factory-method="get"/>

```





## 七、Bean的生命周期

### （1）Bean的生命周期之5步

![image-20240824134407752](.\img\image-20240824134407752.png)

```java
public class User {
    private String name;

    public User() {
        System.out.println("第一步、实例化Bean");
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        System.out.println("第二步、Bean属性赋值");
        this.name = name;
    }

    public void initBean(){
        System.out.println("第三步、初始化Bean");
    }

    public void destroyBean(){
        System.out.println("第五步、销毁Bean");
    }
}

```

```xml
<!--
	init-method属性指定初始化方法
	destroy-method属性指定销毁方法
-->
<bean id="user"
      class="com.powernode.spring6.bean.User"
      init-method="initBean"
      destroy-method="destroyBean">
    <property name="name" value="zhangsan"/>
</bean>
```

```java
@Test
public void testBeanFifeCycleBy5(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    User user = applicationContext.getBean("user", User.class);
    System.out.println("第四步、使用Bean");
    // 需要手动关闭后才能执行第五步
    ClassPathXmlApplicationContext context = (ClassPathXmlApplicationContext) applicationContext;
    context.close();
}
```



### （2）Bean生命周期之7步

在以上的5步中，第三步的前后添加代码，加入Bean处理器

```java
public class LogBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("第三步、Bean后处理的before方法执行，即将开始初始化");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("第五步、Bean后处理的after方法执行，完成初始化");
        return bean;
    }
}

```

```xml
<!--配置Bean后处理器。这个后处理器将作用于当前配置文件中所有的bean。-->
<bean class="com.powernode.spring6.bean.LogBeanPostProcessor"/>
```

![image-20240824135418259](.\img\image-20240824135418259.png)



### （3）Bean生命周期之10步

![image-20240824135538859](.\img\image-20240824135538859.png)

Aware相关的接口包括：BeanNameAware、BeanClassLoaderAware、BeanFactoryAware

​	当Bean实现了BeanNameAware，Spring会将Bean的名字传递给Bean。

​	当Bean实现了BeanClassLoaderAware，Spring会将加载该Bean的类加载器传递给Bean

​	当Bean实现了BeanFactoryAware，Spring会将Bean工厂对象传递给Bean

```java
public class User implements BeanNameAware, BeanClassLoaderAware, BeanFactoryAware, InitializingBean , DisposableBean {
    private String name;

    public User() {
        System.out.println("第一步、实例化Bean");
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        System.out.println("第二步、Bean属性赋值");
        this.name = name;
    }

    public void initBean(){
        System.out.println("第六步、初始化Bean");
    }

    public void destroyBean(){
        System.out.println("第十步、销毁Bean");
    }

    @Override
    public void setBeanClassLoader(ClassLoader classLoader) {
        System.out.println("第三步、类加载器：" + classLoader);
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("第三步、Bean工厂：" + beanFactory);
    }

    @Override
    public void setBeanName(String name) {
        System.out.println("第三步、Bean名字：" + name);
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("第五步、afterPropertiesSet执行");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("第九步、DisposableBean destroy");
    }
}


public class BeanLifeCycleTest {
    @Test
    public void testBeanFifeCycleBy5(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
        User user = applicationContext.getBean("user", User.class);
        System.out.println("第四步、使用Bean");
        ClassPathXmlApplicationContext context = (ClassPathXmlApplicationContext) applicationContext;
        context.close();
    }

    @Test
    public void testBeanFifeCycleBy7(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
        User user = applicationContext.getBean("user", User.class);
        System.out.println("第六步、使用Bean");
        ClassPathXmlApplicationContext context = (ClassPathXmlApplicationContext) applicationContext;
        context.close();
    }

    @Test
    public void testBeanFifeCycleBy10(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
        User user = applicationContext.getBean("user", User.class);
        System.out.println("第八步、使用Bean");
        ClassPathXmlApplicationContext context = (ClassPathXmlApplicationContext) applicationContext;
        context.close();
    }
}


@Test
public void testBeanFifeCycleBy10(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    User user = applicationContext.getBean("user", User.class);
    System.out.println("第八步、使用Bean");
    ClassPathXmlApplicationContext context = (ClassPathXmlApplicationContext) applicationContext;
    context.close();
}

```

```xml
<bean class="com.powernode.spring6.bean.LogBeanPostProcessor"/>
<bean id="user"
      class="com.powernode.spring6.bean.User"
      init-method="initBean"
      destroy-method="destroyBean">
    <property name="name" value="zhangsan"/>
</bean>
```



### （4）Bean的作用域不同，作用方式不同

```xml
<!-- 多例模式 -->
<bean id="userBean" class="com.powernode.spring6.bean.User" init-method="initBean" destroy-method="destroyBean" scope="prototype">
    <property name="name" value="zhangsan"/>
</bean>

<!--配置Bean后处理器。这个后处理器将作用于当前配置文件中所有的bean。-->
<bean class="com.powernode.spring6.bean.LogBeanPostProcessor"/>
```

![image-20240824162011286](.\img\image-20240824162011286.png)

### （5）自己new的对象让spring管理

```java
@Test
public void testBeanRegister(){
    User user = new User();
    System.out.println(user);

    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    factory.registerSingleton("userBean", user);
    User userBean = factory.getBean("userBean", User.class);
    System.out.println(userBean);
}
```

![image-20240824162416069](.\img\image-20240824162416069.png)



## 八、循环依赖

![image-20240824162504304](.\img\image-20240824162504304.png)

A对象中有B属性，B对象中有A属性。



### （1）singleton下的set注入

```java
public class Husband {
    private String name;
    private Wife wife;

    @Override
    public String toString() {
        return "Husband{" +
                "name='" + name + '\'' +
                ", wife=" + wife.getName() +
                '}';
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Wife getWife() {
        return wife;
    }

    public void setWife(Wife wife) {
        this.wife = wife;
    }
}


public class Wife {
    private String name;
    private Husband husband;

    @Override
    public String toString() {
        return "Wife{" +
                "name='" + name + '\'' +
                ", husband=" + husband.getName() +
                '}';
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Husband getHusband() {
        return husband;
    }

    public void setHusband(Husband husband) {
        this.husband = husband;
    }
}
```

```xml
<bean id="husband" class="com.powernode.spring6.bean.Husband" scope="singleton">
    <property name="name" value="zhangsan"/>
    <property name="wife" ref="wife"/>
</bean>
<bean id="wife" class="com.powernode.spring6.bean.Wife" scope="singleton">
    <property name="name" value="liu"/>
    <property name="husband" ref="husband"/>
</bean>
```

```java
@Test
public void testCircularDependency(){
    ApplicationContext context = new ClassPathXmlApplicationContext("spring-circular-dependencies.xml");
    Husband husband = context.getBean("husband", Husband.class);
    Wife wife = context.getBean("wife", Wife.class);
    System.out.println(husband);
    System.out.println(wife);
}
```

![image-20240824163941809](.\img\image-20240824163941809.png)

**在singleton + set注入的情况下，循环依赖是没有问题的。Spring可以解决这个问题**



### （2）prototype下的set注入

```xml
<bean id="husband" class="com.powernode.spring6.bean.Husband" scope="prototype">
    <property name="name" value="zhangsan"/>
    <property name="wife" ref="wife"/>
</bean>
<bean id="wife" class="com.powernode.spring6.bean.Wife" scope="prototype">
    <property name="name" value="liu"/>
    <property name="husband" ref="husband"/>
</bean>
```

错误信息：Error creating bean with name 'husband' defined in class path resource [spring-circular-dependencies.xml]: Cannot resolve reference to bean 'wife' while setting bean property 'wife'

### （3）singleton下的构造注入

```xml
<bean id="husband" class="com.powernode.spring6.bean.Husband" scope="singleton">
    <constructor-arg name="name" value="zhangsan"/>
    <constructor-arg name="wife" ref="wife"/>
</bean>
<bean id="wife" class="com.powernode.spring6.bean.Wife" scope="singleton">
    <constructor-arg name="name" value="liu"/>
    <constructor-arg name="husband" ref="husband"/>
</bean>
```

错误信息：Cannot resolve reference to bean 'wife' while setting constructor argument

原因：通过构造方法注入会导致实例化对象过程和对象属性赋值过程没有分开。



### （4）解决循环依赖机理

根本原因在于：这种方式可以做到将“实例化Bean”和“给Bean属性赋值”这两个动作分开去完成。

实例化Bean的时候：调用无参构造方法完成，**此时可以先不给属性赋值，可以提前将Bean对象“曝光”给外界**

给Bean属性赋值的时候：调用setter方法完成。

Bean都是单例的，我们可以先把所有的单例Bean实例化出来，放到一个集合中（称之为缓存），所有单例实例化完成后，再用setter方法赋值。

三级缓存见语雀https://www.yuque.com/dujubin/ltckqu/kipzgd?singleDoc#wt8oL。





## 九、Spring IoC注解开发

### （1）注解使用







## 十、GoF之代理模式







## 十一、面向切面编程AOP

### （1）AOP介绍

​	一般系统中会有一些系统服务，例如：日志、事务管理、安全等，这些业务被称为：**交叉业务**。

​	如果在每一个业务处理过程当中，都将这些交叉业务代码参杂进去，那么存在两方面问题：

​	1、交叉业务代码在业务流程中反复出现，不能实现复用。

​	2、程序员无法专注业务代码的编写，在编写业务代码的同时还要处理这些交叉业务。

![image-20240826141438442](.\img\image-20240826141438442.png)



### （2）AOP的七大术语

​	**1、连接点Jointpoint**

​		在程序的整个执行流程中，**可以织入**切面的位置。方法的执行前后，异常抛出之后等位置。

​	**2、切点Pointcut**

​		在程序执行流程中，**真正织入**切面的方法。

​	**3、通知Advice**

​		通知又叫增强，就是具体要你织入的代码。

​		通知包括：

​			前置通知

​			后置通知

​			环绕通知

​			异常通知

​			最终通知

​	**4、切面Aspect**

​		**切点 + 通知就是切面**

​	5、织入Weaving

​		把通知应用到目标对象上的过程

​	6、代理对象Proxy

​		一个目标对象被织入通知后产生的新对象

​	7、目标对象Target

​		被织入的通知对象

![image-20240826142208124](.\img\image-20240826142208124.png)

```java
public class UserService{
    public void do1(){
        System.out.println("do 1");
    }
    public void do2(){
        System.out.println("do 2");
    }
    public void do3(){
        System.out.println("do 3");
    }
    public void do4(){
        System.out.println("do 4");
    }
    public void do5(){
        System.out.println("do 5");
    }
    // 核心业务方法
    public void service(){
        try{
            // Jointpoint连接点
            do1(); // Pointcut连接点
            // Jointpoint连接点
            do2(); // Pointcut连接点
            // Jointpoint连接点
            do3(); // Pointcut连接点
            // Jointpoint连接点
            do5(); // Pointcut连接点
            // Jointpoint连接点
        }catch(){
            // Jointpoint连接点
        }finally{
            // Jointpoint连接点
        }
        
    }
}

// 1、连接点（Jointpoint）描述的是位置
// 2、切点（Pointcut）本质上就是方法（真正织入切面的那个方法叫做切点）
// 3、通知（Advice），通知又叫增强，就是具体增强的那个代码
// 		例如，具体的事务代码，日志代码等等
// 		具体的代码就是通知
// 4、切面：切点 + 通知
```



### （3）切点表达式



### （4）使用Spring的AOP

#### 	1、准备工作

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>6.1.6</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>6.1.6</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>6.1.6</version>
</dependency>
```

```xml
<!-- spring配置文件中添加context命名空间和aop命名空间 -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

</beans>
```

#### 	2、基于Aspect的AOP注解开发

```java
@Service
public class OrderService {
    public void generate(){
        System.out.println("订单生成");
    }
}
/**
	前置通知：@Before
	后置通知：@AfterReturning
	环绕通知：@Around目标方法之前和之后添加通知
	异常通知：@AfterThrowing发生异常之后执行通知
	最终通知：@After 放在finally语句中
*/
@Component
@Aspect
public class MyAspect {
    @Before("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void beforeAdvice(){
        System.out.println("前置通知");
    }

    @Around("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void aroundAdvice(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("环绕通知开始");
        // 执行目标方法
        proceedingJoinPoint.proceed();
        System.out.println("环绕通知结束");
    }

    @AfterReturning("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void afterReturningAdvice(){
        System.out.println("后置通知");
    }

    @AfterThrowing("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void afterThrowingAdvice(){
        System.out.println("异常通知");
    }

    @After("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void afterAdvice(){
        System.out.println("最终通知");
    }
}

```

```xml
<context:component-scan base-package="com.powernode.spring6.service"/>
<!-- 开启aspectj的自动代理 -->
<!--
	true：使用CGLIB动态代理
	false：使用JDK动态代理（默认）
-->
<aop:aspectj-autoproxy proxy-target-class="true"/>
```

![image-20240826153510498](.\img\image-20240826153510498.png)

出现异常之后，**后置通知**和**环绕通知的结束部分**不会执行



#### 3、切面的先后顺序

使用**@Order**注解调整优先级，数字越小，优先级越高

```java
@Aspect
@Component
@Order(1) //设置优先级
public class YourAspect{
	...
}

@Aspect
@Component
@Order(2) //设置优先级
public class MyAspect{
	...
}

```

![image-20240826153944932](.\img\image-20240826153944932.png)

互换优先级

![image-20240826154011436](.\img\image-20240826154011436.png)

#### 4、优化使用切点表达式

​	切点表达式重复写了很多次，没有得到复用；如果要修改切点表达式，需要修改多处。

```java
// 切面类
@Component
@Aspect
@Order(2)
public class MyAspect {
    
    @Pointcut("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void pointcut(){}

    @Around("pointcut()")
    public void aroundAdvice(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("环绕通知开始");
        // 执行目标方法。
        proceedingJoinPoint.proceed();
        System.out.println("环绕通知结束");
    }

    @Before("pointcut()")
    public void beforeAdvice(){
        System.out.println("前置通知");
    }

    @AfterReturning("pointcut()")
    public void afterReturningAdvice(){
        System.out.println("后置通知");
    }

    @AfterThrowing("pointcut()")
    public void afterThrowingAdvice(){
        System.out.println("异常通知");
    }

    @After("pointcut()")
    public void afterAdvice(){
        System.out.println("最终通知");
    }

}

```



#### 5、全注解开发AOP

```java
@Configuration
@ComponentScan("com.powernode.spring6.service")
@EnableAspectJAutoProxy(proxyTargetClass = true)
public class SpringConfiguration {
}


@Test
public void testAOPByAnnotation(){
    ApplicationContext applicationContext = new AnnotationConfigApplicationContext(SpringConfiguration.class);
    OrderService orderService = applicationContext.getBean("orderService", OrderService.class);
    orderService.generate();
}
```



### （5）AOP事务处理

```java
@Aspect
@Component
public class TransactionAspect {
    @Around("execution(* com.powernode.spring6.service..*(..))")
    public void aroundAspect(ProceedingJoinPoint proceedingJoinPoint){
        try {
            System.out.println("开启事务");
            proceedingJoinPoint.proceed();
            System.out.println("关闭事务");
        } catch (Throwable e) {
            System.out.println("回滚");
        }
    }
}

@Component
public class AccountService {
    public void transfer(){
        System.out.println("正在进行银行账户转账");
    }

    public void withdraw(){
        System.out.println("正在进行取款操作");
    }
}

@Component
public class OrderService {
    public void generate(){
        System.out.println("订单生成");
    }
    public void cancel(){
        System.out.println("取消订单");
    }
}
```



### （6）AOP安全日志

```java
@Component
//用户业务
public class UserService {
    public void getUser(){
        System.out.println("获取用户信息");
    }
    public void saveUser(){
        System.out.println("保存用户");
    }
    public void deleteUser(){
        System.out.println("删除用户");
    }
    public void modifyUser(){
        System.out.println("修改用户");
    }
}

@Component
public class AccountService {
    public void transfer(){
        System.out.println("正在进行银行账户转账");
    }

    public void withdraw(){
        System.out.println("正在进行取款操作");
    }
}

@Aspect
@Component
public class SecurityAspect {
    @Pointcut("execution(* com.powernode.spring6.service..save*(..))")
    public void savePointCut(){}

    @Pointcut("execution(* com.powernode.spring6.service..delete*(..))")
    public void deletePointcut(){}

    @Pointcut("execution(* com.powernode.spring6.service..modify*(..))")
    public void modifyPointcut(){}

    @Before("savePointCut() || deletePointcut() || modifyPointcut()")
    public void beforeAdvice(JoinPoint joinPoint){
        System.out.println("XXX操作员正在操作" + joinPoint.getSignature().getName() + "方法");
    }
}


```



## 十二、Spring对事务的支持

### （1）事务概述

​	事务的四个处理过程：

​		第一步：开启事务

​		第二步：执行核心业务代码

​		第三步：提交事务

​		第四步：回滚事务

​	事务的四个特性：

​		A原子性：事务最小的工作单元，不可再分

​		B一致性：事务要求要么同时成功，要么同时失败

​		I隔离性：事务和事务之间因为有隔离性，才可以保证互不干扰

​		D持久性：持久性是事务结束的标志

### （2）



## 十三、Spring6整合Junit5

### （1）spring对junit4的支持

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>6.1.6</version>
</dependency>
```

```java
@RunWith(SpringJUnit4ClassRunner.class)
// 若为配置类，只需修改classpath为class
@ContextConfiguration("classpath:spring.xml")
public class SpringJUnit4Test {

    @Autowired
    private User user;

    @Test
    public void testUser(){
        System.out.println(user.getName());
    }
}
```

### （2）spring对junit5的支持

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.9.0</version>
    <scope>test</scope>
</dependency
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>6.1.6</version>
</dependency>
```

```java
@ExtendWith(SpringExtension.class)
// 若为配置类，只需修改classpath为class
@ContextConfiguration("classpath:spring.xml")
public class SpringJUnit5Test {

    @Autowired
    private User user;

    @Test
    public void testUser(){
        System.out.println(user.getName());
    }
}
```

