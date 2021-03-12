---
title: 手把手教你入门AIoT（1）
date: 2020-06-28 01:50:00
author: Zak
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wuzguo
authorDesc: 一个追求进步的「十八线码农」
categories: 物联网
tags: 
	- ioT
	- MQTT
	- QoS
	- Broker
	- 发布订阅
	- 协议
keywords: ioT，MQTT，Broker，MQTT，发布订阅，协议
photos:
	- /blog/images/202006/0.png
description: 一套很良心的AIoT入门教程，讲解详细，附带源码演示
---


​物联网被认为是继计算机、互联网之后，信息技术行业的第三次浪潮。随着基础通讯设施的不断完善，尤其是 5G 的出现，进一步降低了万物互联的门槛和成本。说到物联网不得不讲下物联网通讯。
​物联网通讯是物联网的一个核心内容，目前物联网的通讯协议并没有一个统一的标准，比较常见的有 MQTT、CoAP、DDS、XMPP 等，在这其中，MQTT（消息队列遥测传输协议）应该是应用最广泛的标准之一。
​所以入门物联网，掌握 MQTT 是一个非常必要的步骤。


### 1.1 MQTT 是什么

MQTT 协议是什么？简单地来说 MQTT 协议有以下特性：

- 基于 TCP 协议的应用层协议；
- 采用 C/S 架构；
- 使用订阅/发布模式，将消息的发送方和接受方解耦；
- 提供 3 种消息的 QoS（Quality of Service）: 至多一次，最少一次，只有一次；
- 收发消息都是异步的，发送方不需要等待接收方应答。

虽然 MQTT 协议名称有 Message Queue 两个词，但是它并不是一个像 RabbitMQ 那样的一个消息队列，这是初学者最容易搞混的一个问题。MQTT 跟传统的消息队列相比，有以下一些区别：

1. 在传统消息队列中，在发送消息之前，必须先创建相应的队列；在 MQTT 中，不需要预先创建要发布的主题（可订阅的 Topic）；
2. 在传统消息队列中，未被消费的消息总是会被保存在某个队列中，直到有一个消费者将其消费；在 MQTT 中，如果发布一个没有被任何客户端订阅的消息，这个消息将被直接扔掉；
3. 在传统消息队列中，一个消息只能被一个客户端获取，在 MQTT 中，一个消息可以被多个订阅者获取，MQTT 协议也不支持指定消息被单一的客户端获取。

MQTT 协议可以为大量的低功率、工作网络环境不可靠的物联网设备提供通讯保障。而它的应用范围也不仅如此，在移动互联网领域也大有作为：很多 Android App 的推送功能，都是基于 MQTT 实现的，也有一些 IM 的实现，是基于 MQTT 的。



### 1.2 MQTT 协议的通信模型

就像我们在之前提到的，MQTT 的通信是通过发布/订阅的方式来实现的，消息的发布方和订阅方通过这种方式来进行解耦，它们没有直接地连接，它们需要一个中间方。在 MQTT 里面我们称之为 Broker，用来进行消息的存储和转发。一次典型的 MQTT 消息通信流程如下所示：

![](/blog/images/202006/1.png)

1. 发布方将消息发送到 Broker；
2. Broker 接收到消息以后，检查下都有哪些订阅方订阅了此类消息，然后将消息发送到这些订阅方；
3. 订阅方从 Broker 获取该消息。

接下来的内容我们将发送方称为 Publisher，将订阅方称为 Subscriber。

### 1.3 MQTT Client

任何终端，嵌入式设备也好，服务器也好，只要运行了 MQTT 的库或者代码，我们都称为 MQTT 的 Client。Publisher 和 Subscriber 都属于 Client，Pushlisher 或者 Subscriber 只取决于该 Client 当前的状态——是在发布还是在订阅消息。当然，一个 Client 可以同时是 Publisher 和 Subscriber。



MQTT Client 库在很多语言中都有实现，包括 Android、Arduino、Ruby、C、C++、C#、Go、iOS、Java、JavaScript，以及 .NET 等。如果你要查看相应语言的库实现，可以在这里 (https://github.com/mqtt/mqtt.github.io/wiki/libraries) 找到。

本系列课程我们主要使用 Node.js 的 MQTT Client 库来进行演示，所以需要先安装 Node.js，然后安装 MQTT Client 的 Node.js 包：

```
npm install mqtt -g
```

### 1.4 MQTT Broker

如前面所讲的，Broker 负责接收 Publisher 的消息，并发送给相应的 Subscriber，它是整个 MQTT 订阅/发布的核心。在实际应用中，一个 MQTT Broker 还应该提供以下一些功能：

- 可以横向扩展，比如集群，来满足大量的 Client 接入；
- 可以扩展接入业务系统；
- 易于监控，满足高可用性。



本系列文章我们使用一个公共的 MQTT Broker —— iot.eclipse.org 做演示，同时也会学习如何搭建一个 MQTT Broker。



### 1.5 MQTT 协议数据包

MQTT 协议的数据包格式非常简单，一个 MQTT 协议数据包由下面三个部分组成：

- 固定头（Fixed header）：存在于所有的 MQTT 数据包中，用于表示数据包类型及对应标识，表明数据包大小；
- 可变头（Variable header）：存在于部分类型的 MQTT 数据包中，具体内容由相应类型的数据包决定；
- 消息体（Payload）：存在于部分 MQTT 数据包中，存储消息的具体数据。



接下来看一下固定头的格式，可变头和消息体我们将在讲解各种具体类型的 MQTT 协议数据包的时候 case by case 地讨论。

固定头格式：

| Bit      | 7               | 6                                       | 5    | 4    | 3    | 2    | 1    | 0    |
| -------- | --------------- | --------------------------------------- | ---- | ---- | ---- | ---- | ---- | ---- |
| 字节 1   | MQTT 数据包类型 | MQTT 数据包 Flag， 内容由数据包类型指定 |      |      |      |      |      |      |
| 字节 2…… | 数据包剩余长度  |                                         |      |      |      |      |      |      |

固定头的第一个字节的高 4 位 bit 用于指定该数据包的类型，MQTT 的数据包有以下一些类型：

| 名称        | 值   | 方向             | 描述                     |
| ----------- | ---- | ---------------- | ------------------------ |
| Reserved    | 0    | 不可用           | 保留位                   |
| CONNECT     | 1    | Client 到 Broker | Client 请求连接到 Broker |
| CONNACK     | 2    | Broker 到 Client | 连接确认                 |
| PUBLISH     | 3    | 双向             | 发布消息                 |
| PUBACK      | 4    | 双向             | 发布确认                 |
| PUBREC      | 5    | 双向             | 发布收到                 |
| PUBREL      | 6    | 双向             | 发布释放                 |
| PUBCOMP     | 7    | 双向             | 发布完成                 |
| SUBSCRIBE   | 8    | Client 到 Broker | Client 请求订阅          |
| SUBACK      | 9    | Broker 到 Client | 订阅确认                 |
| UNSUBSCRIBE | 10   | Client 到 Broker | Client 请求取消订阅      |
| UNSUBACK    | 11   | Broker 到 Client | 取消订阅确认             |
| PINGREQ     | 12   | Client 到 Broker | PING 请求                |
| PINGRESP    | 13   | Broker 到 Client | PING 应答                |
| DISCONNECT  | 14   | Client 到 Broker | Client 主动中断连接      |
| Reserved    | 15   | 不可用           | 保留位                   |

固定头的低 4 位 bit 用于指定数据包的 Flag，不同的数据包类型，其 Flag 的定义是不一样的，每种数据包对应的 Flag 如下：

| 数据包      | 标识位          | Bit 3 | Bit 2 | Bit 1 | Bit 0  |
| ----------- | --------------- | ----- | ----- | ----- | ------ |
| CONNECT     | 保留位          | 0     | 0     | 0     | 0      |
| CONNACK     | 保留位          | 0     | 0     | 0     | 0      |
| PUBLISH     | MQTT 3.1.1 使用 | DUP   | QoS   | QoS   | RETAIN |
| PUBACK      | 保留位          | 0     | 0     | 0     | 0      |
| PUBREC      | 保留位          | 0     | 0     | 0     | 0      |
| PUBREL      | 保留位          | 0     | 0     | 0     | 0      |
| PUBCOMP     | 保留位          | 0     | 0     | 0     | 0      |
| SUBSCRIBE   | 保留位          | 0     | 0     | 0     | 0      |
| SUBACK      | 保留位          | 0     | 0     | 0     | 0      |
| UNSUBSCRIBE | 保留位          | 0     | 0     | 0     | 0      |
| UNSUBACK    | 保留位          | 0     | 0     | 0     | 0      |
| PINGREQ     | 保留位          | 0     | 0     | 0     | 0      |
| PINGRESP    | 保留位          | 0     | 0     | 0     | 0      |
| DISCONNECT  | 保留位          | 0     | 0     | 0     | 0      |

从固定头的第 2 字节开始是用于标识 MQTT 数据包长度的字段，最少一个字节，最大四个字节，每一个字节的低 7 位用于标识值，范围为 0~127。最高位的 1 位是标识位，用来说明是否有后续字节来标识长度。例如：标识为 0，代表为没有后续字节；标识为 1，代表后续还有一个字节用于标识包长度。MQTT 协议规定最多可以用四个字节来标识包长度。

所以这四个字节最多可以标识的包长度为：(0xFF, 0xFF, 0xFF, 0x7F) = 268435455 字节，约 256M，这个是 MQTT 协议中数据包的最大长度。

### 1.7 总结

我们在这一课中学习了 MQTT 的通信模型，以及 Client 和 Broker 的概念，同时也学习了 MQTT 数据包的格式。接下来我们开始收发数据的第一步：从 Client 连接到 Broker。