---
title: JMS基本概念及ActiveMQ使用
date: 2016-05-22 12:57:22 
author: Zak
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wuzguo
authorDesc: 一个追求进步的「十八线码农」
categories: 后台
tags: 
- JMS
- ActiveMQ
- MQ
keywords: JMS，ActiveMQ
photos:
- /blog/images/201605/2.png
description: ActiveMQ的基本使用
---


1.JMS介绍
JMS（Java Messaging Service）即Java消息服务，是Java平台上有关面向消息中间件(MOM)的技术规范，是一个Java平台中关于面向消息中间件（MOM）的API，用于在两个应用程序之间，或分布式系统中发送消息，进行异步通信。它便于消息系统中的Java应用程序进行消息交换,并且通过提供标准的产生、发送、接收消息的接口简化企业应用的开发。
JMS是一种与厂商无关的 API，用来访问消息收发系统消息，它类似于JDBC(Java Database Connectivity)。这里，JDBC 是可以用来访问许多不同关系数据库的 API，而 JMS 则提供同样与厂商无关的访问方法，以访问消息收发服务。许多厂商都支持 JMS，包括 IBM 的 MQSeries、BEA的 Weblogic JMS service和 Progress 的 SonicMQ。

JMS 使您能够通过消息收发服务（有时称为消息中介程序或路由器）从一个 JMS 客户机向另一个 JMS客户机发送消息。消息是 JMS 中的一种类型对象，由三部分组成：
1. 报头（head）
每条JMS 消息都必须具有消息头。头由路由信息以及有关该消息的元数据组成。可以通过多种方式来设置消息头的值：
- 由JMS 提供者在生成或传送消息的过程中自动设置
- 由生产者客户机通过在创建消息生产者时指定的设置进行设置
- 由生产者客户机逐一对各条消息进行设置
 
2. 属性（property）
消息可以包含称作属性的可选头字段。他们是以属性名和属性值对的形式制定的。可以将属性归类为消息头得扩展，其中可以包括如下信息：
- 创建数据的进程
- 数据的创建时间
- 每条数据的结构
JMS提供者也可以添加影响消息处理的属性，如是否应压缩消息或如何在消息生命周期结束时废弃消息。
 
3. 主体（body）
包含要发送给接收应用程序的内容。每个消息接口特定于它所支持的内容类型。JMS为不同类型的内容提供了他们各自的消息类型，但是所有消息都派生自Message接口。
- StreamMessage     一种主体中包含Java基元值流的消息。其填充和读取均按顺序进行
- MapMessage	    一种主体中包含一组键--值对的消息。没有定义条目顺序
- TextMessage       一种主体中包含Java字符串的消息（例如，XML消息）
- ObjectMessage     一种主体中包含序列化Java对象的消息
- BytesMessage      一种主体中包含连续字节流的消息

例如:MapMessage 消息格式
 
    MapMessage={  
    Header={  
    ... standard headers ...  
    CorrelationID={123-00001}  
    }  
    Properties={  
    AccountID={Integer:1234}  
    }  
    Fields={  
    Name={String:Mark}  
    Age={Integer:47}  
    }   
    }  
 
2.JMS传递模型
JMS 的编程过程很简单，概括为：应用程序A 发送一条消息到消息服务器（也就是JMS Provider）的某个目得地(Destination)，然后消息服务器把消息转发给应用程序B。因为应用程序A 和应用程序B 没有直接的代码关连，所以两者实现了解偶。如下图：
![](/images/201605/8.jpg)


JMS支持两种消息传递模型：点对点（point-to-point，简称PTP）和发布/订阅（publish/subscribe,简称pub/sub）。这两种消息传递模型非常相似，但有以下区别:
- PTP消息传递模型规定了一条消息之恩能够传递费一个接收方
- Pub/sub消息传递模型允许一条消息传递给多个接收方

每个模型都通过扩展公用基类来实现。例如：javax.jms.Queue和Javax.jms.Topic都扩展自javax.jms.Destination类。
 
1. 点对点消息传递
通过点对点的消息传递模型，一个应用程序可以向另外一个应用程序发送消息。在此传递模型中，目标类型时队列。消息首先被传送至队列目标，然后从改对垒将消息传送至对此队列进行监听的某个消费者，如下图:
![](/images/201605/9.jpg)
 
一个队列可以关联多个队列发送方和接收方，但一条消息仅传递给一个接收方。如果多个接收方正在监听队列上的消息，JMS Provider将根据“先来者优先”的原则确定由哪个价售房接受下一条消息。如果没有接收方在监听队列，消息将保留在队列中，直至接收方连接到队列为止。这种消息传递模型是传统意义上的拉模型或轮询模型。在此列模型中，消息不时自动推动给客户端的，而是要由客户端从队列中请求获得。
 
2. 发布/订阅消息传递
通过发布/订阅消息传递模型，应用程序能够将一条消息发送到多个接收方。在此传送模型中，目标类型是主题。消息首先被传送至主题目标，然后传送至所有已订阅此主题的或送消费者。如下图：
![](/images/201605/10.jpg)

主题目标也支持长期订阅。长期订阅表示消费者已注册了主题目标，但在消息到达目标时改消费者可以处于非活动状态。当消费者再次处于活动状态时，将会接收该消息。如果消费者均没有注册某个主题目标，该主题只保留注册了长期订阅的非活动消费者的消息。与PTP消息传递模型不同，pub/sub消息传递模型允许多个主题订阅者接收同一条消息。JMS一直保留消息，直至所有主题订阅者都接收到消息为止。pub/sub消息传递模型基本上时一个推模型。在该模型中，消息会自动广播，消费者无须通过主动请求或轮询主题的方法来获得新的消息。
 
上面两种消息传递模型里，我们都需要定义消息生产者和消费者，生产者吧消息发送到JMS Provider的某个目标地址（Destination），消息从该目标地址传送至消费者。消费者可以同步或异步接收消息，一般而言，异步消息消费者的执行和伸缩性都优于同步消息接收者，体现在：
1. 异步消息接收者创建的网络流量比较小。单向对东消息，并使之通过管道进入消息监听器。管道操作支持将多条消息聚合为一个网络调用。
2. 异步消息接收者使用线程比较少。异步消息接收者在不活动期间不使用线程。同步消息接收者在接收调用期间内使用线程，结果线程可能会长时间保持空闲，尤其是如果该调用中指定了阻塞超时。
3. 对于服务器上运行的应用程序代码，使用异步消息接收者几乎总是最佳选择，尤其是通过消息驱动Bean。使用异步消息接收者可以防止应用程序代码在服务器上执行阻塞操作。而阻塞操作会是服务器端线程空闲，甚至会导致死锁。阻塞操作使用所有线程时则发生死锁。如果没有空余的线程可以处理阻塞操作自身解锁所需的操作，这该操作永远无法停止阻塞。
 
 
3.JMS Provider（ActiveMQ）
上面介绍了JMS的基本概念，下面介绍一款开源的JMS具体实现ActiveMQ，ActiveMQ是由Apache出品的，一款最流行的，能力强劲的开源消息总线（消息中间件）。ActiveMQ是一个完全支持JMS1.1和J2EE 1.4规范的 JMS Provider实现，它非常快速，支持多种语言的客户端和协议，而且可以非常容易的嵌入到企业的应用环境中，并有许多高级功能。

1. MQ
首先简单的介绍一下MQ，MQ英文名MessageQueue，中文名也就是大家用的消息队列，干嘛用的呢，说白了就是一个消息的接受和转发的容器，可用于消息推送。

2. 消息中间件
我们简单的介绍一下消息中间件，对它有一个基本认识就好，消息中间件（MOM：Message Orient middleware）。


3. 消息中间件有很多的用途和优点： 
- 将数据从一个应用程序传送到另一个应用程序，或者从软件的一个模块传送到另外一个模块
- 负责建立网络通信的通道，进行数据的可靠传送
- 保证数据不重发，不丢失 
- 能够实现跨平台操作，能够为不同操作系统上的软件集成技工数据传送服务


4.ActiveMQ使用

1. 下载ActiveMQ
去官方网站下载：[http://activemq.apache.org/](http://activemq.apache.org/)

2. 运行ActiveMQ
解压缩apache-activemq-5.5.1-bin.zip，然后双击apache-activemq-5.5.1\bin\activemq.bat运行ActiveMQ程序,启动ActiveMQ以后，登陆：http://localhost:8161/admin/,如下图所示：
![](/images/201605/4.png)

3. 创建	intellij IDEA 项目并运行
- 点对点模式
![](/images/201605/5.png)


![](/images/201605/6.png)


- 订阅发布模式
![](/images/201605/7.png)



5.本文测试源码：[https://github.com/wuzguo/jms.git](https://github.com/wuzguo/jms.git)


6.参考博文：
[1. 深入浅出JMS](http://blog.csdn.net/jiuqiyuliang/article/details/46701559)
[2.JMS](http://baike.baidu.com/link?url=niD5orGszw_hottRirC5_piUciGWelrxCNr1izaXMutlFcfNiVwHA0F7AHHjJ8j5r9KsNOG2HI2N_x0hRX6UM_PR8Tg67FuhUsmMsrmdKNK)
		