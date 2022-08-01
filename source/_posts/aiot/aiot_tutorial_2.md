---
title: 手把手教你入门AIoT（二）
date: 2020-06-28 02:50:00
author: Zak
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wuzguo
authorDesc: 一个追求进步的「十八线码农」
categories: IoT
tags: 
	- IoT
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


Client在可以发布和订阅消息之前，必须先连接到Broker，下面我们来看一下Client连接到Broker的流程。本节课核心内容：

- Client 连接到 Broker 的流程
- CONNECT
- CONNACK

### 2.1 Client 连接到 Broker 的流程

Client建立到Broker的连接流程如下图所示：
![](/images/202006/2.png)

### 2.2 CONNECT

​		连接的建立由 Client 端发起，Client 端首先向 Broker 发送一个 CONNECT 数据包，CONNECT 数据包包含以下内容（这里我们略过 Fixedheader）。

#### 2.2.1 可变头（Variable header）

在 CONNECT 数据包可变头中，含有以下信息。

**「协议名称」**（ProtocolName）：值固定为字符 “MQTT”。

**「协议版本」**（ProtocolLevel）：对 MQTT 3.1.1 来说，值为 4。

**「用户名标识」**（UserNameFlag）：消息体中是否有用户名字段，1 bit，0 或者 1。

**「密码标识」**（PasswordFlag）：消息体中是否有密码字段，1 bit，0 或者 1。

**「遗愿消息 Retain 标识」**（Will Retain）：标识遗愿消息是否是Retain消息，1bit，0或者1。

**「遗愿消息 QOS 标识」**（Will QOS）：标识遗愿消息的 QOS，2bit，0、1或者2。

**「遗愿标识」**（Will Flag）：标识是否使用遗愿消息，1 bit，0 或者 1。

**「会话清除标识」**（CleanSession）：标识Client是否建立一个持久化的会话，1 bit，0 或者 1，当 CleanSession 的标识设为0时，代表 Client 希望建立一个持久会的连接，Broker 将存储该 Client 订阅的主题和未接受的消息，否则 Broker 不会存储这些数据，同时在建立连接时清除这个 Client 之前存在的持久化会话所保存的数据。

**「连接保活」**（KeepAlive）:设置一个单位为秒的时间间隔，Client 和 Broker 之间在这个时间间隔之内需要至少有一次消息交互，否则 Client 和 Broker 会认为它们之间的连接已经断开。

#### 2.2.2 消息体（Payload）
CONNECT 数据包的消息体中包含以下数据。

**「客户端标识符」**（Client Identifier）：Client Identifier 是用来标识 Client 身份的字段，在 MQTT 3.1.1 的版本中，这个字段的长度是1到23个字节，而且只能包含数字和26个字母（包括大小写），Broker 通过这个字段来区分不同的 Client。所以在连接的时候，Client 应该保证它的 Identifier 是唯一的，通常我们可以使用比如 UUID，唯一的设备硬件标识，或者 Android 设备的 DEVICE_ID 等作为 Client Identifier 的取值来源。

MQTT 协议中要求 Client 连接时必须带上 Client Identifier，但是也允许 Broker 在实现时 Client Identifier 为空，这时 Broker 会为 Client 分配一个内部唯一的Identifier。如果你需要使用持久化会话，那就必须自己为Client设定一个唯一的Identifier。

**「用户名」**（Username）：如果可变头中的用户名标识设为1，那么消息体中将包含用户名字段，Broker 可以使用用户名和密码来对接入的Client进行验证，只允许已授权的Client接入。注意不同的Client需要使用不同的 ClientIdentifier，但它们可以使用同样的用户名和密码进行连接。

**「密码」**（Password）：如果可变头中的密码标识设为1，那么消息体中将包含密码字段。

**「遗愿主题」**（Will Topic）：如果可变头中的遗愿标识设为1，那么消息体中将包含遗愿主题，当 Client 非正常地中断连接的时候，Broker 将向指定的遗愿主题中发布遗愿消息。

**「遗愿消息」**（Will Message）：如果可变头中的遗愿标识设为1，那么消息体中将包含遗愿消息，当Client非正常地中断连接的时候，Broker 将向指定的遗愿主题中发布由该字段指定的内容。我们会在后续的课程里面详细讨论。

### 2.3 CONNACK

当 Broker 收到 Client 的 CONNECT 数据包之后，将检查并校验 CONNECT 数据包的内容，之后回复 Client 一个 CONNACK 数据包。CONNACK 数据包包含以下内容（这里我们略过 Fixedheader）。

#### 2.3.1 可变头（Variable header）
CONNACK 数据包的可变头中，含有以下信息。

**「会话存在标识」**（Session Present Flag）：用于标识在 Broker上，是否已存在该Client（用 Client Identifier 区分）的持久性会话，1 bit，0 或者 1。

当 Client 在连接时设置 Clean Session = 1，则 CONNACK 中的 Session Presen tFlag 始终为 0；

当 Client 在连接时设置 Clean Session = 0，那么就有两种情况——如果 Broker 上面保存了这个 Client 之前留下的持久性会话，那么 CONNACK 中的 Session Present Flag 值为 1；

如果 Broker 没有保存该 Client 的任何会话数据，那么 CONNACK 中的 Session Present Flag 值为 0。Session Present Flag 这个特性是在 MQTT 3.1.1 版本中新加入的，之前的版本中并没有这个标识。

**「连接返回码」**（Connect Return code）：用于标识 Client 是 Broker 的连接是否建立成功，连接返回码有以下一些值：

| Return Code | 连接状态                             |
| ----------- | ------------------------------------ |
| 0           | 连接已建立                           |
| 1           | 连接被拒绝，不允许的协议版本         |
| 2           | 连接被拒绝，Client Identifier 被拒绝 |
| 3           | 连接被拒绝，服务器不可用             |
| 4           | 连接被拒绝，错误的用户名或密码       |
| 5           | 连接被拒绝，未授权                   |

这里重点讲一下 Return Code 4 和 5 。Return Code 4 在 MQTT 协议中的含义是Username 和 Password 的格式不正确， 但是在大部分的 Broker 实现中， 在使用错误的用户名密码时， 得到的返回码也是 4 。所以这里我们认为 4 就是代表错误的用户名或密码。Return Code 5 一般在 Broker 不使用用户名和密码而使用 IP 地址或者 Client Identifier 进行验证的时候使用， 来标识 Client 没有通过验证。

#### 2.3.2 消息体（Payload）
CONNACK 没有消息体。

当 Client 向 Broker 发送 CONNECT 数据包并获得 Return Code 为 0 的CONNACK 包后， 就代表连接建立成功， 可以发布和接受消息了。

### 2.4 小结

本节课我们了解了 Client 连接到 Broker 的流程，接下来我们学习连接的关闭，以及 MQTT 连接建立与关闭的实例代码。