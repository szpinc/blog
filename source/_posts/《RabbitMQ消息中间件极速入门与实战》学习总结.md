---
title: 《RabbitMQ消息中间件极速入门与实战》学习总结
date: 2019-01-07 15:22:50
tags:
    - 消息队列
categories:
    - 学习笔记
---

## 第一章：RabbitMQ起步
### 1-1 课程导航

课程导航
* RabbitMQ简介及AMQP协议
* RabbitMQ安装与使用
* RabbitMQ核心概念
* 与SpringBoot整合
* 保障100%的消息可靠性投递方案落地实现

### 1-2 RabbitMQ简介

初识RabbitMQ
* RabbitMQ是一个开源的消息代理和队列服务器
* 用来通过普通协议在完全不同的应用之间共享数据
* RabbitMQ是使用Erlang语言来编写的
* 并且RabbitMQ是基于AMQP协议的

RabbitMQ简介
* 目前很多互联网大厂都在使用RabbitMQ
* RabbitMQ底层采用Erlang语言进行编写
* 开源、性能优秀，稳定性保障
* 与SpringAMQP完美的整合、API丰富
* 集群模式丰富，表达式配置，HA模式，镜像队列模型
* 保证数据不丢失的前提做到高可靠性、可用性
* AMQP全称：Advanced Message Queuing Protocol
* AMQP翻译：高级消息队列协议

AMQP协议模型
![](https://image-static.segmentfault.com/285/334/2853347369-5b94bb059cead_articlex)

### 1-3 RabbitMQ安装

学习笔记

``` bash
0.安装准备
官网地址：http://www.rabbitmq.com/
安装Linux必要依赖包<Linux7>
下载RabbitMQ安装包
进行安装，修改相关配置文件
vim /etc/hostname
vim /etc/hosts

1.安装Erlang
wget https://packages.erlang-solutions.com/erlang-solutions_1.0_all.deb
sudo dpkg -i erlang-solutions_1.0_all.deb
sudo apt-get install erlang erlang-nox

2.安装RabbitMQ
echo 'deb http://www.rabbitmq.com/debian/ testing main' | sudo tee /etc/apt/sources.list.d/rabbitmq.list
wget -O- https://www.rabbitmq.com/rabbitmq-release-signing-key.asc | sudo apt-key add -
sudo apt-get install rabbitmq-server

3.安装RabbitMQ web管理插件
sudo rabbitmq-plugins enable rabbitmq_management
sudo systemctl restart rabbitmq-server
访问：http://localhost:15672
默认用户名密码：guest/guest

4.修改RabbitMQ配置
vim /usr/lib/rabbitmq/lib/rabbitmq_server-3.5.7/ebin/rabbit.app
比如修改密码、配置等等；例如：loopback_users中的<<"guest">>，只保留guest
服务启动：rabbitmq-server start &
服务停止：rabbitmqctl app_stop
```

### 1-4 RabbitMQ概念

RabbitMQ的整体架构
![](https://image-static.segmentfault.com/815/706/815706350-5b94bb247c51a_articlex)

RabbitMQ核心概念
* Server：又称Broker，接受客户端的连接，实现AMQP实体服务
* Connection：连接，应用程序与Broker的网络连接
* Channel：网络信道
> 几乎所有的操作都在Channel中进行
Channel是进行消息读写的通道
客户端可建立多个Channel
每个Channel代表一个会话任务
* Message：消息
> 服务器和应用程序之间传送的数据，由Properties和Body组成
Properties可以对消息进行修饰，比如消息的优先级、延迟等高级特性
Body则就是消息体内容
* Virtual host：虚拟机
>用于进行逻辑隔离，最上层的消息路由
一个Virtual host里面可以有若干个Exchange和Queue
同一个Virtual host里面不能有相同名称的Exchange或Queue
* Exchange：交换机，接收消息，根据路由键转发消息到绑定的队列
* Binding：Exchange和Queue之间的虚拟连接，binding中可以包含routing key
* Routing key：一个路由规则，虚拟机可用它来确定如何路由一个特定消息
* Queue：也称为Message Queue，消息队列，保存消息并将它们转发给消费者

RabbitMQ消息的流转过程

![](https://image-static.segmentfault.com/152/410/1524103111-5b94bb2f365ac_articlex)

## 第二章：RabbitMQ使用
### 2-1 发送消息

SpringBoot与RabbitMQ集成
* 引入相关依赖
* 对application.properties进行配置

创建名为rabbitmq-producer的maven工程pom如下

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>47-rabbitmq</artifactId>
        <groupId>com.myimooc</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>rabbitmq-producer</artifactId>

    <properties>
        <spring.boot.version>2.0.4.RELEASE</spring.boot.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-parent</artifactId>
                <version>${spring.boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <!--RabbitMQ依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>

        <!--工具类依赖-->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.5</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.36</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>

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
            </plugin>
        </plugins>
    </build>

</project>
```

1.编写Order类

``` java
package org.szpinc.study.mq.rabbitmq.entity;

import lombok.Data;
import java.io.Serializable;

/**
 * 订单消息对象
 *
 * @author GhostDog
 */
@Data
public class Order implements Serializable {

    private String id;

    private String name;

    private String messageId;
}

```

2.编写OrderSender类

``` java
package org.szpinc.study.mq.rabbitmq.producer;

import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.szpinc.study.mq.rabbitmq.entity.Order;

/**
 * 订单消息发送
 *
 * @author GhostDog
 */
@Component
public class OrderSender {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    /**
     * 消息发送
     *
     * @param order 订单消息对象
     * @throws Exception 异常
     */
    public void send(Order order) throws Exception {
        rabbitTemplate.convertAndSend("order-exchange", "order.demo1", order, new CorrelationData(order.getMessageId()));
    }


}
```

3.编写application.properties类

``` yml
# RabbitMQ配置
server:
  port: 8080
  servlet:
    context-path: /study/rabbitmq/
spring:
  http:
    encoding:
      charset: UTF-8
  jackson:
    time-zone: GMT+8
    date-format: yyyy-MM-dd HH:mm:ss
    default-property-inclusion: non_null
  # rabbitmq配置
  rabbitmq:
    username: szpinc
    password: 123456
    host: 192.168.130.110
    port: 5672
    virtual-host: /
    connection-timeout: 15
    listener:
      simple:
        # 并发数
        concurrency: 5
        # 签收模式（手动）
        acknowledge-mode: manual


# 日志配置
logging:
  level:
    org.szpinc.study.mq.rabbitmq: debug
```

4.编写Application类

``` java
package org.szpinc.study.mq.rabbitmq;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @author GhostDog
 */
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}

```

5.编写OrderSenderTest类

``` java
package org.szpinc.study.mq.rabbitmq.test;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.util.DigestUtils;
import org.szpinc.study.mq.rabbitmq.entity.Order;
import org.szpinc.study.mq.rabbitmq.producer.OrderSender;

import java.util.UUID;

@RunWith(SpringRunner.class)
@SpringBootTest
public class TestSendOrderMessage {

    @Autowired
    private OrderSender orderSender;

    @Test
    public void sendOrder() throws Exception {

        Order order = new Order();
        order.setId(String.valueOf(System.currentTimeMillis()));
        order.setName("order01");
        order.setMessageId(DigestUtils.md5DigestAsHex((System.currentTimeMillis() + "$" + UUID.randomUUID().toString()).getBytes()));
        orderSender.send(order);
    }

}
```

2-2 处理消息

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>47-rabbitmq</artifactId>
        <groupId>com.myimooc</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>rabbitmq-consumer</artifactId>

    <properties>
        <spring.boot.version>2.0.4.RELEASE</spring.boot.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-parent</artifactId>
                <version>${spring.boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <!--RabbitMQ依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>

        <!--工具类依赖-->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.5</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.36</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>

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
            </plugin>
        </plugins>
    </build>

</project>
```

1.编写Order类

``` java
package org.szpinc.study.mq.rabbitmq.entity;

import lombok.Data;

import java.io.Serializable;

/**
 * 订单消息对象
 *
 * @author GhostDog
 */
@Data
public class Order implements Serializable {

    private String id;

    private String name;

    private String messageId;
}
```

2.编写OrderReceiver类

``` java
package org.szpinc.study.mq.rabbitmq.consumer;


import com.rabbitmq.client.Channel;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.annotation.*;
import org.springframework.amqp.support.AmqpHeaders;
import org.springframework.messaging.handler.annotation.Headers;
import org.springframework.messaging.handler.annotation.Payload;
import org.springframework.stereotype.Component;
import org.szpinc.study.mq.rabbitmq.entity.Order;

import java.io.IOException;
import java.util.Map;

/**
 * @author GhostDog
 */
@Component
@Slf4j
public class OrderReceiver {

    @RabbitListener(
            bindings = @QueueBinding(
                    value = @Queue(value = "order-queue", durable = "true"),
                    exchange = @Exchange(value = "order-exchange", durable = "true", type = "topic"),
                    key = "order.#"
            )
    )
    @RabbitHandler
    public void orderHandler(@Payload Order order, @Headers Map<String, Object> headers, Channel channel) throws IOException {
        log.debug("消息开始消费");
        log.debug("消息信息：[{}]", order);
        long deliveryTag = (long) headers.get(AmqpHeaders.DELIVERY_TAG);
        //通知rabbitmq消息已消费
        channel.basicAck(deliveryTag, false);
    }
}

```

3.编写application.properties类

``` yml
server:
  port: 8080
  servlet:
    context-path: /study/rabbitmq/
spring:
  http:
    encoding:
      charset: UTF-8
  jackson:
    time-zone: GMT+8
    date-format: yyyy-MM-dd HH:mm:ss
    default-property-inclusion: non_null
  # rabbitmq配置
  rabbitmq:
    username: szpinc
    password: 123456
    host: 192.168.130.110
    port: 5672
    virtual-host: /
    connection-timeout: 15
    listener:
      simple:
        # 基本并发数
        concurrency: 5
        # 最大并发数
        max-concurrency: 10
        # 签收模式（手动）
        acknowledge-mode: manual
        # 并发策略(同一时间只有1条消息发来消费)
        prefetch: 1

# 日志配置
logging:
  level:
    org.szpinc.study.mq.rabbitmq: debug
```

4.编写Application类

``` java
package org.szpinc.study.mq.rabbitmq;


import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @author GhostDog
 */
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}

```

## 第三章：可靠性投递
### 3-1 设计方案

保障100%消息投递成功设计方案（一）
![](https://image-static.segmentfault.com/100/074/1000745068-5b94bb4f85a5b_articlex)

### 3-2 代码详解

因篇幅限制，源码请到github地址查看，这里仅展示核心关键类

1.编写OrderSender类

``` java
package com.myimooc.rabbitmq.ha.producer;

import com.myimooc.rabbitmq.entity.Order;
import com.myimooc.rabbitmq.ha.constant.Constants;
import com.myimooc.rabbitmq.ha.dao.mapper.BrokerMessageLogMapper;
import com.myimooc.rabbitmq.ha.dao.po.BrokerMessageLogPO;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.rabbit.support.CorrelationData;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

/**
 * <br>
 * 标题: 订单消息发送者<br>
 * 描述: 订单消息发送者<br>
 * 时间: 2018/09/06<br>
 *
 * @author zc
 */
@Component
public class OrderSender {

    @Autowired
    private RabbitTemplate rabbitTemplate;
    @Autowired
    private BrokerMessageLogMapper brokerMessageLogMapper;
    /**
     * 回调方法：confirm确认
     */
    private final RabbitTemplate.ConfirmCallback confirmCallback = new RabbitTemplate.ConfirmCallback() {
        @Override
        public void confirm(CorrelationData correlationData, boolean ack, String cause) {
            System.out.println("correlationData：" + correlationData);
            String messageId = correlationData.getId();
            if (ack) {
                // 如果confirm返回成功，则进行更新
                BrokerMessageLogPO messageLogPO = new BrokerMessageLogPO();
                messageLogPO.setMessageId(messageId);
                messageLogPO.setStatus(Constants.OrderSendStatus.SEND_SUCCESS);
                brokerMessageLogMapper.changeBrokerMessageLogStatus(messageLogPO);
            } else {
                // 失败则进行具体的后续操作：重试或者补偿等
                System.out.println("异常处理...");
            }
        }
    };

    /**
     * 发送订单
     *
     * @param order 订单
     */
    public void send(Order order) {
        // 设置回调方法
        this.rabbitTemplate.setConfirmCallback(confirmCallback);
        // 消息ID
        CorrelationData correlationData = new CorrelationData(order.getMessageId());
        // 发送消息
        this.rabbitTemplate.convertAndSend("order-exchange", "order.a", order, correlationData);
    }
}
```

2.编写OrderService类

``` java
package com.myimooc.rabbitmq.ha.service;

import com.myimooc.rabbitmq.entity.Order;
import com.myimooc.rabbitmq.ha.constant.Constants;
import com.myimooc.rabbitmq.ha.dao.mapper.BrokerMessageLogMapper;
import com.myimooc.rabbitmq.ha.dao.mapper.OrderMapper;
import com.myimooc.rabbitmq.ha.dao.po.BrokerMessageLogPO;
import com.myimooc.rabbitmq.ha.producer.OrderSender;
import com.myimooc.rabbitmq.ha.util.FastJsonConvertUtils;
import org.apache.commons.lang3.time.DateUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.Date;

/**
 * <br>
 * 标题: 订单服务<br>
 * 描述: 订单服务<br>
 * 时间: 2018/09/07<br>
 *
 * @author zc
 */
@Service
public class OrderService {

    @Autowired
    private OrderMapper orderMapper;
    @Autowired
    private BrokerMessageLogMapper brokerMessageLogMapper;
    @Autowired
    private OrderSender orderSender;

    /**
     * 创建订单
     *
     * @param order 订单
     */
    public void create(Order order) {
        // 当前时间
        Date orderTime = new Date();
        // 业务数据入库
        this.orderMapper.insert(order);
        // 消息日志入库
        BrokerMessageLogPO messageLogPO = new BrokerMessageLogPO();
        messageLogPO.setMessageId(order.getMessageId());
        messageLogPO.setMessage(FastJsonConvertUtils.convertObjectToJson(order));
        messageLogPO.setTryCount(0);
        messageLogPO.setStatus(Constants.OrderSendStatus.SENDING);
        messageLogPO.setNextRetry(DateUtils.addMinutes(orderTime, Constants.ORDER_TIMEOUT));
        this.brokerMessageLogMapper.insert(messageLogPO);
        // 发送消息
        this.orderSender.send(order);
    }
}
```

3.编写RetryMessageTask类

``` java
package com.myimooc.rabbitmq.ha.task;

import com.myimooc.rabbitmq.entity.Order;
import com.myimooc.rabbitmq.ha.constant.Constants;
import com.myimooc.rabbitmq.ha.dao.mapper.BrokerMessageLogMapper;
import com.myimooc.rabbitmq.ha.dao.po.BrokerMessageLogPO;
import com.myimooc.rabbitmq.ha.producer.OrderSender;
import com.myimooc.rabbitmq.ha.util.FastJsonConvertUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.util.List;

/**
 * <br>
 * 标题: 重发消息定时任务<br>
 * 描述: 重发消息定时任务<br>
 * 时间: 2018/09/07<br>
 *
 * @author zc
 */
@Component
public class RetryMessageTask {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private OrderSender orderSender;
    @Autowired
    private BrokerMessageLogMapper brokerMessageLogMapper;

    /**
     * 启动完成3秒后开始执行，每隔10秒执行一次
     */
    @Scheduled(initialDelay = 3000, fixedDelay = 10000)
    public void retrySend() {
        logger.debug("重发消息定时任务开始");
        // 查询 status = 0 和 timeout 的消息日志
        List<BrokerMessageLogPO> pos = this.brokerMessageLogMapper.listSendFailureAndTimeoutMessage();
        for (BrokerMessageLogPO po : pos) {
            logger.debug("处理消息日志：{}",po);
            if (po.getTryCount() >= Constants.MAX_RETRY_COUNT) {
                // 更新状态为失败
                BrokerMessageLogPO messageLogPO = new BrokerMessageLogPO();
                messageLogPO.setMessageId(po.getMessageId());
                messageLogPO.setStatus(Constants.OrderSendStatus.SEND_FAILURE);
                this.brokerMessageLogMapper.changeBrokerMessageLogStatus(messageLogPO);
            } else {
                // 进行重试，重试次数+1
                this.brokerMessageLogMapper.updateRetryCount(po);
                Order reSendOrder = FastJsonConvertUtils.convertJsonToObject(po.getMessage(), Order.class);
                try {
                    this.orderSender.send(reSendOrder);
                } catch (Exception ex) {
                    // 异常处理
                    logger.error("消息发送异常：{}", ex);
                }
            }
        }
        logger.debug("重发消息定时任务结束");
    }
}
```

4.编写ApplicationTest类

``` java
package com.myimooc.rabbitmq.ha;

import com.myimooc.rabbitmq.entity.Order;
import com.myimooc.rabbitmq.ha.service.OrderService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.UUID;

/**
 * <br>
 * 标题: 订单创建测试<br>
 * 描述: 订单创建测试<br>
 * 时间: 2018/09/07<br>
 *
 * @author zc
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class ApplicationTest {

    @Autowired
    private OrderService orderService;

    @Test
    public void testCreateOrder(){
        Order order = new Order();
        order.setId(String.valueOf(System.currentTimeMillis()));
        order.setName("测试创建订单");
        order.setMessageId(System.currentTimeMillis() + "$" + UUID.randomUUID().toString().replaceAll("-",""));
        this.orderService.create(order);
    }

}
```