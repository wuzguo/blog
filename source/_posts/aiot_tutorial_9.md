---
title: 手把手教你入门AIoT（9）
date: 2020-06-29 09:50:00
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


这一课我们来学习 MQTT 协议中的 Keep Alive 机制。本节课核心内容：

- Keep Alive
- 代码实践
- 如何在移动端保持 MQTT 连接

### 9.1 Keep Alive

在上一课中，我们提到过 Broker 需要知道 Client 是否非正常地断开了和它的连接，以发送遗愿消息。实际上 Client 也需要能够很快地检测到它失去了和 Broker 的连接，以便重新连接。

MQTT 协议是基于 TCP 的一个应用层协议，理论上 TCP 协议在丢失连接时会通知上层应用，但是 TCP 有一个半打开连接的问题（half-open connection）。这里我不打算深入分析 TCP 协议，需要记住的是，在这种状态下，一端的 TCP 连接已经失效，但是另外一端并不知情，它认为连接依然是打开的，它需要很长的时间才能感知到对端连接已经断开了，这种情况在使用移动或者卫星网络的时候尤为常见。

仅仅依赖 TCP 层的连接状态监测是不够的，于是 MQTT 协议设计了一套 Keep Alive 机制。回忆一下，在建立连接的时候，我们可以传递一个 Keep Alive 参数，它的单位为秒，MQTT 协议中约定：**在 1.5\*Keep Alive 的时间间隔内，如果 Broker 没有收到来自 Client 的任何数据包，那么 Broker 认为它和 Client 之间的连接已经断开；同样地, 如果 Client 没有收到来自 Broker 的任何数据包，那么 Client 认为它和 Broker 之间的连接已经断开。**

MQTT 还有一对 PINGREQ/PINGRESP 数据包，当 Broker 和 Client 之间没有任何数据包传输的时候，可以通过 PINGREQ/PINGRESP 来满足 Keep Alive 的约定和侦测连接状态。

#### 9.1.1 PINGREQ

PINGREQ 数据包没有可变头（Variable header）和消息体（Payload），当 Client 在一个 Keep Alive 时间间隔内没有向 Broker 发送任何数据包，比如 PUBLISH 和 SUBSCRIBE 的时候，它应该向 Broker 发送 PINGREQ 数据包。

#### 9.1.2 PINGRESP

PINGRESP 数据包没有可变头（Variable header）和消息体（Payload），当 Broker 收到来自 Client 的 PINGREQ 数据包，它应该回复 Client 一个 PINGRESP 数据包。

对于 Keep Alive 机制，我们还需要记住以下几点：

- 如果在一个 Keep Alive 时间间隔内，Client 和 Broker 有过数据包传输，比如 PUBLISH，Client 就没有必要再使用 PINGREQ 了，在网络资源比较紧张的情况下这点很重要；
- Keep Alive 值是由 Client 指定的，不同的 Client 可以指定不同的值；
- Keep Alive 的最大值为 18 小时 12 分 15 秒；
- Keep Alive 值如果设为 0 的话，代表不使用 Keep Alive 机制。

### 9.2 代码实践

我们首先来完成一个 Client 的代码， 它会把发送和收到的 PINGREQ/PINGRESP 打印出来。

完整代码 keepalive.js 如下：

```
var mqtt = require('mqtt')

var dateTime = require('node-datetime');
var client = mqtt.connect('mqtt://iot.eclipse.org', {
    clientId: "mqtt_sample_id_chapter_9",
    clean: false,
    keepalive: 5
})

client.on('connect', function () {
    client.on('packetsend', function (packet) {
        console.log(`${dateTime.create().format('H:M:S')}: send ${packet.cmd}`)
    })

    client.on('packetreceive', function (packet) {
        console.log(`${dateTime.create().format('H:M:S')}: receive ${packet.cmd}`)
    })
})
```

这里为了演示起见，把 Keep Alive 的时间间隔设为了 5 秒。

运行 `node keepalive.js`，会得到以下输出：

```
19:42:44: send pingreq
19:42:44: receive pingresp
19:42:49: send pingreq
19:42:49: receive pingresp
19:42:54: send pingreq
19:42:54: receive pingresp
.........
```

可以看到，每隔 5 秒就会有一个 PINGREQ/PINGRESP 的交互。

接下来 Client 每隔 4 秒钟发送一个 PUBLISH 数据包，我们来看看是否还会触发 PINGREQ/PINGRESP。

完整的代码 keepalive_with_publish.js 如下：

```
var mqtt = require('mqtt')

var dateTime = require('node-datetime');
var client = mqtt.connect('mqtt://iot.eclipse.org', {
    clientId: "mqtt_sample_id_chapter_9",
    clean: false,
    keepalive: 5
})

client.on('connect', function () {
    client.on('packetsend', function (packet) {
        console.log(`${dateTime.create().format('H:M:S')}: send ${packet.cmd}`)
    })

    client.on('packetreceive', function (packet) {
        console.log(`${dateTime.create().format('H:M:S')}: receive ${packet.cmd}`)
    })

    setInterval(function () {
        client.publish("foo/bar", "test")
    }, 4 * 1000)
})
```

运行 `node chapter-9/keepalive_with_publish.js`，会得到以下输出：

```
19:54:37: send publish
19:54:41: send publish
19:54:45: send publish
......
```

正如之前讲的，如果在一个 Keep Alive 的时间间隔内，Client 和 Broker 之间有传输过数据包，那么就不会触发 PINGREQ/PINGRESP。

### 9.3 如何在移动端保持 MQTT 连接

通常在移动端使用 MQTT 的时候，都会碰到一个问题：App 被切入后台后，怎么才能保持 MQTT 的连接并继续接收消息呢？接下来我们就 Android 和 iOS 分别来讲一下。

#### 9.3.1 Android

在 Android 上，我们可以在一个 Service 中创建和保持 MQTT 连接，这样即使 App 被切入后台了，这个 Service 还在运行，那么 MQTT 的连接还存在，也能接收消息。类似如下的代码：

```
public class MQTTService extends Service{ 
    
    ........

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
       .......
       
       mqttClient.connect(...)
       
       .......
   }
   .......
}
```

接收到消息后，我们可以通过一些方式，比如广播，来通知 App 来处理这些消息。

### 9.3.2 iOS

iOS 的机制与 Android 不同，在 App 被切入后台时，你没有办法在后台运行 App 的任何代码（当然，iOS 提供了几种可以后台运行的类型，比如 Download、Audio 等，但是如果你的 App 假借这些方式来运行后台程序，是过不了审核的，所以这里只讨论正常的情况），所以无法通过 MQTT 的连接来获取消息。

在 iOS 上面，App 切入后台以后，正确接收 MQTT 消息的方式是：

1. Publisher 发布一条或者多条消息；
2. Publisher 通过某种渠道（比如 HTTP API）告知 App 的应用服务器，然后服务器通过苹果的 APNs 向对应的 iOS 订阅者推送一条消息；
3. 用户点击推送，App 进入前台；
4. App 重新建立和 Broker 的连接；
5. App 收到 Publisher 刚刚发送的一条或多条消息。

App 端的代码类似如下：

```
-(void)application:(UIApplication *)app didReceiveRemoteNotification:(NSDictionary *)userInfo {

  if([app applicationState] == UIApplicationStateInactive) {
        [mqttClient connect]
     }
}...
```

注意： 实际上，当下国内主流的 Android 系统都有后台清理功能， App 切入后台以后，它的服务，即使是前台服务（Foreground Service）也会很快地被杀掉，除非 App 被厂商或者用户加入白名单。所以在 Android 上最好还是利用厂商的推送通道，比如华为推送、小米推送等，在 App 被切入后台时采用和 iOS 上一样的机制来接收 MQTT 的消息。

### 9.4 小结

到此为止我们学习完了 MQTT 协议及其所有特性，你可在 https://github.com/sufish/mqtt-nodejs-sample 找到所有的示例代码，接下来我们进行一个 IoT+AI 的实战。