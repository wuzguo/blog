---
title: 手把手教你入门AIoT（七）
date: 2020-06-28 07:50:00
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


QoS0 和 QoS1 是相对简单的 QoS 等级，QoS2 不仅要确保 Receiver 能收到 Sender 发送的消息，还要保证消息不重复。它的重传和应答机制就要复杂一些，同时开销也是最大的。下面就让我们来看一下 QoS2 的机制。本节课核心内容：

- QOS2
- QoS 和会话（Session）
- 如何选择 QoS

### 7.1 QOS2

在 QoS2 下，一条消息的传递流程如下：
![img](/images/202006/7.png)

QoS 使用 2 套请求/应答流程（一个 4 段的握手）来确保 Receiver 收到来自 Sender 的消息，且不重复：

1. Sender 发送 QoS 为 2 的 PUBLISH 数据包，数据包 Packet Identifier 为 P，并在本地保存该 PUBLISH 包；
2. Receiver 收到 PUBLISH 数据包以后，在本地保存 PUBLISH 包的 Packet Identifier P，并回复 Sender 一个 PUBREC 数据包，PUBREC 数据包可变头中的 Packet Identifier 为 P，没有消息体（Payload）；
3. 当 Sender 收到 PUBREC，它就可以安全地丢弃掉初始的 Packet Identifier 为 P 的 PUBLISH 数据包，同时保存该 PUBREC 数据包，同时回复 Receiver 一个 PUBREL 数据包，PUBREL 数据包可变头中的 Packet Identifier 为 P，没有消息体；如果 Sender 在一定时间内没有收到 PUBREC，它会把 PUBLISH 包的 DUP 标识设为 1，重新发送该 PUBLISH 数据包（Payload）；
4. 当 Receiver 收到 PUBREL 数据包，它可以丢弃掉保存的 PUBLISH 包的 Packet Identifier P，并回复 Sender 一个 PUBCOMP 数据包，PUBCOMP 数据包可变头中的 Packet Identifier 为 P，没有消息体（Payload）；
5. 当 Sender 收到 PUBCOMP 包，那么它认为数据包传输已完成，它会丢弃掉对应的 PUBREC 包。如果 Sender 在一定时间内没有收到 PUBCOMP 包，它会重新发送 PUBREL 数据包。

我们可以看到在 QoS2 中，完成一次消息的传递，Sender 和 Reciever 之间至少要发送四个数据包，QoS2 是最安全也是最慢的一种 QoS 等级了。

我们可以运行上一课中的 publish_with_qos.js 和 subscribe_with_qos.js 来验证一下：

`node publish_with_qos.js --qos=2` 输出为：

```
send: publish
receive: pubrec

send: pubrel
receive: pubcomp
```

`node subscribe_with_qos.js --qos=2` 输出为：

```
receive: publish
send: pubrec

receive: pubrel
send: pubcomp
```

当然，和上一课讲的一样，如果 Publish QoS 为 1，Subscribe QoS 为 2，或者 Publish QoS 为 2，Subscribe QoS 为 1，那么实际 Subscribe 接收消息的 QoS 仍然为 1。

### 7.2 QoS 和会话（Session）

如果 Client 想接收离线消息，必须使用持久化的会话（Clean Session = 0）连接到 Broker，这样 Broker 才会存储 Client 在离线期间没有确认接收的 QoS 大于 1 的消息。

### 7.3 如何选择 QoS

**在以下情况下你可以选择 QoS0**：

- Client 和 Broker 之间的网络连接非常稳定，例如一个通过有线网络连接到 Broker 的测试用 Client；
- 可以接受丢失部分消息，比如你有一个传感器以非常短的间隔发布状态数据，所以丢一些也可以接受；
- 你不需要离线消息。

**在以下情况下你应该选择 QoS1：**

- 你需要接收所有的消息，而且你的应用可以接受并处理重复的消息；
- 你无法接受 QoS2 带来的额外开销，QoS1 发送消息的速度比 QoS2 快很多。

**在以下情况下你应该选择 QoS2：**

- 你的应用必须接收到所有的消息，而且你的应用在重复的消息下无法正常工作，同时你也能接受 QoS2 带来的额外开销。

实际上，QoS1 是应用最广泛的 QoS 等级，QoS1 发送消息的速度很快，而且能够保证消息的可靠性。虽然使用 QoS1 可能会收到重复的消息，但是在应用程序里面处理重复消息，通常并不是件难事。在实战的课程里面我们会看到如何在应用里对消息去重。

### 7.4 小结

在本节课里面我们学习了 QoS2 消息传递的流程，以及选择 QoS 的最佳实践，下一课我们将学习 Retained Message 和 LWT（Last Will and Testament）。