---
title: 手把手教你入门AIoT（12）
date: 2020-06-29 12:50:00
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


到目前为止，我们使用的都是一个公有的 Broker，对于学习和演示来说，应该是足够的。但是对于实际生产来说，我们需要有一个私有、可控的 Broker。

正如本课程开头所说，现在很多云服务商都提供了 MQTT Broker 服务，在这里我列举几个较大的：

- 阿里云的物联网套件
- 腾讯云的 IoT Hub
- 青云的 EMQ IoT Hub
- 百度天工

云服务商的 MQTT Broker 服务是一个很好的选择，接入和配置也很简单，你只需要阅读相应的产品文档，照着步骤一步一步来就可以了。

如果你因为某种原因无法使用公有云的服务，或者你需要可控性、定制性更强的 Broker，你也可以选择自行搭建 MQTT Broker。

这一课我们来学习如何自行搭建 MQTT Broker。本节课核心内容：

- EMQTT
- Client 身份验证
- 传输层安全

### 12.1 EMQTT

首先我想给大家推荐的 Broker 是 [EMQTT](https://github.com/emqtt/emqttd)，它是由国人开发和维护的一款开源 MQTT Broker。推荐它的理由是：

- 性能和可靠性，EMQTT 是用 Erlang 语言编写的，在电信行业工作的同学可能了解，电信行业里很多核心的应用系统都是用 Erlang 编写的；
- 纵向扩展能力，在 8 核 32G 的主机上，可以容纳超过 100 万 MQTT Client 接入；
- 横向扩展能力，支持多机组成集群；
- 基于插件的功能扩展，官方提供了很多插件来扩展功能和与其他业务系统集成，如果你熟悉 Erlang 语言，也可以自行通过插件的方式扩展；
- 项目由公司开发和维护，并提供商业服务。

实际上，上面提到的青云的 IoT Hub 就是基于 EMQTT 实现的，我自己在实际生产中也使用 EMQTT 多年了，对它的性能和稳定性是相当认可的。

12.1.1 安装和运行

这里我以 Mac 系统上的安装流程为例，来讲解如何安装和使用 EMQTT Broker。如果你没有使用 Mac 系统也不用担心，除了下载的二进制文件有区别以外，大部分使用方法在类 Unix 系统都是一样的。

**安装 Erlang 运行环境**

首先我们需要安装 Erlang 运行环境，在 Mac 上的安装方式是使用 Erlang 源代码编译：

1. 下载 Erlang/OTP 21.0 源文件包 [otp_src_21.0.tar.gz](http://erlang.org/download/otp_src_21.0.tar.gz)
2. 解压缩 `tar -xzf otp_src_21.0.tar.gz`
3. `cd otp_src_21.0.tar.gz`
4. `./configure`
5. `make`
6. `sudo make install`

安装完毕以后在终端输入 `erl`，即可进入 Erlang 的交互 Shell。

12.1.1.2 **安装 EMQTT**

安装 EMQTT 非常简单，下载 Mac 上的二进制包 [emqttd-macosx-v2.3.11.zip](http://emqtt.com/downloads/2318/macosx)，之后解压缩：

```
unzip emqttd-macosx-v2.3.11.zip && cd emqttd
```

你可以通过 `bin/emqttd start` 来运行 EMQTT Broker。如果没有问题，你就会看到终端输出：

```
emqttd 2.3.11 is started successfully!
```

现在 EMQTT Broker 以守护进程的方式运行。

可以查看当前 Broker 状态：

```
bin/emqttd_ctl status
```

没有问题的话会输出：

```
node 'emq@127.0.0.1' is started

emqttd 2.3.11 is running
```

关闭 Broker：

```
bin/emqttd stop
```

默认情况下，EMQTT Broker 的 MQTT 端口是 1883，WebSocket 的端口是 8083。

### 12.2 Client 身份验证

在前面课程里，我们使用的 Public Broker 没有对 Client 的身份做任何检验，任何 Client 都可以接入。而在实际生产中，我们应该对接入 Client 的身份进行校验，只允许通过校验的 Client 接入。

EMQTT 以插件的方式提供了多种 Client 授权方式，这里我以 PostgresSQL 数据库授权插件（emqx-auth-pgsql）为例，讲解一下如何进行 Client 授权的设置。

PostgresSQL 授权插件的原理很简单。在 PostgresSQL 数据库中建一张用户表来存储用于校验身份的用户名和密码，Broker 使用 Client 在 CONNECT 数据包里指定的用户名和密码，在这张表里面进行匹配，如果匹配通过，则运行接入；否则断开连接，不允许接入。

继续之前请先安装 PostgresSQL。

emqx-auth-pgsql 的配置文件位于 etc/plugins/emqx-auth-pgsql.conf，以下是需要配置的几项：

- `auth.pgsql.server = 127.0.0.1:5432`，PostgresSQL 的 IP 地址和端口，使用默认值；
- `auth.pgsql.username = mqtt`，数据库用户名；
- `auth.pgsql.password = mqtt`，数据库密码；
- `auth.pgsql.database = mqtt`，数据库名称；
- `auth.pgsql.auth_query = select password from mqtt_user where username = '%u' limit 1`，匹配时使用的查询 SQL，使用默认值；
- `auth.pgsql.password_hash = plain`，mqtt_user 表 password 字段的加密方式，这里为了演示简单起见，设为 plain 不加密，可选的值还有 md5 | sha | sha256 | bcrypt。

然后用 `#` 注销掉 `auth.pgsql.acl_query`，这个是控制 Client 对某个主题的 Publish 和 Subscribe 的权限。我们暂时不用这个功能。

接下来我们在 PostgresSQL 中创建相应的数据库：

```
CREATE DATABASE mqtt
    WITH OWNER = mqtt
    ENCODING = 'UTF8'
    TABLESPACE = pg_default
    LC_COLLATE = 'C'
    LC_CTYPE = 'C'
    CONNECTION LIMIT = -1;
```

然后创建用户表，并插入一条数据：

```
CREATE TABLE mqtt_user
(
    id   SERIAL primary key,
    is_superuser boolean,
    username     character varying(100),
    password     character varying(100)
);

INSERT INTO mqtt_user(is_superuser， username, password) VALUES (false， 'testuser', '123456')
```

加载 emqx-auth-pgsql 插件：

```
bin/emqttd_ctl plugins load emq_auth_pgsql
```

不出意外的话，会得到这样的输出：

```
Start apps: [emq_auth_pgsql]

Plugin emq_auth_pgsql loaded successfully.
```

接下来我们来测试一下。

运行代码：

```
var mqtt = require('mqtt')

var client = mqtt.connect('mqtt://127.0.0.1:1883')

client.on('connect', function (connack) {
    console.log(`return code: ${connack.returnCode}, sessionPresent: ${connack.sessionPresent}`)
    client.end()
})

client.on("error", function (error) {
    console.log(`${error}`)
})
```

会得到以下输出：

```
Error: Connection refused: Bad username or password
```

然后在连接的时候我们把正确的用户名和密码加进去：

```
var mqtt = require('mqtt')

var client = mqtt.connect('mqtt://127.0.0.1:1883', {
    username: 'testuser',
    password: '123456'
})

client.on('connect', function (connack) {
    console.log(`return code: ${connack.returnCode}, sessionPresent: ${connack.sessionPresent}`)
    client.end()
})
```

那么 Client 可以正确地连接到 Broker，得到以下输入：

```
return code: 0, sessionPresent: false
```

这样我们就完成了 Client 的身份验证。

### 12.3 传输层安全

到目前为止，本课程中的 MQTT 代码都是使用明文来传输 MQTT 数据包，包括含有 username、password 的 CONNECT 数据包。在实际的生产环境中，这样显然是不安全的。

MQTT 支持传输层加密。通常我们在生产环境中，都需要使用 SSL 来传输 MQTT 数据包，使用传输层加密的 MQTT 被称为 MQTTS（类似于 HTTP 之于 HTTPS）。

我们使用的 Public Broker 支持 MQTTS，你只需要在连接时修改一下 Broker URL，将 “mqtt” 换成 “mqtts” 就可以了：

```
var client = mqtt.connect('mqtts://iot.eclipse.org'）
```

EMQTT 当然也是支持 MQTTS 的，你可以在 etc/certs 下配置你的 SSL 证书。EMQTT 也自带了一份自签署的证书，可以开箱即用地在 8883 端口使用 MQTTS。但是因为是自签署的证书，所以你需要关闭客户端的证书验证。

```
var client = mqtt.connect('mqtts://127.0.0.1:8883', {

    rejectUnauthorized: false
}）
```

### 12.4 小结

这一课里我们学习了如何搭建自己的 MQTT Broker，如何进行 Client 身份验证，以及如何使用 MQTTS 加强 Broker 的安全性。EMQTT 还提供了很多其他扩展功能，有兴趣的话可以查看 [EMQTT 文档](http://emqtt.com/docs/v2/)进一步了解。下一课我们来学习 MQTT 的最新版本，MQTT 5.0 的新特性。