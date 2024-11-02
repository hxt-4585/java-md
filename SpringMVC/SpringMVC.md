# [SpringMVC ](https://www.yuque.com/dujubin/java/myxi54xu063hgsl4)



## 二、RequestMapping

**@RequestMapping**  用于将请求映射到相应处理方法上，可以出现在方法和类上。

### （一）value属性

```java
// value属性填写的是请求路径，通过请求路径将对应的控制器方法绑定在一起
/*
	@AliasFor("value")
    String[] path() default {};
    可以提供多个路径
*/
@RequestMapping(value = {"/testValue1", "/testValue2"})
```

### （二）Ant风格的value

value支持模糊匹配，通配符包括：

​	1、?，代表任意一个字符

​	2、*，代表0到N个任意字符

​	3、* *，代表0到N个任意字符，且路径中可以出现路径分隔符，注意：/x**z/ 实际上并没有使用通配符 \**，本质上还是使用的 *，因为通配符 ** 在使用的时候，左右两边都不能有任何字符，必须是 /，如 / ** /testValueAnt，以上写法在Spring5的时候是支持的，但是在Spring6中进行了严格的规定，** 通配符只能出现在路径的末尾，如 /testValueAnt/**



### （三）value中的占位符

普通的请求路径：localhost:8080/springmvc/login?username=admin&password=123&age=20

RESTful风格的请求路径：localhost:8080/springmvc/login/admin/123/20

如果使用RESTful风格的请求路径，在控制器中应该如何获取请求中的数据呢？可以在value属性中使用占位符，例如：/login/{id}/{username}/{password}

```java
@RequestMapping("/testRESTful/{id}/{username}/{age}")
public String testRESTful(
        @PathVariable("id")int id,
        @PathVariable("username")String username,
        @PathVariable("age")int age
){
    System.out.println(id + "," + username + "," + age);
    return "testRESTful";    }
```



### （四）method属性

```java
public enum RequestMethod {
    GET,
    HEAD,
    POST,
    PUT,
    PATCH,
    DELETE,
    OPTIONS,
    TRACE;
    
    ...
    
}
```

```java
// @PostMapping("login")
@RequestMapping(value="/login", method = RequestMethod.POST)
public String testMethod(){
    return "testMethod";
}
```



### （五）web的请求方式

前端向服务器发送请求的方式包括哪些？共9种，前5种常用，后面作为了解：

​	1、GET：获取资源，只允许读取数据，不影响数据的状态和功能。使用URL中传递的参数或者在HTTP请求的头部中使用参数，服务器返回请求的资源。

​	2、POST：向服务器提交资源，可能会改变数据的状态和功能

​	3、PUT：更新资源，用于更新指定的资源上所有可编辑内容。通过请求体发送需要被更新的全部内容，服务器接收数据后，将被更新的资源进行替换或修改。

​	4、DELETE：删除资源，用于删除指定的资源。将要被删除的资源标识符放在 URL 中或请求体中。

​	5、HEAD：请求服务器返回资源的头部，与 GET 命令类似，但是所有返回的信息都是头部信息，不能包含数据体。主要用于资源检测和缓存控制。

​	6、PATCH：部分更改请求。当被请求的资源是可被更改的资源时，请求服务器对该资源进行部分更新，即每次更新一部分。

​	7、OPTIONS：请求获得服务器支持的请求方法类型，以及支持的请求头标志。“OPTIONS *”则返回支持全部方法类型的服务器标志。

​	8、TRACE：服务器响应输出客户端的 HTTP 请求，主要用于调试和测试。

​	9、CONNECT：建立网络连接，通常用于加密 SSL/TLS 连接。

注意：
1. 使用超链接以及原生的form表单只能提交get和post请求，put、delete、head请求可以使用发送ajax请求的方式来实现。
2. 使用超链接发送的是get请求
3. 使用form表单，如果没有设置method，发送get请求
4. 使用form表单，设置method="get"，发送get请求
5. 使用form表单，设置method="post"，发送post请求
6. 使用form表单，设置method="put/delete/head"，仍然发送get请求。意味着后端接受需要设置为 method = GET

#### （六）GET和POST的区别

​	1、get发生请求的时候，数据挂在URI后面，post发生请求，是在请求体中发送。

​	2、get请求只能发生普通的字符串，且长度有限制，长度要看使用什么浏览器；post请求可以发送任何类型的数据包括字符串，流媒体等信息：视频、声音、图片。

​	3、get适合从服务端获取数据；post适合向服务端提交数据。

​	4、get请求是安全的，不会对服务器上的数据进行修改；post是危险的，会修改服务端的资源。

​	5、get请求支持缓存，第二次发送get请求时，会走浏览器上次的缓存结果，不再是正真的服务器（可以添加一个时间戳避免）；post不支持缓存。



#### （七）params属性

```java
// 请求参数中必须包含username、password
@RequestMapping(value="/login", params={"username", "passward"})

// 请求参数中不包含username，且包含password
@RequestMapping(value="/login", params={"!username", "passward"})

// 请求参数中必须包含username，值等于admin，也包含password
@RequestMapping(value="/login", params={"username=admin", "password"}) 

// 请求参数中必须包含username参数，但参数的值不能是admin，另外也必须包含password参数
@RequestMapping(value="/login", params={"username!=admin", "password"})

// 注意：如果前端提交的参数，和后端要求的请求参数不一致，则出现400错误！！！
// HTTP状态码400的原因：请求参数格式不正确而导致的。
```





#### （八）headers属性

```java
// 表示：请求头信息中必须包含Referer和Host，才能与当前标注的方法进行映射。
@RequestMapping(value="/login", headers={"Referer", "Host"}) 

// 表示：请求头信息中必须包含Referer，但不包含Host，才能与当前标注的方法进行映射。
@RequestMapping(value="/login", headers={"Referer", "!Host"}) 

// 表示：请求头信息中必须包含Referer和Host，并且Referer的值必须是http://localhost:8080/springmvc/，才能与当前标注的方法进行映射。
@RequestMapping(value="/login", headers={"Referer=http://localhost:8080/springmvc/", "Host"}) 

// 表示：请求头信息中必须包含Referer和Host，并且Referer的值不是http://localhost:8080/springmvc/，才能与当前标注的方法进行映射。
@RequestMapping(value="/login", headers={"Referer!=http://localhost:8080/springmvc/", "Host"}) 

// 注意：如果前端提交的请求头信息，和后端要求的请求头信息不一致，则出现404错误！！！
```





## 三、获取请求数据

### （一）使用原生Servlet API获取

```

```



### （二）RequestParam注解标注

RequestParam注解和PathVariable注解：

PathVariable用于restful风格中。



#### 1、RequestRaram的基本使用

```java
// @RequestParam不能省
@PostMapping("/register")
public String register(
        @RequestParam("username")String username,
        @RequestParam("password")String password,
        @RequestParam("sex")String sex,
        @RequestParam("hobby")String[] hobby,
        @RequestParam("intro")String intro
){
    System.out.println(username);
    System.out.println(password);
    System.out.println(sex);
    System.out.println(Arrays.toString(hobby));
    System.out.println(intro);
    return "success";
}
```



#### 2、RequestParam注解的required属性

​	required属性用来设置该方法参数是否为必须的。默认为true，前端界面不能为空，但可以为“”，即空字符串。

```java
@RequestParam(value = "intro", required = false)String intro
```

![image-20240920150310748](.\img\image-20240920150310748.png)



#### 3、defaultValue属性

```java
// defaultValue用于设置形参的默认值，当没有提供请求参数（空）或者该请求的参数是空字符串
// 特殊例子，数组
@RequestParam(value = "hobby", defaultValue = "smoke,drink")String[] hobby
```



### （三）依靠控制器方法上的形参名来接收

@RequestParam是可以省略的，如果方法形参的名字和提交数据时的名字相同，可以省略。

如果是spring5版本，则不需要添加配置，但如果是spring6+版本，需要在pom.xml中添加：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.12.1</version>
            <configuration>
                <source>21</source>
                <target>21</target>
                <compilerArgs>
                    <arg>-parameters</arg>
                </compilerArgs>
            </configuration>
        </plugin>
    </plugins>
</build>
```





### （四）RequestHeader注解

将请求头信息映射到方法的形参上，同样是有三个属性：value、required、defaultValue

```java
@PostMapping("/registerByObject")
public String registerByObject(User user, @RequestHeader("accept")String accept){
    System.out.println(user);
    System.out.println(accept);
    return "success";
}
```

请求头参数如下（大小写可以不在意）：

![image-20240920201257045](.\img\image-20240920201257045.png)



### （五）CookieValue注解

将请求提交的Cookie数据映射到方法形参上，有三个属性：value、required、defaultValue

```html
<script type="text/javascript">
    function sendCookie(){
        document.cookie = "id=123456789; expires=Thu, 18 Dec 2025 12:00:00 UTC; path=/";
        document.location = "/springmvc/register";
    }
</script>
<button onclick="sendCookie()">向服务器端发送Cookie</button>
```

```java
@GetMapping("/register")
public String register(User user,
                       @RequestHeader(value="Referer", required = false, defaultValue = "")
                       String referer,
                       @CookieValue(value="id", required = false, defaultValue = "2222222222")
                       String id){
    System.out.println(user);
    System.out.println(referer);
    System.out.println(id);
    return "success";
}
```



## 四、三个域对象

### （一）Servlet中的三个域对象

#### 1、request

接口名：HttpServletRequest

request对象代表一次请求，一次请求一个request。

使用请求域的业务场景：在A资源中通过转发的方式跳转到B资源，因为是转发，因此从A到B是一次请求，如果想让A资源和B资源共享同一个数据，可以将数据存储到request域中。



#### 2、session

接口名：HttpSession

session对象代表了一次会话。从打开浏览器开始访问，到最终浏览器关闭，这是一次完整的会话。每个会话session对象都对应一个JSESSIONID，而JSESSIONID生成后以cookie的方式存储在浏览器客户端。浏览器关闭，JSESSIONID失效，会话结束。

使用会话域的业务场景：

1. 在A资源中通过重定向的方式跳转到B资源，因为是重定向，因此从A到B是两次请求，如果想让A资源和B资源共享同一个数据，可以将数据存储到session域中。
2. 登录成功后保存用户的登录状态。



#### 3、application

接口名：ServletContext

application对象代表了整个web应用，服务器启动时创建，服务器关闭时销毁。对于一个web应用来说，application对象只有一个。

使用应用域的业务场景：记录网站的在线人数。



### （二）request域对象

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
  <meta charset="UTF-8">
  <title>index</title>
</head>
<body>
<h1>Index Page</h1>
<a th:href="@{/testServletAPI}">在SpringMVC中使用原生Servlet API实现request域数据共享</a><br>
<a th:href="@{/testModel}">在SpringMVC中使用model实现request域数据共享</a><br>
<a th:href="@{/testMap}">在SpringMVC中使用map实现request域数据共享</a><br>
<a th:href="@{/testModelMap}">在SpringMVC中使用modelMap实现request域数据共享</a><br>
<a th:href="@{/testModelAndView}">在SpringMVC中使用modelAndView实现request域数据共享</a><br>
<a th:href="@{/testSessionScope}">在SpringMVC中使用原生Servlet API实现session域数据共享</a><br>
<a th:href="@{/testSessionScopeByAnno}">在SpringMVC中使用原生Servlet API实现session域数据共享</a><br>
<a th:href="@{/testApplicationScope}">在SpringMVC中使用ServletAPI实现application域共享数据</a><br>
</body>
</html>
```

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
  <meta charset="UTF-8">
  <title>view</title>
</head>
<body>
<div th:text="${testRequestScope}"></div>
<div th:text="${session.testSessionScope}"></div>
<div th:text="${session.x}"></div>
<div th:text="${session.y}"></div>
<div th:text="${application.applicationScope}"></div>
</body>
</html>
```



#### 1、使用原生Servlet API方式

```java
@RequestMapping("/testServletAPI")
public String testServletAPI(HttpServletRequest request){
    request.setAttribute("testRequestScope", "在SpringMVC中使用原生Servlet API实现request域数据共享");
    return "view";
}
```



#### 2、使用Model接口

```java
@RequestMapping("/testModel")
public String testModel(Model model){
    // 向request域中存储数据
    model.addAttribute("testRequestScope", "在SpringMVC中使用Model接口实现request域数据共享");
    System.out.println(model.getClass().getName());
    return "view";
}
```





#### 3、使用Map接口

```java
@RequestMapping("/testMap")
public String testMap(Map<String, Object> map){
    // 向request域中存储数据
    map.put("testRequestScope", "在SpringMVC中使用Map接口实现request域数据共享");
    System.out.println(map.getClass().getName());
    return "view";
}
```





#### 4、使用ModelMap接口

```java
@RequestMapping("/testModelMap")
public String testModelMap(ModelMap modelMap){
    // 向request域中存储数据
    modelMap.addAttribute("testRequestScope", "在SpringMVC中使用ModelMap实现request域数据共享");
    System.out.println(modelMap.getClass().getName());
    return "view";
}
```

##### Model、Map、ModelMap的关系

![image-20240921145241592](.\img\image-20240921145241592.png)

![image-20240921145328730](.\img\image-20240921145328730.png)

三个类均继承自BindingAwareModelMap类，也就是底层一样。



#### 5、使用ModelAndView类

ModelAndView这个类实际上封装了Model和View，也就是说这个类即封装了处理之后的数据，也体现了跳转到哪一个页面

```java
@RequestMapping("/testModelAndView")
public ModelAndView testModelAndView(){
    ModelAndView modelAndView = new ModelAndView();
    modelAndView.setViewName("view");
    modelAndView.addObject("testRequestScope", "在SpringMVC中使用ModelAndView实现request域数据共享");
    return modelAndView;
}
```

##### ModelAndView源码解析

Model、Map、ModelMap、ModelAndView四种的底层DispatcherServlet调用我们的Controller之后，返回的对象都是ModelAndView



### （三）session域对象

#### 1、原生Servlet API

```java
@RequestMapping("/testSessionScope")
public String testSessionScope(HttpSession session){
    session.setAttribute("testSessionScope", "使用原生Servlet API实现session域共享数据");
    return "view";
}
```





#### 2、使用注解方式

```java
@Controller
@SessionAttributes(value = {"x", "y"})
public class SessionScopeTestController {
    @RequestMapping("/testSessionScopeByAnno")
    public String testSessionScopeByAnno(ModelMap modelMap){
        modelMap.addAttribute("x","I am x");
        modelMap.addAttribute("y","I am y");
        return "view";
    }
}
```

注意：SessionAttributes注解使用在Controller类上。标注了当key是 x 或者 y 时，数据将被存储到会话session中。如果没有 SessionAttributes注解，默认存储到request域中。



### （四）application域对象

```java
@Controller
public class ApplicationScopeTestController {

    @RequestMapping("/testApplicationScope")
    public String testApplicationScope(HttpServletRequest request){
        
        // 获取ServletContext对象
        ServletContext application = request.getServletContext();

        // 向应用域中存储数据
        application.setAttribute("applicationScope", "我是应用域当中的一条数据");

        return "view";
    }
}

```



## 五、视图view

### （一）Spring MVC视图实现原理

#### 1、Spring MVC支持的常见视图

​	（1）InternalResourceView：内部资源视图（springmvc框架内置的，专门为JSP模板语法准备）

​	（2）RedirectView：重定向视图（springmvc框架内置的，用来完成重定向效果）

​	（3）ThymeleafView：Thymeleaf视图（第三方，为Thymeleaf模板语法准备的）

​	（4）FreeMarkerView：FreeMarker视图（第三方的，为`FreeMarker模板语法`准备的）

​	（5）VelocityView：Velocity视图（第三方的，为`Velocity模板语法`准备的）

​	（6）PDFView：PDF视图（第三方的，专门用来生成pdf文件视图）

​	（7）ExcelView：Excel视图（第三方的，专门用来生成excel文件视图

​	......



#### 2、实现视图机制核心类

​	1、DispatcherServlet，前端控制器

​		核心方法：doDispatch

​		负责接收前端的请求

​		根据请求路径找到对应的处理方法（controller）

​		执行处理方法

​		最终返回ModelAndView对象

```java
public class DispatcherServlet extends FrameworkServlet {
        protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        // 根据请求路径映射处理器的方法，处理器方法执行结束之后，返回逻辑视图名称
        // 返回逻辑视图名称之后，DispatcherServlet会将视图名称ViewName和model封装为ModelAndView对象
        mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
        // ...
        // 处理视图
        this.processDispatchResult(processedRequest, response, mappedHandler, mv, (Exception)dispatchException);
    }

    private void processDispatchResult(HttpServletRequest request, HttpServletResponse response, @Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv, @Nullable Exception exception) throws Exception {
        //...
        // 将模板字符串转化为html代码
        this.render(mv, request, response);
        //...
    }

    protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
        //...
        // 将逻辑视图名称转换为物理视图名称，并且最终返回视图对象View
        view = this.resolveViewName(viewName, mv.getModelInternal(), locale, request);
    }

    protected View resolveViewName(String viewName, @Nullable Map<String, Object> model, Locale locale, HttpServletRequest request) throws Exception {

        // ...
        // 这行代码才是起真正的作用，将逻辑视图转换为物理视图，并且最终返回视图对象View
        View view = viewResolver.resolveViewName(viewName, locale);
    }
    
}


```

​	2、ViewResolver接口，视图解析器接口（ThymeleafViewResolver实现了ViewResolver）

​		将逻辑视图名称 转换为 物理视图名称

​		并且最终返回一个View接口对象

```java
public interface ViewResolver {// 如果使用的是Thymeleaf，那么接口的实现类是：ThmeleafViewResolver
    @Nullable
    // 将逻辑视图转换为物理视图，并且最终返回视图对象View
    View resolveViewName(String viewName, Locale locale) throws Exception;
}

```

​	3、View接口，视图接口（ThymeleafView实现了View接口......）

​		负责将模板语法的字符串转化成html代码，交给浏览器渲染。

​		核心方法：

​			void render(......)

```java
public interface View {
	// ...
    void render(@Nullable Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception;
}
```



#### 3、实现试图机制的原理描述

第一步：浏览器发送请求给web服务器

第二步：Spring MVC中的DispatcherServlet接收到请求

第三步：DispatcherServlet根据请求路径分发到对应的controller

第四步：调用controller方法

第五步：Controller执行后返回一个`逻辑视图名`给DispatcherServlet

第六步：DispatcherServlet调用ThymeleafViewResolver的resolveViewName方法，将`逻辑视图名`转换为`物理视图名`，并创建ThymeleafView对象给DispatcherServlet

第七步：DispatcherServlet调用ThymeleafView的render方法，render方法将模板语言转换为HTML代码，响应给浏览器，完成最终的渲染。



### （二）JSP视图

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!--组件扫描-->
    <context:component-scan base-package="com.powernode.springmvc.controller"/>

    <!--视图解析器-->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```





### （三）转发和重定向

#### 1、转发和重定向的区别

​	（1）转发是一次请求，浏览器地址栏不会发生变化；重定向是两次请求，浏览器地址栏会发生变化。

​	（2）转发代码：request.getRequestDispatcher("/index").forward(request, response);

​		重定向代码：response.sendRedirect("/webapproot/index");

​	（3）转发是内部资源跳转，由服务器控制，且不可跨域；重定向可以完成内部资源跳转，也可以跨域跳转。

​	（4）转发的方式可以访问WEB-INF目录下受保护的资源；重定向无法直接访问WEB-INF目录下受保护的资源。

​	（5）转发原理：

​			a、假设发送了/a请求，执行力AServlet

​			b、在AServlet中通过request.getRequestDispatcher("/b").forward(request, response);转发到BServlet

​			c、从AServlet跳转到BServlet是服务器内部来控制的。对于浏览器而言，浏览器只发送了一个 /a 请求。

​	（6）重定向原理：

​			a、假设发送了/a请求，执行力AServlet

​			b、在AServlet中通过response.sendRedirect("/webapproot/b")重定向到了BServlet

​			c、此时服务器会将请求路径 /webapproot/b 响应给浏览器

​			d、浏览器会再次发送 /webapproot/b 请求来访问BServlet

#### 2、forward

在Spring MVC中默认就是转发的方式，我们之前所写的程序，都是转发的方式。

```java
@RequestMapping("/a")
public String toA(){
    return "forward:/b";
}

@RequestMapping("/b")
public String toB(){
    return "view";
}
```

流程见语雀



#### 3、redirect

```java
@RequestMapping("/a")
public String toA(){
    return "redirect:/b";
}

@RequestMapping("/b")
public String toB(){
    return "view";
}
```

```java
// 跨域
@RequestMapping("/a")
public String a(){
    return "redirect:http://localhost:8080/springmvc/b";
}
```



### （四）\<mvc:view-controller/> 和\<mvc:annotation-driven/>

```java
// 当controller中没有任何业务逻辑代码，如
@RequestMapping("/test")
public String toTest(){
    return "test";
}

// 那么只需要删掉这块代码，在springmvc.xml中配置
/*
	如果只包含第一行代码，那么会让所有的注解失效包括@Controller，就会访问不到页面404，所以需要加上第二行代码
	<mvc:view-controller path="/test" view-name="test"/>
    <mvc:annotation-driven/>

*/
```



### （五）访问静态资源

​	一个项目可能会包含大量的静态资源，比如：css、js、images等。

​	由于我们DispatcherServlet的url-pattern配置的是“/”，之前我们说过，这个"/"代表的是除jsp请求之外的所有请求，也就是说访问应用中的静态资源，也会走DispatcherServlet，这会导致404错误，无法访问静态资源

#### 1、使用默认Servlet处理静态资源

在springmvc.xml中添加即可

```xml
<!-- 开启注解驱动 -->
<mvc:annotation-driven />

<!--开启默认Servlet处理-->
<mvc:default-servlet-handler>
```

在tomcat服务器中配有一个默认的servlet：DefaultServlet，当发生404时，DefaultServlet会对该请求进行处理，该类会根据请求的 URL 去查询 Web 应用的静态资源（如 HTML、CSS、JavaScript 和图片等），并将其返回给用户。



#### 2、使用 mvc:resources 标签配置静态资源

```xml
<!-- 开启注解驱动 -->
<mvc:annotation-driven />

<!-- 配置静态资源处理 -->
<!-- WEB-INF目录下的资源仍旧收到保护无法通过 mapping="/WEB-INF/**" ，访问-->
<mvc:resources mapping="/static/**" location="/static/" />
```





## 六、RESTFul编程风格



## 七、HttpMessageConverter

### （一）HttpMessageConverter

#### 1、转换器转换的是什么

转换的是HTTP协议和Java对象

![image-20240923143155147](.\img\image-20240923143155147.png)

底层实际上使用了HttpMessageConverter接口的其中一个实现类FormHttpMessageConverter，将请求协议转换为java对象。



<img src=".\img\无标题.png" style="zoom:150%;" />

上图中StringHttpMessageConverter将java对象转换为响应协议。



### （二）SpringMVC中的AJAX请求

​	传统的请求：controller返回一个逻辑视图，然后交给视图解析器解析，最后跳转页面。而AJAX是局部刷新，以前我们在servlet中使用response.getWriter().print("message")的方式响应，现在也能使用：

```java
@Controller
public class HelloController {

    @RequestMapping(value = "/hello")
    public void hello(HttpServletResponse response) throws IOException {
        response.getWriter().print("hello");
    }
}

```



### （三）@ResponseBody

#### 1、StringHttpMessageConverter

上述AJAX代码可以修改为：

```java
@Controller
public class HelloController {

    @RequestMapping(value = "/hello")
    @ResponseBody
    public String hello(){
        // 由于你使用了 @ResponseBody 注解
        // 以下的return语句返回的字符串则不再是“逻辑视图名”了
        // 而是作为响应协议的响应体进行响应。
        return "hello";
    }
}
```

这里的 **hello** 并不是逻辑视图名，而是作为响应体进行响应。



#### 2、MappingJackson2HttpMessageConverter

启用 MappingJackson2HttpMessageConverter 消息转换器的步骤如下：

第一步：引入Jackson依赖

```xml
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>
  <version>2.17.0</version>
</dependency>
```

第二步：开启注解驱动

```xml
<mvc:annotation-driven/>
```



测试：

```java
@Controller
public class HelloController {

    @RequestMapping(value = "/hello")
    @ResponseBody
    public User hello(){
        User user = new User("zhangsan", "22222");
        return user;
    }
}
```



### （四）@RestController

为了方便，Spring MVC中提供了一个注解 @RestController。这一个注解代表了：@Controller + @ResponseBody。



### （五）RequestBody

只能作用在形参上

其作用是：直接将请求体传递给java程序，在java中可以使用String类型的变量接受这个请求体的内容

#### 1、FormHttpMessageConverter

**需要注意的是：这边发送过来的请求必须要有请求体，post可行。普通的get请求貌似不行**

```java
// springmvc仍然会使用FomrHttpMessageConverter转换器，直接将请求体以字符串的形式传递给requestBodyStr
@PostMapping(value = "/save")
public String saveByPost(@RequestBody String requestBodyStr){
    System.out.println("请求体：" + requestBodyStr);
    return "ok";
}
```



#### 2、MappingJackson2HttpMessageConverter

如果请求体中提交的是一个JSON格式的字符串，这个JSON字符串传递给SpringMVC之后，可以将**JSON字符串**转换为POJO对象。

```

```



### （六）RequestEntity









## 八、文件上传和下载

### （一）文件上传

使用springmvc6版本，不需要添加以下依赖：

```xml
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.5</version>
</dependency>
```



```xml
<!--前端控制器-->
<servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
    <multipart-config>
        <!--设置单个支持最大文件的大小-->
        <max-file-size>102400</max-file-size>
        <!--设置整个表单所有文件上传的最大值-->
        <max-request-size>102400</max-request-size>
        <!--设置最小上传文件大小-->
        <file-size-threshold>0</file-size-threshold>
    </multipart-config>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

在DispatcherServlet配置时，添加multipart-config配置信息。（这是Spring6，如果是Spring5，则不是这样配置，而是在springmvc.xml文件中配置：CommonsMultipartResolver）

SpringMVC6中把这个类已经删除了，废弃了。



Spring MVC专门为文件上传提供了一个类，MultipartFile，这个类代表从客户端传过来的那个文件

```java
@PostMapping("/file/up")
public String fileUp(@RequestParam("fileName")MultipartFile multipartFile, HttpServletRequest request) throws IOException {
    String name = multipartFile.getName();
    // 获取前端表单中上传文件字段的name值
    System.out.println(name);

    String originalFilename = multipartFile.getOriginalFilename();
    // 源文件名字
    System.out.println(originalFilename);

    InputStream inputStream = multipartFile.getInputStream();
    // 创建一个File对象，指向服务器上的upload文件夹。
    // getRealPath("/upload")返回的是项目根目录下的upload文件夹的绝对路径
    File file = new File(request.getServletContext().getRealPath("/upload"));

    // 如果该文件夹不存在，则创建该目录
    if(!file.exists()){
        file.mkdirs();
    }

    BufferedOutputStream out = new BufferedOutputStream(new FileOutputStream(file.getAbsolutePath() + "/" + UUID.randomUUID().toString() + originalFilename.substring(originalFilename.lastIndexOf("."))));
    byte[] bytes = new byte[1024 * 100];
    int readCount = 0;

    while ((readCount = inputStream.read(bytes)) != -1){
        out.write(bytes, 0, readCount);
    }

    out.flush();

    inputStream.close();
    out.close();

    return "ok";
}
```



### （二）文件下载

```java
@GetMapping("/download")
public ResponseEntity<byte[]> downloadFile(HttpServletResponse response, HttpServletRequest request) throws IOException {
    File file = new File(request.getServletContext().getRealPath("/upload") + "/1.jpeg");
    // 创建响应头对象
    HttpHeaders headers = new HttpHeaders();
    // 设置响应内容类型
    headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
    // 设置下载文件的名称
    headers.setContentDispositionFormData("attachment", file.getName());

    // 下载文件
    ResponseEntity<byte[]> entity = new ResponseEntity<byte[]>(Files.readAllBytes(file.toPath()), headers, HttpStatus.OK);
    return entity;
}
```



## 九、异常处理器

### （一）什么是异常处理器

在执行处理器方法时出现了异常，可以采用异常处理器进行应对，通过异常处理器跳转到对应的视图，展示友好的信息



提供了一个接口：HandlerExceptionResolver

核心方法是resolveException，该方法用来编写具体的处理方案，返回ModelAndView，表示处理完异常后跳转到哪个视图。



HandlerExceptionResolver接口有两个常用的默认实现：

DefaultHandlerResolver

SimpleMappingExceptionResolver



### （二）默认异常处理器

DefaultHandlerExceptionResolver核心方法：

![image-20240925202531433](.\img\image-20240925202531433.png)

常见的是404等页面



### （三）自定义异常处理器

SimpleMappingExceptionResolver有两种语法：

通过XML配置文件

通过注解

#### 1、配置文件方式

```xml
<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
    <property name="exceptionMappings">
        <props>
            <!--用来指定出现异常后，跳转的视图tip-->
            <prop key="java.lang.Exception">tip</prop>
        </props>
    </property>
    <!--将异常信息存储到request域，value属性用来指定存储时的key。-->
    <property name="exceptionAttribute" value="e"/>
</bean>
```

tip视图

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>出错了</title>
</head>
<body>
<h1>出错了，请联系管理员！</h1>
<div th:text="${e}"></div>
</body>
</html>
```



#### 2、注解方式

```java
@ControllerAdvice
public class ExceptionController {

    @ExceptionHandler
    public String tip(Exception e, Model model){
        model.addAttribute("e", e);
        return "tip";
    }
}
```





## 十、拦截器

### （一）拦截器概述

拦截器（Interceptor）类似于过滤器（filter）

拦截器的应用场景：

1、登陆验证：判断用户是否已经登陆

2、权限校验：根据用户权限对对部分网址进行访问控制。

3、请求日志：记录请求信息，如请求地址、请求参数等

4、更改响应：可以对响应的内容进行修改，例如添加头信息、调整响应内容格式等。 拦截器和过滤器的区别在于它们的作用层面不同。

![](.\img\image.png)

**注意：对于这种基本配置来说，拦截器是拦截所有请求的。**



### （二）拦截器的创建与配置

#### 1、定义拦截器

实现**org.springframework.web.servlet.HandlerInterceptor**接口，共有三个方法：

​	preHandler：处理器方法调用之前。只有该方法有返回值，返回布尔类型，true放行。

​	postHandler：处理器方法调用之后执行

​	afterCompletion：页面渲染之后执行



在Spring的拦截器（Interceptor）链中，如果拦截器A的`preHandle`方法返回`true`，则表示请求可以继续处理，后续的拦截器的`preHandle`方法将会被调用。如果后续拦截器的`preHandle`方法返回`false`，则请求处理流程会被中断，后续的处理（如控制器方法）将不会被执行。

然而，不论后续拦截器的`preHandle`返回什么，拦截器A的`afterCompletion`方法都会被调用。这是因为`afterCompletion`方法是在请求处理完成后（无论成功与否）执行的，用于执行一些清理工作或记录日志等。



#### 2、拦截器基本配置

##### Ⅰ、方式一：

```xml
<!-- 拦截器可以配置多个 -->
<mvc:interceptors>
    <bean class="com.powernode.springmvc.interceptors.Interceptor1"/>
</mvc:interceptors>
```



##### Ⅱ、方式二：

```xml
<mvc:interceptors>
    <ref bean="interceptor1"/>
</mvc:interceptors>
```

前提要对Interceptor1类用@Component注解



#### 3、拦截器源码分析

```java
// DispatcherServlet
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
			// ...
			// 调用所有拦截器的 preHandle 方法
                if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                    return;
                }
			// 调用处理器方法
                mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
                if (asyncManager.isConcurrentHandlingStarted()) {
                    return;
                }
                this.applyDefaultViewName(processedRequest, mv);
    		// 调用所有拦截器的 postHandle 方法
                mappedHandler.applyPostHandle(processedRequest, response, mv);
            // ...
}

private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
        @Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
        @Nullable Exception exception) throws Exception {
    // 渲染页面
    render(mv, request, response);
    // 调用所有拦截器的 afterCompletion 方法
    mappedHandler.triggerAfterCompletion(request, response, null);
}
```

### （三）拦截器的高级配置

采用以上基本配置方式，拦截器是拦截所有请求路径的，如果要针对某些路径进行拦截，某些不拦截，可以采用高级配置：

```xml
<mvc:interceptors>
    <mvc:interceptor>
        <!-- 拦截所有请求路径 -->
        <mvc:mapping path="/**"/>
        <!-- 除 /test 路径之外 -->
        <mvc:exclude-mapping path="/test"/>
        <ref bean="interceptor1"/>
        <!-- 
            或者
            <bean class="com.powernode.springmvc.incerceptors.Interceptor1"/>
         -->
    </mvc:interceptor>
</mvc:interceptors>
```





### （四）拦截器的执行顺序

#### 1、执行顺序

如果所有拦截器prehandle都返回true，那么自上而下调用prehandler

```java
<mvc:interceptors>
    <ref bean="interceptor1"/>
    <ref bean="interceptor2"/>
</mvc:interceptors>
```



如果其中一个拦截器prehandle返回false，任何一个postHandle都不执行。但是返回false的拦截器的前面的拦截器按照逆序执行afterCompletion

```java
// HandlerExceptionChain
boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
    for(int i = 0; i < this.interceptorList.size(); this.interceptorIndex = i++) {
        HandlerInterceptor interceptor = (HandlerInterceptor)this.interceptorList.get(i);
        if (!interceptor.preHandle(request, response, this.handler)) {
            this.triggerAfterCompletion(request, response, (Exception)null);
            return false;
        }
    }

    return true;
}

void applyPostHandle(HttpServletRequest request, HttpServletResponse response, @Nullable ModelAndView mv) throws Exception {
    for(int i = this.interceptorList.size() - 1; i >= 0; --i) {
        HandlerInterceptor interceptor = (HandlerInterceptor)this.interceptorList.get(i);
        interceptor.postHandle(request, response, this.handler, mv);
    }

}

void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, @Nullable Exception ex) {
    for(int i = this.interceptorIndex; i >= 0; --i) {
        HandlerInterceptor interceptor = (HandlerInterceptor)this.interceptorList.get(i);

        try {
            interceptor.afterCompletion(request, response, this.handler, ex);
        } catch (Throwable var7) {
            logger.error("HandlerInterceptor.afterCompletion threw exception", var7);
        }
    }

}
```



## 十一、springmvc执行顺序



## 十二、手写springmvc



## 十三、SSM整合

### （一）web.xml文件的代替

#### 1、Servlet3.0新特性

web.xml可以不用写，规范中提供了一个接口ServletContainerInitializer

```java
public interface ServletContainerInitializer {
    void onStartup(Set<Class<?>> var1, ServletContext var2) throws ServletException;
}
```

服务器在启动的时候会自动从容器中找到该接口的实现类，并调用它的onStartup方法来完成Servlet上下文的初始化。



在spring3.1的时候，提供了**ServletContainerInitializer**的实现类：

```java
public class SpringServletContainerInitializer implements ServletContainerInitializer{
	// ...
	
    // 核心方法
	public void onStartup(@Nullable Set<Class<?>> webAppInitializerClasses, ServletContext servletContext) throws ServletException {
	//...
                        ((List)initializers).add((WebApplicationInitializer)ReflectionUtils.accessibleConstructor(waiClass, new Class[0]).newInstance());
                    //...
    }

}

//其中WebApplicationInitializer如下，是一个接口，该接口有一个子类AbstractAnnotationConfigDispatcherServletInitializer，其为抽象类，需要

public interface WebApplicationInitializer {
    void onStartup(ServletContext servletContext) throws ServletException;
}
```

![](.\img\image (1).png)

当我们编写类继承`AbstractAnnotationConfigDispatcherServletInitializer`之后，web服务器在启动的时候会根据它来初始化Servlet上下文。

![](.\img\未命名文件.png)

#### 2、编写WebAppInitializer

