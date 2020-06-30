---
title: 手把手教你入门AIoT（10）
date: 2020-06-29 10:50:00
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


在接下来的课程里，我们来完成一个 IoT+AI 的实战项目。本节课核心内容：

- 如何在 MQTT 里面传输大文件
- 消息去重
- 消息数据编码
- 实现 Android 发布端
- 发布识别结果

### 10.1 如何在 MQTT 里面传输大文件

我们前面提到过，一个 MQTT 数据包最大可以达到约 256M，所以对于传输图片的需求，最简单直接的方式就把图片数据直接包含在 PUBLISH 包里面进行传输。

还有一种更好的做法。在发布数据之前，先把图片上传到云端的某个图片存储里，然后 PUBLISH 包里面只包含图片的 URL，当订阅端接收这个数据之后，它再通过图片的 URL 来获取图片。这样的做法较前面有几个优点。

- **对订阅端来说，它可以在有需要的时候再下载图片数据**，而第一种做法，每次都必须接收图片的全部数据。
- **这种做法可以处理文件大于 256M 的情况**，而第一种做法，必须把文件分割为多个 PUBLISH 包，订阅端接收后再重新组合，非常麻烦。
- **节省带宽**，如果图片数据直接放在 PUBLISH 包中，那么 Broker 就需要预留相对大的带宽。目前在中国，带宽还是比较贵的。PUBLISH 包中只包含 URL 的话，每一个 PUBLISH 包都很小，那么 Broker 的带宽需求就很小了。虽然上传图片也需要带宽，但是如果你使用云存储，比如阿里云 OSS、七牛等，从它们那里购买上传和下载图片的带宽要便宜很多。同时，这些云存储服务商建设了很多 CDN，通常上传和下载图片比直接通过 PUBLISH 包传输要快一些。
- **节约存储和处理能力**，因为 Broker 需要存储 Client 未接收的消息，所以如果图片包含在 PUBLISH 包里面，Broker 需要预留相当的存储空间；而使用云存储的话，存储的成本比自建要便宜得多。

在这个实战项目里，我们用第二种方式来传输图片，使用[七牛](https://www.qiniu.com/)作为图片存储。

### 10.2 消息去重

为了兼顾效率和可靠性，我们使用 QoS1 来传输消息。QoS1 有一个问题，就是可能会收到重复的消息，所以需要在应用里面手动对消息进行去重。

我们可以在消息数据里面带一个唯一的消息 ID，通常是 UUID。订阅端需要保存已接收消息的 ID，当收到新消息的时候，通过消息的 ID 来判断是否是重复的消息，如果是，则丢弃。

### 10.3 消息数据编码

我们需要对消息数据进行编码，方便订阅端对数据进行解析。通常可以使用 JSON 格式，如果设备的计算和存储能力有限，也可以使用 Protocol Buffer 这类对内存消耗更少、解析更快的编码方式。

这个项目里面，我们使用 JSON 作为数据编码格式。一个消息数据格式如下：

```
{'id': <消息 ID>, timestamp:<UNIX 时间戳>, image_url: <图片>, objects:[图片中物体名称的数组]}
```

接下来我们先来实现数据的发布端。

### 10.4 实现 Android 发布端

10.4.1 连接到 Broker

首先包含 MQTT Android 库的依赖：

```
repositories {

    maven {
        url "https://repo.eclipse.org/content/repositories/paho-snapshots/"
    }
}

dependencies {

    compile 'org.eclipse.paho:org.eclipse.paho.client.mqttv3:1.1.0'
}
```

接下来连接到 Broker，注意这里使用 AndriodID 作为 Client Identifier 的一部分，保证 Client Identifier 不会冲突：

```
String clientId = "client_" + Settings.Secure.getString(getApplicationContext().getContentResolver(), Settings.Secure.ANDROID_ID);

  mqttAsyncClient = new MqttAsyncClient("tcp://iot.eclipse.org:1883", clientId, 
          new MqttDefaultFilePersistence(getApplicationContext().getApplicationInfo().dataDir));

            mqttAsyncClient.connect(null, new IMqttActionListener() {

                @Override
                public void onSuccess(IMqttToken asyncActionToken) {

                    runOnUiThread(new Runnable() {

                        @Override
                        public void run() {
                            Toast.makeText(getApplicationContext(), "已连接到 Broker", Toast.LENGTH_LONG).show();
                        }
                    });
                }

                @Override
                public void onFailure(IMqttToken asyncActionToken, final Throwable exception) {

                    runOnUiThread(new Runnable() {

                        @Override
                        public void run() {
                            Toast.makeText(getApplicationContext(), "连接 Broker 失败:" + exception.getMessage(), Toast.LENGTH_LONG).show();

                        }
                    });
                }
            });
```

10.4.2 上传图片

首先包含七牛安卓库的依赖：

```
compile 'com.qiniu:qiniu-android-sdk:7.3.+'
```

然后上传图片：

```
uploadManager.put(getBytesFromBitmap(image), null, upToken, new UpCompletionHandler() {

     @Override
     public void complete(String key, ResponseInfo info, JSONObject response) {

         if(info.isOK()){
             //上传成功以后 PUBLISH
          }
     }
}, null);
```

### 10.5 发布识别结果

最后，在图片上传成功以后，将识别结果发布到 “front_door/detection/objects” 这个主题：

```
JSONObject jsonMesssage = new JSONObject();

jsonMesssage.put("id", randomUUID());
jsonMesssage.put("timestamp", timestamp);
jsonMesssage.put("objects", objects);
jsonMesssage.put("image_url", "http://" + QINIU_DOMAIN + "\\" + response.getString("key"));
	
mqttAsyncClient.publish("front_door/detection/objects", new MqttMessage(jsonMesssage.toString().getBytes()));
```

### 10.6 小结

本节课我们完成了 Android 发布端的代码，你可以在 https://github.com/sufish/object_detection_mqtt 找到全部代码。