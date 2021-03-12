---
title: AngularJS 控制器

date: 2017-09-09 20:17:06

author: Zak

avatar: /blog/images/avatar.png

authorLink: http://www.wuzguo.com

authorAbout: https://github.com/wuzguo

authorDesc: 一个追求进步的「十八线码农」

categories: 前端

tags:
	- AngularJS

	- 控制器

	- Controller

keywords: 控制器，Controller，AngularJS

photos:
	- /blog/images/avatar.png

description: AngularJS 控制器介绍和正确使用
---

### 一、定义

​	AngularJS 控制器控制 AngularJS 应用程序的数据。 AngularJS 控制器是常规的 JavaScript 对象，由标准的 JavaScript 对象的构造函数 创建。**ng-controller** 指令定义了应用程序控制器。

在AngularJS中可以通过以下格式自定义控制器：

```javascript
module.controller(name,controllerFactory)
```

或者

```javascript
 $controllerProvider.register(name,controllerFactory);
```


### 二、使用实例

​	AngularJS 应用程序由 ng-app 定义。`ng-controller="myCtrl"` 属性是一个 AngularJS 指令。用于定义一个控制器。`myCtrl` 函数是一个 JavaScript 函数。**AngularJS 使用`$scope` 对象来调用控制器。**

​	在 AngularJS 中， `$scope` 是一个应用对象（属于应用变量和函数）。控制器的 `$scope` （相当于作用域、控制范围）用来保存AngularJS Model（模型）的对象。

```html
<div ng-app="myApp" ng-controller="myCtrl">
  名: <input type="text" ng-model="firstName"><br>
  姓: <input type="text" ng-model="lastName"><br>
  <br>
  姓名: {{firstName + " " + lastName}}
</div>

<script>
var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope) {
    $scope.firstName = "John";
    $scope.lastName = "Doe";
});
</script>
```

控制器也可以有方法（变量和函数）：

```html
<div ng-app="myApp" ng-controller="personCtrl">

名: <input type="text" ng-model="firstName"><br>
姓: <input type="text" ng-model="lastName"><br>
<br>
姓名: {{fullName()}}

</div>

<script>
var app = angular.module('myApp', []);
app.controller('personCtrl', function($scope) {
    $scope.firstName = "John";
    $scope.lastName = "Doe";
    $scope.fullName = function() {
        return $scope.firstName + " " + $scope.lastName;
    }
});
</script>
```

### 三、依赖注入

AngularJS中控制器使用的对象可以通过显示注入和隐式注入两种方式进行，隐式依赖注入在代码发布过程中经过某些压缩工具压缩后可能不能正常使用，一般情况下建议使用显示注入的方式。实例如下：

- 隐式注入

```javascript
var myApp = angular.module('myApp', [])
  .factory('CustomService', ['$window', function (a) {
    console.log(a);
  }])
  // 隐示的依赖注入
  .controller('firstController', function ($scope, CustomService) {
    console.log(CustomService);
  });
```

- 显示注入（建议使用）

```javascript
var myApp = angular.module('myApp', [])
  // 显示的依赖注入
  .controller('secondController', ['$scope', '$filter', function (a, b) {
    console.log(b('json')([1, 2, 3, 4, 5]));
  }]);
```
显示依赖注入第二种方式，一般不建议使用：
```javascript
function otherController(a) {
    console.log(a);
}
otherController.$inject = ['$scope'];
```

然后在HTML中调用控制器：

```html
<div ng-app="myApp">
    <div ng-controller="secondController">
    </div>
    <div ng-controller="otherController">
    </div>
</div>
```

### 四、正确使用

​	Controller不应该尝试做太多的事情。它应该仅仅包含单个视图所需要的业务逻辑，保持Controller的简单性，常见办法是抽出那些不属于Controller的工作到service中，在Controller通过依赖注入来使用这些 service。

不要在Controller中做以下的事情：

- 任何类型的DOM操作，Controller应该仅仅包含业务逻辑，任何表现逻辑放到Controller中，大大地影响了应用逻辑的可测试性。angular为了自动操作（更新）DOM，提供的数据绑定。如果希望执行我们自定义的DOM操作，可以把表现逻辑抽取到directive中。


- Input formatting（输入格式化），使用angular form controls 代替。
- Output filtering （输出格式化过滤），使用angular filters 代替。
- 执行无状态或有状态的、controller共享的代码，使用angular services 代替。
- 实例化或者管理其他组件的生命周期（例如创建一个服务实例）。


### 五、参考博客

- kittencup 的AngularJS教学视频 [http://kittencup.com/](http://kittencup.com/)