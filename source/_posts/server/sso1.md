---
title: CAS单点登录
date: 2016-05-13 11:11:14 
author: Zak
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wuzguo
authorDesc: 一个追求进步的「十八线码农」
categories: 后台
tags: 
	- SSO
	- CAS
	- SSL协议
keywords: SSO，CAS
photos:
	- /blog/images/201605/2.png
description: 配置Tomcat使用https协议，CAS单点登录(SSO)
---

1.创建证书
需要安装JDK1.4以上版本并配置JAVA_HOME和PATH环境变量。

切换到命令窗口，并切换到某个目录下（比如C:\）


- 生成密钥

    keytool -genkey -alias tomcat -keyalg RSA -keypass changeit -storepass changeit -keystore server.keystore -validity 3600
在交互命令行中，第一项“您的名字与姓氏是什么？”需要填写服务器域名（本机用localhost）


- 导出证书
    keytool -export -trustcacerts -alias tomcat -file server.cer -keystore  server.keystore -storepass changeit
    

- 加入JDK受信任库
    keytool -import -trustcacerts -alias tomcat -file server.cer -keystore  %JAVA_HOME%/jre/lib/security/cacerts -storepass changeit

还有两个辅助命令，如果需要的时候再使用：

查看受信任库内容
    keytool -list -keystore %JAVA_HOME%/jre/lib/security/cacerts

从受信任库中删除
    keytool -delete -trustcacerts -alias tomcat  -keystore  %JAVA_HOME%/jre/lib/security/cacerts -storepass changeit

![](/images/201605/2.png)

2.修改Tomcat配置
在%TOMCAT_HOME%/conf/server.xml中找到Connector配置群，增加一项

    <Connector protocol="org.apache.coyote.http11.Http11NioProtocol"   
       port="8443" minSpareThreads="5" maxSpareThreads="75"   
       enableLookups="true" disableUploadTimeout="true" 
       acceptCount="100"  maxThreads="200"   
       scheme="https" secure="true" SSLEnabled="true"   
       clientAuth="false" sslProtocol="TLS"   
       keystoreFile="C:/server.keystore" 
       keystorePass="changeit"/>

其中keystoreFile配置为密钥库所在路径，keystorePass配置为密钥库文件保护密码。

3.验证访问
重启Tomcat，在浏览器中访问配置好的Web应用，如果弹出“证书错误”页面，选“继续浏览”即可，不影响使用。

![](/images/201605/3.png)



---后续将连载单点登录，敬请期待...