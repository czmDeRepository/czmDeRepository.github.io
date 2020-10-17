---
title: RabbitMQ入门
date: 2020-09-25 20:32:13
tags: 
  - 消息中间件
---



## 消息中间件（MQ）

### 简介

​	消息中间件是基于队列与消息传递技术，在网络环境中为应用系统提供同步或异步、可靠的消息传输的支撑性软件系统。——百度百科

​	MQ全称为Message Queue，消息队列是程序和程序之间的通信方法。

### 使用场景

- 实现项目通讯与解耦   

MQ相当于一个中介，消息生产方将消息发给MQ，消息消费方接收消息并进行相应逻辑处理，它将两应用程序进行解耦合。

-  处理异步任务

​		在项目中，可将一些无需即时返回且耗时的操作提取出来，进行异步处理，而这种异步处理的方式大大的节省了服务器的请求响应时间，从而提高了系统的吞吐量。将不需要同步处理的并且耗时长的操作由消息队列通知消息接收方进行异步处理。提高了应用程序的响应时间。

- 
     削峰填谷


  如订单抢票系统，开始抢票瞬间高请求，高并发，若此时都操作数据库，需要大量IO操作，消耗系统性能，系统很可能崩溃，我们可以先将订单存消息队列里，然后系统就可以避开高峰期再按照自己的消费能力消费消息队列里的消息。



## 主流实现方式

### AMQP

​	AMQP，即Advanced Message Queuing Protocol，一个提供统一消息服务的应用层标准高级[消息](https://baike.baidu.com/item/消息/1619218)队列协议，是[应用层](https://baike.baidu.com/item/应用层/4329788)协议的一个开放标准，为面向消息的中间件设计。基于此协议的客户端与消息中间件可传递消息，并不受客户端/[中间件](https://baike.baidu.com/item/中间件/452240)不同产品，不同的开发语言等条件的限制。[Erlang](https://baike.baidu.com/item/Erlang)中的实现有[RabbitMQ](https://baike.baidu.com/item/RabbitMQ)等。                                                                               												——百度百科

AMQP不从API层进行限定，而是直接定义网络交换的数据格式。

### JMS

​	JMS即Java消息服务（Java Message Service）应用程序接口，是一个[Java平台](https://baike.baidu.com/item/Java平台)中关于面向[消息中间件](https://baike.baidu.com/item/消息中间件/5899771)（MOM）的[API](https://baike.baidu.com/item/API/10154)，用于在两个应用程序之间，或[分布式系统](https://baike.baidu.com/item/分布式系统/4905336)中发送消息，进行[异步通信](https://baike.baidu.com/item/异步通信/2273903)。Java消息服务是一个与具体平台无关的API，绝大多数MOM提供商都对JMS提供支持。																																																	——百度百科

### AMQP 与 JMS 区别

- JMS是定义了统一的接口，来对消息操作进行统一；AMQP是通过规定协议来统一数据交互的格式
- JMS限定了必须使用Java语言；AMQP只是协议，不规定实现方式，因此是跨语言的。
- JMS规定了两种消息模式；而AMQP的消息模式更加丰富

### 常见的消息队列

- ActiveMQ：基于JMS
- ZeroMQ：基于C语言开发
- RabbitMQ：基于AMQP协议，erlang语言开发，稳定性好
- RocketMQ：基于JMS，阿里巴巴产品
- Kafka：类似MQ的产品；分布式消息系统，高吞吐量

## RabbitMQ

[官网](http://www.rabbitmq.com/)

#### 简介

​		**RabbitMQ**是实现了高级消息队列协议（AMQP）的开源消息代理软件（亦称面向消息的中间件）。RabbitMQ服务器是用[Erlang](https://baike.baidu.com/item/Erlang)语言编写的，而集群和故障转移是构建在[开放电信平台](https://baike.baidu.com/item/开放电信平台)框架上的。所有主要的编程语言均有与代理接口通讯的客户端[库](https://baike.baidu.com/item/库)。		——百度百科

### 6种模式

- 简单模式：一个生产者发送消息到队列,一个消费者接收，不需要设置交换机（使用默认的交换机）

- work工作队列模式，一个生产者，多个消费者，每个消费者获取到的消息唯一，多个消费者竞争同一个队列的消息

- Publish/Subscribe发布与订阅模式：一个生产者发送的消息会被多个消费者获取。一个生产者、一个交换机、多个队列、多个消费者

- Routing路由模式（direct模式）：生产者发送消息到交换机并且要指定路由key，消费者将队列绑定到交换机时需要指定路由key

- Topics通配符模式:  生产者发送消息到交换机，交换机类型设置topic，交换机根据绑定队列的routing key的值进行通配符匹配，

    "#"：匹配零个或者多个词topic.# 可以匹配topic，topic.text，topic.test.queue

    "*"：匹配l零个或一个词，topic.* 可以匹配topic，topic.text或topic.queue

- RPC远程调用模式：功能如名，调用远程项目的功能并等待结果。

### 入门使用

#### **maven坐标**

```xml
 <dependency>
     <groupId>com.rabbitmq</groupId>
     <artifactId>amqp-client</artifactId>
     <version>3.6.5</version>
</dependency>
```

#### java代码示例

注：这里以direct模式示例，其他模式类似

##### 消息生产者

```java
//创建连接工厂
ConnectionFactory connectionFactory = new ConnectionFactory();
//设置Rabbitmq主机地址,默认localhost
connectionFactory.setHost("localhost");
//设置端口,默认5672
connectionFactory.setPort(5672);
//设置虚拟主机，默认 /
connectionFactory.setVirtualHost("/");
//连接用户名；默认为guest
connectionFactory.setUsername("czm");
//连接密码；默认为guest
connectionFactory.setPassword("123456");
//通过连接工厂创建连接
Connection connection = connectionFactory.newConnection();
// 通过connection创建一个Channel通道
Channel channel = connection.createChannel();

// 要发送的信息
String message = "hello world";

AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
				.deliveryMode(2)//2消息持久化；1重启消息丢失
				.contentEncoding("UTF-8")
				.expiration("10000")//十秒失效
				.headers(headers)
				.build();
/**
* 参数1：交换机名称，如果没有指定则使用默认Default Exchage
* 参数2：路由key,简单模式可以传递队列名称
* 参数3：消息其它属性
* 参数4：消息内容
*/
channel.basicPublish("test_direct_exchange", "test.direct", properties, message.getBytes());

// 关闭资源
channel.close();
connection.close();
```

##### 消息消费者

```java
ConnectionFactory connectionFactory = new ConnectionFactory() ;  

connectionFactory.setHost("localhost");
connectionFactory.setPort(5672);
connectionFactory.setVirtualHost("/");
connectionFactory.setUsername("czm");
connectionFactory.setPassword("123456");
//自动恢复连接
connectionFactory.setAutomaticRecoveryEnabled(true);
//自动恢复在尝试重新连接之前要等待多长时间，默认5000ms
connectionFactory.setNetworkRecoveryInterval(3000);

Connection connection = connectionFactory.newConnection();

Channel channel = connection.createChannel();  
//4 声明
String exchangeName = "test_direct_exchange";
String exchangeType = "direct";
String queueName = "test_direct_queue";
String routingKey = "test.direct";

//表示声明了一个交换机
channel.exchangeDeclare(exchangeName, exchangeType, true, false, false, null);
// 声明（创建）队列
/**
* 参数1：队列名称
* 参数2：是否持久化队列
* 参数3：是否独占连接
* 参数4：是否在不使用的时候自动删除队列
* 参数5：队列其它参数
*/
channel.queueDeclare(queueName, false, false, false, null);
//建立一个绑定关系:
channel.queueBind(queueName, exchangeName, routingKey);

//durable 是否持久化消息
QueueingConsumer consumer = new QueueingConsumer(channel);
//参数：队列名称、是否自动ACK、Consumer
channel.basicConsume(queueName, true, consumer);
//循环获取消息  
while(true){  
    //获取消息，如果没有消息，这一步将会一直阻塞  
    Delivery delivery = consumer.nextDelivery();  
    String msg = new String(delivery.getBody());    
    System.out.println("收到消息：" + msg);  
} 
```

### 用xml配置方式与spring整合

#### 生产端

##### xml配置

注：以下配置rabbitMQ可靠性投递用到，消息的延迟投递，做二次确认，回调检查

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
                http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
                http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/rabbit
                http://www.springframework.org/schema/rabbit/spring-rabbit.xsd">

   <!--引入spring配置文件 ，便于下方配置变量引用-->
    <context:property-placeholder location="classpath:application.properties"/>
    <!-- 创建连接类 -->
    <!--<bean id="rabbitmqConnectionFactory"  class="org.springframework.amqp.rabbit.connection.CachingConnectionFactory">-->
        <!--<constructor-arg value="localhost" />-->
        <!-- username,访问RabbitMQ服务器的账户,默认是guest -->
        <!--<property name="username" value="${rmq.manager.user}" />-->
        <!-- username,访问RabbitMQ服务器的密码,默认是guest -->
        <!--<property name="password" value="${rmq.manager.password}" />-->
        <!-- host,RabbitMQ服务器地址，默认值"localhost" -->
        <!--<property name="host" value="${rmq.ip}" />-->
        <!-- port，RabbitMQ服务端口，默认值为5672 -->
        <!--<property name="port" value="${rmq.port}" />-->
        <!-- channel-cache-size，channel的缓存数量，默认值为25 -->
        <!--<property name="channel-cache-size" value="50" />-->
        <!--cache-mode，缓存连接模式，默认值为CHANNEL(单个connection连接，连接之后关闭，自动销毁) -->
        <!--<property name="cache-mode" value="CHANNEL" />-->
    <!--</bean>-->
    
    <!--或者这样配置，connection-factory元素实际就是注册一个org.springframework.amqp.rabbit.connection.CachingConnectionFactory实例-->
    <rabbit:connection-factory id="rabbitmqConnectionFactory" 
                               host="${rmq.ip}" port="${rmq.port}"
    							virtual-host="${rmq.manager.virtual}" 
                               username="${rmq.manager.user}" 
                               password="${rmq.manager.password}" 
                               publisher-confirms="true" publisher-returns="true"/>
	<!--注：publisher-confirms="true" publisher-returns="true"用于开启消息投递的回调监听，若想使用必须配置-->
    
    
    <rabbit:admin connection-factory="rabbitmqConnectionFactory"/>

    <!-- 生产者部分 -->
    <!-- 发送消息的producer类，也就是生产者 -->
    <bean id="RabbitmqProduct" class="生产者类路径">
        <!--routingKey的 value中的值就是producer中的的routingKey，它与上面的rabbit:bindings标签中的key必须相同 ->
        <property name="exchange" value="${rmq.manager.exchange}"/>
        <property name="routingKey" value="${rmq.manager.routingKey}"/>
        <property name="delayKey" value="${rmq.manager.key_delay}"/>
        <property name="rabbitTemplate" ref="rabbitTemplate"/>
    </bean>

    <!-- spring amqp默认的是jackson 的一个插件,目的将生产者生产的数据转换为json存入消息队列，由于fastjson的速度快于jackson,这里替换为fastjson的一个实现 -->
    <!--<bean id="jsonMessageConverter" class="com.jy.utils.FastJsonMessageConverter"></bean>-->
    <!-- 或者配置jackson -->

    <bean id="jsonMessageConverter" class="org.springframework.amqp.support.converter.Jackson2JsonMessageConverter" />
	<!--消息成功投递后回调类-->
    <bean id="confirmCallBackListener" class="回调类路径"/>
	<!--消息找不到对应交换机或routingkey，投递失败回调类-->
    <bean id="returnCallBackListener" class="回调类路径" />
    <!--exchange="Anyview_exchange"-->
    <rabbit:template  id="rabbitTemplate" connection-factory="rabbitmqConnectionFactory"
                      message-converter="jsonMessageConverter"
                      confirm-callback="confirmCallBackListener" return-callback="returnCallBackListener" mandatory="true"/>


    <!-- 消费者部分 -->

    <!--定义消息队列,durable:是否持久化，
    如果想在RabbitMQ退出或崩溃的时候，不会失去所有的queue和消息，需要同时标志队列(queue)和交换机(exchange)是持久化的，	即rabbit:queue标签和rabbit:direct-exchange中的durable=true,而消息(message)默认是持久化的可以看类			org.springframework.amqp.core.MessageProperties中的属性
	public static final MessageDeliveryMode DEFAULT_DELIVERY_MODE = MessageDeliveryMode.PERSISTENT;
    exclusive: 仅创建者可以使用的私有队列，断开后自动删除；
    auto_delete: 当所有消费客户端连接断开后，是否自动删除队列 -->
    <rabbit:queue name="${rmq.manager.queue}" id="rabbitQueue" durable="true" auto-delete="false" exclusive="false" />


    <!--延时队列-->
    <rabbit:queue id="queueDelay" name="${rmq.manager.queue_delay}" durable="true" auto-delete="false" exclusive="false" >
        <rabbit:queue-arguments>
            <!--  队列过期时间 10秒-->
            <entry key="x-message-ttl" value="20000" value-type="java.lang.Long" />
            <!-- 过期后消息将通过以下交换机和routingkey发送到死信队列-->
            <entry key="x-dead-letter-exchange" value="${rmq.manager.exchange_dead}" />
            <entry key="x-dead-letter-routing-key" value="${rmq.manager.key_dead}" />
        </rabbit:queue-arguments>
    </rabbit:queue>


    <!--绑定队列,
rabbitmq的exchangeType常用的三种模式：direct，fanout，topic三种,
我们用direct模式，即rabbit:direct-exchange标签，Direct交换器很简单，
如果是Direct类型，就会将消息中的RoutingKey与该Exchange关联的所有Binding中的BindingKey进行比较，
如果相等，则发送到该Binding对应的Queue中。有一个需要注意的地方：如果找不到指定的exchange，就会报错。
但routing key找不到的话，不会报错，这条消息会直接丢失，所以此处要小心,
auto-delete:自动删除，如果为Yes，则该交换机所有队列queue删除后，自动删除交换机，默认为false -->
    <rabbit:direct-exchange id="exchange" name="${rmq.manager.exchange}" durable="true" auto-delete="false">
        <rabbit:bindings>
            <rabbit:binding queue="rabbitQueue" key="${rmq.manager.routingKey}"></rabbit:binding>
            <rabbit:binding queue="queueDelay" key="${rmq.manager.key_delay}"></rabbit:binding>
        </rabbit:bindings>
    </rabbit:direct-exchange>

    <!--死信队列-->
    <rabbit:queue name="${rmq.manager.queue_dead}" id="queueDead" durable="true" auto-delete="false" exclusive="false" />


    <rabbit:direct-exchange id="exchange_dead" name="${rmq.manager.exchange_dead}" durable="true" auto-delete="false">
        <rabbit:bindings>
            <rabbit:binding queue="queueDead" key="${rmq.manager.key_dead}"></rabbit:binding>
        </rabbit:bindings>
    </rabbit:direct-exchange>




    <!-- 用于消息的监听的代理类MessageListenerAdapter -->
    <!--<bean id="testQueueListenerAdapter" class="org.springframework.amqp.rabbit.listener.adapter.MessageListenerAdapter" >-->
        <!-- 类名 -->
        <!--<constructor-arg ref="Handler" />-->
        <!-- 方法名 -->
        <!--<property name="defaultListenerMethod" value="handlerTest"></property>-->
        <!--<property name="messageConverter" ref="jsonMessageConverter"></property>-->
    <!--</bean>-->



    <!--<rabbit:direct-exchange id="${rmq.manager.exchange}" name="${rmq.manager.exchange}"  durable="false" auto-delete="false" >-->

    <!--</rabbit:direct-exchange>-->

    <!-- 自定义接口类 -->
    <bean name="Handler" class="消息类路径"></bean>

    <!-- 配置监听acknowledeg="manual"设置手动应答，它能够保证即使在一个worker处理消息的时候用CTRL+C来杀掉这个		worker，或者一个consumer挂了(channel关闭了、connection关闭了或者TCP连接断了)，也不会丢失消息。
    因为RabbitMQ知道没发送ack确认消息导致这个消息没有被完全处理，将会对这条消息做re-queue处理。
    如果此时有另一个consumer连接，消息会被重新发送至另一个consumer会一直重发,直到消息处理成功,
    监听容器acknowledge="auto" concurrency="30"设置发送次数,最多发送30次 -->
    <!--concurrency="20"-->
    <rabbit:listener-container connection-factory="rabbitmqConnectionFactory" acknowledge="manual" >
        <rabbit:listener queues="${rmq.manager.queue_dead}" ref="Handler"/>
    </rabbit:listener-container>

</beans>
```

##### 生产者代码

```java
/**
 * @author CZM
 */
public class RabbitmqProduct {

    private RabbitTemplate rabbitTemplate;
    private String exchange;
    private String routingKey;

    private String delayKey;
	/**
	*此服务用于将消息持久化
	*/
 	@Resource
    SocketMsgService socketMsgService;

    /**
     * rabbitmq发送消息
     * @param socketMsg 消息实体
     * @param isFirst 是否第一次发送
     */
    public void sendMessage(SocketMsg socketMsg, boolean isFirst){
        //第一次发送前先持久化
        if(isFirst){
            socketMsgService.insert(socketMsg);
        }

        //CorrelationData消息唯一ID
        try {
            Object jsonSocketMsg = JSON.toJSON(socketMsg);
            rabbitTemplate.convertAndSend(exchange,routingKey, jsonSocketMsg, 
                                          new CorrelationData(socketMsg.getId().toString()));
            //发送到延时队列
            rabbitTemplate.convertAndSend(exchange,delayKey, jsonSocketMsg,
                                          new CorrelationData(socketMsg.getId().toString()+"delay"));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public void setExchange(String exchange) {
        this.exchange = exchange;
    }

    public void setRoutingKey(String routingKey) {
        this.routingKey = routingKey;
    }

    public void setRabbitTemplate(RabbitTemplate rabbitTemplate) {
        this.rabbitTemplate = rabbitTemplate;
    }

    public void setDelayKey(String delayKey) {
        this.delayKey = delayKey;
    }
}
```

##### 延时队列消费者代码

```java
@Slf4j
public class Handler implements ChannelAwareMessageListener {

    @Resource
    SocketMsgService socketMsgService;

    @Resource
    RabbitmqProduct rabbitmqProduct;

    @Override
    public void onMessage(Message message, Channel channel) {

        log.info(new String(message.getBody()));
        Object parse = JSONObject.parse(message.getBody());
        log.info(parse.toString());
        SocketMsg socketMsg = JSONObject.parseObject(message.getBody(), SocketMsg.class);
        log.info(socketMsg.toString());
        try {
            //消息的标识，false只确认当前一个消息收到，true确认所有consumer获得的消息
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
        } catch (IOException e) {
            e.printStackTrace();
        }

        //消息未成功投递
        if(socketMsgService.findById(socketMsg.getId()).getStatus() == 0){
            //未超过重试次数
            if (socketMsgService.getTryCountById(socketMsg.getId()) < 3){
                //添加重试次数
                socketMsgService.addTryCountById(socketMsg.getId());
                rabbitmqProduct.sendMessage(socketMsg, false);
            }
        }
    }
}
```

##### 投递成功回调类

```java
/**
 * @Author CZM
 * @create 2020/7/31 10:58
 */
@Slf4j
@Service("confirmCallBackListener")
public class ConfirmCallBackListener implements ConfirmCallback {
    @Resource
    SocketMsgService socketMsgService;

    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        if (ack){
            log.info("################发送到exchange成功:"+correlationData);
            //非延时队列的消息确认
            if (!correlationData.getId().endsWith("delay")){
                //将数据库消息标志置为成功发送
                socketMsgService.succeedSendById(Long.parseLong(correlationData.getId()));
            }
        }else {
            log.error("#########发送失败:"+correlationData);
        }
    }
}
```

##### 投递失败回调类

```java
/**
 * @Author CZM
 * @create 2020/7/31 10:59
 */
@Slf4j
public class ReturnCallBackListener implements ReturnCallback {
    @Override
    public void returnedMessage(Message message, int replyCode, 
                                String replyText, String exchange, String routingKey) {
        log.error("无对应key队列，发送失败了"+message);
        log.info(new String(message.getBody()));
    }
}
```

#### 消费端

##### 配置与上面延时队列的消息接收方类似，此处不再累赘



## END