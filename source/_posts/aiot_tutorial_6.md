---
title: 手把手教你入门AIoT（6）
date: 2020-06-28 06:50:00
author: Zak
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wuzguo
authorDesc: 一个自强不息，艰苦奋斗的「十八线码农」
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


在前面的课程中我们多次提到了 QoS（Quality of Service）的概念，CONNECT、PUBLISH、SUBSCRIBE 中都有 QoS 的标识，那么 MQTT 提供的 QoS 是什么呢？本节课核心内容：

- MQTT 中的 QoS 等级
- QoS0
- QoS1
- 代码实践

### 6.1 MQTT 中的 QoS 等级

作为最初用来在网络带宽窄、信号不稳定的环境下传输数据的协议，MQTT 设计了一套保证消息稳定传输的机制，包括消息应答、存储和重传。在这套机制下，提供了三种不同层次 QoS：

- QoS0，At most once，至多一次；
- QoS1，At least once，至少一次；
- QoS2，Exactly once，确保只有一次。

什么意思呢，QoS 是消息的发送方（Sender）和接受方（Receiver）之间达成的一个协议：

- QoS0 代表，Sender 发送的一条消息，Receiver 最多能收到一次，也就是说 Sender 尽力向 Receiver 发送消息，如果发送失败，也就算了；
- QoS1 代表，Sender 发送的一条消息，Receiver 至少能收到一次，也就是说 Sender 向 Receiver 发送消息，如果发送失败，会继续重试，直到 Receiver 收到消息为止，但是因为重传的原因，Receiver 有可能会收到重复的消息；
- QoS2 代表，Sender 发送的一条消息，Receiver 确保能收到而且只收到一次，也就是说 Sender 尽力向 Receiver 发送消息，如果发送失败，会继续重试，直到 Receiver 收到消息为止，同时保证 Receiver 不会因为消息重传而收到重复的消息。

要注意的是，QoS 是 Sender 和 Receiver 之间达成的协议，不是 Publisher 和 Subscriber 之间达成的协议。也就是说 Publisher 发布一条 QoS1 的消息，只能保证 Broker 能至少收到一次这个消息；至于对应的 Subscriber 能否至少收到一次这个消息，还要取决于 Subscriber 在 Subscribe 的时候和 Broker 协商的 QoS 等级。

接下来我们来看一下 QoS0 和 QoS1 的机制，并讨论一下什么是 QoS 降级。

### 6.2 QoS0

QoS0 是最简单的一个 QoS 等级了，在这个 QoS 等级下，Sender 和 Receiver 之间一次消息的传递流程如下：
![img](/blog/images/202006/5.png)

Sender 向 Receiver 发送一个包含消息数据的 PUBLISH 包，然后不管结果如何，丢弃掉已发送的 PUBLISH 包，一条消息的发送完成。

### 6.3 QoS1

QoS 要保证消息至少到达 Sender 一次，所以有一个应答的机制，在 Qos1 等级下的 Sender 和 Receiver 的一次消息的传递流程如下。
![img](/blog/images/202006/6.png)

1. Sender 向 Receiver 发送一个带有消息数据的 PUBLISH 包， 并在本地保存这个 PUBLISH 包。
2. Receiver 收到 PUBLISH 包以后，向 Sender 发送一个 PUBACK 数据包，PUBACK 数据包没有消息体（Payload），在可变头中（Variable header）中有一个包标识（Packet Identifier），和它收到的 PUBLISH 包中的 Packet Identifier 一致。
3. Sender 收到 PUBACK 之后，根据 PUBACK 包中的 Packet Identifier 找到本地保存的 PUBLISH 包，然后丢弃掉，一次消息的发送完成。
4. 如果 Sender 在一段时间内没有收到 PUBLISH 包对应的 PUBACK，它将该 PUBLISH 包的 DUP 标识设为 1（代表是重新发送的 PUBLISH 包），然后重新发送该 PUBLISH 包。重复这个流程，直到收到 PUBACK，然后执行第 3 步。

### 6.4 代码实践

这里我们实现一个发布端和一个订阅端，可以通过命令行参数来指定发布和订阅的 QoS，同时，通过捕获“packetsend”和“packetreceive”事件，将发送和接受到的 MQTT 数据包的类型打印出来。

完整的代码 publish_with_qos.js 如下：

```
var args = require('yargs').argv;
var mqtt = require('mqtt')

var client = mqtt.connect('mqtt://mqtt.eclipse.org:1883', {
    clientId: "mqtt_sample_publisher_2",
    clean: false
})

client.on('connect', function (connack) {
    if (connack.returnCode == 0) {
        client.on('packetsend', function (packet) {
            console.log(`send: ${packet.cmd}`)
        })

        client.on('packetreceive', function (packet) {
            console.log(`receive: ${packet.cmd}`)
        })

        client.publish("home/sample_topic", JSON.stringify({data: 'test'}), {qos: args.qos})
    } else {
        console.log(`Connection failed: ${connack.returnCode}`)
    }
})
```

完整的代码 subscribe_with_qos.js 如下：

```
var args = require('yargs').argv;

var mqtt = require('mqtt')
var client = mqtt.connect('mqtt://mqtt.eclipse.org:1883', {
    clientId: "mqtt_sample_subscriber_id_2",
    clean: false
})

client.on('connect', function (connack) {
    if (connack.returnCode == 0) {
        client.subscribe("home/sample_topic", {qos: args.qos}, function () {
            client.on('packetsend', function (packet) {
                console.log(`send: ${packet.cmd}`)
            })

            client.on('packetreceive', function (packet) {
                console.log(`receive: ${packet.cmd}`)
            })
        })
    } else {
        console.log(`Connection failed: ${connack.returnCode}`)
    }
})
```



在 subscribe_with_qos.js 中， Client 每次连接到 Broker 之后都会按照参数指定的 QoS 重新订阅主题，订阅成功以后才开始捕获接收和发送的数据包，所以 Client 在连接之后，重新订阅之前收到的离线消息不会被打印出来。

我们可以通过 node publish_with_qos.js --qos=xxx 和 node subscribe_with_qos.js --qos=xxx 来运行这两个 JS 程序。

接下来我们用 4 种参数组合来运行这两 JS 程序，看看输出分别是什么。

注意：需要先运行 subscribe_with_qos.js 再运行 publish_with_qos.js，确保接收到消息可以打印出来。

#### 6.4.1 发布使用 QoS0，订阅使用 QoS0

node publish_with_qos.js --qos=0 输出为：

```
send: publish
```

node subscribe_with_qos.js --qos=0 输出为：

```
receive: publish
```

结果显而易见，Publisher 到 Broker，Broker 到 Subscriber 都是用的 QoS0。

#### 6.4.2 发布使用 QoS1，订阅使用 QoS1

node publish_with_qos.js --qos=1 输出为：

```
send: publish
receive: puback
```

node subscribe_with_qos.js --qos=1 输出为：

```
receive: publish
send: puback
```

同样地，结果显而易见，Publisher 到 Broker，Broker 到 Subscriber 都是用的 QoS1。

#### 6.4.3 发布使用 QoS0，订阅使用 QoS1

node publish_with_qos.js --qos=0 输出为：

```
send: publish
```

node subscribe_with_qos.js --qos=1 输出为：

```
receive: publish
```

这里就有点奇怪了， 很明显 Broker 到 Subscriber 这段使用的是 QoS0，和 Subscriber 订阅时指定的 QoS 不一样。原因我们后面来讲。

6.4.4 发布使用 QoS1，订阅使用 QoS0

node publish_with_qos.js --qos=1 输出为：

```
send: publish
receive: puback
```

node subscribe_with_qos.js --qos=0 输出为：

```
receive: publish
```

和设定的一样， Publisher 到 Broker 使用 QoS1，Broker 到 Subscriber 使用的 QoS0。Publisher 使用 QoS1 发布消息，但是消息到 Subscriber 却是 QoS0。也就是说有可能无法收到消息，这种现象叫做 QoS 的降级（QoS Degrade）。

这里有一个很重要的计算方法，**在 MQTT 协议中，从 Broker 到 Subscriber 这段消息传递的实际 QoS 等于：Publisher 发布消息时指定的 QoS 等级和 Subscriber 在订阅时与 Broker 协商的 QoS 等级，这两个 QoS 等级中的最小那一个。**

***\*Actual Subscribe QoS = MIN(Publish QoS, Subscribe QoS)\****

这也就解释了“publish qos=0, subscribe qos=1”的情况下 Subscriber 的实际 QoS 为 0，以及“publish qos=1, subscribe qos=0”时出现 QoS 降级的原因。

理解了实际 Subscriber QoS 的计算方法，你才能很好地设计你系统里面 Publisher 和 Subscriber 使用的 QoS。例如，如果你希望 Subscriber 至少收到一次 Publisher 的消息，那么你要确保 Publisher 和 Subscriber 都使用不小于 1 的 QoS 等级。

### 6.5 小结

在这一课里面，我们学习了相对比较简单的两种 QoS 等级，同时学习了实际 QoS 的计算方法，接下来我们学习相对复杂的 QoS2 以及 QoS 的最佳实践。