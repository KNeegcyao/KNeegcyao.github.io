---
title: 深入浅出消息队列(RabbitMQ)
tags:
  - 中间件
top_img: 'https://haowallpaper.com/link/common/file/previewFileImg/15918089447001472'
cover: 'https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image.png'
abbrlink: 93c6a719
date: 2025-03-14 22:44:55
---

# 深入浅出消息队列 (RabbitMQ)

## 1. 前言

在分布式系统中，**消息队列 (Message Queue, MQ)** 经常被用来实现异步解耦、流量削峰等功能。它能够让系统在高并发环境下更具弹性，并且在各个模块之间实现“松耦合”的交互。

- **什么是消息队列？**  
  消息队列是指在消息传输过程中保存消息的容器，可以帮助不同进程/应用之间异步通信。
- 你可以把消息队列想象成一个“菜鸟驿站”。就像你网购时，快递先送到驿站，你再去取货一样，消息队列里先存储消息，等消费者来“取走”处理。这样，生产消息的部分（就像快递员）和处理消息的部分（就像取快递的人）就不会因为时间不匹配而互相拖累。简单来说，菜鸟驿站帮助解决快递和取件的时差问题，消息队列则解决了系统中生产和消费的速度差异问题。
- ![image-20250314212026844](https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20250314212026844.png)
- **为什么要使用消息队列？**  
  - 系统解耦：生产者和消费者不必直接调用。  
  - 异步处理：生产者把任务丢到队列即可，后续由消费者异步执行。  
  - 流量削峰：将突发的请求平滑化处理，防止服务被压垮。  
  - 数据分发：在分布式环境中，能快速地把数据分发给不同的消费者。

---

## 2. MQ 的基本概念

- 异步调用方式其实就是基于消息通知的方式，一般包含三个角色：

  - 消息发送者：投递消息的人，就是原来的调用方
  - 消息Broker：管理、暂存、转发消息，你可以把它理解成微信服务器
  - 消息接收者：接收和处理消息的人，就是原来的服务提供方

  ![image-20250408094739702](https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20250408094739702.png)

  在异步调用中，发送者不再直接同步调用接收者的业务接口，而是发送一条消息投递给消息Broker。然后接收者根据自己的需求从消息Broker那里订阅消息。每当发送方发送消息后，接受者都能获取消息并处理。

  这样，发送消息的人和接收消息的人就完全解耦了。

  ***消息Broker，目前常见的实现方案就是消息队列（MessageQueue），简称为MQ.***

---

## 3. 常见的消息队列实现

|            | RabbitMQ                | ActiveMQ                       | RocketMQ   | Kafka      |
| ---------- | ----------------------- | ------------------------------ | ---------- | ---------- |
| 公司/社区  | Rabbit                  | Apache                         | 阿里       | Apache     |
| 开发语言   | Erlang                  | Java                           | Java       | Scala&Java |
| 协议支持   | AMQP，XMPP，SMTP，STOMP | OpenWire,STOMP，REST,XMPP,AMQP | 自定义协议 | 自定义协议 |
| 可用性     | 高                      | 一般                           | 高         | 高         |
| 单机吞吐量 | 一般                    | 差                             | 高         | 非常高     |
| 消息延迟   | 微秒级                  | 毫秒级                         | 毫秒级     | 毫秒以内   |
| 消息可靠性 | 高                      | 一般                           | 高         | 一般       |

**追求可用性：**Kafka、 RocketMQ 、RabbitMQ

**追求可靠性：**RabbitMQ、RocketMQ

**追求吞吐能力：**RocketMQ、Kafka

**追求消息低延迟：**RabbitMQ、Kafka

----



## 4. 消息队列的使用场景

### 4.1异步处理



![image-20250314223318585](https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20250314223318585.png)

如果没有引入MQ进行架构改造，每次支付成功后的大量同步接口调用，耗时大，通过引入MQ，当更新完订单状态和扣减成功库存后，就发一条“支付成功”消息到MQ中，然后立即返回。而信息、积分、优惠券系统会订阅MQ中的该类消息，它们收到通知后就会去异步处理。整个系统的响应时间可以大大缩短,

### 4.2 **削峰/限流**。 

 先将短时间高并发产生的事务消息存储在消息队列中，然后后端服务再慢慢根据自己的能力去消费这些消息，这样就避免直接把后端服务打垮掉。

![image-20250314222512088](https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20250314222512088.png)

像之前黑马点评项目中的**异步秒杀下单**，判断库存，检验一人一单，只要满足这两个条件就一定可以下单成功，不用等数据真的写进数据库中，可以直接告诉用户下单成功，只需要将订单id等信息引入异步队列记录，后台再开一个线程慢慢去执行队列中的消息就行，有效提高效率。

### 4.3 解耦

解耦和异步是同生同源的，我们经过上述的异步化改造后，自然而然的就已经将各个系统解耦出去了.如果模块之间不存在直接调用，那么新增模块或者修改模块就对其他模块影响较小，这样系统的可扩展性无疑更好一些。

## 5. 示例：如何简单使用 RabbitMQ

### 安装与配置

1. [RabbitMQ 官方下载](https://www.rabbitmq.com/download.html)  
2. 安装后可通过管理控制台查看队列状态。

![image-20250408101153944](https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20250408101153944.png)

- **`publisher`**：生产者，也就是发送消息的一方
- **`consumer`**：消费者，也就是消费消息的一方
- **`queue`**：队列，存储消息。生产者投递的消息会暂存在消息队列中，等待消费者处理
- **`exchange`**：交换机，负责消息路由。生产者发送的消息由交换机决定投递到哪个队列。
- **`virtual host`**：虚拟主机，起到数据隔离的作用。每个虚拟主机相互独立，有各自的exchange、queue

#### WorkQueues模型

Work queues，任务模型。简单来说就是让**多个消费者** 绑定到一个队列，共同消费队列中的消息。

![image-20250408102511471](https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20250408102511471.png)



当消息处理比较耗时的时候，可能生产消息的速度会远远大于消息的消费速度。长此以往，消息就会堆积越来越多，无法及时处理。

此时就可以使用work 模型，**多个消费者共同处理消息处理，消息处理的速度就能大大提高**了。

#### 交换机类型

没有交换机，生产者直接发送消息到队列。而一旦引入交换机，消息发送的模式会有很大变化：

![image-20250408102756551](https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20250408102756551.png)

 可以看到，在订阅模型中，多了一个exchange角色，而且过程略有变化：

  - **Publisher**：生产者，不再发送消息到队列中，而是发给交换机
  - **Exchange**：交换机，一方面，接收生产者发送的消息。另一方面，知道如何处理消息，例如递交给某个特别队列、递交给所有队列、或是将消息丢弃。到底如何操作，取决于Exchange的类型。
- **Queue**：消息队列也与以前一样，接收消息、缓存消息。不过队列一定要与交换机绑定。
- **Consumer**：消费者，与以前一样，订阅队列，没有变化

**Exchange（交换机）只负责转发消息，不具备存储消息的能力**，因此如果没有任何队列与Exchange绑定，或者没有符合路由规则的队列，那么消息会丢失！

交换机的类型有四种：

- **Fanout**：广播，将消息交给所有绑定到交换机的队列。我们最早在控制台使用的正是Fanout交换机
- **Direct**：订阅，基于RoutingKey（路由key）发送给订阅了消息的队列
- **Topic**：通配符订阅，与Direct类似，只不过RoutingKey可以使用通配符
- **Headers**：头匹配，基于MQ的消息头匹配，用的较少。

### 第一步，配置pom包。

创建Spring Boot项目并在pom.xml文件中添加spring-bootstarter-amqp等相关组件依赖：

```
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

在上面的示例中，引入Spring Boot自带的amqp组件spring-bootstarter-amqp。

SpringAMQP提供了三个功能：

- 自动声明队列、交换机及其绑定关系
- 基于注解的监听器模式，异步接收消息
- 封装了RabbitTemplate工具，用于发送消



### 第二步，修改配置文件。

修改application.yml配置文件，配置rabbitmq的host地址、端口以及账户信息。

```yaml
  spring:
    rabbitmq:
      host: localhost
      port: 5672
      username: guest
      password: guest
```

### 声明队列和交换机

***基于API***由程序启动时检查队列和交换机是否存在，如果不存在自动创建

创建一个类，声明队列和交换机

#### fanout示例

```java
package com.itheima.consumer.config;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.FanoutExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FanoutConfig {
    /**
     * 声明交换机
     * @return Fanout类型交换机
     */
    @Bean
    public FanoutExchange fanoutExchange(){
        return new FanoutExchange("hmall.fanout");
    }

    /**
     * 第1个队列
     */
    @Bean
    public Queue fanoutQueue1(){
        return new Queue("fanout.queue1");
    }

    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue1(Queue fanoutQueue1, FanoutExchange fanoutExchange){
        return BindingBuilder.bind(fanoutQueue1).to(fanoutExchange);
    }

    /**
     * 第2个队列
     */
    @Bean
    public Queue fanoutQueue2(){
        return new Queue("fanout.queue2");
    }

    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue2(Queue fanoutQueue2, FanoutExchange fanoutExchange){
        return BindingBuilder.bind(fanoutQueue2).to(fanoutExchange);
    }
}
```

#### direct示例

direct模式由于要绑定多个KEY，会非常麻烦，每一个Key都要编写一个binding：

```Java
package com.itheima.consumer.config;

import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class DirectConfig {

    /**
     * 声明交换机
     * @return Direct类型交换机
     */
    @Bean
    public DirectExchange directExchange(){
        return ExchangeBuilder.directExchange("hmall.direct").build();
    }

    /**
     * 第1个队列
     */
    @Bean
    public Queue directQueue1(){
        return new Queue("direct.queue1");
    }

    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue1WithRed(Queue directQueue1, DirectExchange directExchange){
        return BindingBuilder.bind(directQueue1).to(directExchange).with("red");
    }
    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue1WithBlue(Queue directQueue1, DirectExchange directExchange){
        return BindingBuilder.bind(directQueue1).to(directExchange).with("blue");
    }

    /**
     * 第2个队列
     */
    @Bean
    public Queue directQueue2(){
        return new Queue("direct.queue2");
    }

    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue2WithRed(Queue directQueue2, DirectExchange directExchange){
        return BindingBuilder.bind(directQueue2).to(directExchange).with("red");
    }
    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue2WithYellow(Queue directQueue2, DirectExchange directExchange){
        return BindingBuilder.bind(directQueue2).to(directExchange).with("yellow");
    }
}
```

#### 基于注解声明

基于@Bean的方式声明队列和交换机比较麻烦，Spring还提供了基于注解方式来声明。

例如，我们同样声明Direct模式的交换机和队列：

```Java
@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "direct.queue1"),
    exchange = @Exchange(name = "hmall.direct", type = ExchangeTypes.DIRECT),
    key = {"red", "blue"}
))
public void listenDirectQueue1(String msg){
    System.out.println("消费者1接收到direct.queue1的消息：【" + msg + "】");
}

@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "direct.queue2"),
    exchange = @Exchange(name = "hmall.direct", type = ExchangeTypes.DIRECT),
    key = {"red", "yellow"}
))
public void listenDirectQueue2(String msg){
    System.out.println("消费者2接收到direct.queue2的消息：【" + msg + "】");
}
```

Topic模式：

```Java
@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "topic.queue1"),
    exchange = @Exchange(name = "hmall.topic", type = ExchangeTypes.TOPIC),
    key = "china.#"
))
public void listenTopicQueue1(String msg){
    System.out.println("消费者1接收到topic.queue1的消息：【" + msg + "】");
}

@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "topic.queue2"),
    exchange = @Exchange(name = "hmall.topic", type = ExchangeTypes.TOPIC),
    key = "#.news"
))
public void listenTopicQueue2(String msg){
    System.out.println("消费者2接收到topic.queue2的消息：【" + msg + "】");
}
```

### 消息转换器

#### 默认转换器

![image-20250408110042858](https://cdn.jsdelivr.net/gh/KNeegcyao/picdemo/img/image-20250408110042858.png)

Spring的消息发送代码接收的消息体是一个Object：而在数据传输时，它会把你发送的消息序列化为字节发送给MQ，接收消息的时候，还会把字节反序列化为Java对象。只不过，默认情况下Spring采用的序列化方式是JDK序列化。

JDK序列化存在下列问题：

- 数据体积过大
- 有安全漏洞
- 可读性差

#### 配置JSON转换器

显然，JDK序列化方式并不合适。我们希望消息体的体积更小、可读性更高，因此可以使用JSON方式来做序列化和反序列化

启动类中添加一个Bean即可

```java
@Bean
public MessageConverter messageConverter(){
    // 1.定义消息转换器
    Jackson2JsonMessageConverter jackson2JsonMessageConverter = new Jackson2JsonMessageConverter();
    // 2.配置自动创建消息id，用于识别不同消息，也可以在业务中基于ID判断是否是重复消息
    jackson2JsonMessageConverter.setCreateMessageIds(true);
    return jackson2JsonMessageConverter;
}
```

#### 消费者接收Object

我们定义一个新的消费者，publisher是用Map发送，那么消费者也一定要用Map接收，格式如下：

```Java
@RabbitListener(queues = "object.queue")
public void listenSimpleQueueMessage(Map<String, Object> msg) throws InterruptedException {
    System.out.println("消费者接收到object.queue消息：【" + msg + "】");
}
```