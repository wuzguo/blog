---
title: 手把手教你入门AIoT（5）
date: 2020-06-28 05:50:00
author: Zak
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wzguo
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



接下来我们来学习如何订阅一个主题，并接收消息。本节课核心内容：

- 订阅
- 取消订阅
- 代码实践

### 5.1 订阅

订阅主题的流程如下：
![](/blog/images/202006/3.png)

1. Client 向 Broker 发送一个 SUBSCRIBE 数据包，其中包含了 Client 想要订阅的主题以及其他一些参数；
2. Broker 收到 SUBSCRIBE 数据包后，向 Client 发送一个 SUBACK 数据包作为应答。

接下来我们看数据包的具体内容。

#### 5.1.1 SUBSCRIBE

##### 5.1.1.1 **可变头（Variable header）**

**数据包标识（Packet Identifier）**：两个字节，用来唯一标识一个数据包，数据包标识只需要保证在从 Sender 到 Receiver 的一次消息交互中保持唯一。

##### 5.1.1.2 **消息体（Payload）**

**订阅列表（List of Subscriptions）**：SUBSCRIBE 的消息体中包含 Client 想要订阅的主题列表，列表中的每一项由订阅主题名和对应的 QoS 组成。主题名中可以包含通配符，单层通配符“+”和多层通配符“#”。使用包含通配符的主题名可以订阅满足匹配条件的所有主题。为了和 PUBLISH 中的主题区分，我们叫 SUBSCRIBE 中的主题名为主题过滤器（Topic Filter）。

单层通配符“+”：就如之前我们讲的，MQTT 的主题是具有层级概念的，不同的层级之间用“/”分割，“+”可以用来指代任意一个层级。

例如“home/2ndfloor/+/temperature”，可匹配：

- home/2ndfloor/201/temperature
- home/2ndfloor/202/temperature

不可匹配：

- home/2ndfloor/201/livingroom/temperature
- home/3ndfloor/301/temperature

多层通配符“#”：“#”和“+”的区别在于，“#”可以用来指定任意多个层级，但是“#”必须是 Topic Filter 的最后一个字符，同时它必须跟在“/”后面，除非 Topic Filter 只包含“#”这一个字符。

例如“home/2ndfloor/#”，可匹配：

- home/2ndfloor
- home/2ndfloor/201
- home/2ndfloor/201/temperature
- home/2ndfloor/202/temperature
- home/2ndfloor/201/livingroom/temperature

不可匹配：

- home/3ndfloor/301/temperature

> 注意：“#”是一个合法的 Topic Filter，代表所有的主题;而“home#”不是一个合法的 Topic Filter，因为“#”号需要跟在“/”后面。

SUBSCRIBE 数据包中 QoS 代表针对某一个或者一组主题，Client 希望 Broker 在发送来自这些主题的消息给它时，消息使用的 QoS 级别，我们在《第06课：QoS0 和 QoS1》里面再详细讨论。

#### 5.1.2 SUBACK

为了确认每一次的订阅，Broker 收到 SUBSCRIBE 之后会回复一个 SUBACK 数据包作为应答。

##### 5.1.2.1 **可变头（Variable header）**

**数据包标识（Packet Identifier）**：两个字节，用来唯一标识一个数据包，数据包标识只需要保证在从 Sender 到 Receiver 的一次消息交互中保持唯一。

##### 5.1.2.2 **消息体（Payload）**

**返回码（return codes）**：SUBBACK 数据包包含了一组返回码，返回码的数量和顺序和 SUBSCRIBE 数据包的订阅列表对应，用于标识订阅类别中的每一个订阅项的订阅结果。

| 返回码 | 含义                         |
| :----- | :--------------------------- |
| 0      | 订阅成功， 最大可用 QoS 为 0 |
| 1      | 订阅成功，最大可用 QoS 为 1  |
| 2      | 订阅成功， 最大可用 QoS 为 2 |
| 128    | 订阅失败                     |

返回码 0~2 代表订阅成功，同时 Broker 授予 Subscriber 不同的 QoS 等级，这个等级可能会和 Subscriber 在 SUBSCRIBE 数据包中要求的不一样。

返回码 128 代表订阅失败，比如 Client 没有权限订阅某个主题，或者要求订阅的主题格式不正确等。

#### 5.1.3 代码实践：订阅一个主题

接下来我们来写订阅并处理消息的代码，我们订阅在上一课中的 publisher.js 中的主题，并通过捕获 “message” 事件获取接收的消息并打印出来。

通常我们在建立和 Broker 的连接之后就可以开始订阅了，但是这里有一个小小的优化，如果你建立的是持久会话的连接，那么有可能 Broker 已经保存你在之前的连接时订阅的主题，你就没有必要再发起 SUBSCRIBE 请求了，这个小优化在网络带宽或者设备处理能力较差的情况尤为重要。

完整的代码 subscriber.js 如下：

```
var mqtt = require('mqtt')

var client = mqtt.connect('mqtt://mqtt.eclipse.org:1883', {
    clientId: "mqtt_sample_subscriber_id_1",
    clean: false
})

client.on('connect', function (connack) {
    if(connack.returnCode == 0) {
        if (connack.sessionPresent == false) {
            console.log("subscribing")
            client.subscribe("home/2ndfloor/201/temperature", {
                qos: 1
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


client.on("message", function (_, message, _) {
    var jsonPayload = JSON.parse(message.toString())
    console.log(`current temperature is ${jsonPayload.current}`)
})
```

在终端上运行 `node subscriber.js` 我们会得到以下输出：

```
subscribing

subscribe succeeded with home/2ndfloor/201/temperature, qos: 1
```

第一次运行这个代码的时候，Broker 上面没有保存这个 Client 的会话，所以需要进行订阅，现在 `CTRL+C` 终止这段代码的运行，然后重新运行，因为 Broker 上面已经保存了这个 Client 的会话，所以就不需要再订阅了，你就不会看到订阅相关的输出了。

在上一课中，我们运行过 publisher.js，向“home/2ndfloor/201/temperature”这个主题发布过一个消息，但是这发生在 subscriber.js 订阅该主题之前，所以现在 Subscriber 不会收到任何消息，我们需要再运行一次 publish.js，然后在运行 subscriber.js 的终端上会输出：

```
current temperature is 25
```

好了，我们终于通过 MQTT 协议完成了一次点到点的消息传递，同时我们也验证了，建立持久性会话连接之后，Broker 会保存 Client 的订阅信息。

### 5.2 取消订阅

Subcriber 也可以取消对某些主题的订阅，取消订阅的流程如下：
![](/blog/images/202006/4.png)

1. Client 向 Broker 发送一个 UNSUBSCRIBE 数据包，其中包含了 Client 想要取消订阅的主题；
2. Broker 收到 UNSUBSCRIBE 数据包后，向 Client 发送一个 UNSUBACK 数据包作为应答。

接下来我们看数据包的具体内容。

#### 5.2.1 UNSUBSCRIBE

##### 5.2.1.1 **可变头（Variable header）**

**数据包标识（Packet Identifier）**：两个字节，用来唯一标识一个数据包，数据包标识只需要保证在从 Sender 到 Receiver 的一次消息交互中保持唯一。

##### 5.2.1.2 **消息体（Payload）**

**主题列表（List of Topics）**：UNSUBSCRIBE 的消息体中包含 Client 想要取消订阅的主题过滤器列表，这些主题过滤器和 SUBSCRIBE 数据包中一样，可以包含通配符。UNSUBSCRIBE 消息体里面不再包含主题过滤器对应的 QoS 了。

#### 5.2.2 UNSUBACK

Broker 收到 UNSUBSCRIBE 之后会回复一个 UNSUBACK 数据包作为应答：

##### 5.2.2.1 **可变头（Variable header）**

**数据包标识（Packet Identifier）**：两个字节，用来唯一标识一个数据包，数据包标识只需要保证在从 Sender 到 Receiver 的一次消息交互中保持唯一。

##### 5.2.2.2 **消息体（Payload）**

UNSUBACK 数据包没有消息体。

### 5.3 代码实践：取消订阅

我们要完成的代码很简单，在建立连接之后取消对之前订阅的主题。

完整的代码 unsubscribe.js 如下：

```
var mqtt = require('mqtt')

var client = mqtt.connect('mqtt://mqtt.eclipse.org:1883', {
    clientId: "mqtt_sample_subscriber_id_1",
    clean: false
})

client.on('connect', function (connack) {
    if (connack.returnCode == 0) {
        console.log("unsubscribing")
        client.unsubscribe("home/2ndfloor/201/temperature", function (err) {
            if (err != undefined) {
                console.log("unsubscribe failed")
            } else {
                console.log("unsubscribe succeeded")
            }

            client.end()
        })
    } else {
        console.log(`Connection failed: ${connack.returnCode}`)
    }
})
```

在终端上运行 `node unsubscribe.js`，会得到以下输出：

```
unsubscribing

unsubscribe succeeded
```

在这里取消了对“home/2ndfloor/201/temperature”的订阅，所以再运行 subscriber.js 和 publisher.js，再运行 subscribe.js 的终端不会再有消息的打印信息了。如何要使 subscriber.js 重新订阅这个主题，读者可以动下脑筋然后自己动手实现一下。

### 5.4 小结

我们终于完成了发布订阅的学习，并第一次实现了消息的点到点传输， 接下来我们开始学习 MQTT 中的一个非常重要的特性：三种 QoS 等级。