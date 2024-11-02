# Java

## 一、反射机制

### （一）反射机制作用

​	通过java语言中的反射机制可以操作字节码文件（class文件）

### （二）获取Class文件三种方式

#### 	1、forName

```java
try {
    Class c = Class.forName("java.lang.String");
} catch (ClassNotFoundException e) {
    throw new RuntimeException(e);
}
```

#### 	2、getClass

```java
String s = "123";
Class c = s.getClass();
```

#### 	3、class

```java
Class c = String.class;
```



#### （三）实例化

```java
try {
    Class c = Class.forName("java.util.Date");
    // 调用无参
    Object o = c.newInstance();
    System.out.println(o);
} catch (ClassNotFoundException | InstantiationException | IllegalAccessException e) {
    throw new RuntimeException(e);
}
```

```java
try {
    FileReader reader = new FileReader("javase-001-reflect\\src\\main\\resources\\classInfo.properties");
    Properties properties = new Properties();
    properties.load(reader);
    reader.close();
    String className = properties.getProperty("className");
    Class c = Class.forName(className);
    Object o = c.newInstance();
    System.out.println(o);

} catch (Exception e) {
    throw new RuntimeException(e);
}
```



```java
try {
    FileReader reader = new FileReader("javase-001-reflect\\src\\main\\resources\\classInfo.properties");
    Properties properties = new Properties();
    properties.load(reader);
    reader.close();
    String className = properties.getProperty("className");
    // 会导致类加载
    Class c = Class.forName(className);


} catch (Exception e) {
    throw new RuntimeException(e);
}

public class User {
    static {
        System.out.println("静态代码块执行");
    }
}
```



#### （四）获取类路径下文件的绝对路径

```java
// 方式一
String path = Thread.currentThread().getContextClassLoader().getResource("classInfo.properties").getPath();
FileReader reader = new FileReader(path);
Properties properties = new Properties();
properties.load(reader);
reader.close();
String className = properties.getProperty("className");
Class c = Class.forName(className);

// 方式二：直接获取流
InputStream stream = Thread.currentThread().getContextClassLoader().getResourceAsStream("classInfo.properties");
Properties properties = new Properties();
properties.load(stream);
stream.close();
String className = properties.getProperty("className");
Class c = Class.forName(className);

```



#### （五）资源绑定器

```java
// 只能绑定xxx.properties文件，并且这个文件必须在类路径下
// 写路径的时候，不需要写扩展名
ResourceBundle bundle = ResourceBundle.getBundle("classInfo");
String className = bundle.getString("className");
System.out.println(className);
```



#### （六）Field

```java
// 获取所有实例（静态也包含）属性
try {
    Class c = Class.forName("com.powernode.reflect.Student");
    Field[] fields = c.getDeclaredFields();
    for (Field field : fields){
        System.out.println(field.getType());
        System.out.println(field.getName());
        // 返回的是一个数字，代表各自的限定符
        System.out.println(Modifier.toString(field.getModifiers()));
        System.out.println("========================");
    }
} catch (ClassNotFoundException e) {
    throw new RuntimeException(e);
}

public class Student {
    public static int no;
    private String name;
    protected int age;
    boolean sex;
}
```

```
try {
    Class studentClass = Class.forName("com.powernode.reflect.Student");
    Object student = studentClass.newInstance();
    Field no = studentClass.getDeclaredField("no");
    no.set(student, 111);
    student = (Student) student;
    System.out.println(student);
} catch (ClassNotFoundException | InstantiationException | IllegalAccessException | NoSuchFieldException e) {
    throw new RuntimeException(e);
}
```



#### （七）Method

##### 	1、可变长参数

```java
// 可变长参数要求的参数个数是0~N
// 可变长参数在参数列表上必须在最后一个位置上，且仅有一个，如 method(int i, String... arg)
// 可以当成数组来看待
public static void main(String[] args) {
    String[] strings = new String[]{"我", "是", "中", "国", "人"};
    method(strings);

    method("我", "是", "中", "国", "人");
}

public static void method(String... arg){
    for (String str : arg){
        System.out.println(str);
    }
}
```



##### 	2、反射Method

```java
try {
    Class studentClass = Class.forName("com.powernode.reflect.Student");
    Object student = studentClass.newInstance();
    Method[] methods = studentClass.getDeclaredMethods();
    for (Method method : methods) {
        System.out.println(method.getName());
        System.out.println(Modifier.toString(method.getModifiers()));
        System.out.println(method.getReturnType());
        Class[] parameterTypes = method.getParameterTypes();
        for (Class parameterType : parameterTypes) {
            System.out.println(parameterType.getSimpleName());
        }
        System.out.println("================");
    }
} catch (ClassNotFoundException | InstantiationException | IllegalAccessException e) {
    throw new RuntimeException(e);
}

public class Student {
//    public static int no;
    public int no;
    private String name;
    protected int age;
    boolean sex;

    public boolean login(String name, String pwd){
        if (name.equals("zhangsan") && pwd.equals("123")){
            System.out.println("登陆成功");
            return true;
        }
        System.out.println("登陆失败");
        return false;
    }

    public void logout(){
        System.out.println("登出");
    }

    @Override
    public String toString() {
        return "Student{" +
                "no=" + no +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", sex=" + sex +
                '}';
    }
}

```

```java
try {
    Class studentClass = Class.forName("com.powernode.reflect.Student");
    Object student = studentClass.newInstance();
    Method method = studentClass.getMethod("login", String.class, String.class);
    // invoke调用method方法
    Object ret = method.invoke(student, "zhangsan", "123");
    System.out.println(ret);
} catch (ClassNotFoundException | InstantiationException | IllegalAccessException | NoSuchMethodException |
         InvocationTargetException e) {
    throw new RuntimeException(e);
}

public class Student {
//    public static int no;
    public int no;
    private String name;
    protected int age;
    boolean sex;

    public boolean login(String name, String pwd){
        if (name.equals("zhangsan") && pwd.equals("123")){
            System.out.println("登陆成功");
            return true;
        }
        System.out.println("登陆失败");
        return false;
    }

    public void logout(){
        System.out.println("登出");
    }

    @Override
    public String toString() {
        return "Student{" +
                "no=" + no +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", sex=" + sex +
                '}';
    }
}
```







## 二、代理

### （一）代理模式作用

​	1、功能增强：在原有的功能上增加了新的功能

​	2、控制访问：代理类不让你访问目标

### （二）静态代理

​	1、代理类自己手工实现，自己创建一个java类

​	2、所要代理的目标是确定的

```java
public interface UsbSell {
    float sell(int amount);
}

public class TaoBao implements UsbSell{

    private UsbSell factory = new Factory();

    @Override
    public float sell(int amount) {
        float price = factory.sell(1);
        price = price + 25.0f;
        return price;
    }
}

public class Factory implements UsbSell{
    @Override
    public float sell(int amount) {
        return 85.0f * amount;
    }
}

public class Buyer {
    public static void main(String[] args) {
        UsbSell taoBao = new TaoBao();
        System.out.println(taoBao.sell(1));
    }
}
```

缺点：

​	1、当目标类（factory）增加了，代理类（shangjian）可能也需要成倍的增加

![image-20240917194256542](D:\java development\java\img\image-20240917194256542.png)

​	2、当你的接口中功能在增加了，或者修改了，会影响众多的实现类、厂家类，代理都需要修改，影响比较多



#### （三）动态代理

使用jdk的反射机制，创建对象的能力，创建的是代理类对象，而不是创建类文件，不用写java文件

优点：

​	1、代理类数量可以很少

​	2、当修改接口中的方法，不会影响代理类

```java
public interface UsbSell {
    float sell(int amount);
    float sale();
}

public class UsbKingFactory implements UsbSell {
    @Override
    public float sell(int amount) {
        return amount * 85.0f;
    }

    @Override
    public float sale() {
        return 50f;
    }
}


public class SaleHandler implements InvocationHandler {
    private Object target;
    public SaleHandler(Object target){
        this.target = target;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        Object res = null;
        res = method.invoke(target, args);

        if (res != null){
            return (Float)res;
        }

        return 0f;
    }
}

public class SellHandler implements InvocationHandler {

    // 传入是谁的对象，就给谁创建代理
    private Object target;
    public SellHandler(Object target){
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        Object res = null;

        res = method.invoke(target, args);

        if (res != null){
            Float price = (Float) res;
            price += 25;
            res = price;
        }

        return res;
    }
}


public class MainShop {
    public static void main(String[] args) {
        UsbSell factory = new UsbKingFactory();
        InvocationHandler sellHandler = new SellHandler(factory);
        InvocationHandler saleHandler = new SaleHandler(factory);

        UsbSell proxySell = (UsbSell) Proxy.newProxyInstance(factory.getClass().getClassLoader(), factory.getClass().getInterfaces(), sellHandler);
        UsbSell proxySale = (UsbSell) Proxy.newProxyInstance(factory.getClass().getClassLoader(), factory.getClass().getInterfaces(), saleHandler);

        float sell = proxySell.sell(2);
        float sale = proxySale.sale();
        System.out.println(sell);
        System.out.println(sale);
    }
}
```

InvocationHandler接口的实现类：

​	1、调用目标方法

​	2、功能增强

## 三、类加载器

### （一）概述

​	代码在开始执行前，会将所需要的类全部加载到JVM当中。

​	如何加载？

​		1、首先通过“启动类加载器”加载

​			启动类加载器专门加载："E:\Program Files\java\jdk1.8.0_321\jre\lib\rt.jar"

​			rt.jar中都是JDK最核心的类库。

​		

​		2、如果通过“启动类加载器”加载不到的时候

​			会通过“扩展类加载器”加载：E:\Program Files\java\jdk1.8.0_321\jre\lib\ext



​		3、如果通过“扩展类加载器”加载不到的时候

​			会通过“应用类加载器”加载：classpath中的类



### （二）双亲委派机制

​	优先从**“启动类加载器”（父）**和**“扩展类加载器”（母）**加载，先从父再从母
