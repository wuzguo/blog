---
title: 手把手教你入门AIoT（3）
date: 2020-06-28 03:50:00
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


在上一课中我们详细地了解了从Client到Broker的连接建立，接下来看一下如何关闭连接。本节课核心内容：


### 3.1 Client 主动关闭连接
Client 主动关闭连接的流程非常简单，只需要向 Broker 发送一个 DISCONNECT 数据包就可以了。DISCONNECT数据包没有可变头（Variableheader）和消息体
（Payload）。在 Client 发送完 DISCONNECT 之后，就可以关闭底层的 TCP 连接了，不需要等待 Broker 的回复（Broker 也不会对 DISCONNECT 数据包回复）。
这里读者可能有一个疑问，为什么需要在关闭TCP连接之前，发送一个和 Broker 没有交互的 DISCONNECT 数据包，而不是直接关闭底层的 TCP 连接？
这里涉及到 MQTT 协议的一个特性，Broker 需要判断 Client 是否正常地断开连接。当 Broker 收到 Client 的 DISCONNECT 数据包的时候，它认为 Client 是正常地断开连接，那么它会丢弃当前连接指定的遗愿消息（Will Message）。如果B roker 检测到 Client 连接丢失，但又没有收到 DISCONNECT 消息包，它会认为 Client 是非正常断开连接，就会向在连接的时候指定的遗愿主题（Will Topic）发布遗愿消息（WillMessage）

### 3.2 Broker主动关闭连接
MQTT 协议规定 Broker 在没有收到 Client 的 DISCONNECT 数据包之前都应该保持和 Client 连接，只有 Broker 在 KeepAlive 的时间间隔里，没有收到 Client 的任何 MQTT 数据包的时候会主动关闭连接。一些 Broker 的实现在 MQTT 协议上做了一些拓展，支持 Client 的连接管理，可以主动地断开和某个 Client 的连接。
Broker 主动关闭连接之前不会向 Client 发送任何 MQTT 数据包，直接关闭底层的 TCP 连接就完事了。

### 3.3 代码实践
下面就到了大家最喜欢的代码环节了，这里我们将用代码来展示MQTT连接的建立，和断开各种情况下的示例。

在这里我们使用 Node.js 的 MQTT 库，请确保已安装 Node.js，并通过 npm install mqtt --save 安装了 MQTT 库。

这里我们使用一个公共的 Broker：mqtt.eclipse.org。

#### 3.3.1 建立持久会话的连接

首先引用MQTT库：

```
var mqtt = require ('mqtt')
```

然后建立连接：

```
var client = mqtt.connect( 'mqtt://mqtt.eclipse.org:1883', {
	clientId: "mqtt_sample_id_1",
	clean: false
})
```

这里我们通过 Client ID 选项指定 Client Identifier，并通过 Clean 选项设定 Clean Session 为 false，代表我们要建立一个持久化会话的连接。

接下来我们通过捕获 connect 事件将 CONNACK 包 Return Code 和 Session Present Flag 打印出来，然后断开连接：

```
client.on('connect', function ( connack ) {
	console.log (`returncode: ${connack.returnCode},
	sessionPresent:${connack.sessionPresent}`)
client.end()
```

完整的代码 persistent_connection.js 如下：

```
var mqtt = require('mqtt')
var client = mqtt.connect('mqtt://mqtt.eclipse.org:1883',{
	clientId:"mqtt_sample_id_1",
	clean:false
})

client.on('connect',function(connack){
	console.log(`returncode: ${connack.returnCode},
	sessionPresent: ${connack.sessionPresent}`)
	client.end()
})
```

我们在终端上运行：

```
node persistent_connection.js
```

会得到以下输出：

```
returncode: 0, sessionPresent: false
```

连接成功，因为是 “mqtt_sample_id_1” 的 Client 第一次建立连接，所以 SessionPresent 为 false。

再次运行 node persistent_connection.js，输出就会变成：

```
returncode: 0, sessionPresent: true
```


#### 3.3.2 建立非持久会话的连接
我们只需要将 Clean 选项设为 true，就可以建立一个非持久会话的连接了，完整的代码 non_persistent_connetion.js 如下：

```
var mqtt = require('mqtt')
var client = mqtt.connect('mqtt://mqtt.eclipse.org:1883',{
	clientId: "mqtt_sample_id_1",
	clean: true
})

client.on('connect', function(connack ) {
	console.log(`returncode:${connack.returnCode},
	sessionPresent: ${connack.sessionPresent}`)
	client.end()
})
```

我们在终端上运行：

```
node persistent_connection.js
```

会得到以下输出：

```
returncode: 0, sessionPresent: false
```

无论运行多少次，SessionPresent 都将为 false。

#### 3.3.3 使用相同的 Client Identifier 进行连接

接下来我们看一下如果两个 Client 使用同样的 Client Identifier 会发生什么。我们把代码稍微调整下，在连接成功的时候保持连接，然后捕获 offline 事件，在 Client 的连接被关闭的时候打印出来。

完整的代码 identifcal.js 如下：

```
var mqtt = require('mqtt')
var client = mqtt.connect('mqtt://mqtt.eclipse.org:1883',{
	clientId:"mqtt_identical_1",
})

client.on('connect', function(connack) {
	console.log(`returncode: ${connack.returnCode},
	sessionPresent: ${connack.sessionPresent}`)
})

client.on('offline', function() {
	console.log("client went offline")
})
```

然后我们打开两个终端，分别在上面运行 node identifcal.js，然后我们会看到在两个终端上不停地出现以下打印：

```
returncode: 0, sessionPresent: false
client went offline
returncode: 0, sessionPresent: false
client went offline
returncode: 0, sessionPresent: false.......
```

在 MQTT 中，在两个 Client 使用相同的 ClientI dentifier 进行连接时，如果第二个 Client 连接成功，Broker 会关闭和第一个已经连接上的 Client 连接。由于我们使用的 MQTT 库实现了断线重连的功能，所以当连接被 Broker 关闭时，它又会尝试重新连接，结果就是这两个 Client 交替地把对方顶下线，我们就会看到这样的打印输出。因此在实际应用中，一定要保证每一个设备使用的 Client Identifier 是唯一的。

如果你观察到一个 Client 不停地上线下线，那么有很大可能是 Client Identifier 冲突的问题。

### 3.4 小结

在本节课中我们学习了 MQTT 连接关闭的过程，并且学习了连接建立和关闭的相关代码，下一课我们来学习发布和订阅的概念，实现消息在 Client 之间的传输。