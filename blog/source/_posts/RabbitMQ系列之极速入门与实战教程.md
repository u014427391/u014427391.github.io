title: SpringBoot系列之RabbitMQ使用实用教程
date: 2017-12-03 00:00:00
---
<Excerpt in index | 首页摘要> 

消息队列（Message Queue，简称MQ），其本质是个队列，FIFO（First In First OUT，先入先出），MQ主要用于不同线程之间的线程通信。大多应用中，可通过消息服务中间件来提升系统异步通信、扩展解耦能力

两个重要概念：
* 消息代理（message broker）和目的地（destination）
（消息发送者发送消息以后，将由消息代理broker接管，然后再传递到指定目的地）
<!-- more -->
<The rest of contents | 余下全文>

SpringBoot系列之RabbitMQ使用实用教程
@[toc]
## 1. 消息队列概述
### 1.1 MQ的概述

消息队列（Message Queue，简称MQ），其本质是个队列，FIFO（First In First OUT，先入先出），MQ主要用于不同线程之间的线程通信。大多应用中，可通过消息服务中间件来提升系统异步通信、扩展解耦能力

两个重要概念：
* 消息代理（message broker）和目的地（destination）
（消息发送者发送消息以后，将由消息代理broker接管，然后再传递到指定目的地）

### 1.2 MQ目的地形式
主要两种形式的目的地：
* 1.队列（queue）：也可以称作为点对点式，即点对点消息通信（point-to-point），主要特点是消息只有唯一的发送者和接收者，但是不能说只有一个接收者，因为有可能是主从模式

* 2.主题（topic）：也可以称作发布订阅式，发送者（发布者）发送消息到主题，多个接收者（订阅者）监听（订阅）这个主题

## 2. 消息队列实现方式
### 2.1 常见MQ框架
MQ框架很多，比较流行的有RabbitMq、ActiveMq、ZeroMq、kafka，以及阿里开源的RocketMQ等等

### 2.2 MQ实现方式
MQ框架的实现方式有多种，比如jms、amqp、mqtt等等，本文主要对比一下JMS和AMQP

JMS（Java Message Service）JAVA消息服务：
* 基于JVM消息代理的规范。ActiveMQ、HornetMQ是JMS实现

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200409171432802.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0MjczOTE=,size_16,color_FFFFFF,t_70)
图来自：https://www.javatpoint.com/jms-tutorial


AMQP（Advanced Message Queuing Protocol）
* 高级消息队列协议，也是一个消息代理的规范，兼容JMS， RabbitMQ是AMQP的实现
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200409173631448.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0MjczOTE=,size_16,color_FFFFFF,t_70)
引用尚硅谷视频教程的总结图示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200409171642331.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0MjczOTE=,size_16,color_FFFFFF,t_70)

## 3. RabbitMQ简介
### 3.1 RabbitMQ简介
RabbitMQ 是一个由 Erlang 语言开发的 AMQP 的开源实现。

开发语言：Erlang – 面向并发的编程语言。


### 3.2 核心概念
引用尚硅谷的视频教程的归纳：
> * Message
消息由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-
该消息可能需要持久性存储）等。
> * Publisher
消息的生产者，也是一个向交换器发布消息的客户端应用程序。
> * Exchange
交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。
Exchange有4种类型：direct(默认)，fanout, topic, 和headers，不同类型的Exchange转发消息的策略有所区别
> * Queue
消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。
> * Binding
 绑定，用于消息队列和交换器之间的关联。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。Exchange 和Queue的绑定可以是多对多的关系。
>  * Connection
网络连接，比如一个TCP连接。
> * Channel
信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内的虚拟连接，AMQP 命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁 TCP 都是非常昂贵的开销，所以引入了信道的概念，以复用一条 TCP 连接
> * Consumer
消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。
> * Virtual Host
虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加
密环境的独立服务器域。每个 vhost 本质上就是一个 mini 版的 RabbitMQ 服务器，拥有
自己的队列、交换器、绑定和权限机制。vhost 是 AMQP 概念的基础，必须在连接时指定，
RabbitMQ 默认的 vhost 是 / 。
> * Broker
表示消息队列服务器实体

学习尚硅谷课件的这些理论知识后，就可以很容易地理解RabbitMQ的体系结构如图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200409173505666.png)
### 3.3 RabbitMQ运行机制
**RabbitMQ是基于AMQP协议，AMQP 中增加了Exchange 和 Binding这两种角色，生产者发布消息后，发给代理Broker，主要还是由Exchange交换器处理，决定将消息发往那个消费者队列**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200409220818827.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0MjczOTE=,size_16,color_FFFFFF,t_70#pic_center)

### 3.4 Exchange类型
RabbitMQ目前共四种交换器类型：direct、fanout、topic、headers。headers 交换器和 direct 交换器完全一致，但性能差很多，用的比较少，所以只介绍三种类型

Direct Exchange：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200409174930566.png)
图片来源：https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_MRG/2/html-single/Messaging_Programming_Reference/index.html

**这种模式根据路由键（routing key）去匹配Bindings中的 binding key，如果完全一致，就发送消息到对应Queue**

Fanout Exchange：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200409175428892.png)

**这种模式是常见的发布订阅模式，发消息方式类似于子网广播，队列只要绑定到对应的Exchange，生产者发送消息过来，有绑定的队列都能接收消息**

Topic Exchange：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200409175725692.png)
**这种模式和Direct exchange有点像，不过Direct exchange是完全匹配，这种匹配方式是，先将路由键、bindings键根据点号隔开，# 表示匹配 0 个或多个单词， “*”表示匹配一个单词**


## 4. RabbitMQ安装部署
本文介绍基于Docker系统的RabbitMQ安装部署


### 4.1 Docker版本部署RabbitMQ
查询rabbitMQ镜像：management版本，不指定默认为最新版本latest
```shell
 docker search rabbitmq:management
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/202004081815519.png)
拉取RabbitMQ镜像：

```shell
docker pull rabbitmq:management
```
查看docker镜像列表：
```shell
docker images
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200409221430388.png)

启动RabbitMQ：做下端口隐射

```powershell
docker run -d -p 15672:15672  -p  5672:5672  -e RABBITMQ_DEFAULT_USER=guest -e RABBITMQ_DEFAULT_PASS=guest --name rabbitmq --hostname=rabbitmqhostone  rabbitmq:management
```
* -d 后台运行
* -p 隐射端口
* --name 指定rabbitMQ名称
* RABBITMQ_DEFAULT_USER 指定用户账号
* RABBITMQ_DEFAULT_PASS 指定账号密码


执行如上命令后访问：http://ip:15672/

输入默认账号密码：guest/guest
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200408181214452.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200409164212323.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0MjczOTE=,size_16,color_FFFFFF,t_70)

### 4.2 Admin新增用户
用户管理和权限管理都在Admin页签里
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200410103408754.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0MjczOTE=,size_16,color_FFFFFF,t_70)

>* 1、超级管理员(administrator)
可登陆管理控制台，可查看所有的信息，并且可以对用户，策略(policy)进行操作。
>* 2、监控者(monitoring)
可登陆管理控制台，同时可以查看rabbitmq节点的相关信息(进程数，内存使用情况，磁盘使用情况等)
>* 3、策略制定者(policymaker)
可登陆管理控制台, 同时可以对policy进行管理。但无法查看节点的相关信息
>* 4、普通管理者(management)
仅可登陆管理控制台，无法看到节点信息，也无法对策略进行管理。
>* 5、其他
无法登陆管理控制台，通常就是普通的生产者和消费者。

### 4.3 设置用户权限
默认是Vitual host如图所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200410103722419.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0MjczOTE=,size_16,color_FFFFFF,t_70)
设置topic permissions
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200410104744353.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0MjczOTE=,size_16,color_FFFFFF,t_70)
### 4.4 创建Virtual Hosts
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200410104850589.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0MjczOTE=,size_16,color_FFFFFF,t_70)
新增后，记得对应用户也要设置权限，SpringBoot的yaml配置文件也得修改
### 4.5 其它管理配置
![在这里插入图片描述](https://img-blog.csdnimg.cn/202004101102583.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0MjczOTE=,size_16,color_FFFFFF,t_70)
## 5. SpringBoot集成RabbitMQ
### 5.1 引入spring-boot-starter-amqp

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
```

### 5.2 RabbitMQ YAML配置

注意spring-boot-starter-amqp有自动配置类，有些配置可以不需要配，详情跟一下源码
```yaml
spring:
  rabbitmq:
    host: 192.168.7.135
    port: 5672
    username: guest
    password: guest
    virtual-host: /
    #  支持发布确认
    publisher-confirms: true
    #  支持发布返回
    publisher-returns: true
    listener:
      simple:
        #  采用手动应答
        acknowledge-mode: manual
        #  当前监听容器数
        concurrency: 1
        #  最大数
        max-concurrency: 1
        #  是否支持重试
        retry:
          enabled: true

```
### 5.3 RabbitMQ Boot支持
开启支持RabbitMQ @EnableRabbit，同时配置自定义的AmqpTemplate Bean

```c

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.core.*;
import org.springframework.amqp.rabbit.annotation.EnableRabbit;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;


/**
 * <pre>
 *      RabbitMQ配置类
 * </pre>
 *
 * <pre>
 * @author mazq
 * 修改记录
 *    修改后版本:     修改人：  修改日期: 2020/04/07 11:48  修改内容:
 * </pre>
 */
@Configuration
@EnableRabbit
public class RabbitMQConfig {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Bean
    //@Primary
    public AmqpTemplate amqpTemplate(){
        Logger LOG = LoggerFactory.getLogger(AmqpTemplate.class);
        //使用jackson 消息转换器(发送对象时候才开启)
        //rabbitTemplate.setMessageConverter(new Jackson2JsonMessageConverter());
        rabbitTemplate.setEncoding("UTF-8");
        rabbitTemplate.setMandatory(true);
        // 开启returncallback    yml 需要配置publisher-returns: true
        rabbitTemplate.setReturnCallback(((message,  replyCode, replyText, exchange, routingKey) -> {
            String correlationId = message.getMessageProperties().getCorrelationId();
            LOG.info("消息：{} 发送失败, 应答码：{} 原因：{} 交换机: {}  路由键: {}", correlationId, replyCode, replyText, exchange, routingKey);
        }));
        //开启消息确认  yml 需要配置   publisher-returns: true
        rabbitTemplate.setConfirmCallback(((correlationData, ack, cause) ->{
            if (ack) {
               LOG.info("消息发送到交换机成功,correlationId:{}",correlationData.getId());
            } else {
                LOG.info("消息发送到交换机失败,原因:{}",cause);
            }
        } ));
        return rabbitTemplate;
    }
}
```

### 5.4 Direct Exchange例子

```java
 /**
     * 声明直连交换机 支持持久化.
     * @return the exchange
     */
    @Bean("directExchange")
    public Exchange directExchange() {
        return ExchangeBuilder.directExchange("amq.direct").durable(true).build();
    }

    @Bean("directQueue")
    public Queue directQueue(){
        return new Queue("directQueue", true, true, true);
        //return QueueBuilder.durable("directQueue").build();
    }

    @Bean
    public Binding directBinding(@Qualifier("directQueue")Queue queue,@Qualifier("directExchange")Exchange directExchange){
        return BindingBuilder.bind(queue).to(directExchange).with("direct_routingKey").noargs();
    }
```
在RabbitMQ管理平台，新增对应队列，并新增绑定如图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200409230802130.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0MjczOTE=,size_16,color_FFFFFF,t_70)
消息生产者：
```java
package com.example.springboot.rabbitmq.component.direct;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.UUID;

/**
 * <pre>
 *   消息生产者
 * </pre>
 *
 * <pre>
 * @author mazq
 * 修改记录
 *    修改后版本:     修改人：  修改日期: 2020/04/07 13:42  修改内容:
 * </pre>
 */
@Component
public class DirectSender {

    Logger LOG = LoggerFactory.getLogger(DirectSender.class);

    @Autowired
    private RabbitTemplate rabbitTemplate;
    public void send(int i) {
        String date = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date());
        String content = i+":hello!"+date;
        CorrelationData correlationData = new CorrelationData(UUID.randomUUID().toString());
        LOG.info("class:{},message:{}","DirectSender",content);
        this.rabbitTemplate.convertAndSend("amq.direct","direct_routingKey",content,correlationData);
    }
}

```
消息接收者：
```java
package com.example.springboot.rabbitmq.component.direct;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

/**
 * <pre>
 *   消息消费者
 * </pre>
 *
 * <pre>
 * @author mazq
 * 修改记录
 *    修改后版本:     修改人：  修改日期: 2020/04/07 13:47  修改内容:
 * </pre>
 */
@Component
@RabbitListener(queues = {"directQueue"})
public class DirectReceiver {
    Logger LOG = LoggerFactory.getLogger(DirectReceiver.class);

    @RabbitHandler
    public void receiverMsg(String msg){
        LOG.info("class:{},message:{}","DirectReceiver",msg);
    }
}

```
Junit测试：
```java
@Test
    void directSend(){
        directSender.send(1);
    }
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200409230543589.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0MjczOTE=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200409230611411.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0MjczOTE=,size_16,color_FFFFFF,t_70)
查询一下message：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200409230640990.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0MjczOTE=,size_16,color_FFFFFF,t_70)

### 5.5 Fanout Exchange例子
配置开启

```java
@Bean("fanoutQueueA")
    public Queue fanoutQueueA(){
        return new Queue("fanoutQueueA", true, true, true);
    }

    @Bean("fanoutQueueB")
    public Queue fanoutQueueB(){
        return new Queue("fanoutQueueB", true, true, true);
    }

    @Bean("fanoutQueueC")
    public Queue fanoutQueueC(){
        return new Queue("fanoutQueueC", true, true, true);
    }

    /**
     * 声明一个Fanout类型的交换器
     * @Author mazq
     * @Date 2020/04/08 11:25
     * @Param []
     * @return org.springframework.amqp.core.FanoutExchange
     */
    @Bean("fanoutExchange")
    public FanoutExchange fanoutExchange(){
        return new FanoutExchange("fanoutExchange");
    }

    @Bean
    public Binding fanoutABinding(@Qualifier("fanoutQueueA")Queue queue,FanoutExchange fanoutExchange){
        return BindingBuilder.bind(queue).to(fanoutExchange);
    }

    @Bean
    public Binding fanoutBBinding(@Qualifier("fanoutQueueB")Queue queue,FanoutExchange fanoutExchange){
        return BindingBuilder.bind(queue).to(fanoutExchange);
    }

    @Bean
    public Binding fanoutCBinding(@Qualifier("fanoutQueueC")Queue queue,FanoutExchange fanoutExchange){
        return BindingBuilder.bind(queue).to(fanoutExchange);
    }
```
新增3个接收者A、B、C：

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
@RabbitListener(queues = {"fanoutQueueA"})
public class FanoutReceiverA {

    Logger LOG = LoggerFactory.getLogger(FanoutReceiverA.class);

    @RabbitHandler
    public void process(String hello) {
        LOG.info("AReceiver  : " + hello + "/n");
    }
}
```
FanoutReceiverB、FanoutReceiverC代码类似，不贴代码

Fanout模式是发布订阅模式，不需要绑定路由键，`this.rabbitTemplate.convertAndSend("amq.fanout","",content,correlationData);`，只要和fanout exchange绑定就可以，只要队列绑定了fanout exchange，发送者发消息后，exchange都会将消息发给对应消费者队列
```java
import com.example.springboot.rabbitmq.component.direct.DirectSender;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.UUID;

@Component
public class FanoutSender {

    Logger LOG = LoggerFactory.getLogger(DirectSender.class);

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void send() {
        String date = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date());
        String content = "hello!"+date;
        CorrelationData correlationData = new CorrelationData(UUID.randomUUID().toString());
        LOG.info("class:{},message:{}","FanoutSender",content);
        this.rabbitTemplate.convertAndSend("amq.fanout","",content,correlationData);
    }

}

```
同理在RabbitMQ管理新增对应队列和绑定

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200410111835600.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0MjczOTE=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200410111800831.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0MjczOTE=,size_16,color_FFFFFF,t_70)
用Junit进行测试消息发送，ReceiverA、B、C都可以接收到消息

### 5.6 Topic Exchange例子

新增两个队列，规则为topic.msg和topic.#，#表示匹配0或多个字符
```java
@Bean("topicQueueA")
    public Queue topicQueueA(){
        return new Queue("topicQueueA",true, true, true);
    }

    @Bean("topicQueueB")
    public Queue topicQueueB(){
        return new Queue("topicQueueB",true, true, true);
    }

    @Bean("topicExchange")
    public TopicExchange topicExchange(){
        return new TopicExchange("topicExchange");
    }

    @Bean
    public Binding topicABinding(@Qualifier("topicQueueA")Queue queue,TopicExchange topicExchange){
        return BindingBuilder.bind(queue).to(topicExchange).with("topic.msg");
    }

    @Bean
    public Binding topicBBinding(@Qualifier("topicQueueB")Queue queue,TopicExchange topicExchange){
        return BindingBuilder.bind(queue).to(topicExchange).with("topic.#");
    }

```
接收者A代码：

```java
import com.example.springboot.rabbitmq.component.direct.DirectReceiver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
@RabbitListener(queues = {"topicQueueA"})
public class TopicReceiverA {

    Logger LOG = LoggerFactory.getLogger(DirectReceiver.class);

    @RabbitHandler
    public void receiverMsg(String msg){
        LOG.info("class:{},message:{}","TopicReceiverA",msg);
    }
}
```
TopicB代码类似，不贴代码，给出两个发送者代码：

```java
import com.example.springboot.rabbitmq.component.direct.DirectSender;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.UUID;


@Component
public class TopicSender {

    Logger LOG = LoggerFactory.getLogger(DirectSender.class);

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void send1() {
        String date = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date());
        String content = "hello!"+date;
        CorrelationData correlationData = new CorrelationData(UUID.randomUUID().toString());
        LOG.info("class:{},message:{}","TopicSender",content);
        this.rabbitTemplate.convertAndSend("amq.topic","topic.msg",content,correlationData);
    }

    public void send2() {
        String date = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date());
        String content = "hello!"+date;
        CorrelationData correlationData = new CorrelationData(UUID.randomUUID().toString());
        LOG.info("class:{},message:{}","TopicSender",content);
        this.rabbitTemplate.convertAndSend("amq.topic","topic.msg1",content,correlationData);
    }
}

```
同理进行队列绑定
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200410112836254.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0MjczOTE=,size_16,color_FFFFFF,t_70)

TopicA：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200410112745398.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0MjczOTE=,size_16,color_FFFFFF,t_70)
topicB：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200410112806277.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0MjczOTE=,size_16,color_FFFFFF,t_70)
**路由键是topic.msg、topic.msg1，所以send1方法执行后，两个绑定键分别为topic.msg、topic.#的都可以收到消息，send2方法执行后，只有绑定键为topic.#的队列能收到消息**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200410112844825.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0MjczOTE=,size_16,color_FFFFFF,t_70)
### 5.7 MQ对象支持例子
上面例子都是基于字符串的发送，接着可以进行对象数据的发送

```java
import lombok.*;

import java.io.Serializable;

/**
 * User信息类
 * @Author mazq
 * @Date 2020/04/08 15:12
 */
@Data
@AllArgsConstructor
@ToString
public class User implements Serializable{

    private String name;

    private String pwd;

//    @Override
//    public String toString() {
//        return "User{" +
//                "name='" + name + '\'' +
//                ", pwd='" + pwd + '\'' +
//                '}';
//    }
}
```


```java
//发送者
    public void send(User user) {
        LOG.info("Sender object: " + user.toString());
        CorrelationData correlationData = new CorrelationData(UUID.randomUUID().toString());
        this.rabbitTemplate.convertAndSend("amq.direct","direct_routingKey",user,correlationData);
    }
```

发送者：

```java
import com.example.springboot.rabbitmq.model.User;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

/**
 * <pre>
 *   消息消费者
 * </pre>
 *
 * <pre>
 * @author mazq
 * 修改记录
 *    修改后版本:     修改人：  修改日期: 2020/04/07 13:47  修改内容:
 * </pre>
 */
@Component
@RabbitListener(queues = {"directQueue"})
public class DirectReceiver {
    Logger LOG = LoggerFactory.getLogger(DirectReceiver.class);

    //接收者
    @RabbitHandler
    public void process(User user) {
        LOG.info("Receiver object : " + user);
    }
}

```
修改配置类，需要换消息转换器
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200410113412726.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0MjczOTE=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200410113529973.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0MjczOTE=,size_16,color_FFFFFF,t_70)
### 5.8 参考资料和代码例子
参考博客：
[CSDN RabbitMQ教程](https://blog.csdn.net/hellozpc/article/details/81436980)
[Springboot：RabbitMQ 详解](http://www.ityouknow.com/springboot/2016/11/30/spring-boot-rabbitMQ.html)

代码下载：[github下载链接](https://github.com/u014427391/springbootexamples/tree/master/springboot-rabbitmq)


<!--more-->
