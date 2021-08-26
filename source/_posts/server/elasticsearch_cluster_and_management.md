---
title: Elasticsearch集群搭建及管理
date: 2017-08-02 21:13:54
author: Zak
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wuzguo
authorDesc: 一个追求进步的「十八线码农」
categories: 后台
tags: 
- Elasticsearch
- Elasticsearch-head
keywords: Elasticsearch，Elasticsearch-head
photos:
- /blog/images/201708/2.png
description: Elasticsearch集群搭建及管理以及可视化查看Elasticsearch数据
---

### 安装Elasticsearch-head
- 安装nodejs

- 安装phantomjs，国内下载地址：https://npm.taobao.org/mirrors/phantomjs

- 下载elasticsearch-head源码
  ```shell
   git clone https://github.com/mobz/elasticsearch-head.git
   cd elasticsearch-head/
   npm install
  ```

### 修改 `elasticsearch` 配置文件

- 修改`elasticsearch.yml`，增加跨域的配置(需要重启es才能生效)
  ```properties
   http.cors.enabled: true
   http.cors.allow-origin: "*"
  ```
### 修改`elasticsearch-head`配置

- 编辑`head/Gruntfile.js`，修改服务器监听地址，增加`hostname`属性，将其值设置为 `*`

   ```json
       connect: {
           hostname: '*',
           server: {
                options: {
                        port: 9100,
                        base: '.',
                        keepalive: true
                }
          }
          
       }
   ```

- 或者

   ```json
          connect: {
          	server: {
                      options: {
                              hostname: '*',
                              port: 9100,
                              base: '.',
                              keepalive: true
                      }
              }
          }
   ```

- 编辑`head/_site/app.js`，修改 `head` 连接 `es` 的地址，将 `localhost` 修改为 `es` 的 `IP` 地址：

   原配置

    `this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://localhost:9200";`

   将localhost修改为ES的IP地址

   `this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://YOUR-ES-IP:9200";`

### 启动elasticsearch-head

`cd elasticsearch-head/ && ./node_modules/grunt/bin/grunt server`

或者：

`npm run start`

- 注意：
   - 此时elasticsearch-head为前台启动，如果终端退出，那么elasticsearch-head服务也会随之关闭。
   - 在非elasticsearch-head目录中启动server会失败。因为grunt需要读取目录下的Gruntfile.js。

### 搭建集群

#### 注意事项

- 拷贝`elasticsearch`安装包时不要时不要带`data`目录，否则你永远永远不可能搭建成功。
  ![](/blog/images/201708/2.png)

  解决问题的链接 :https://github.com/elastic/elasticsearch/issues/21405

#### 配置文件

- 配置es的集群名称, es会自动发现在同一网段下的es,如果在同一网段下有多个集群,就可以用这个属性来区分不同的集群

  ```properties
  cluster.name: es-hongling-test
  ```

- 节点名称

  ```properties
  node.name: hongling-node-0
  ```

- 设置绑定的ip地址还有其它节点和该节点交互的ip地址

  ```properties
  network.host: 127.0.0.1
  ```

- 指定http端口,你使用head､kopf等相关插件使用的端口

  ```properties
  http.port: 9200
  ```

- 跨域，如果要使用head,那么需要增加新的参数,使head插件可以访问es

  ```properties
  http.cors.enabled: true
  http.cors.allow-origin: "*"
  ```

- 指定该节点是否有资格被选举成为node

  ```properties
  node.master: true 
  ```

- 指定该节点是否存储索引数据,默认为true

  ```properties
  node.data: true
  ```

- 设置节点间交互的tcp端口,默认是9300

  ```properties
  transport.tcp.port: 9300
  network.publish_host: 127.0.0.1
  ```

- 这是在因为Centos6不支持SecComp，而ES5.2.0默认bootstrap.system_call_filter为true进行检测，所以导致检测失败，失败后直接导致ES不能启动

  ```properties
  bootstrap.memory_lock: false
  bootstrap.system_call_filter: false
  ```

- 通过配置大多数节点（主节点总数/ 2 + 1）来防止“分裂大脑”：

  ```properties
  discovery.zen.minimum_master_nodes: 2
  ```

- 设置集群中master节点的初始列表,可以通过这些节点来自动发现新加入集群的节点

  ```properties
  discovery.zen.ping.unicast.hosts: ["127.0.0.1:9300","127.0.0.1:9301","127.0.0.1:9302"]
  ```

### 集群的管理

- 格式化返回结果

  ```shell
  curl -XGET localhost:9200/twitter?pretty
  ```

- 查看集群`state`

  ```shell
  curl -XGET 'http://localhost:9200/_cluster/state
  ```

- 查看集群 health 状态

  ```shell
  curl -XGET 'http://localhost:9200/_cluster/health?pretty
  ```

- 查看某个索引的 health 状态

  ```shell
  curl -XGET 'http://localhost:9200/_cluster/health/movie
  ```

- 批量加载数据

  ```shell
  curl -XPOST 'localhost:9200/bank/account/_bulk?pretty' --data-binary @accounts.json
  ```

- 查看_type的映射

  ```shell
  curl -XGET 'localhost:9200/shakespeare/_mapping/act'?pretty
  ```


### 插件安装

- analysis-ik

  - download or compile

    download pre-build package from here: https://github.com/medcl/elasticsearch-analysis-ik/releases

     or compiled from the source:

     checkout ik version respective to your elasticsearch version

    ```shell
     git checkout tags/{version}
     mvn package
    ```

     copy and unzip target/releases/elasticsearch-analysis-ik-{version}.zip to your-es-root/plugins/ik
  - restart elasticsearch

### Java Client

- 连接ES集群

   ```java
     // on startup
     Settings settings = Settings.builder()
             .put("cluster.name", "es-hongling-test").build();

     TransportClient client = new PreBuiltTransportClient(Settings.EMPTY)
             .addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"), 9300))
             .addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"), 9301))
             .addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName("127.0.0.1"), 9302));

     List<DiscoveryNode> nodeList = client.connectedNodes();
     for (DiscoveryNode node : nodeList) {
         System.out.println(" node: " + node.getName());
     }
   ```

- 注意：使用的IP地址和端口号对应配置文件中的 `network.host` 和 `transport.tcp.port`， 否则将保错：

  ```java
  Exception in thread "main" NoNodeAvailableException[None of the configured nodes are available: [{#transport#-1}{9ibol1jmRlCVKFW8PjeJbQ}{127.0.0.1}{127.0.0.1:9200}]]
  at org.elasticsearch.client.transport.TransportClientNodesService.ensureNodesAreAvailable(TransportClientNodesService.java:348)
  at org.elasticsearch.client.transport.TransportClientNodesService.execute(TransportClientNodesService.java:246)
  ......
  ```
### 常见错误及解决方案

- 聚合查询时报错

  ```java
  Exception in thread "main" Failed to execute phase [dfs], all shards failed; shardFailures {[IHX-j5mQRLeH6DK6pATl-A][logstash-logs][0]: RemoteTransportException[[hongling-node-2][127.0.0.1:9302][indices:data/read/search[phase/dfs]]]; nested: IllegalArgumentException[Fielddata is disabled on text fields by default. Set fielddata=true on [extension] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead.]; }{[IHX-j5mQRLeH6DK6pATl-A][logstash-logs][1]: RemoteTransportException[[hongling-node-2][127.0.0.1:9302][indices:data/read/search[phase/dfs]]]; nested: IllegalArgumentException[Fielddata is disabled on text fields by default. Set fielddata=true on [extension] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead.]; }{[iHjuxsoZQOiQwSB2Y-rYkw][logstash-logs][2]: RemoteTransportException[[hongling-node-0][127.0.0.1:9300][indices:data/read/search[phase/dfs]]]; nested: IllegalArgumentException[Fielddata is disabled on text fields by default. Set fielddata=true on [extension] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory. Alternatively use a keyword field instead.]; }

     	at org.elasticsearch.action.search.AbstractSearchAsyncAction.onPhaseFailure(AbstractSearchAsyncAction.java:272)
     	at org.elasticsearch.action.search.AbstractSearchAsyncAction.executeNextPhase(AbstractSearchAsyncAction.java:130)
     	at org.elasticsearch.action.search.AbstractSearchAsyncAction.onPhaseDone(AbstractSearchAsyncAction.java:241)
     	at org.elasticsearch.action.search.InitialSearchPhase.onShardFailure(InitialSearchPhase.java:87)
     	at org.elasticsearch.action.search.InitialSearchPhase.access$100(InitialSearchPhase.java:47)
     	at org.elasticsearch.action.search.InitialSearchPhase$1.onFailure(InitialSearchPhase.java:155)
     	at org.elasticsearch.action.ActionListenerResponseHandler.handleException(ActionListenerResponseHandler.java:51)
     	at org.elasticsearch.transport.TransportService$ContextRestoreResponseHandler.handleException(TransportService.java:1050)
     	at org.elasticsearch.transport.TcpTransport.lambda$handleException$17(TcpTransport.java:1451)
     	at org.elasticsearch.common.util.concurrent.EsExecutors$1.execute(EsExecutors.java:110)
  ```

  解决方案：

  - 原因： Fielddata is disabled on text fields by default. Set `fielddata=true` on [`your_field_name`] in order to load fielddata in memory by uninverting the inverted index. Note that this can however use significant memory.

    ```json
    curl -XPUT http://localhost:9202/logstash-logs/_mapping/log -d '{
      "log": {
        "properties": {
          "extension": {
            "type": "text",
            "fielddata": true
          }
        }
      }	
    }
    ```

- 向已存在的索引中添加自定义filter/analyzer，报错

  ```json
  {
     "error": "IndexAlreadyExistsException[[symbol] already exists]",
     "status": 400
  }
  ```

  解决方案：

  ```json
  POST /symbol/_close
  PUT /symbol/_settings
  {
    "settings": {
  	....    
    }
  }

  POST /symbol/_open
  ```
  这样就避免了需要重建索引的麻烦。有了新添加的filter和analyzer，就可以根据需要再对types中的mappings进行更新了。


