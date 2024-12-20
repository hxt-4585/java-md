# RabbitMQ

附：[MQ的相关概念 | xustudyxu's Blog](https://frxcat.fun/pages/e645d9/)

## 一、RabbitMQ简介

### 1.1 简介

RabbitMQ 是个中间件 , 负责 接收/存储/转发 消息数据 . 类似 平时快递派送的过程



**是什么?**

MQ 本质是个队列 , 先进先出 消息推送 , 是种跨进程通信机制的上下游传递消息 . 主要解决不同对象通信的序列

**为什么用?**

- **流量消锋 :** 如果系统最大多处理1W条请求 , 且还是高峰期的时候 , 很有可能会突破1W导致宕机 . MQ可以队列缓冲 , 防止宕机 (高峰期体验差 , 但能保障了不会宕机)
- **应用解耦 :** 系统有多个子系统 , 在业务涉及多个子系统完成时 , 当中的一个子系统宕机了 , 导致该业务异常无法运作 . MQ可以将要处理的业务缓存到消息队列中 , 由消息队列进行访问子系统执行业务 , 防止不一致运作问题 , 提高可用性
- **异步处理 :** 假如 A调用B , 但B需要执行很长一段时间 , 但A想知道B执行的进度 , 以往 会通过 A调用API查B进度 , 显然不优雅 . MQ可以使用消息总线 , A调用B后 , MQ会监控B进度(A实时得到B进度) , B处理完后 会发消息给MQ , 由MQ转发至A .



### 1.2 四大核心概念

![image-20241116145612709](.\img\image-20241116145612709.png)

一个交换机可以对应多个队列，一个队列对应一个消费者

### 1.3 工作原理

![image-20241116145808311](.\img\image-20241116145808311.png)

**Broker**：接收和分发消息的应用，RabbitMQ Server就是Message Broker

**Virtual host**：出于多租户和安全因素设计的，把 AMQP 的基本组件划分到一个虚拟的分组中，类似于网络中的 namespace 概念。当多个不同的用户使用同一个 RabbitMQ server 提供的服务时，可以划分出多个 vhost，每个用户在自己的 vhost 创建 exchange／queue 等

**Connection**：publisher／consumer 和 broker 之间的 TCP 连接

**Channel**：如果每一次访问 RabbitMQ 都建立一个 Connection，在消息量大的时候建立 TCP Connection 的开销将是巨大的，效率也较低。Channel 是在 connection 内部建立的逻辑连接，如果应用程序支持多线程，通常每个 thread 创建单独的 channel 进行通讯，AMQP method 包含了 channel id 帮助客 户端和 message broker 识别 channel，所以 channel 之间是完全隔离的。Channel 作为轻量级的 Connection 极大减少了操作系统建立 TCP connection 的开销

**Exchange**：message 到达 broker 的第一站，根据分发规则，匹配查询表中的 routing key，分发 消息到 queue 中去。常用的类型有：direct (point-to-point)，topic (publish-subscribe) and fanout (multicast)

**Queue**：消息最终被送到这里等待 consumer 取走

**Binding**：exchange 和 queue 之间的虚拟连接，binding 中可以包含 routing key，Binding 信息被保 存到 exchange 中的查询表中，用于 message 的分发依据



### 1.4 安装

```sh
# 下载以下两个文件，通过xftp传到目录/usr/local/software
# erlang-23.2.7-2.el7.x86_64.rpm
# rabbitmq-server-3.8.14-1.el7.noarch.rpm
# 进入/usr/local/software后，执行以下命令

# Erlang
rpm -ivh erlang-21.3-1.el7.x86_64.rpm
	# 检查版本 quit退出
	erl -v
# socat 依赖插件
yum install socat -y
# RabbitMQ
rpm -ivh rabbitmq-server-3.8.8-1.el7.noarch.rpm
	# 启动服务
	systemctl start rabbitmq-server
	# 查看服务状态 (active绿色表示成功)
	systemctl status rabbitmq-server
	
	
# 添加开机自启
chkconfig rabbitmq-server on
# 启动服务
/sbin/service rabbitmq-server start
# 查看状态
/sbin/service rabbitmq-server status
# 关闭服务
/sbin/service rabbitmq-server stop

# 安装web管理插件
# 默认使用 http://{ip}:15672访问
rabbitmq-plugins enable rabbitmq_management
# 用插件访问时，需要关闭防火墙
firewall-cmd --zone=public --add-port=15672/tcp --permanent
```

安装时常见问题

```
# 在centos7中使用yum命令时候报错
# https://www.cnblogs.com/kohler21/p/18331060

# 解决 rabbitmq-plugins enable rabbitmq_management 报 {:query, :rabbit@xxxxxx, {:badrpc, :timeout}}
# https://blog.csdn.net/qq_39879542/article/details/115258799
```



### 1.3 设置权限

```
# 创建账号
rabbitmqctl add_user admin 123
# 设置用户角色
rabbitmqctl set_user_tags admin administrator
# 设置用户权限
# 为用户添加资源权限，添加配置、写、读权限
# rabbitmqctl set_permissions [-p <vhostpath>] <user> <conf> <write> <read>
rabbitmqctl set_permissions -p "/" bozhu ".*" ".*" ".*"
# 当前用户和角色
rabbitmqctl list_users

```





## 二、RabbitMQ入门案例

```xml
<!--rabbitmq 依赖客户端-->
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.8.0</version>
</dependency>
<!--操作文件流的一个依赖-->
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.6</version>
</dependency>
```

![image-20241116150428639](.\img\image-20241116150428639.png)



### 2.1 消息生产者

步骤：

​	1、创建 RabbitMQ 连接工厂

​	2、进行 RabbitMQ 工厂配置信息

​	3、创建 RabbitMQ 连接

​	4、创建 RabbitMQ 信道

​	5、生成一个队列

​	6、发送一个消息到交换机，交换机发送到队列。""代表默认交换机

```java
public class Producer {

    // 队列名称
    public static final String QUEUE_NAME = "hello";

    // 发消息
    public static void main(String[] args) throws IOException, TimeoutException {
        // 创建一个连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 工厂ip 连接rabbitmq队列
        String host = "192.168.116.128";
        factory.setHost(host);

        // 用户名
        factory.setUsername("admin");
        // 密码
        factory.setPassword("20170440");

        // 创建连接
        Connection connection = factory.newConnection();
        // 创建信道
        Channel channel = connection.createChannel();
        /**
         * 生产一个队列
         * 1、队列名称
         * 2、队列里面的消息是否持久化，默认情况下，消息存储在内存中
         * 3、该队列是否只提供一个消费者进行消费，是否进行消息共享，false可以多个
         * 4、是否自动删除，最后一个消费者端断开后，该队列是否自动删除，true自动删除
         * 5、其他参数
         */
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        // 发消息
        String message = "Hello World!";
        /**
         * 发送一个消息
         * 1、发送到哪个交换机
         * 2、路由的key值是哪个，本次是队列的名称
         * 3、其他参数信息
         * 4、发生消息的消息体
         */
        channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
        System.out.println("消息发生完成");
    }
}

```

![image-20241116170222916](.\img\image-20241116170222916.png)

**方法解释**



A: 声明队列

```java
channel.queueDeclare(队列名/String, 持久化/boolean, 共享消费/boolean, 自动删除/boolean, 配置参数/Map);
```

配置参数现在是 null，后面死信队列延迟队列等会用到，如：

队列的优先级

队列里的消息如果没有被消费，何去何从？（死信队列）

```java
Map<String, Object> params = new HashMap();
// 设置队列的最大优先级 最大可以设置到 255 官网推荐 1-10 如果设置太高比较吃内存和 CPU
params.put("x-max-priority", 10);
// 声明当前队列绑定的死信交换机
params.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE);
// 声明当前队列的死信路由 key
params.put("x-dead-letter-routing-key", "YD");
channel.queueDeclare(QUEUE_NAME, true, false, false, params);

```



B: 发布消息

```java
channel.basicPublish(交换机名/String, 队列名/String, 配置参数/Map, 消息/String);
```

配置参数现在是 null，后面死信队列、延迟队列等会用到，如：

发布的消息优先级

发布的消息标识符 id

```java
// 给消息赋予 优先级 ID 属性
AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().priority(10).messageId("1")build();
channel.basicPublish("", QUEUE_NAME, properties, message.getBytes());

```



### 2.2 消息消费者

创建一个类作为消费者，消费 RabbitMQ 队列的消息

```java
public class Consumer {
    //队列的名称
    public static final String QUEUE_NAME="hello";
    // 接受消息
    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.116.128");
        factory.setUsername("admin");
        factory.setPassword("20170440");
        Connection connection = factory.newConnection();

        Channel channel = connection.createChannel();

        DeliverCallback deliverCallback = (consumerTag, message) -> {
            System.out.println(new String(message.getBody()));
        };

        CancelCallback cancelCallback = (consumerTag) -> {
            System.out.println("消息消费被中断");
        };
        /**
         * 消费者消息信息
         * 1、消费哪个队列
         * 2、消费成功后是否要自动应答true，代表自动应答   false：代表手动应答
         * 3、消费者成功消费的回调
         * 4、消费者未成功消费的回调
         */
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
    }
}
```



### 2.3 轮询消费

轮询消费消息指的是轮流消费消息，即每个工作队列都会获取一个消息进行消费，并且获取的次数按照顺序依次往下轮流。

案例中生产者叫做 Task，一个消费者就是一个工作队列，启动两个工作队列消费消息，这个两个工作队列会以轮询的方式消费消息。



![image-20241116175851241](.\img\image-20241116175851241.png)

封装

```java
public class RabbitMQUtil {
    public static Channel getChannel() throws IOException, TimeoutException {

        //创建一个连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //工厂IP 连接RabbitMQ对列
        factory.setHost("192.168.116.128");
        //用户名
        factory.setUsername("admin");
        //密码
        factory.setPassword("20170440");

        //创建连接
        Connection connection = factory.newConnection();
        //获取信道
        Channel channel = connection.createChannel();

        return channel;
    }
}
```

生产者

```java
public class Task01 {
    //队列名称
    public static final String QUEUE_NAME="hello";

    //发送大量消息
    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = RabbitMQUtil.getChannel();
        //队列的声明
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);

        //发送消息
        //从控制台当中接受信息
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()) {
            String message = scanner.next();
            channel.basicPublish("",QUEUE_NAME,null,message.getBytes());
            System.out.println("消息发送完成:"+message);
        }
    }
}
```



消费者

```java
/**
 * 这是一个工作线程，相当于之间的Consumer
 */
public class Worker01 {
    // 队列名称
    public static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();

        DeliverCallback deliverCallback = (consumerTag, message) -> {
            System.out.println("接收到的消息：" + new String(message.getBody()));
        };

        CancelCallback cancelCallback = (consumerTag) -> {
            System.out.println("消息消费被中断");
        };
        /**
         * 消费者消息信息
         * 1、消费哪个队列
         * 2、消费成功后是否要自动应答true，代表自动应答   false：代表手动应答
         * 3、消费者成功消费的回调
         * 4、消费者未成功消费的回调
         */

        System.out.println("C2等待接收消息......");
        // 消息的接受
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
    }
}
```

模拟多线程（此处实际上是多客户端），勾选 `Allow multiple instances`

![image-20241116180111421](.\img\image-20241116180111421.png)

![image-20241116180344941](.\img\image-20241116180344941.png)



## 三、RabbitMQ消息应答

### 3.1 消息应答

消费者完成一个任务需要一定的时间，如果其中一个消费者处理一个长的任务并仅仅只完成了部分就挂掉了，会发生什么情况。RabbitMQ一旦向消费者传递一条信息，便立即将该消息标记为删除，在这种情况下，突然有个消费者挂掉了，我们将丢失正在处理的消息，以及后续发送给消费者的信息。

**消息应答**：消费者在接收到消息并且处理该消息之后，告诉 rabbitmq 它已经处理了，rabbitmq 可以把该消息删除了



### 3.2 自动应答

消息发送后立即被认为已经传送成功，这种模式需要在 **高吞吐量和数据传输安全性方面做权衡**，因为这种模式如果消息在接收到之前，消费者那边出现或者channel关闭，那么消息就丢失了，当然另一方面这种模式消费者那边可以传递过载信息， **没有对传递的消息数量进行限制**，当然这样有可能使得消费者这边由于接收太多还来不及处理的消息，导致这些消息的积压，最终使得内存耗尽，最终这些消费者线程被操作系统杀死，**所以这种模式仅适用在消费者可以高效并以 某种速率能够处理这些消息的情况下使用。**



### 3.3 手动应答方式

`channel.basicAck`（肯定确认应答）

```java
basicAck(long deliveryTag, boolean multiple);
```

第一个参数是消息的标记，第二个参数表示是否应用于多消息，RabbitMQ已经知道该消息成功处理，那么就可以将其丢弃



`channel.basicReject`（否定确认应答）

```java
basicReject(long deliveryTag, boolean requeue)
```

第一个参数表示拒绝 `deliveryTag` 对应的消息，第二个参数表示是否 `requeue`：true 则重新入队列，false 则丢弃或者进入死信队列。

该方法 reject 后，该消费者还是会消费到该条被 reject 的消息。



`channel.basicNack`（用于否定确认）：表示己拒绝处理该消息，可以将其丢弃了

```java
basicNack(long deliveryTag, boolean multiple, boolean requeue);
```

第一个参数表示拒绝 `deliveryTag` 对应的消息，第二个参数是表示否应用于多消息，第三个参数表示是否 `requeue`，与 basicReject 区别就是同时支持多个消息，可以 拒绝签收 该消费者先前接收未 ack 的所有消息。拒绝签收后的消息也会被自己消费到。



`Channel.basicRecover`

```java
basicRecover(boolean requeue);
```

是否恢复消息到队列，参数是是否 `requeue`，true 则重新入队列，并且尽可能的将之前 `recover` 的消息投递给其他消费者消费，而不是自己再次消费。false 则消息会重新被投递给自己。



**Multiple 的解释：**

手动应答的好处是可以批量应答并且减少网络拥堵

- true 代表批量应答 channel 上未应答的消息

​		比如说 channel 上有传送 tag 的消息 5,6,7,8 当前 tag 是 8 那么此时 5-8 的这些还未应答的消息都会被确认收到消息应答

- false 同上面相比只会应答 tag=8 的消息 5,6,7 这三个消息依然不会被确认收到消息应答

![image-20241117130329173](.\img\image-20241117130329173.png)



### 3.4 消息自动重新入队

如果消费者由于某些原因失去连接（其通道已关闭，连接已关闭或TCP连接丢失），导致消息未发送ACK确认，RabbitMQ 将了解到消息未完全处理，并将对其重新排队。如果此时其他消费者可以处理，它将很快将其重新分发给另一个消费者。这样，即使某个消费者偶尔死亡，也可以确保不会丢失任何消息。

![image-20241117130928160](.\img\image-20241117130928160.png)



### 3.5 手动应答案例

默认消息采用的是自动应答，所以我们要想实现消息消费过程中不丢失，需要把自动应答改为手动应答

![image-20241117132159712](.\img\image-20241117132159712.png)

```java
// 生产者
public class Task02 {

    public static final String TASK_QUEUE_NAME = "ACK_QUEUE";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();

        channel.queueDeclare(TASK_QUEUE_NAME, false, false, false, null);
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入信息：");
        while (scanner.hasNext()){
            String message = scanner.next();
            channel.basicPublish("",TASK_QUEUE_NAME,null,message.getBytes("UTF-8"));
            System.out.println("生产者发出消息:"+ message);
        }
    }
}

// 消费者
public class Work03 {
    //队列名称
    public static final String TASK_QUEUE_NAME = "ACK_QUEUE";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();

        System.out.println("worker03等待接受消息处理时间较短");

        DeliverCallback deliverCallback =(consumerTag, message) ->{

            //沉睡1S
            SleepUtil.sleep(1);
            System.out.println("接受到的消息:"+new String(message.getBody(),"UTF-8"));
            //手动应答
            /**
             * 1.消息的标记Tag
             * 2.是否批量应答 false表示不批量应答信道中的消息
             */
            channel.basicAck(message.getEnvelope().getDeliveryTag(),false);

        };

        CancelCallback cancelCallback = (consumerTag -> {
            System.out.println(consumerTag + "消费者取消消费接口回调逻辑");
        });

        //采用手动应答
        boolean autoAck = false;
        channel.basicConsume(TASK_QUEUE_NAME,autoAck,deliverCallback,cancelCallback);
    }
}

public class Work04 {
    //队列名称
    public static final String TASK_QUEUE_NAME = "ACK_QUEUE";

    //接受消息
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();
        System.out.println("worker04等待接受消息处理时间较长");

        DeliverCallback deliverCallback =(consumerTag,message) ->{

            //沉睡1S
            SleepUtil.sleep(30);
            System.out.println("接受到的消息:"+new String(message.getBody(),"UTF-8"));
            //手动应答
            /**
             * 1.消息的标记Tag
             * 2.是否批量应答 false表示不批量应答信道中的消息
             */
            channel.basicAck(message.getEnvelope().getDeliveryTag(),false);

        };

        CancelCallback cancelCallback = (consumerTag -> {
            System.out.println(consumerTag + "消费者取消消费接口回调逻辑");

        });
        //采用手动应答
        boolean autoAck = false;
        channel.basicConsume(TASK_QUEUE_NAME,autoAck,deliverCallback,cancelCallback);
    }
}

```

首先发送 `aa` 和 `bb`，`worker03`在 1s 内就接收到，`worker04`等待 30s 后才接收到。再次发送 `cc、dd`，并手动关闭 `worker04` 进程，可以见到消息 `dd` 被转发给了 `worker03`

![image-20241117140156461](.\img\image-20241117140156461.png)



### 3.6 RabbitMQ持久化

当 RabbitMQ 服务停掉以后，消息生产者发送过来的消息不丢失要如何保障？默认情况下 RabbitMQ 退出或由于某种原因崩溃时，它忽视队列和消息，除非告知它不要这样做。确保消息不会丢失需要做两件事：**我们需要将队列和消息都标记为持久化**

#### 3.6.1 队列持久化

将 `durable` 设置为`true`，队列持久化。

```java
public class Task02 {

    public static final String TASK_QUEUE_NAME = "ACK_QUEUE";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();

        boolean durable = true; // 让队列持久化

        channel.queueDeclare(TASK_QUEUE_NAME, durable, false, false, null);
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入信息：");
        while (scanner.hasNext()){
            String message = scanner.next();
            channel.basicPublish("",TASK_QUEUE_NAME,null,message.getBytes("UTF-8"));
            System.out.println("生产者发出消息:"+ message);
        }
    }
}
```

由于队列名称为 `ACK_QUEUE` ，在手动应答案例中已将被创建，且未被持久化，会发生报错，所以需要将之前的队列删除。

重新生成队列后，如下，`D` 代表持久化，虚拟机经过充气后仍然存在

![image-20241117143123368](.\img\image-20241117143123368.png)



#### 3.6.2 消息持久化

想让消息持久化需要修改消息生产者的代码，添加`MessageProperties.PERSISTENT_TEXT_PLAIN`

```java
channel.basicPublish("",TASK_QUEUE_NAME,MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes("UTF-8"));
```

将消息持久化并不能完全保证不会丢失消息。尽管它告诉 RabbitMQ 将消息保存到磁盘，但是这里依然存在当消息刚准备存储在磁盘的时候 但是还没有存储完，消息还在缓存的一个间隔点。此时并没 有真正写入磁盘。持久性保证并不强，但是对于我们的简单任务队列而言，这已经绰绰有余了。

**更加强有力的持久化策略在发布确认章节**



#### 3.6.3 basicQos

在最开始的时候我们学习到 RabbitMQ 分发消息采用的轮询分发，但是在某种场景下这种策略并不是很好，比方说有两个消费者在处理任务，其中有个**消费者 1** 处理任务的速度非常快，而另外一个**消费者 2** 处理速度却很慢，这个时候我们还是采用轮询分发的化就会到这处理速度快的这个消费者很大一部分时间处于空闲状态，而处理慢的那个消费者一直在干活，这种分配方式在这种情况下其实就不太好。

**公平分发一定要手动应答**

为了避免这种情况，**在消费者中消费消息之前**，设置参数 `channel.basicQos();`

```
// 需要设置在消费者端
// consumer 03 处理时间是5s
channel.basicQos(2);
// consumer 04 处理时间是30s
channel.basicQos(5);

// 处理流程
// 在0~5s内，task02连续发送了7条消息，其中两条被03接收，5条被04接收，如果在处理期间某个消费者断线，那么其正在处理和未处理的消息会被转发到其他消费者，若消费者再次恢复，如果另一个消费者的消息队列已满，且还有未处理的消息，则会再次分配给回复的消费者
```

![image-20241126212611720](.\img\image-20241126212611720.png)



## 四、RabbitMQ发布确认

### 4.1 发布确认原理

![image-20241126222952065](.\img\image-20241126222952065.png)

生产者发布消息到 RabbitMQ 后，需要 RabbitMQ 返回「ACK（已收到）」给生产者，这样生产者才知道自己生产的消息成功发布出去

**开启发布确认**

```java
//开启发布确认
channel.confirmSelect();
```



### 4.2 发布确认的策略

#### 4.2.1 单个发布确认

这是一种 **同步确认发布** 的方式，也就是发布一个消息后只有它被确认发布，后续的消息才能继续发布，`waitForConfirmsOrDie(long)` 这个方法只有在消息被确认的时候才返回，如果在指定时间范围内这个消息没有被确认那么它将抛出异常。

这种确认方式有一个最大的缺点就是：**发布速度特别的慢**，因为如果没有确认发布的消息就会阻塞所有后续消息的发布，这种方式最多提供每秒不超过数百条发布消息的吞吐量。当然对于某些应用程序来说这可能已经足够了。

```java
/**
 * 发布确认模式：
 *  1、单个确认
 *  2、批量确认
 *  3、异步批量确认
 */
public class ConfirmMessage {

    public static final int MESSAGE_COUNT = 1000;


    public static void main(String[] args) throws Exception {
        ConfirmMessage.publishMessageIndividually();
    }

    public static void publishMessageIndividually() throws Exception{
        Channel channel = RabbitMQUtil.getChannel();
        // 队列声明
        String queueName = UUID.randomUUID().toString();
        channel.queueDeclare(queueName, false, false, false, null);

        // 开启发布确认
        channel.confirmSelect();
        // 开始时间
        long begin = System.currentTimeMillis();

        // 批量发消息
        for (int i = 0; i < MESSAGE_COUNT; i++) {
            String message = i + "";
            channel.basicPublish("", queueName, null, message.getBytes());
            // 单个消息就马上进行发布确认
            boolean flag = channel.waitForConfirms();
            if (flag) {
                System.out.println("发消息成功");
            }
        }

        long end = System.currentTimeMillis();
        System.out.println("发布"+MESSAGE_COUNT+"个单独确认消息，耗时:"+(end-begin)+"ms");
    }
}
```

![image-20241127194427924](.\img\image-20241127194427924.png)

#### 4.2.2 批量发布确认

单个确认发布方式 **非常慢** ，与单个等待确认消息相比，先发布一批消息然后一起确认可以极大地提高吞吐量，当然这种方式的 **缺点** 就是：当发生故障导致发布出现问题时，不知道是哪个消息出问题了，我们必须将整个批处理保存在内存中，以记录重要的信息而后重新发布消息。当然这种方案仍然是同步的，也一样阻塞消息的发布。

```java
public static void publishMessageBatch() throws Exception{
    Channel channel = RabbitMQUtil.getChannel();
    // 队列声明
    String queueName = UUID.randomUUID().toString();
    channel.queueDeclare(queueName, false, false, false, null);

    // 开启发布确认
    channel.confirmSelect();
    // 开始时间
    long begin = System.currentTimeMillis();

    // 批量确认消息的大小
    int batchSize = 100;

    // 批量发消息
    for (int i = 0; i < MESSAGE_COUNT; i++) {
        String message = i + "";
        channel.basicPublish("", queueName, null, message.getBytes());
        // 批量确认
        if( ( i + 1 ) % batchSize == 0 ){
            //发布确认
            boolean flag = channel.waitForConfirms();
            if (flag) {
                System.out.println("发消息成功");
            }
        }

    }

    long end = System.currentTimeMillis();
    System.out.println("发布"+MESSAGE_COUNT / batchSize+"个批量确认消息，耗时:"+(end-begin)+"ms");
}
```

![image-20241127194537175](.\img\image-20241127194537175.png)

#### 4.2.3 异步确认发布

![image-20241127194630263](.\img\image-20241127194630263.png)

给每一条消息编号，`broker`接收到消息后会进行应答，**异步通知**，生产者只需一直发送消息

```java
public static void publishMessageAsync() throws Exception{
    Channel channel = RabbitMQUtil.getChannel();
    // 队列声明
    String queueName = UUID.randomUUID().toString();
    channel.queueDeclare(queueName, false, false, false, null);

    // 开启发布确认
    channel.confirmSelect();
    // 开始时间
    long begin = System.currentTimeMillis();

    //消息确认回调的函数
    ConfirmCallback ackCallback = (deliveryTag, multiple) ->{
        System.out.println("确认的消息:" + deliveryTag);
    };
    ConfirmCallback nackCallback= (deliveryTag,multiple) ->{
        System.out.println("未确认的消息:" + deliveryTag);
    };
    channel.addConfirmListener(ackCallback,nackCallback);//异步通知


    // 批量发消息
    for (int i = 0; i < MESSAGE_COUNT; i++) {
        String message = i + "";
        channel.basicPublish("", queueName, null, message.getBytes());

    }

    long end = System.currentTimeMillis();
    System.out.println("发布"+MESSAGE_COUNT+"个异步发布确认消息，耗时:"+(end-begin)+"ms");

}
```



![image-20241127200812355](.\img\image-20241127200812355.png)

从结果看到，1000条消息已经发完，但是监听器还在监听

#### 4.2.5 处理异步未确认消息

最好的解决的解决方案就是把未确认的消息放到一个基于内存的能被发布线程访问的队列，比如说用 `ConcurrentLinkedQueue` 这个队列在 `confirm` 和`callbacks `与发布线程之间进行消息的传递。

```java
public static void publishMessageAsync() throws Exception{
    Channel channel = RabbitMQUtil.getChannel();
    // 队列声明
    String queueName = UUID.randomUUID().toString();
    channel.queueDeclare(queueName, false, false, false, null);

    // 开启发布确认
    channel.confirmSelect();
    /**
         * 线程安全有序的哈希表 适用于高并发场景
         *  1、轻松地将序号与消息进行关联
         *  2、轻松删除批量条目
         *  3、支持高并发（多线程）
         */
    ConcurrentSkipListMap<Long, String> outstandingConfirms = new ConcurrentSkipListMap<>();

    // 开始时间
    long begin = System.currentTimeMillis();

    //消息确认回调的函数
    ConfirmCallback ackCallback = (deliveryTag, multiple) ->{
        // 删除已确认的消息
        // 如果是批量删除
        if (multiple){
            ConcurrentNavigableMap<Long, String> confirmed = outstandingConfirms.headMap(deliveryTag);
            confirmed.clear();
        }else {
            outstandingConfirms.remove(deliveryTag);
        }
        System.out.println("确认的消息:" + deliveryTag);
    };
    ConfirmCallback nackCallback= (deliveryTag,multiple) ->{
        System.out.println("未确认的消息:" + deliveryTag);
    };
    channel.addConfirmListener(ackCallback,nackCallback);//异步通知


    // 批量发消息
    for (int i = 0; i < MESSAGE_COUNT; i++) {
        String message = i + "";
        channel.basicPublish("", queueName, null, message.getBytes());
        outstandingConfirms.put(channel.getNextPublishSeqNo(), message);
    }

    long end = System.currentTimeMillis();
    System.out.println("发布"+MESSAGE_COUNT+"个异步发布确认消息，耗时:"+(end-begin)+"ms");

}
```



## 五、交换机

### 5.1 概念

RabbitMQ的核心思想是： **生产者生产的消息从不会直接发送到队列**（不指明交换机则使用默认交换机）。实际上，通常消费者甚至都不知道这些消息传递到哪些队列当中。

相反， **生产者只能将消息发送到交换机中** ，交换机的工作内容简单，一方面接受来自生产者的消息，另一方面将消息推入队列。交换机必须确切知道如何处理收到的消息。是把消息推送到队列还是丢弃，由交换机决定。

简单模式、工作模式

![image-20241128174627174](.\img\image-20241128174627174.png)

发布、订阅模式

![image-20241128174704267](.\img\image-20241128174704267.png)



### 5.2 Exchange类型

**直接（direct）**：处理路由键。需要将一个队列绑定到交换机上，要求该消息与一个特定的路由键完全匹配。

**主题（topic）**：

**标题（headers，不常用）**

**扇出（fanout）**



### 5.3 默认exchange

通过空字符串("")进行标识的交换机是默认交换

```java
channel.basicPublish("", TASK_QUEUE_NAME, null, message.getBytes("UTF-8"));
```

第一个参数是交换机的名称。空字符串 表示默认或无名称交换机：消息能路由发送到队列中其实是由 routingKey(bindingkey) 绑定指定的 key



### 5.4  临时队列

队列的名称至关重要，我们需要指定我们的消费者去哪个队列消费消息

每当我们连接到Rabbit时，我们都需要一个全新的空队列，为此我们可以创建一个具有随机队列名称的队列，或者能让服务器为我们选择一个随机队列名称。其次一旦我们断开了消费者的连接，队列将被自动删除。

以下内容先参考 **5.6.2 实战**

程序运行时有 `amg` 开头的两条临时队列，终止进程后，队列自动消除。实际上，在生产者中并未创建队列，是在消费者中创建的队列

![image-20241128193800254](.\img\image-20241128193800254.png)

![image-20241128193722373](.\img\image-20241128193722373.png)

### 5.5 绑定（bingings）

binding 其实是 exchange 和 queue 之间的桥梁，它告诉我们 exchange 和那个队列进行了绑定关系。比如说下面这张图告诉我们的就是 X 与 Q1 和 Q2 进行了绑定

![image-20241128185607973](.\img\image-20241128185607973.png)

**新增一个队列**

![image-20241128185837274](.\img\image-20241128185837274.png)

**新增一个交换机**

![image-20241128185805199](.\img\image-20241128185805199.png)

**与队列进行绑定**

![image-20241128190009569](.\img\image-20241128190009569.png)

可以根据 `RoutingKey` 指定向哪个队列发

![image-20241128190116950](.\img\image-20241128190116950.png)



### 5.6 Fantout Exchange

#### 5.6.1 Fanout介绍

它是将接收到的消息**广播**到他知道的所有队列中，系统中有默认的，也可以自己创建

![image-20241128190545492](.\img\image-20241128190545492.png)



#### 5.6.2 实战

![image-20241128190716388](.\img\image-20241128190716388.png)

##### 5.6.2.1 Fanout消费者

```java
public class ReceiveLogs01 {

    // 交换机名称
    public static final String EXCHANGE_NAME = "logs";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();
        // 声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
        // 声明一个临时队列
        /**
         * 名称随机
         * 当删除的时候队列自动删除
         */
        String queueName = channel.queueDeclare().getQueue();

        // 绑定交换机和队列
        channel.queueBind(queueName, EXCHANGE_NAME, "");
        System.out.println("等待接收消息，把接收到的消息打印在屏幕上...");

        DeliverCallback deliverCallback = (consumerTag, message) ->{
            System.out.println("ReceiveLogs01控制台打印接收到的消息:"+new String(message.getBody(),"UTF-8"));
        };
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> {});
    }
}

```



##### 5.6.2.2 Fanout生产者

```java
public class EmitLog {

    //交换机名称
    public static final String EXCHANGE_NAME = "logs";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();

        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");

        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()) {
            String message = scanner.next();
            channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes("UTF-8"));
            System.out.println("生产者发出消息:"+message);
        }
    }
}

```



##### 5.6.2.3 结果

![image-20241128192804259](.\img\image-20241128192804259.png)

经过实验其实下面一条语句并不是每一个消费者或生产者都要写

```java
channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
```

因为只要有一个声明，其余两个都是在重复声明。因此即使某个生产者或者消费者代码中没有写这该行代码，只要在发送或者接收消息前，有一方能够提前声明交换机就行了。



### 5.7 Direct Exchange

![image-20241129184741082](.\img\image-20241129184741082.png)

![image-20241129184755492](.\img\image-20241129184755492.png)



```java
public class DirectLogs {

    //交换机名称
    public static final String EXCHANGE_NAME = "direct_logs";
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();


        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()){
            String message = scanner.next();
            channel.basicPublish(EXCHANGE_NAME,"error",null,message.getBytes("UTF-8"));
            System.out.println("生产者发出消息:"+message);
        }
    }

}

// 消费者1
public class ReceiveLogsDirect01 {
    public static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();
        // 声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
        // 声明队列
        channel.queueDeclare("console", false, false, false, null);

        // 绑定交换机和队列
        /**
         * 队列名称
         * 交换机名称
         * routingKey
         */
        channel.queueBind("console", EXCHANGE_NAME, "info");
        channel.queueBind("console", EXCHANGE_NAME, "warning");
        System.out.println("等待接收消息，把接收到的消息打印在屏幕上...");

        DeliverCallback deliverCallback = (consumerTag, message) ->{
            System.out.println("ReceiveLogsDirect01控制台打印接收到的消息:"+new String(message.getBody(),"UTF-8"));
        };
        channel.basicConsume("console", true, deliverCallback, consumerTag -> {});
    }
}

// 消费者2
public class ReceiveLogsDirect02 {
    public static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();
        // 声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
        // 声明一个临时队列
        channel.queueDeclare("disk", false, false, false, null);

        // 绑定交换机和队列
        /**
         * 队列名称
         * 交换机名称
         * routingKey
         */
        channel.queueBind("disk", EXCHANGE_NAME, "error");
        System.out.println("等待接收消息，把接收到的消息打印在屏幕上...");

        DeliverCallback deliverCallback = (consumerTag, message) ->{
            System.out.println("ReceiveLogsDirect02控制台打印接收到的消息:"+new String(message.getBody(),"UTF-8"));
        };
        channel.basicConsume("disk", true, deliverCallback, consumerTag -> {});
    }
}
```

设置生产者的路由键为 `info`

![image-20241129185054533](.\img\image-20241129185054533.png)

设置生产者的路由键为 `warning`

![image-20241129185139088](.\img\image-20241129185139088.png)

设置生产者的路由键为 `error`

![image-20241129185220870](.\img\image-20241129185220870.png)



### 5.8 Topics Exchange

交换机想同时发送消息给Q1和Q2，做不到

![image-20241129185412189](.\img\image-20241129185412189.png)

#### 5.8.1 Topic要求

发送到类型是 topic 交换机的消息的 routing_key 不能随意写，必须满足一定的要求，它必须是**一个单词列表**，**以点号分隔开**。这些单词可以是任意单词，最多不能超过 255 个字节

- ***(星号)可以代替一个位置**
- **#(井号)可以替代零个或多个位置**



#### 5.8.2  Topic匹配案例

![image-20241129190145232](.\img\image-20241129190145232.png)

| **例子**                 | **说明**                                   |
| ------------------------ | ------------------------------------------ |
| quick.orange.rabbit      | 被队列 Q1Q2 接收到                         |
| azy.orange.elephant      | 被队列 Q1Q2 接收到                         |
| quick.orange.fox         | 被队列 Q1 接收到                           |
| lazy.brown.fox           | 被队列 Q2 接收到                           |
| lazy.pink.rabbit         | 虽然满足两个绑定但只被队列 Q2 接收一次     |
| quick.brown.fox          | 不匹配任何绑定不会被任何队列接收到会被丢弃 |
| quick.orange.male.rabbit | 是四个单词不匹配任何绑定会被丢弃           |
| lazy.orange.male.rabbit  | 是四个单词但匹配 Q2                        |

**当一个队列绑定键是 #，那么这个队列将接收所有数据，就有点像 fanout 了**

**如果队列绑定键当中没有 # 和 * 出现，那么该队列绑定类型就是 direct 了**



#### 5.8.3 实战

![image-20241129190504412](.\img\image-20241129190504412.png)

![image-20241129190517774](.\img\image-20241129190517774.png)

```java
public class EmitLogTopic {

    private static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();

        HashMap<String, String> bindingKeyMap = new HashMap<>();
        bindingKeyMap.put("quick.orange.rabbit", "被队列 Q1Q2 接收到");
        bindingKeyMap.put("lazy.orange.elephant", "被队列 Q1Q2 接收到");
        bindingKeyMap.put("quick.orange.fox", "被队列 Q1 接收到");
        bindingKeyMap.put("lazy.brown.fox", "被队列 Q2 接收到");
        bindingKeyMap.put("lazy.pink.rabbit", "虽然满足两个绑定但只被队列 Q2 接收一次");
        bindingKeyMap.put("quick.brown.fox", "不匹配任何绑定不会被任何队列接收到会被丢弃");
        bindingKeyMap.put("quick.orange.male.rabbit", "是四个单词不匹配任何绑定会被丢弃");
        bindingKeyMap.put("lazy.orange.male.rabbit", "是四个单词但匹配 Q2");

        for (Map.Entry<String, String> bindingKeyEntry : bindingKeyMap.entrySet()) {
            String routingKey = bindingKeyEntry.getKey();
            String message = bindingKeyEntry.getValue();

            channel.basicPublish(EXCHANGE_NAME, routingKey, null, message.getBytes("UTF-8"));
            System.out.println("生产者发出消息:"+message);
        }

    }
}

// 消费者01
public class ReceiveLogsTopic01 {

    //交换机的名称
    public static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();
        // 声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);

        // 声明队列
        String queueName = "Q1";
        channel.queueDeclare(queueName, false, false, false, null);
        channel.queueBind(queueName, EXCHANGE_NAME, "*.orange.*");

        DeliverCallback deliverCallback = (consumerTag, message) -> {
            System.out.println(new String(message.getBody(),"UTF-8"));
            System.out.println("接收队列："+queueName+"  绑定键:"+message.getEnvelope().getRoutingKey());
        };

        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> {});

    }
}

// 消费者02
public class ReceiveLogsTopic02 {

    //交换机的名称
    public static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();
        // 声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);

        // 声明队列
        String queueName = "Q2";
        channel.queueDeclare(queueName, false, false, false, null);
        channel.queueBind(queueName, EXCHANGE_NAME, "*.*.rabbit");
        channel.queueBind(queueName, EXCHANGE_NAME, "lazy.*");

        DeliverCallback deliverCallback = (consumerTag, message) -> {
            System.out.println(new String(message.getBody(),"UTF-8"));
            System.out.println("接收队列："+queueName+"  绑定键:"+message.getEnvelope().getRoutingKey());
        };

        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> {});

    }
}

```



![image-20241129191928120](.\img\image-20241129191928120.png)





## 六、死信队列

### 6.1 死信的概念

死信队列是指**无法被消费的队列**，producer将消息投递到broker或者直接到queue，consumer从queue取出消息进行消费，但某些时候由于特定的原因导致queue中的某些消息无法被消费，这样的消息如果没有后续的处理，就变成了死信。

应用场景：为了保证订单业务消息数据不丢失，需要用到RabbitMQ的死信队列机制，当消息发生异常时，将将消息投入死信队列。还有比如说：用户在商城下单成功并点击去支付后在指定时间未支付时自动失效。



### 6.2 死信的来源

- 消息 TTL 过期

  TTL是 Time To Live 的缩写, 也就是生存时间

- 队列达到最大长度

​		队列满了，无法添加数据到MQ中

- 消息被拒绝

  (basic.reject 或 basic.nack) 并且 requeue = false



### 6.3 实战

#### 6.3.1 架构图

![image-20241130192858033](.\img\image-20241130192858033.png)



#### 6.3.2 消息TTL过期

```java
public class Consumer01 {

    // 普通交换机
    private static final String NORMAL_EXCHANGE = "normal_exchange";
    // 死信交换机
    private static final String DEAD_EXCHANGE = "dead_exchange";
    //普通队列的名称
    public static final String NORMAL_QUEUE = "normal_queue";
    //死信队列的名称
    public static final String DEAD_QUEUE = "dead_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();

        channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
        channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);

		// 不可以随意更改 "x-dead-letter-exchange" 和 "x-dead-letter-routing-key" 
        HashMap<String, Object> arguments = new HashMap<>();
        // 正常的队列设置死信交换机
        arguments.put("x-dead-letter-exchange", DEAD_EXCHANGE);
        // 设置死信的routingKey
        arguments.put("x-dead-letter-routing-key", "lisi");

        // 声明normal和dead队列
        channel.queueDeclare(NORMAL_QUEUE, false, false, false, arguments);
        channel.queueDeclare(DEAD_QUEUE, false, false, false, null);

        channel.queueBind(NORMAL_QUEUE, NORMAL_EXCHANGE, "zhangsan");
        channel.queueBind(DEAD_QUEUE, DEAD_EXCHANGE, "lisi");

        System.out.println("等待接收消息...");
        DeliverCallback deliverCallback = (consumerTag, message) ->{
            System.out.println("Consumer01接受的消息是："+new String(message.getBody(),"UTF-8"));
        };
        channel.basicConsume(NORMAL_QUEUE, true, deliverCallback, consumerTag -> {});
    }
}

public class Consumer02 {
    //死信队列的名称
    public static final String DEAD_QUEUE = "dead_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();
        System.out.println("等待接收死信消息...");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println("Consumer02接受的消息是：" + message);
        };
        channel.basicConsume(DEAD_QUEUE, true, deliverCallback, consumerTag -> {});
    }
}


public class Producer {
    private static final String NORMAL_EXCHANGE = "normal_exchange";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();

        // 死信消息 设置ttl时间 单位ms
        AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().expiration("10000").build();
        for (int i = 0; i < 11; i++) {
            String message = "info" + i;
            channel.basicPublish(NORMAL_EXCHANGE, "zhangsan", properties, message.getBytes());
        }
    }

}
```



正常运行

![image-20241130201651327](.\img\image-20241130201651327.png)

关闭Consumer01进程，10s后出现以下结果

![image-20241130201511692](.\img\image-20241130201511692.png)



#### 6.3.3 死信最大长度

```java
public class Producer {
    private static final String NORMAL_EXCHANGE = "normal_exchange";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();
        for (int i = 0; i < 11; i++) {
            String message = "info" + i;
            channel.basicPublish(NORMAL_EXCHANGE, "zhangsan", null, message.getBytes());
        }
    }
}

// consumer01中添加  arguments.put("x-max-length", 6);  02不做修改
public class Consumer01 {

    // 普通交换机
    private static final String NORMAL_EXCHANGE = "normal_exchange";
    // 死信交换机
    private static final String DEAD_EXCHANGE = "dead_exchange";
    //普通队列的名称
    public static final String NORMAL_QUEUE = "normal_queue";
    //死信队列的名称
    public static final String DEAD_QUEUE = "dead_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();

        channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
        channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);


        // 不可以随意更改 "x-dead-letter-exchange" 和 "x-dead-letter-routing-key"
        HashMap<String, Object> arguments = new HashMap<>();
        // 正常的队列设置死信交换机
        arguments.put("x-dead-letter-exchange", DEAD_EXCHANGE);
        // 设置死信的routingKey
        arguments.put("x-dead-letter-routing-key", "lisi");
        // 队列达到最大长度
        arguments.put("x-max-length", 6);


        // 声明normal和dead队列
        channel.queueDeclare(NORMAL_QUEUE, false, false, false, arguments);
        channel.queueDeclare(DEAD_QUEUE, false, false, false, null);

        channel.queueBind(NORMAL_QUEUE, NORMAL_EXCHANGE, "zhangsan");
        channel.queueBind(DEAD_QUEUE, DEAD_EXCHANGE, "lisi");

        System.out.println("等待接收消息...");
        DeliverCallback deliverCallback = (consumerTag, message) ->{
            System.out.println("Consumer01接受的消息是："+new String(message.getBody(),"UTF-8"));
        };
        channel.basicConsume(NORMAL_QUEUE, true, deliverCallback, consumerTag -> {});
    }
}
```



需要把消费者01和02都关掉，这边 `arguments.put("x-max-length", 6);` 表示消息积压的数量，并不是说消费六条消息后就将其余消息放到死信队列，如果开启01，积压的消息会被立刻消费掉，如果有新的消息进来，01仍可继续消费，02同理。

![image-20241130203035970](.\img\image-20241130203035970.png)



#### 6.3.4 死信消息被拒

需求：消费者 C1 拒收消息 "info5"，开启手动应答

```
// 还要修改channel.basicConsume(NORMAL_QUEUE, false, deliverCallback, consumerTag -> {});中的autoAck
public class Consumer01 {

    // 普通交换机
    private static final String NORMAL_EXCHANGE = "normal_exchange";
    // 死信交换机
    private static final String DEAD_EXCHANGE = "dead_exchange";
    //普通队列的名称
    public static final String NORMAL_QUEUE = "normal_queue";
    //死信队列的名称
    public static final String DEAD_QUEUE = "dead_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();

        channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
        channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);


        // 不可以随意更改 "x-dead-letter-exchange" 和 "x-dead-letter-routing-key"
        HashMap<String, Object> arguments = new HashMap<>();
        // 正常的队列设置死信交换机
        arguments.put("x-dead-letter-exchange", DEAD_EXCHANGE);
        // 设置死信的routingKey
        arguments.put("x-dead-letter-routing-key", "lisi");
        // 队列达到最大长度
//        arguments.put("x-max-length", 6);


        // 声明normal和dead队列
        channel.queueDeclare(NORMAL_QUEUE, false, false, false, arguments);
        channel.queueDeclare(DEAD_QUEUE, false, false, false, null);

        channel.queueBind(NORMAL_QUEUE, NORMAL_EXCHANGE, "zhangsan");
        channel.queueBind(DEAD_QUEUE, DEAD_EXCHANGE, "lisi");

        System.out.println("等待接收消息...");
        DeliverCallback deliverCallback = (consumerTag, message) ->{
//            System.out.println("Consumer01接受的消息是："+new String(message.getBody(),"UTF-8"));
            String msg = new String(message.getBody(), "UTF-8");
            if(msg.equals("info5")){
//                System.out.println("Consumer01接受的消息是："+msg+"： 此消息是被C1拒绝的");
                //requeue 设置为 false 代表拒绝重新入队 该队列如果配置了死信交换机将发送到死信队列中
                channel.basicReject(message.getEnvelope().getDeliveryTag(), false);
            }else {
                System.out.println("Consumer01接受的消息是："+msg);
                 channel.basicAck(message.getEnvelope().getDeliveryTag(), false);
            }
        };
        //开启自动应答，也就是关闭手动应答
        channel.basicConsume(NORMAL_QUEUE, false, deliverCallback, consumerTag -> {});
    }
}
```

![image-20241130204346395](.\img\image-20241130204346395.png)



## 七、延时队列

### 7.1 延时队列概念

延时队列内部是有序的，最重要的特性就体现在它的延时属性上，延时队列中的元素是希望 **在指定时间到了以后或之前取出和处理**，延时队列就是用来存放需要在指定时间被处理的元素的队列

**延时队列使用场景**

1. 订单在十分钟之内未支付则自动取消
2. 新建的店铺，如果在十天内都没有上传过商品，则自动发送消息提醒
3. 用户注册成功以后如果三天内没有登录则进行短信提醒
4. 用户发起退款，如果三天内没有得到处理则通知相关运营人员
5. 预定会议后，需要在预定的时间点前十分钟通知各个与会人员参加会议



这些场景都有一个特点，需要在某个事件发生之后或者之前的指定时间点完成某一项任务，如： 发生订单生成事件，在十分钟之后检查该订单支付状态，然后将未支付的订单进行关闭；那我们一直轮询数据，每秒查一次，取出需要被处理的数据，然后处理不就完事了吗？

**如果数据量比较少，确实可以这样做**，比如：对于「如果账单一周内未支付则进行自动结算」这样的需求， 如果对于时间不是严格限制，而是宽松意义上的一周，那么每天晚上跑个定时任务检查一下所有未支付的账单，确实也是一个可行的方案。但对于数据量比较大，并且时效性较强的场景，如：「订单十分钟内未支付则关闭」，短期内未支付的订单数据可能会有很多，活动期间甚至会达到百万甚至千万级别，对这么庞大的数据量仍旧使用轮询的方式显然是不可取的，很可能在一秒内无法完成所有订单的检查，同时会给数据库带来很大压力，无法满足业务要求而且性能低下。



### 7.2 队列TTL

#### 7.2.1 架构图

![image-20241130211640172](.\img\image-20241130211640172.png)

 创建两个队列QA和QB，两者队列TTL分别设置10s和40s，然后在创建一个交换机X和死信交换机Y，他们的类型都是direct，创建一个死信队列QD



#### 7.1.1 配置文件类代码

```java
@Configuration
public class TTLQueueConfig {

    // 死信交换机
    private static final String Y_EXCHANGE = "Y";
    // 普通交换机
    private static final String X_EXCHANGE = "X";

    // 创建队列QA和AB
    private static final String QA_QUEUE = "QA";
    private static final String QB_QUEUE = "QB";
    private static final String QD_QUEUE = "QD";

    @Bean("xExchange")
    public DirectExchange xExchange() {
        return new DirectExchange(X_EXCHANGE);
    }

    @Bean("yExchange")
    public DirectExchange yExchange() {
        return new DirectExchange(Y_EXCHANGE);
    }

    @Bean("queueA")
    public Queue queueA() {
        HashMap<String, Object> arguments = new HashMap<>(3);

        arguments.put("x-dead-letter-exchange", Y_EXCHANGE);
        arguments.put("x-dead-letter-routing-key", "YD");
        arguments.put("x-message-ttl", 10000);

        return QueueBuilder.durable(QA_QUEUE).withArguments(arguments).build();

    }

    @Bean("queueB")
    public Queue queueB() {
        HashMap<String, Object> arguments = new HashMap<>(3);

        arguments.put("x-dead-letter-exchange", Y_EXCHANGE);
        arguments.put("x-dead-letter-routing-key", "YD");
        arguments.put("x-message-ttl", 40000);
        return QueueBuilder.durable(QB_QUEUE).withArguments(arguments).build();
    }

    @Bean("queueD")
    public Queue queueD() {
        return QueueBuilder.durable(QD_QUEUE).build();
    }

    @Bean
    public Binding queueABindingX(@Qualifier("queueA") Queue queueA,@Qualifier("xExchange") DirectExchange xExchange) {
        return BindingBuilder.bind(queueA).to(xExchange).with("XA");
    }

    @Bean
    public Binding queueBBindingX(@Qualifier("queueB") Queue queueB,@Qualifier("xExchange") DirectExchange xExchange) {
        return BindingBuilder.bind(queueB).to(xExchange).with("XB");
    }

    @Bean
    public Binding queueDBindingY(@Qualifier("queueD") Queue queueD,@Qualifier("yExchange") DirectExchange yExchange) {
        return BindingBuilder.bind(queueD).to(yExchange).with("YD");
    }
}


@Slf4j
@Component
public class DeadLetterQueueConsumer {

    @RabbitListener(queues = "QD")
    public void receiveD(Message message, Channel channel) {
        String msg = new String(message.getBody());
        log.info("time {}, received dead letter message:{}", new Date().toString(),msg);

    }
    
    
//    @RabbitListener(queues = "QA")
//    public void receiveA(Message message, Channel channel) {
//        String msg = new String(message.getBody());
//        log.info("time {}, received dead letter message:{}", new Date().toString(),msg);
//    }
//
//    @RabbitListener(queues = "QB")
//    public void receiveB(Message message, Channel channel) {
//        String msg = new String(message.getBody());
//        log.info("time {}, received dead letter message:{}", new Date().toString(),msg);
//    }
}


@Slf4j
@RestController
@RequestMapping("/ttl")
public class SendMsgController {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/sendMsg/{message}")
    public void sendMsg(@PathVariable("message") String message) {
        log.info("当前时间:{},发送一条信息给两个TTL队列：{}",new Date().toString(),message);

        rabbitTemplate.convertAndSend("X", "XA", "消息来自ttl为10s的队列:"+message);
        rabbitTemplate.convertAndSend("X", "XB", "消息来自ttl为40s的队列:"+message);
    }

}
```



先注释掉 `@RabbitListener(queues = "QA")` 和 `@RabbitListener(queues = "QB")` ，由于两条消息不能被及时消费，分别过了10s和40s后才被死信队列消费

![image-20241201191653285](.\img\image-20241201191653285.png)

不注释的话，两条消息被立即消费



### 7.3 延时队列优化

如果像上述代码这样使用的话，岂不是每增加一个新的时间需求，就要新增一个队列，这里只有 10S 和 40S 两个时间选项，如果需要一个小时后处理，那么就需要增加 TTL 为一个小时的队列，如果是预定会议室然后提前通知这样的场景，岂不是要增加无数个队列才能满足需求？

#### 7.3.1 代码架构图

![image-20241201192025416](.\img\image-20241201192025416.png)

#### 7.3.2 配置文件类代码

当时写的时候会有疑惑，为什么 `arguments.put("x-dead-letter-routing-key","YD")` 中必须是 `YD` ，其实流程是这样的，TTL时间到了，根据 `x-dead-letter-exchange` 发送给死信交换机，死信交换机根据 `routingKey` 发给死信队列

```java
@Configuration
public class MsgTTLQueueConfig {

    //普通队列的名称
    public static final String QUEUE_C = "QC";

    //死信交换机的名称
    public static final String Y_DEAD_LETTER_EXCHANGE="Y";

    //声明QC
    @Bean("queueC")
    public Queue QueueC(){
        Map<String,Object> arguments = new HashMap<>(3);
        //设置死信交换机
        arguments.put("x-dead-letter-exchange",Y_DEAD_LETTER_EXCHANGE);
        //设置死信RoutingKey
        arguments.put("x-dead-letter-routing-key","YD");
        return QueueBuilder.durable(QUEUE_C).withArguments(arguments).build();
    }

    //声明队列 QC 绑定 X 交换机
    @Bean
    public Binding queueCBindingX(@Qualifier("queueC") Queue queueC,
                                  @Qualifier("xExchange") DirectExchange xExchange){
        return BindingBuilder.bind(queueC).to(xExchange).with("XC");
    }
}


//开始发消息 发TTL
    @GetMapping("/sendExpirationMsg/{message}/{ttlTime}")
    public void sendExpirationMsg(@PathVariable("message") String message, @PathVariable("ttlTime") String ttlTime) {
        log.info("当前时间:{},发送一条时长是{}毫秒TTL信息给队列QC：{}",
                new Date().toString(),ttlTime,message);
        rabbitTemplate.convertAndSend("X", "XC", message, msg -> {
            //发送消息的时候的延迟时长
            msg.getMessageProperties().setExpiration(ttlTime);
            return msg;
        });
    }
```

在浏览器中依次请求

[localhost:8888/ttl/sendExpirationMsg/你好2/20000](http://localhost:8888/ttl/sendExpirationMsg/你好2/2000)

[localhost:8888/ttl/sendExpirationMsg/你好2/2000](http://localhost:8888/ttl/sendExpirationMsg/你好2/2000)

![image-20241201195349660](.\img\image-20241201195349660.png)

发现即使第二条消息的时延短，它也在20s后被消费，这是因为他排在第一条请求之后，要等第一条请求被消费了才能被消费



### 7.4 插件实现延时队列

#### 7.4.1 安装延时插件

![image-20241202181702921](.\img\image-20241202181702921.png)

选择releases，选择版本下载

![image-20241202181735429](.\img\image-20241202181735429.png)

![image-20241202184206137](.\img\image-20241202184206137.png)

最后可以看到交换机的类型多了一种

![image-20241202184245747](.\img\image-20241202184245747.png)



不使用插件的延时设置在队列，使用插件后是在交换机中

![image-20241202184439104](.\img\image-20241202184439104.png)



#### 7.4.2 代码架构图

![image-20241202184702188](.\img\image-20241202184702188.png)



#### 7.4.3 配置文件类代码

```java
@Configuration
public class DelayedQueueConfig {

    public static final String DELAYED_QUEUE_NAME = "delayed.queue";
    public static final String DELAYED_EXCHANGE_NAME = "delayed.exchange";
    public static final String DELAYED_ROUTING_KEY = "delayed.routingKey";

    @Bean
    public Queue delayedQueue() {
        return new Queue(DELAYED_QUEUE_NAME);
    }

    @Bean
    public CustomExchange delayedExchange() {
        Map<String, Object> arguments = new HashMap<>();
        arguments.put("x-delayed-type", "direct");
        /**
         * 交换机名称
         * 交换机类型 x-delayed-type
         * 是否需要持久化
         * 是否需要自动删除
         * 其他参数
         */
        return new CustomExchange(DELAYED_EXCHANGE_NAME, "x-delayed-message", true, false, arguments);
    }

    @Bean
    public Binding delayedQueueBindingDelayedExchange(@Qualifier("delayedQueue") Queue delayedQueue, @Qualifier("delayedExchange") CustomExchange delayedExchange ) {
        // .noargs()：表示在创建绑定时不需要额外的参数
        return BindingBuilder.bind(delayedQueue).to(delayedExchange).with(DELAYED_ROUTING_KEY).noargs();
    }

}

// 监听器中
    @RabbitListener(queues = DelayedQueueConfig.DELAYED_QUEUE_NAME)
    public void receiveDelayedQueue(Message message){
        String msg = new String(message.getBody());
        log.info("当前时间：{},收到延时队列的消息：{}", new Date().toString(), msg);
    }
    
    // 生产者
    @GetMapping("/sendDelayMsg/{message}/{delayTime}")
    public void sendMsg(@PathVariable("message") String message, @PathVariable("delayTime") Integer  delayTime){
        log.info("当前时间:{},发送一条时长是{}毫秒TTL信息给延迟队列delayed.queue：{}",
                new Date().toString(),delayTime,message);

        rabbitTemplate.convertAndSend(DelayedQueueConfig.DELAYED_EXCHANGE_NAME, DelayedQueueConfig.DELAYED_ROUTING_KEY, message, msg -> {
            //发送消息的时候的延迟时长 单位ms
            msg.getMessageProperties().setDelay(delayTime);
            return msg;
        });
    }    
```

`CustomExchange`不是一种固定的类型，而是用来配合插件一起使用的。它允许开发者根据特定的插件和业务需求来实现自定义的交换机类型和行为

![image-20241202191922300](.\img\image-20241202191922300.png)



### 7.5 总结

延时队列在需要延时处理的场景下非常有用，使用 RabbitMQ 来实现延时队列可以很好的利用 RabbitMQ 的特性，如：消息可靠发送、消息可靠投递、死信队列来保障消息至少被消费一次以及未被正确处理的消息不会被丢弃。另外，通过 RabbitMQ 集群的特性，可以很好的解决单点故障问题，不会因为单个节点挂掉导致延时队列不可用或者消息丢失。

当然，延时队列还有很多其它选择，比如利用 Java 的 DelayQueue，利用 Redis 的 zset，利用 Quartz 或者利用 kafka 的时间轮，这些方式各有特点,看需要适用的场景。



## 八、RabbitMQ发布确认高级

在生产环境中由于一些不明原因，导致rabbitmq重启，在RabbitMQ重启期间生产者投递消息失败，导致消息丢失，需要手动处理和恢复；RabbitMQ集群不可用的时候，无法投递消息该怎么处理？



### 8.1 发布确认springboot版本

#### 8.1.1 确认机制方案

首先发布确认后进行备份，放入缓存，如果消息成功发布确认到交换机，则从缓存删除该消息，如果没有发布成功，则设定一个定时任务，重新从缓存里获取消息并发布到交换机



![image-20241202193625583](.\img\image-20241202193625583.png)

#### 8.1.2 代码架构图

![image-20241202193916977](.\img\image-20241202193916977.png)



#### 8.1.3 配置文件类

在配置文件中添加

```yml
server:
  port: 8888
spring:
  rabbitmq:
    host: 192.168.91.200
    port: 5672
    username: root
    password: 123
    publisher-confirm-type: correlated
```

- `NONE` 值是禁用发布确认模式，是默认值
- `CORRELATED` 值是发布消息成功到交换器后会触发回调方法
- `SIMPLE` 值经测试有两种效果，其一效果和 CORRELATED 值一样会触发回调方法，其二在发布消息成功后使用 rabbitTemplate 调用 waitForConfirms 或 waitForConfirmsOrDie 方法等待 broker 节点返回发送结果，根据返回结果来判定下一步的逻辑，要注意的点是 waitForConfirmsOrDie 方法如果返回 false 则会关闭 channel，则接下来无法发送消息到 broker

```java
@Configuration
public class ConfirmConfig {

    public static final String CONFIRM_QUEUE = "confirm.queue";
    public static final String CONFIRM_EXCHANGE = "confirm.exchange";
    public static final String CONFIRM_ROUTINGKEY = "key1";

    @Bean
    public Queue confirmQueue() {
        return new Queue(CONFIRM_QUEUE);
    }

    @Bean
    public DirectExchange confirmExchange() {
        return new DirectExchange(CONFIRM_EXCHANGE);
    }

    @Bean
    public Binding confirmBinding(@Qualifier("confirmQueue") Queue confirmQueue, @Qualifier("confirmExchange") DirectExchange confirmExchange) {
        return BindingBuilder.bind(confirmQueue).to(confirmExchange).with(CONFIRM_ROUTINGKEY);
    }
}


@Slf4j
@Component
public class MyCallBack implements RabbitTemplate.ConfirmCallback {

    @Autowired
    private RabbitTemplate rabbitTemplate;
    // 当Spring容器完成Bean的装配（包括属性注入和依赖注入）后，它会寻找所有带有@PostConstruct注解的方法，并按声明的顺序执行它们。
    @PostConstruct
    public void init() {
        rabbitTemplate.setConfirmCallback(this);
    }
    /**
     * 交换机确认回调方法
     * 1.发消息 交换机收到了  回调
     *  1.1 correlationData 保存回调消息的ID及相关信息
     *  1.2 交换机收到消息 ack=true
     *  1.3 cause null
     * 2.发消息  交换机接收失败了  回调
     *  2.1 correlationData 保存回调消息的ID及相关信息
     *  2.2 交换机收到消息  ack=false
     *  2.3 cause失败的原因
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        String id = correlationData != null ? correlationData.getId() : "";
        if (ack){
            log.info("交换机已经收到ID为{}的消息", id);
        }else {
            log.info("交换机还未收到ID为:{}的消息，由于原因:{}",id,cause);
        }
    }
}


    @RabbitListener(queues = ConfirmConfig.CONFIRM_QUEUE)
    public void receiveConfirmMessage(Message message){
        String msg = new String(message.getBody());
        log.info("接受到的队列confirm.queue消息:{}",msg);
    }
    
    
    @GetMapping("/send/{message}")
    public void sendMessage(@PathVariable("message") String message){
        CorrelationData correlationData1 = new CorrelationData("1");
        rabbitTemplate.convertAndSend(ConfirmConfig.CONFIRM_EXCHANGE, ConfirmConfig.CONFIRM_ROUTINGKEY, message + "key1", correlationData1);
        log.info("发送消息内容:{}",message+"key1");

        CorrelationData correlationData2 = new CorrelationData("2");
        String CONFIRM_ROUTING_KEY = "key2";
        rabbitTemplate.convertAndSend(ConfirmConfig.CONFIRM_EXCHANGE, CONFIRM_ROUTING_KEY, message + "key2", correlationData2);
        log.info("发送消息内容:{}",message+"key2");

        CorrelationData correlationData3 = new CorrelationData("3");
        rabbitTemplate.convertAndSend(ConfirmConfig.CONFIRM_EXCHANGE + "123", ConfirmConfig.CONFIRM_ROUTINGKEY, message + "key3", correlationData3);
        log.info("发送消息内容:{}",message+"key3");
    }
```

在浏览器键入`http://localhost:8888/ttl/send/大家好1`

![image-20241203175012065](.\img\image-20241203175012065.png)

消息一：对应的交换机和路由键均已存在，所以成功

消息二：交换机名称正确，但路由键名字不正确，不存在以该路由键为名称的消息队列，但是回调结果 `ack=true` ，没有被消费

消息三：交换机名称错误，回调 `ack=false`



### 8.2 回退消息

#### 8.2.1 Mandatory参数

在仅仅开生产者确认机制的情况下，交换机接收到消息后，会直接给消息生产者发送确认消息，但**如果消息不可路由，那么消息就会被丢弃，正如上述消息二，此时生产者是不知道消息被丢弃这个事件的**。通过设置 `mandatory` 参数可以当消息传递过程中不可达目的时将消息返回给生产者。

配置文件开启

```yml
# 新版
spring:
  rabbitmq:
  	template:
      mandatory: true
      
# 旧版
spring:
  rabbitmq:
    mandatory: true

```

代码中开启

```java
rabbitTemplate.setMandatory(true);
```



#### 8.2.2 实战

**修改配置文件**

```
spring:
    rabbitmq:
        host: 192.168.91.200
        port: 5672
        username: root
        password: 123
        publisher-confirm-type: correlated
        publisher-returns: true
        template:
            mandatory: true
server:
    port: 8888

```

**修改回调接口**

```java
@Slf4j
@Component
public class MyCallBack implements RabbitTemplate.ConfirmCallback, RabbitTemplate.ReturnsCallback {

    @Autowired
    private RabbitTemplate rabbitTemplate;
    // 当Spring容器完成Bean的装配（包括属性注入和依赖注入）后，它会寻找所有带有@PostConstruct注解的方法，并按声明的顺序执行它们。
    @PostConstruct
    public void init() {
        rabbitTemplate.setConfirmCallback(this);
        rabbitTemplate.setReturnsCallback(this);
    }
    /**
     * 交换机确认回调方法
     * 1.发消息 交换机收到了  回调
     *  1.1 correlationData 保存回调消息的ID及相关信息
     *  1.2 交换机收到消息 ack=true
     *  1.3 cause null
     * 2.发消息  交换机接收失败了  回调
     *  2.1 correlationData 保存回调消息的ID及相关信息
     *  2.2 交换机收到消息  ack=false
     *  2.3 cause失败的原因
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        String id = correlationData != null ? correlationData.getId() : "";
        if (ack){
            log.info("交换机已经收到ID为{}的消息", id);
        }else {
            log.info("交换机还未收到ID为:{}的消息，由于原因:{}",id,cause);
        }
    }

    // 可以当消息在传递过程中不可达目的地时将消息返回给生产者
    // 只有不可达目的时 才进行回退

    /**
     * 当消息无法路由的时候回调
     * message      消息
     * replyCode    编码
     * replyText    退回原因
     * exchange     从哪个交换机退回
     * routingKey   通过哪个路由 key 退回
     */

    @Override
    public void returnedMessage(ReturnedMessage returned) {
        log.error("消息{}，被交换机{}返回，退回原因：{}，路由key：{}", returned.getMessage(), returned.getExchange(), returned.getReplyText(),returned.getRoutingKey());
    }
}

```



**结果**

![image-20241203194058811](.\img\image-20241203194058811.png)

```
2024-12-03 18:09:17.134  INFO 13464 --- [nio-8888-exec-1] com.hxt.controller.SendMsgController     : 发送消息内容:大家好1key1
2024-12-03 18:09:17.138  INFO 13464 --- [nectionFactory1] com.hxt.config.MyCallBack                : 交换机已经收到ID为1的消息
2024-12-03 18:09:17.139  INFO 13464 --- [nio-8888-exec-1] com.hxt.controller.SendMsgController     : 发送消息内容:大家好1key2
2024-12-03 18:09:17.139  INFO 13464 --- [nio-8888-exec-1] com.hxt.controller.SendMsgController     : 发送消息内容:大家好1key3
2024-12-03 18:09:17.140  INFO 13464 --- [ntContainer#1-1] c.hxt.consumer.DeadLetterQueueConsumer   : 接受到的队列confirm.queue消息:大家好1key1
2024-12-03 18:09:17.140 ERROR 13464 --- [nectionFactory1] com.hxt.config.MyCallBack                : 消息(Body:'大家好1key2' MessageProperties [headers={spring_returned_message_correlation=2}, contentType=text/plain, contentEncoding=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, deliveryTag=0])，被交换机confirm.exchange返回，退回原因：NO_ROUTE，路由key：key2
2024-12-03 18:09:17.140  INFO 13464 --- [nectionFactory2] com.hxt.config.MyCallBack                : 交换机已经收到ID为2的消息
2024-12-03 18:09:17.141 ERROR 13464 --- [68.116.132:5672] o.s.a.r.c.CachingConnectionFactory       : Shutdown Signal: channel error; protocol method: #method<channel.close>(reply-code=404, reply-text=NOT_FOUND - no exchange 'confirm.exchange123' in vhost '/', class-id=60, method-id=40)
2024-12-03 18:09:17.141  INFO 13464 --- [nectionFactory1] com.hxt.config.MyCallBack                : 交换机还未收到ID为:3的消息，由于原因:channel error; protocol method: #method<channel.close>(reply-code=404, reply-text=NOT_FOUND - no exchange 'confirm.exchange123' in vhost '/', class-id=60, method-id=40)
```

可见消息二、和三均被返回，且是error，但是消息二的`ack=true`



### 8.3 备份交换机

有了 mandatory 参数和回退消息，我们获得了对无法投递消息的感知能力，有机会在生产者的消息无法被投递时发现并处理。但有时候，我们并不知道该如何处理这些无法路由的消息，最多打个日志，然后触发报警，再来手动处理。而通过日志来处理这些无法路由的消息是很不优雅的做法，特别是当生产者所在的服务有多台机器的时候，手动复制日志会更加麻烦而且容易出错。而且设置 mandatory 参数会增加生产者的复杂性，需要添加处理这些被退回的消息的逻辑。如果既不想丢失消息，又不想增加生产者的复杂性，该怎么做呢？

前面在设置死信队列的文章中，我们提到，可以为队列设置死信交换机来存储那些处理失败的消息，可是这些不可路由消息根本没有机会进入到队列，因此无法使用死信队列来保存消息。 在 RabbitMQ 中，有一种备份交换机的机制存在，可以很好的应对这个问题。

什么是备份交换机呢？备份交换机可以理解为 RabbitMQ 中交换机的“备胎”，当我们为某一个交换机声明一个对应的备份交换机时，就是为它创建一个备胎，当交换机接收到一条不可路由消息时，将会把这条消息转发到备份交换机中，由备份交换机来进行转发和处理，通常备份交换机的类型为 Fanout ，这样就能把所有消息都投递到与其绑定的队列中，然后我们在备份交换机下绑定一个队列，这样所有那些原交换机无法被路由的消息，就会都进 入这个队列了。当然，我们还可以建立一个报警队列，用独立的消费者来进行监测和报警。

#### 8.3.1 代码架构图

![image-20241203185723070](.\img\image-20241203185723070.png)



#### 8.3.2 实现

**修改高级确认发布 配置类**

```java
@Configuration
public class ConfirmConfig {

    public static final String CONFIRM_QUEUE = "confirm.queue";
    public static final String CONFIRM_EXCHANGE = "confirm.exchange";
    public static final String CONFIRM_ROUTINGKEY = "key1";
    public static final String BACKUP_EXCHANGE = "backup.exchange";
    public static final String BACKUP_QUEUE = "backup.queue";
    public static final String WARNING_QUEUE = "warning.queue";

    @Bean
    public Queue confirmQueue() {
        return new Queue(CONFIRM_QUEUE, true);
    }

    @Bean
    public DirectExchange confirmExchange() {
        Map<String, Object> args = new HashMap<String, Object>();
        args.put("alternate-exchange", BACKUP_EXCHANGE);
        return new DirectExchange(CONFIRM_EXCHANGE, true, false, args);
    }

    @Bean
    public Binding confirmBinding(@Qualifier("confirmQueue") Queue confirmQueue, @Qualifier("confirmExchange") DirectExchange confirmExchange) {
        return BindingBuilder.bind(confirmQueue).to(confirmExchange).with(CONFIRM_ROUTINGKEY);
    }

    @Bean
    public Queue backupQueue() {
        return new Queue(BACKUP_QUEUE, true);
    }

    @Bean
    public Queue warningQueue() {
        return new Queue(WARNING_QUEUE, true);
    }

    @Bean
    public FanoutExchange backupExchange() {
        return new FanoutExchange(BACKUP_EXCHANGE);
    }

    @Bean
    public Binding backupQueueBingBackupExchange(@Qualifier("backupExchange")FanoutExchange backupExchange, @Qualifier("backupQueue") Queue backupQueue) {
        return BindingBuilder.bind(backupQueue).to(backupExchange);
    }

    @Bean
    public Binding warningQueueBingBackupExchange(@Qualifier("backupExchange")FanoutExchange backupExchange, @Qualifier("warningQueue") Queue warningQueue) {
        return BindingBuilder.bind(warningQueue).to(backupExchange);
    }
}

```

**报警消费者**

```java
@Slf4j
@Component
public class WarningConsumer {
    @RabbitListener(queues = ConfirmConfig.WARNING_QUEUE)
    public void confirm(Message message) {
        String msg = new String(message.getBody());
        log.error("报警发现不可路由消息:{}",msg);
    }
}
```

消息三由于连交换机都没找到所有不会发出报警信息

![image-20241203194246187](.\img\image-20241203194246187.png)



**优先级**

![image-20241203194357747](.\img\image-20241117130928165.png)

从结果中可以看到没有在输出消息回退中 `"消息{}，被交换机{}返回，退回原因：{}，路由key：{}"` 的语句，说明Mandatory 参数与备份交换机可以一起使用的时候，如果两者同时开启，**备份交换机优先级高**。



## 九、RabbitMQ其他知识点

### 9.1 幂等性

用户对于同一操作发起的一次请求或者多次请求的结果是一致的，不会因为多次点击而产生了副作用。举个最简单的例子，那就是支付，用户购买商品后支付，支付扣款成功，但是返回结果的时候网络异常，此时钱已经扣了，用户再次点击按钮，此时会进行第二次扣款，返回结果成功，用户查询余额发现多扣钱了，流水记录也变成了两条。在以前的单应用系统中，我们只需要把数据操作放入事务中即可，发生错误立即回滚，但是再响应客户端的时候也有可能出现网络中断或者异常等等。

可以理解为验证码，只能输入一次，再次重新输入会刷新验证码，原来的验证码失效。

####  9.1.1 消息重复消费

消费者在消费 MQ 中的消息时，MQ 已把消息发送给消费者，消费者在给 MQ 返回 ack 时网络中断， 故 MQ 未收到确认信息，该条消息会重新发给其他的消费者，或者在网络重连后再次发送给该消费者，但实际上该消费者已成功消费了该条消息，造成消费者消费了重复的消息。

#### 9.1.2 解决思路

MQ 消费者的幂等性的解决一般使用全局 ID 或者写个唯一标识比如时间戳 或者 UUID 或者订单消费者消费 MQ 中的消息也可利用 MQ 的该 id 来判断，或者可按自己的规则生成一个全局唯一 id，每次消费消息时用该 id 先判断该消息是否已消费过。

##### 9.1.2.1 唯一ID+指纹码机制

指纹码：我们的一些规则或者时间戳加别的服务给到的唯一信息码,它并不一定是我们系统生成的，基本都是由我们的业务规则拼接而来，但是一定要保证唯一性，然后就利用查询语句进行判断这个 id 是否存在数据库中，优势就是实现简单就一个拼接，然后查询判断是否重复；劣势就是在高并发时，如果是单个数据库就会有写入性能瓶颈当然也可以采用分库分表提升性能，但也不是我们最推荐的方式。

##### 9.1.2.2 Redis的原子性

利用Redis执行 `setnx`  命令，天然具有幂等性。



### 9.2 优先级队列

#### 9.2.1 使用场景

在我们系统中有一个订单催付的场景，我们的客户在天猫下的订单，淘宝会及时将订单推送给我们，如果在用户设定的时间内未付款那么就会给用户推送一条短信提醒，很简单的一个功能对吧。

但是，tmall 商家对我们来说，肯定是要分大客户和小客户的对吧，比如像苹果，小米这样大商家一年起码能给我们创造很大的利润，所以理应当然，他们的订单必须得到优先处理，而曾经我们的后端系统是使用 redis 来存放的定时轮询，大家都知道 redis 只能用 List 做一个简简单单的消息队列，并不能实现一个优先级的场景，所以订单量大了后采用 RabbitMQ 进行改造和优化，如果发现是大客户的订单给一个相对比较高的优先级， 否则就是默认优先级。



#### 9.2.2 添加方法

##### 9.2.2.1 Web页面添加

![image-20241205184141719](.\img\image-20241205184141719.png)

1. 进入 Web 页面，点击 Queue 菜单，然后点击 `Add a new queue`
2. 点击下方的 `Maximum priority`，数字越大优先级越大
3. 执行第二步，则会自动在 `Argument` 生成 `x-max-priority` 字符串
4. 点击 `Add queue` 即可添加优先级队列成功



##### 9.2.2.2 声明队列的时候添加优先级

设置队列的最大优先级 最大可以设置到 255 官网推荐 1-10 如果设置太高比较吃内存和 CPU

```java
Map<String, Object> params = new HashMap();
// 优先级为 10
params.put("x-max-priority", 10);
channel.queueDeclare("hello", true, false, false, params);
```



#### 9.2.3 实战

生产者发送十个消息，如果消息为 `info5`，则优先级是最高的，当消费者从队列获取消息的时候，优先获取 `info5` 消息

**生产者**

```java
public class PriorityProducer {
    public static final String QUEUE_NAME = "priority_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();

        // 设置消息的优先级
        AMQP.BasicProperties properties = new AMQP.BasicProperties().builder().priority(1).priority(10).build();

        for (int i = 1; i < 11; i++) {
            String message = "info" + i;
            if (i == 5){
                channel.basicPublish("", QUEUE_NAME, properties, message.getBytes());
            }else {
                channel.basicPublish("",QUEUE_NAME,null,message.getBytes());
            }
            System.out.println("消息发送完成："+message);
        }
    }
}
```

**消费者**

```java
public class PriorityConsumer {

    public static final String QUEUE_NAME = "priority_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtil.getChannel();
		
        // 设置队列的最大优先级，即消息优先级可取的范围
        Map<String, Object> params = new HashMap<>();
        params.put("x-max-priority",10);
        channel.queueDeclare(QUEUE_NAME,true,false,false,params);

        //推送消息如何进行消费的接口回调
        DeliverCallback deliverCallback = (consumerTag, delivery) ->{
            String message = new String(delivery.getBody());
            System.out.println("消费的消息: "+message);
        };

        //取消消费的一个回调接口 如在消费的时候队列被删除掉了
        CancelCallback cancelCallback = (consumerTag) ->{
            System.out.println("消息消费被中断");
        };

        channel.basicConsume(QUEUE_NAME,true,deliverCallback,cancelCallback);
    }
}

```



![image-20241205190119324](.\img\image-20241205190119324.png)





### 9.3 惰性队列

![image-20241205190715377](.\img\image-20241205190715377.png)

生产者发送消息过来，先保存到磁盘中，消费者要消费时将消息加载到内存



#### 9.3.1 使用场景

RabbitMQ 从 3.6.0 版本开始引入了惰性队列的概念。惰性队列会尽可能的将消息存入磁盘中，而在消费者消费到相应的消息时才会被加载到内存中，它的一个重要的设计目标是能够支持更长的队列，即支持更多的消息存储。当消费者由于各种各样的原因(比如消费者下线、宕机亦或者是由于维护而关闭等)而致使长时间内不能消费消息造成堆积时，惰性队列就很有必要了。

默认情况下，当生产者将消息发送到 RabbitMQ 的时候，队列中的消息会尽可能的存储在内存之中，这样可以更加快速的将消息发送给消费者。即使是持久化的消息，在被写入磁盘的同时也会在内存中驻留一份备份。当 RabbitMQ 需要释放内存的时候，会将内存中的消息换页至磁盘中，这个操作会耗费较长的时间，也会阻塞队列的操作，进而无法接收新的消息。虽然 RabbitMQ 的开发者们一直在升级相关的算法， 但是效果始终不太理想，尤其是在消息量特别大的时候。



#### 9.3.2 两种模式

队列具备两种模式：default 和 lazy。默认的为 default 模式，在 3.6.0 之前的版本无需做任何变更。lazy 模式即为惰性队列的模式，可以通过调用 `channel.queueDeclare` 方法的时候在参数中设置，也可以通过 Policy 的方式设置，如果一个队列同时使用这两种方式设置的话，那么 Policy 的方式具备更高的优先级。如果要通过声明的方式改变已有队列的模式的话，那么只能先删除队列，然后再重新声明一个新的。

在队列声明的时候可以通过 `x-queue-mode` 参数来设置队列的模式，取值为 default 和 lazy。下面示例中演示了一个惰性队列的声明细节：

```java
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-queue-mode", "lazy");
channel.queueDeclare("myqueue", false, false, false, args);
```

也可以在 Web 页面添加队列时，选择 `Lazy mode`

![image-20241205191316239](.\img\image-20241205191316239.png)



### 9.4 集群模式

#### 9.4.1 集群原理

​		最开始我们介绍了如何安装及运行 RabbitMQ服务，不过这些是单机版的，无法满足目前真实应用的要求。如果 RabbitMQ 服务器遇到内存崩溃、机器掉电或者主板故障等情况，该怎么办?单台 RabbitMQ服务器可以满足每秒 1000 条消息的吞吐量，那么如果应用需要 RabbitMQ 服务满足每秒 10 万条消息的吞吐量呢?

​		购买昂贵的服务器来增强单机 RabbitMQ务的性能显得捉襟见肘，搭建一个 RabbitMQ 集群才是解决实际问题的关键.

#### 9.4.2 搭建集群

![image-20241205201740682](.\img\image-20241205201740682.png)

也可以 `node2` 连 `node3`，如

![image-20241205203224137](.\img\image-20241205203224137.png)

```shell
# 修改3台主机的名称
# root@RedHatHXT中RedHatHXT即为主机名称
vim /etc/hostname

# 配置各个节点的hosts文件，让各个节点能够访问对方
# 添加192.168.116.130 node3  192.168.116.131 node2  192.168.116.132 RedHatHXT
vim /etc/hosts
```

![image-20241205202127297](.\img\image-20241205202127297.png)



```shell
# 这边设置RedHatHXT为主节点，并在RedHatHXT主机上键入，为了实现上图所示。也可以修改连接方式。目的是确保各个节点的cookie与RedHatHXT相同
scp /var/lib/rabbitmq/.erlang.cookie root@node2:/var/lib/rabbitmq/.erlang.cookie

scp /var/lib/rabbitmq/.erlang.cookie root@node3:/var/lib/rabbitmq/.erlang.cookie

# 启动rabbitMQ服务，顺带启动Eelang虚拟机和RabbitMQ应用服务（在三台机器以上分别执行以下命令）
rabbitmq-server -detached

# 节点2执行
# rabbitmqctl stop命令会将Erlangxnj关闭，此命令只关闭RabbitMQ服务
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster rabbit@RedHatHXT
rabbitmqctl start_app

# 节点2执行
# rabbitmqctl stop命令会将Erlangxnj关闭，此命令只关闭RabbitMQ服务
rabbitmqctl stop_app
rabbitmqctl reset
# 若改为rabbitmqctl join_cluster rabbit@node2，则会连到node2上，效果需要探索一下
rabbitmqctl join_cluster rabbit@RedHatHXT
rabbitmqctl start_app
```

```shell
# 集群状态
rabbitmqctl cluster_status
# 下图是部分信息，
```

![image-20241205203130193](.\img\image-20241205203130193.png)

```
# 解除集群节点，比如node2，在node2上执行
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl start_app
rabbitmqctl cluster
# 在RedHatHXT上执行
rabbitmqctl forget_cluster_node rabbit@node2
```



#### 9.4.3 镜像队列

![image-20241205195907512](.\img\image-20241205195907512.png)

`name`：任意取

`Pattern`：填一个正则表达式，图中是以 **mirror** 开头的队列

1. **ha-mode**:
   - 这个参数定义了队列的高可用性模式。
   - 可能的值包括：
     - `exactly`：表示队列将被镜像到指定数量的节点上。至于被备份到哪一个节点由服务器决定
     - `all`：表示队列将被镜像到集群中的所有节点。
     - 其他可能的值取决于RabbitMQ的版本和配置。
2. **ha-params**:
   - 这个参数用于指定`ha-mode`的额外参数。
   - 当`ha-mode`设置为`exactly`时，`ha-params`通常是一个数字，表示队列应该被镜像到多少个节点上。在图中，这个值被设置为`2`，意味着队列将被镜像到两个节点上。
3. **ha-sync-mode**:
   - 这个参数控制镜像队列之间的同步模式。
   - 可能的值包括：
     - `automatic`：表示RabbitMQ将自动选择同步模式。这是图中所选的选项。
     - `manual`：表示同步需要手动触发。
   - `automatic`模式下，RabbitMQ会根据网络状况和队列的活动自动调整同步策略，以平衡性能和数据一致性。



可以看到队列 `mirror_hello` 中有 `+1` ，说明这个队列备份了两份，在节点`RedHatHXT` 和 `node2`

![image-20241205200755086](.\img\image-20241205200755086.png)

![image-20241205200931891](.\img\image-20241205200931891.png)



#### 9.4.4 高可用负载均衡



#### 9.4.5 Federation Exchange

​		(broker 北京)，(broker 深圳)彼此之间相距甚远，网络延迟是一个不得不面对的问题。有一个在北京的业务(Client 北京)需要连接(broker 北京),向其中的交换器 exchangeA 发送消息,此时的网络延迟很小(Client 北京)可以迅速将消息发送至 exchangeA 中，就算在开启了 publisherconfirm 机制或者事务机制的情况下，也可以迅速收到确认信息。此时又有个在深圳的业务(Client 深圳)需要向 exchangeA 发送消息,那么(Client 深圳)(broker 北京)之间有很大的网络延迟，(Client 深圳)将发送消息至 exchangeA 会经历定的延迟，尤其是在开启了publisherconfrm 机制或者事务机制的情况下，(Client 深圳)会等待很长的延迟时间来接收(broker 北京)的确认信息，进而必然造成这条发送线程的性能降低，甚至造成一定程度上的阻塞。
​		将业务(Client 深圳)部署到北京的机,房可以解决这个问题，但是如果(Client 深圳)调用的另些服务都部署在深圳，那么又会引发新的时延问题，总不见得将所有业务全部部署在一个机房,那么容灾又何以实现?这里使用 Federation 插件就可以很好地解决这个问题,

![image-20241205203928658](.\img\image-20241205203928658.png)

北京的用户访问北京的交换机，但是出现数据不一致，短时间内不能实现同步



##### 9.4.5.1 搭建步骤

1、保证每台节点单独运行

2、在每台机器上开启 `federation` 相关插件

```shell
rabbitmq-plugins enable rabbitmq_federation
rabbitmq-plugins enable rabbitmq_federation_management
```

出现插件

![image-20241205204437630](.\img\image-20241205204437630.png)

##### 9.4.5.2 原理

![image-20241205204742474](.\img\image-20241205204742474.png)



##### 9.4.5.3 实现

```java
public static final String QUEUE_NAME = "mirror_hello";
public static final String FED_EXCHANGE = "fed.exchange";

// 发消息
public static void main(String[] args) throws IOException, TimeoutException {
    // 创建一个连接工厂
    ConnectionFactory factory = new ConnectionFactory();
    // 工厂ip 连接rabbitmq队列
    // node2
    String host = "192.168.116.130";
    factory.setHost(host);

    // 用户名
    factory.setUsername("admin");
    // 密码
    factory.setPassword("20170440");

    // 创建连接
    Connection connection = factory.newConnection();
    // 创建信道
    Channel channel = connection.createChannel();

    channel.exchangeDeclare(FED_EXCHANGE, BuiltinExchangeType.DIRECT);
    channel.queueDeclare("node2_queue", true, false, false, null);
    channel.queueBind("node2_queue", FED_EXCHANGE, "routingKey");
    // ...
}
```

上图中`node1` 为 `RedHatHXT` 节点，下图URI中是`RedHatHXT`

![image-20241205211031576](.\img\image-20241205211031576.png)

添加策略

![image-20241205210535359](.\img\image-20241205210535359.png)



**结果**

![image-20241205211052434](.\img\image-20241205211052434.png)

`Upstream`：RedHatHXT

下流：node2



#### 9.4.6 Federation Queue

​		联邦队列可以在多个 Broker 节点(或者集群)之间为单个队列提供均衡负载的功能。一个联邦队列可以连接一个或者多个上游队列(upstream queue)，并从这些上游队列中获取消息以满足本地消费者消费消息的需求。



##### 9.4.6.1 搭建步骤

![image-20241205211250177](.\img\image-20241205211250177.png)

图中节点2联邦到节点1，但数据是从节点1发送到节点2

在联邦交换机的基础上

![image-20241205211846602](.\img\image-20241205211846602.png)



#### 9.4.7 Shovel

​		Federation 具备的数据转发功能类似，Shovel 够可靠、持续地从一个 Broker 中的队列(作为源端，即source)拉取数据并转发至另一个 Broker 中的交换器(作为目的端，即 destination)。作为源端的队列和作为目的端的交换器可以同时位于同一个 Broker，也可以位于不同的 Broker 上。Shovel 可以翻译为"铲子"是一种比较形象的比喻，这个"铲子"可以将消息从一方"铲子"另一方。Shovel行为就像优秀的客户端应用程序能够负责连接源和目的地、负责消息的读写及负责连接失败问题的处理。

![image-20241205212024478](.\img\image-2024111713092999.png)

##### 9.4.7.1 搭建步骤

```
# 每台机器安装shovel插件
rabbitmq-plugins enable rabbitmq_shovel
rabbitmq-plugins enable rabbitmq_shovel_management
```

![image-20241205212319474](.\img\image-20241205212319474.png)

![image-20241205212735999](.\img\image-20241205212735999.png)

![image-20241205212642297](.\img\image-20241205212642297.png)
