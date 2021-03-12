---
title: AngularJS与服务端交互
date: 2017-09-13 21:40:12
author: Zak
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wuzguo
authorDesc: 一个自强不息，艰苦奋斗的「十八线码农」
categories: 前端
tags:
	- AngularJS
	- $http
	- $resource
	- Restangular
keywords: AngularJS，$http，$resource，Restangular
photos:
	- /blog/images/avatar.png

description: AngularJS中使用$http、$resource、Restangular与服务端交互
---


### 一、`$http` 服务

在AngularJS中与远程HTTP服务器交互时会用一个非常关键的服务`$http`。`$http`是对原生的`XMLHttpRequest`对象的封装。`$http`的各种方式的请求更趋近于Rest风格。

### 二、参数说明

以下是$http服务的使用方式：

```javascript
    $http(config)
        .success(
            function (data, status, headers, config) {
                // ...
            })
        .error(
            function (data, status, headers, config) {
                // ... 
            });
```

参数注释：

- config 参数为一个json对象，如：

  ```json
  {url:"login.do",method:"post",data:{name:"12346",pwd:"123"}}
  ```

  config中可以设置如下参数：

  - method：请求方式，如：GET、POST、PUT、head、delete、jsonp。
  - url：请求的URL地址。
  - params：请求参数，将在URL上被拼接成`?key=value` ，健值对格式。
  - data：数据，将被放入请求中发送至服务端，健值对格式。
  - cache：若为`true`，在http中GET请求时采用默认的 `$http cache`，否则使用`$cacheFactory`的实例
  - timeout： 设置数值或promise对象，如果是数值，则延迟指定的毫秒数在发送请求，如果是promise，那么如果promise对象被查询数据时`resolve()`，则当前请求会被终止。
  - withCredentials：设置布尔值，默认情况下CORS请求不会发送cookie，所以如果设置为`true`，则就会将目标域的cookie包含在请求中。
  - xsrfHeaderName：cross site request forgery（跨站请求伪造名）。
  - xsrfCookieName：cross site request （跨站请求伪造cookieName）。
  - responseType：指定客户端接受数据的类型字符串，分别可以设置以下值：
    - ""
    - arraybuffer
    - blob
    - document（http文档）
    - json
    - text
    - moz-blob(Firefox接收进度事件）
    - moz-chunked-text(文本流）
    - moz-chunked-arraybuffer(ArrayBuffer流）

- success为请求成功后的回调函数，这里主要是对返回的四个参数进行说明。

  - data 响应体，包含服务端返回的数据。
  - status Http相应的状态值。
  - headers 获取相应Header数据的函数。

  ​        如获取跨域属性配置的值： `console.log(headers('Access-Control-Allow-Origin'));`

  - config 请求中的config对象，同上。

- error为请求失败后的回调函数，参数同success相同。


为了方便大家与HTTP服务器进行交互，AngularJS提供了各个请求方式的简写方法：

-  `$http.put/post(url,data,config);` ，其中 url、data必填，config 可选。
-  `$http.get/delete/jsonp/head(url,config);` ， 其中url必填，config可选。

### 三、`then`方法

#### 1. 使用方式

​	AngularJS 可以使用 `then() ` 方法来处理 `$http` 服务的回调，`then()` 方法接受两个可选的函数作为参数，表示请求成功（`success`）和发生错误（`error`）状态时的处理：　

`promise.then(successFn, errFn, notifyFn);`

​	无论请求成功与否，当结果可用之后都会立刻异步调用`successFn`或者`errFn`。这些方法只传递一个参数来调用回调函数，参数对象中包含相应体、状态码、相应头等信息。

​	在promise被执行时， `notifyFn`回调可能会被调用 0 到多次，以提供过程状态的提示。

```javascript
promise.then(
  function(resp){
	//响应成功时调用，resp是一个响应对象
  }, 
  function(resp) {
    // 响应失败时调用，resp带有错误信息
  }
);
```

`then()` 方法接收的响应对象（resp）包含5个属性：　

- data：（字符串或对象），响应体。
- status：相应http的状态码,如200。
- headers(函数)：头信息的Getter函数，可以接受一个参数，用来获取对应名字的值。
- config(对象)：生成原始请求的完整设置对象。
- statusText：相应的http状态文本，如 "`ok`"。

#### 2. 应用场景

上面介绍了两种获取请求结果的方式：`success()`和`then()`。

`success()` 就是典型的回调嵌套，代码格式不够美观，而且当出现多级嵌套的时候容易出错。

`then()`方法就清晰很多且性能较好，适合在链式编程的场景中使用。

在高版本的AngularJS中已经弃用`success()`的处理方式。

### 四、客户端缓存

当服务端数据很少修改时，可是使用客户端缓存提高请求相应速度，提升客户体验。在AngularJS中可以通过以下两种方式实现缓存：

- 设置cache字段为true

```javascript
$http.get('data/users.json',{cache:true}).success(function (data) {
    $scope.users = data.users;
});
```

- 自定义缓存对象，通过$cacheFactory服务获取实例

```javascript
var app = angular.module('myApp',[]);
//创建MyController，测试$http如何与服务器交互
app.controller('MyController', function ($scope,$http,$cacheFactory) {
    var lru;
    $scope.getUsers = function () {
        if (!lru) {
            lru = $cacheFactory('lru', {capacity: 20});
        }
        $http.get('data/users.json', {cache: lru}).success(function (data) {
            $scope.users = data.users;
        });
    }
});
```

缓存对象可使用的方法：

- info()  获取缓存对象信息。
- put(key, value)  存放数据。
- get(key)  获取数据。
- remove(key)  删除对应key的项。
- removeAll()  删除所有项。
- destroy()  销毁缓存。

```javascript
$http.get('data/users.json', {cache: lru}).success(function (data) {
	$scope.users = data.users;
	var info = lru.info();
     console.info(info);
     lru.put('2', "Tom");
});
```

### 五、全局配置

AngularJS可以设置$httpProvider完成http请求的全局配置，可以设置以下属性：

- 修改AngularJS默认的Content-Type值（application/json）。
- 设置默认缓存。
- 拦截器：在任何请求发送给服务器之前或者从服务器返回时可以对其拦截，从而为请求添加全局的功能。
  - 场景：身份验证、错误处理等。
  - 拦截器的类型：request、response、requestError、responseError。
  - 创建拦截器：1、创建拦截器工厂服务；2、注册该服务。

代码如下：

```javascript
var app = angular.module('myApp',[]);

//声明一个拦截器工厂服务
app.factory('myInterceptor', function () {
    //此处可以设置4个属性：request、response、requestError、responseError
    return {
        request: function (config) {
            //可以修改配置对象
            config.headers.customInfo='Tom';
            console.log(config);
            return config;
        },
        response: function (config) {
            console.log(config);
            return config;
        },
        requestError: function (rejection) {
            console.log(rejection);
            return rejection;
        },
        responseError: function (rejection) {
            console.log(rejection);
            return rejection;
        }
    }
});

app.config(function ($httpProvider) {
    //修改默认Content-Type为url-encoded
    $httpProvider.defaults.headers.post['Content-Type']='application/x-www.form-urlencoded;charset=utf-8';
    $httpProvider.defaults.headers.put['Content-Type']='application/x-www.form-urlencoded;charset=utf-8';

    //设置默认缓存
    $httpProvider.defaults.cache= true;

    //注册一个拦截器
    $httpProvider.interceptors.push('myInterceptor');
});
```

### 六、RESTful类型API调用

RESTful类型API调用：Representational State Transfer（表征状态转移）。AngularJS提供$resource服务可以同支持RESTful服务器端数据源进行交互。

####  1. 使用 `$resource`

- 引入`angular-resource.js`
- 添加依赖 `'ngResource'`
- 获取资源实例：`$resource(‘data/:name.json’, {name:’@username’})` 。
- 基于GET的HTTP方法：
  - `get(params, successFn, errorFn)` （get方法可以不需要设置回调函数）。
- `query(params, successFn, errorFn)` 。
- get和query的唯一不同是AngularJS期望query返回的是数组。

```javascript
var app = angular.module('myApp',['ngResource']);

//创建控制器获取用户Tom的数据并显示
//此处需要注入$resource，通过RESTful的形式查询数据
app.controller('MyController', function ($scope,$resource) {
    //获取资源实例
    var User = $resource('data/:name.json',{name:'@username'});
    //可以直接使用返回结果，好像是一个同步调用
    $scope.user = User.get({username:'tom'});

    //query方法的使用，它和get方法基本一致，唯一一点不同时AngularJS期待返回结果为数组
    User.query({name:'users'}, function (result) {
        console.log(result);
    });
});
```

#### 2. 配置 `$resource`

- 自定义 `$resource` 方法：设置 `$resource` 第三个参数对象。`$resource`提供了五种默认操作：`get`, `query`, `save`, `remove`, `delete`。你可以配置一个`update`操作来完成HTTP PUT：

```javascript
$resource(url,{},{
	'update':{method:'PUT'}
});
```

- 附加属性：资源有两个属性可以同底层数据定义交互

`$promise`： 为`$resource`生成的原始`promise`对象，特别用来同`$routeProvider.when()`在 resolve 时进行连接。
`$resolved` ：服务器首次响应时被设置为 true。

```javascript
console.info(User.$promise);
console.info(User.$resolved);
```

- 配置对象：

```javascript
$resource(url, {}, {
    'update':{
      method:'PUT'
    },
    'get':{
        isArray:true, //以数组形式返回结果，等同于query
        interceptor：{
          //设置拦截器，此处只能设置response和responseError
          response:function(){
   		 },
          responseError:function(){
          }
       }
	}
});
```

### 七、使用Restangular

#### 1.  特点

Restangular通过完全不同的途径实现了XHR通信，弥补了$http和$resource内置服务的缺点，主要优点如下：

- 支持promise：使用起来符合angularjs习惯并支持链式操作。
- promise展开：可以同时操作promise对象。
- 简单明了的API。
- 全HTTP方法的支持。
- 忘记URL：不需要提前指定URL（除了基础URL以外）。
- 资源嵌套：Restangular可以直接处理嵌套资源，无需创建新的实例。
- 单例：使用过程中仅需创建一个Restangular资源对象实例。

#### 2. 安装使用

- 引入Restangular：[https://github.com/mgonto/restangular](https://github.com/mgonto/restangular)

  引入Lo-Dash(或underscore)：[https://raw.github.com/lodash/lodash/3.10.1/lodash.min.js](https://raw.github.com/lodash/lodash/3.10.1/lodash.min.js)

- 添加依赖，并且注入服务`Restangular` 。

- 创建Restangular主对象：

  ```javascript
  Restangular.all('accounts');      //url: /accounts/
  Restangular.one('accounts', 1);   //url: /accounts/1
  Restangular.several('accounts',  1, 2, 3);
  ```

  - GET方法使用

  ```javascript
  var baseAccounts = Restangular.all('accounts');
  baseAccounts.getList().then(function(accounts){...});
  ```

  ```javascript
  scope.accouts = Restangular.all('accounts').getList().object;
  ```

  ```javascript
  //GET - ONE /account/1?p=1
  Restangular.one('accounts',1).get({p:1}).then(function(account){
  	$scope.curAccount = account;
  });
  ```

  - POST方法使用

  ```javascript
  baseAccounts.post(newAccount);
  ```

  - PUT方法使用

  ```javascript
  // UPDATE to /account/1
  // 前面GET方法使用的时候有赋值：$scope.curAccount = account;
  $scope.curAccount.name = 'Jerry';
  $scope.curAccount.put();
  ```

  - DELETE方法使用

  ```javascript
  // DELETE to /account/1
  // 前面GET方法使用的时候有赋值：$scope.curAccount = account;
  $scope.curAccount.remove();
  ```



### 八、参考博客

- AngularJS之$http与服务器交互：  [http://www.cnblogs.com/sytsyt/p/3297872.html](http://www.cnblogs.com/sytsyt/p/3297872.html) 
- AngularJS的$http服务的应用：  [http://www.cnblogs.com/whh412/p/5644458.html](http://www.cnblogs.com/whh412/p/5644458.html)
- ​