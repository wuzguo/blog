---
title: 手把手教你入门AIoT（八）
date: 2020-06-28 08:50:00
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


在这一课里面我们来学习一下 Retained 消息和 LWT。本节课核心内容：

- Retained 消息
- 代码实践：发布和接收 Retained 消息
- LWT
- 代码实践：监控 Client 连接状态

### 8.1 Retained 消息

让我们来看一下这个场景：

你有一个温度传感器，它每三个小时向一个 Topic 发布当前的温度。那么问题来了，有一个新的订阅者在它刚刚发布了当前温度之后订阅了这个主题，那么这个订阅端什么时候能才能收到温度消息？

对的，它必须等到三个小时以后，温度传感器再次发布消息的时候才能收到。在这之前，这个新的订阅者对传感器的温度数据一无所知。

怎么来解决这个问题呢？

这个时候就轮到 Retained 消息出场解决这个问题了。Retained 消息是指在 PUBLISH 数据包中 Retain 标识设为 1 的消息，Broker 收到这样的 PUBLISH 包以后，将保存这个消息，当有一个新的订阅者订阅相应主题的时候，Broker 会马上将这个消息发送给订阅者。

Retain 消息有以下一些特点：

- 一个 Topic 只能有 1 条 Retained 消息，发布新的 Retained 消息将覆盖老的 Retained 消息；
- 如果订阅者使用通配符订阅主题，它会收到所有匹配的主题上的 Retained 消息；
- 只有新的订阅者才会收到 Retained 消息，如果订阅者重复订阅一个主题，也会被当做新的订阅者，然后收到 Retained 消息；
- Retained 消息发送到订阅者时，消息的 Retain 标识仍然是 1，订阅者可以判断这个消息是否是 Retained 消息，以做相应的处理。

注意：Retained 消息和持久性会话没有任何关系，Retained 消息是 Broker 为每一个 Topic 单独存储的，而持久性会话是 Broker 为每一个 Client 单独存储的。

如果你想删除一个 Retained 消息也很简单，只要向这个主题发布一个 Payload 长度为 0 的 Retained 消息就可以了。

那么开头我们提到的那个场景的解决方案就很简单了，温度传感器每 3 个小时发布当前的温度的 Retained 消息，那么无论新的订阅者什么时候进行订阅，它都能收到温度传感器上一次发布的数据。

### 8.2 代码实践：发布和接收 Retained 消息

在这里我们编写一个发布 Retained 消息的发布者，和一个接受消息的订阅端，订阅端在接收消息的时候将消息的 Retain 标识和内容打印出来。

完整的代码 publish_retained.js 如下：

```
var mqtt = require('mqtt')

var client = mqtt.connect('mqtt://mqtt.eclipse.org:1883', {
    clientId: "mqtt_sample_publisher_1",
    clean: false
})

client.on('connect', function (connack) {
    if(connack.returnCode == 0){
        client.publish("home/2ndfloor/201/temperature", JSON.stringify({current: 25}), {qos: 0, retain: 1}, function (err) {

            if(err == undefined) {
                console.log("Publish finished")
                client.end()
            }else{
                console.log("Publish failed")
            }
        })

    }else{
        console.log(`Connection failed: ${connack.returnCode}`)
    }
})
```

完整的代码 subscribe_retained.js 如下：

```
var mqtt = require('mqtt')

var client = mqtt.connect('mqtt://mqtt.eclipse.org:1883', {
    clientId: "mqtt_sample_subscriber_id_chapter_8",
    clean: false
})

client.on('connect', function (connack) {
    if(connack.returnCode == 0) {
        if (connack.sessionPresent == false) {
            console.log("subscribing")
            client.subscribe("home/2ndfloor/201/temperature", {
                qos: 0
            }, function (err, granted) {
                if (err != undefined) {
                    console.log("subscribe failed")
                } else {
                    console.log(`subscribe succeeded with ${granted[0].topic}, qos: ${granted[0].qos}`)
                }
            })
        }
    }else {
        console.log(`Connection failed: ${connack.returnCode}`)
    }
})

client.on("message", function (_, message, packet) {
    var jsonPayload = JSON.parse(message.toString())
    console.log(`retained: ${packet.retain}, temperature: ${jsonPayload.current}`)
})
```

我们首先运行 node publish_retained.js，再运行 node subscribe_retained.js，会得到以下输出：

retained: true, temperature: 25

可见我们在 Publisher 发布之后再订阅主题也能收到 Retained 消息。

然后我们再运行一次 node publish_retained.js，在运行 subscribe_retained.js 的终端会有以下输出：

retained: false, temperature: 25

Broker 收到 Retained 消息以后，只是保存这个消息，然后按照正常的转发逻辑转发给订阅者，所以对订阅者来说，这个是一个普通的 MQTT 消息，所以 Retain 标识为 0。

然后 Ctrl+C 关闭掉 subscribe_retained.js，然后重新运行，终端不会有任何输出，可见 Retained 消息只对新订阅的订阅者有效。

### 8.3 LWT

LWT 全称为 Last Will and Testament，也就是我们在连接到 Broker 时提到的遗愿，包括遗愿主题、遗愿 QoS、遗愿消息等。

顾名思义，当 Broker 检测到 Client 非正常地断开连接的时候，就会向遗愿主题里面发布一条消息。遗愿相关的设置是在建立连接的时候，在 CONNECT 数据包里面指定的。

- Will Flag：是否使用 LWT
- Will Topic：遗愿主题名，不可使用通配符
- Will Qos：发布遗愿消息时使用的 QoS
- Will Retain：遗愿消息的 Retain 标识
- Will Message：遗愿消息内容

Broker 在以下情况下认为 Client 是非正常断开连接的：

1. Broker 检测到底层的 I/O 异常；
2. Client 未能在 Keep Alive 的间隔内和 Broker 之间有消息交互；
3. Client 在关闭底层 TCP 连接前没有发送 DISCONNECT 数据包；
4. Broker 因为协议错误关闭和 Client 的连接，比如 Client 发送了一个格式错误的 MQTT 数据包。

如果 Client 通过发布 DISCONNECT 数据包断开连接，这个属于正常断开连接，不会触发 LWT 的机制，同时，Broker 还会丢弃掉这个 Client 在连接时指定的 LWT 参数。

通常，如果我们关心一个设备，比如传感器的连接状态，可以使用 LWT。在接下来的代码实践里面，我们会使用 LWT 和 Retained 消息来实现对一个 Client 的连接状态监控。

### 8.4 代码实践：监控 Client 连接状态

实现 Client 连接状态监控的原理很简单：

1. Client 在连接的时候指定 Will Topic 为“client/status”，遗愿消息为“offline”，Will Retain=1；
2. Client 在连接成功以后向同一个主题“client/status”，发布一个内容为“online”的 Retained 消息。

那么订阅者在任何时候订阅“client/status”，都会获取 Client 当前的连接状态。

client.js 代码如下：

```
var mqtt = require('mqtt')

var client = mqtt.connect('mqtt://iot.eclipse.org', {
    clientId: "mqtt_sample_publisher_chapter_8",
    clean: false,
    will:{
        topic : 'client/status',
        qos: 1,
        retain: true,
        payload: JSON.stringify({status: 'offline'})
    }
})

client.on('connect', function (connack) {
    if(connack.returnCode == 0){
        client.publish("client/status", JSON.stringify({status: 'online'}), {qos: 1, retain: 1})
    }else{
        console.log(`Connection failed: ${connack.returnCode}`)
    }
})
```

monitor.js 代码如下：

```
var mqtt = require('mqtt')

var client = mqtt.connect('mqtt://iot.eclipse.org', {
    clientId: "mqtt_sample_subscriber_id_chapter_8_2",
    clean: false
})

client.on('connect', function () {
    client.subscribe("client/status", {qos: 1})
})

client.on("message", function (_, message) {
    var jsonPayload = JSON.parse(message.toString())
    console.log(`client is ${jsonPayload.status}`)
})
```

在 monitor.js 中，我们每次连接的时候都重新订阅 “client/status”，这样的话每次运行都能收到关于 Client 连接状态的 Retained 消息。

首先运行 node client.js，然后运行 node monitor.js，会得到以下输出：

```
client is online
```

在运行 client.js 的终端上，使用 Ctrl+C 终止 client.js，之后在运行 monitor.js 的终端上会得到以下输出：

```
client is offline
```

然后重新运行 node client.js，在运行 monitor.js 的终端上会得到以下输出：

```
client is offline
```

Ctrl+C 终止 monitor.js，然后重新运行 node monitor.js，会得到以下输出：

```
client is offline
```

这样我们就完美地监控了 Client 的连接状态。

### 8.5 小结

在这一课我们学习了 Retained 消息和 LWT，并利用这两个特性完成了对 Client 连接状态进行监控的功能，下一课我们将学习 Keep Alive 和在移动端的连接保活。