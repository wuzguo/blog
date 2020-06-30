---
title: AngularJS的模块和服务

date: 2017-09-09 16:56:06

author: Zak

avatar: /blog/images/avatar.png

authorLink: http://www.wuzguo.com

authorAbout: https://github.com/wzguo

authorDesc: 一个自强不息，艰苦奋斗的「十八线码农」

categories: 前端

tags:
	- AngularJS
        - 服务

	- Service

keywords: 服务，Service，AngularJS

photos:
	
       - /blog/images/avatar.png

description: AngularJS的模块和服务使用说明
---

### 一、模块

#### 1.  什么是模块

​	大部分应用都有一个主方法`main`用来实例化、组织、启动应用。AngularJS应用没有主方法，而是使用模块来声明应用应该如何启动。模块允许通过声明的方式来描述应用中的依赖关系，以及如何进行组装和启动。

#### 2. Angular 的模块

​	模块是组织业务的封装，在一个模块当中定义多个服务。当引入了一个模块的时候，就可以使用这个模块提供的一种或多种服务了。AngularJS 本身的一个默认模块叫做 ng ，它提供了 `$http` ， `$scope`等等服务。服务只是模块提供的多种机制中的一种，其它的还有指令`directive` ，过滤器`filter`，及其它配置信息。

​	也可以在已有的模块中新定义一个服务，也可以先新定义一个模块，然后在新模块中定义新服务。服务是需要显式地的声明依赖（引入）关系的，让 ng 自动地做注入。

#### 3. 模块的优点

- 启动过程是声明式的，更容易懂。
- 在单元测试是不需要加载全部模块的，因此这种方式有助于写单元测试。
- 可以在特定情况的测试中增加额外的模块，这些模块能更改配置，能帮助进行端对端的测试。
- 第三方代码可以作为可复用的module打包到angular中。
- 模块可以以任何先后或者并行的顺序加载（因为模块的执行本身是延迟的）。

#### 4. 启动

​	通过ng-app指定对应的模块应用启动。

#### 5. 定义模块

```javascript
angular.module(name[, requires],configFn);
```

configFn 会在模块初始化时执行，可以在里配置模块的服务。

### 二、服务

#### 1. 概念

​	服务本身是一个任意的对象。

​	AngularJS通过依赖注入机制来使用服务。用 `$provider` 对象来实现自动依赖注入机制，注入机制通过调用一个provider 的 `$get()` 方法，把得到的对象作为参数进行相关调用。

```javascript
var myApp = angular.module('myApp',[],function($provide){
    // 自定义服务
    $provide.provider('CustomProvider',function(){

        this.$get = function(){
            return {
                message : 'CustomService Message'
            }
        }
    });

    // 自定义工厂
    $provide.factory('CustomFactory',function(){
        return [1,2,3,4,5,6,7];
    });

    // 自定义服务
    $provide.service('CustomService',function(){
        this.name = "service";
        this.func = function (x) {
            return x.toString(16);//转16进制
        }
    })
});
```

#### 2. 模块的定义

- `$provider.provider ` 

provider是唯一一种可以传进`.config()` 函数的 service。当你想要在service对象启用之前，先进行模块范围的配置，那就应该用provider。

需要注意的是：在config 函数里注入 provider 时，名字应该是：`providerName+Provider`。
使用 provider 的优点就是，你可以在 provider 对象传递到应用程序的其他部分之前在`app.config`函数中对其进行修改。 当你使用provider 创建一个 service 时，唯一的可以在你的控制器中访问的属性和方法是通过`$get()`函数返回内容。

```javascript
<body>
<div ng-app="myApp" ng-controller="myCtrl">
</div>

<script>
    var app = angular.module('myApp', []);

    //需要注意的是：在注入provider时，名字应该是：providerName+Provider   
    app.config(function(myProviderProvider){
        myProviderProvider.setName("大圣");       
    });
    app.provider('myProvider', function() {
        var name="";
        var test={"a":1,"b":2};
        //注意的是，setter方法必须是(set+变量首字母大写)格式
        this.setName = function(newName){
            name = newName  
        }

        this.$get = function($http,$q){
            return {
                getData : function(){
                    var d = $q.defer();
                    $http.get("url") //读取数据的函数。
                        .success(function(response) {
                            d.resolve(response);
                        })
                        .error(function(){
                            d.reject("error");
                        });
                    return d.promise;
                },
                "lastName":name,
                "test":test
            }   
        }

    });
    app.controller('myCtrl', function($scope,myProvider) {
        alert(myProvider.lastName);
        alert(myProvider.test.a)
        myProvider.getData().then(function(data){
            //alert(data)
        },function(data){
            //alert(data)
        });
    });
</script>
</body>
```

- `$provider.factory`

factory是直接把一个函数当成一个对象的`$get`方法，可以直接返回字符串。用factory就是创建一个对象，为它添加属性，然后把这个对象返回出来。你把service传进controller之后，在controller里这个对象里的属性就可以通过factory使用了。

```javascript
<body>
<div ng-app="myApp" ng-controller="myCtrl">
    <p>{{r}}</p>
</div>

<script>
    //创建模型
    var app = angular.module('myApp', []);

    //通过工厂模式创建自定义服务
    app.factory('myFactory', function() {
        var service = {};//定义一个Object对象'
        service.name = "张三";
        var age;//定义一个私有化的变量
        //对私有属性写getter和setter方法
        service.setAge = function(newAge){
            age = newAge;
        }
        service.getAge = function(){
            return age; 
        }
        return service;//返回这个Object对象
    });
    //创建控制器
    app.controller('myCtrl', function($scope, myFactory) {
        myFactory.setAge(20);
        $scope.r =myFactory.getAge();
        alert(myFactory.name);
    });
</script>
</body>
```

- `$provider.service`

service是用`new`关键字实例化的。因此，你应该给`this`添加属性，然后service返回`this`。你把service传进controller之后，在controller里`this`上的属性就可以通过service来使用了。

```javascript
<div ng-app="myApp" ng-controller="myCtrl">
    <h1>{{r}}</h1>
</div>
<script>
    var app = angular.module('myApp', []);
	// 注意这里没有返回值
    app.service('myService', function($http,$q) {
        this.name = "service";
        this.myFunc = function (x) {
            return x.toString(16);//转16进制
        }
        this.getData = function(){
            var d = $q.defer();
            $http.get("ursl")//读取数据的函数。
                .success(function(response) {
                d.resolve(response);
            })
                .error(function(){
                alert(0)
                d.reject("error");
            });
            return d.promise;
        }
    });
    app.controller('myCtrl', function($scope, myService) {
        $scope.r = myService.myFunc(255);
        myService.getData().then(function(data){
            console.log(data);//正确时走这儿
        },function(data){
            alert(data)//错误时走这儿
        });
    });
</script>
```

### 三、包装器

包装器decorator：`$provide`服务提供了在服务实例创建时对其进行拦截的功能，可以对服务进行扩展，甚至完全替代它。常见使用场景：

- 对服务进行扩展。
- 对服务封装以便开发时调试和跟踪。

使用方式：`$provide.decorator('UserService',fn)` 。

```javascript
var app = angular.module('myApp',[]);
//创建一个provider类型的服务
app.provider('UserService', {
    //默认url
    url:'users.json',
    //添加setUrl方法提供修改url的方法
    setUrl: function (newUrl) {
        this.url = newUrl;
    },
    $get: function ($http) {//可以在此处注入其他服务
        var currentUser = {};
        var self = this;//将外层引用保存下来
        return {
            getCurrentUser:function() {
                //通过$http请求数据
                return $http.get(self.url);
            },
            setCurrentUser:function(user) {
                currentUser = user;
            }
        }
    }
});

//可以通过config函数配置provider类型的服务
//此处要对UserService进行包装，因此我们在config函数中注入$provide
app.config(function ($provide, UserServiceProvider) {
    MyUserServiceProvider.setUrl('data/users.json');
  
    //对UserService包装,第一个参数指定为拦截服务的名称
    //此处需要在构造函数中注入$delegate
    //因为需要输出控制台信息，所以注入$log
    $provide.decorator('UserService', function ($delegate, $log) {
        return {
            delCurrentUser: function () {
                //执行所拦截服务之前的方法，这里返回的是Promise对象
                var p = $delegate.getCurrentUser();
                //额外做些事情：记录一下该次请求所消耗时间
                //首先记录请求发起时间
                var start = new Date();
                p.finally(function () {
                    $log.info('执行[getCurrentUser]方法消耗时间：'+(new Date()-start)+' 毫秒');
                });
                return p;
            }
        }
    })
});
```

### 四、参考博客

- 自定义服务详解（factory、service、provider）[http://blog.csdn.net/zcl_love_wx/article/details/51404390)](http://blog.csdn.net/zcl_love_wx/article/details/51404390)
- kittencup 的AngularJS教学视频 [http://kittencup.com/](http://kittencup.com/)

