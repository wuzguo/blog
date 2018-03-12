---
title: AngularJS的指令
date: 2017-09-03 21:25:24
author: wúzguó
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wzguo
authorDesc: 一个自强不息，艰苦奋斗的「十八线码农」
categories: 前端
tags:
	- AngularJS
	- 指令
	- directive
keywords: 指令，directive，AngularJS
photos:
	- /blog/images/avatar.png
description: AngularJS指令的使用介绍和自定义指令时各个参数的函数的介绍
---

### 一、定义

当我们开发应用程序实现业务需求时都需要扩展HTML，在AngularJS中通过指令来实现。AngularJS 内置了很多常用的指令，也可以根据业务需求自定义指令。

### 二、指令的执行过程

1. 浏览器得到 HTML 字符串内容，解析得到 DOM 结构。
2. ng 引入，把 DOM 结构扔给 $compile 函数处理
3. 找出 DOM 结构中有变量占位符
4. 匹配找出 DOM 中包含的所有指令引用
5. 把指令关联到 DOM
6. 关联到 DOM 的多个指令按权重排列
7. 执行指令中的 compile 函数（改变 DOM 结构，返回 link 函数）
8. 得到的所有 link 函数组成一个列表作为 $compile 函数的返回
9. 执行 link 函数（连接模板的 scope）


### 三、内置指令

AngularJS 内置了很多常用的指令，下面简单介绍几个常用的指令。

1. `ng-app`

`ng-app` 指令初始化一个 AngularJS 应用程序。`ng-app` 指令用于告诉 AngularJS 应用当前这个元素是根元素。所有 AngularJS 应用都必须要要一个根元素。HTML 文档中只允许有一个 `ng-app` 指令，如果有多个 `ng-app` 指令，则只有第一个会被使用。

```html
<element ng-app="modulename">
...
 在 ng-app 根元素上的内容可以包含 AngularJS 代码
...
</element>
```

```javascript

<div ng-app="myApp" ng-controller="myCtrl">
    {{ firstName + " " + lastName }}
</div>

<script>
var app = angular.module("myApp", []);
app.controller("myCtrl", function($scope) {
    $scope.firstName = "John";
    $scope.lastName = "Doe";
});
</script>
```

2. `ng-init`

`ng-init` 指令初始化应用程序数据。

```html
<div ng-app="" ng-init="myText='Hello World!'">

<h1>{{myText}}</h1>
```

3. `ng-model`

`ng-model` 指令把元素值（比如输入域的值）绑定到应用程序。`<input>` ,  `<select>`,  `<textarea>`, 元素支持该指令。

```html
<element ng-model="name"></element>
```

```html
<div ng-app="myApp" ng-controller="myCtrl">
    <input ng-model="name">
</div>

<script>
var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope) {
    $scope.name = "John Doe";
});
</script>
```

4. `ng-bind`

`ng-bind` 指令告诉 AngularJS 使用给定的变量或表达式的值来替换 HTML 元素的内容。

如果给定的变量或表达式修改了，指定替换的 HTML 元素也会修改。所有的 HTML 元素都支持该指令。

```html
<element ng-bind="expression"></element>
```

或作为 CSS 类：

```html
<element class="ng-bind: expression"></element>
```

```html
<div ng-app="" ng-init="myText='Hello World!'">

<p ng-bind="myText"></p>

</div>
```

### 四、自定义指令

除了 AngularJS 内置的指令外，我们还可以创建自定义指令。你可以使用 `.directive`函数来添加自定义的指令。要调用自定义指令，HTML 元素上需要添加自定义指令名。

使用驼峰法来命名一个指令， `runoobDirective` ，但在使用它时需要以 `-` 分割，`runoob-directive` ：

```html
<body ng-app="myApp">

<runoob-directive></runoob-directive>

<script>
var app = angular.module("myApp", []);
app.directive("runoobDirective", function() {
    return {
        template : "<h1>自定义指令!</h1>"
    };
});
</script>

</body>
```

你可以通过以下方式来调用指令：

- 元素名：  `<runoob-directive></runoob-directive>`
- 属性：  `<div runoob-directive></div>`
- 类名：  `<div class="runoob-directive"></div>`
- 注释：  ` <!-- directive: runoob-directive -->`

#### 1.  指令的属性

- restrict（限制使用）

你可以限制你的指令只能通过特定的方式来调用。通过添加 `restrict`属性,并设置只值为 `A`, 来设置指令只能通过属性的方式来调用：

```javascript
var app = angular.module("myApp", []);
app.directive("runoobDirective", function() {
    return {
        restrict : "A",
        template : "<h1>自定义指令!</h1>"
    };
});
```

restrict 值可以是以下几种:

- `E` 作为元素名使用
- `A` 作为属性使用
- `C` 作为类名使用
- `M` 作为注释使用

restrict 默认值为 `EA`, 即可以通过元素名和属性名来调用指令。

**注意：从浏览器的兼容性方面考虑，建议使用A或者E。**

- template


模板内容，这个内容会根据replace参数的设置替换节点或只替换节点的内容。


- replace

  - true ：替换节点，会将指令的标签会替换掉
  - false（默认值）：则把当前指令追加到所在元素内部。

  对应 restrict 属性为元素时（E）在最终结果中是多余的，所有replace通常设置为 true。

- priority

设置指令在模版中的执行顺序，顺序是相对于元素上其他执行而言，默认为0，从大到小的顺序依次执行。


- terminal


是否以当前指令的权重为结束界限。如果这值设置为 true，则节点中权重小于当前指令的其它指令不会被执行。相同权重的会执行。如如果当前指令的 priority 值为 0， terminal 为true，那么其它priority 值小于0 的指令不会被执行。


- templateUrl


当模板没有定义在指令中时，可以通过templateUrl属性链接外部模板，模板不一定是一个完整的HTML文件。


- scope


指定指令的作用域。

1. false

   指令直接使用父作用域的属性和方法。

2. true

   指令要创建一个新的作用域，这个作用域继承自我们的父作用域（我们修改指令继承过来的属性时，父作用域中对应的属性值不会发生变化）。

3. {}

   创建的一个新的与父作用域隔离的新的作用域，这使我们在不知道外部环境的情况下，就可以正常工作，不依赖外部环境。


```javascript
var app = angular.module("MyApp", []);
    app.controller("MyController", function ($scope) {
        $scope.name = "dreamapple";
        $scope.age = 20;
        $scope.changeAge = function () {
            $scope.age = 0;
        }
    });
    app.directive("myDirective", function () {
        return {
            restrict: "AE",
            scope: {
                name: '@myName',
                age: '=',
                changeAge: '&changeMyAge'
            },
            replace: true,
            template:
            "<div class='my-directive'>" +
            "<h3>-------------------------------</h3>" +
            "我的名字是：<span ng-bind='name'></span>" +
            "<br/>" +
            "我的年龄是：<span ng-bind='age'></span>" +
            "<br/>" +
            "修改名字：<input type='text' ng-model='name'>" +
            "<br/>" +
            "<button ng-click='changeAge()'>修改年龄</button>" +
            "</div>"
        }
    });
```

```html
<div ng-app="MyApp">
    <div class="container" ng-controller="MyController">
        <div class="my-info">我的名字是：<span ng-bind="name"></span>

            <br/>我的年龄是：<span ng-bind="age"></span>
            <br/>
        </div>
        <div class="my-directive" my-directive my-name="{{name}}" age="age" change-my-age="changeAge()"></div>
    </div>
</div>
```

   执行结果：
   ![](/blog/images/201709/1.gif)

   我们使用了隔离的作用域，不代表我们不可以使用父作用域的属性和方法。我们可以通过向`scope`的`{}`中传入特殊的前缀标识符（即`prefix`）来进行数据的绑定，前缀标识符有如下几种：

   - `@`

   单项数据绑定标识符
   使用方法：在元素中使用属性，如：

```html
<div my-directive my-name="{{name}}"></div>
```

注意：属性的名字要用`-`将两个单词连接，因为是数据的单项绑定所以要通过使用双大括号来绑定数据。

   - `=`

   双向数据绑定标识符
   使用方法：在元素中使用属性，如：

```html
<div my-directive age="age"></div>
```

注意：数据的双向绑定要通过`=`前缀标识符实现，所以不可以使用双大括号。

   - `&`

   绑定函数方法的标识符
   使用方法：在元素中使用属性，如：

```html
<div my-directive change-my-age="changeAge()"></div>
```

注意：属性的名字要用`-`将多个个单词连接。

注意：在新创建指令的作用域对象中，使用属性的名字进行绑定时，要使用驼峰命名标准，比如下面的代码。

```javascript
scope: {
	// 'myName' 就是原来元素中的 'my-name' 属性
	    name: '@myName', 
	    age: '=',
	 // 'changeMyAge'就是原来元素中的'change-my-age'属性
	    changeAge: '&changeMyAge' 
	}
```

- controller
  他会暴露一个方法其他指令可以注入该方法或者指令（通过指令调用这个方法），利用这个API可以在多个指令之间通过依赖注入进行通信。


- controllerAs
  给controller方法起个别名，方便使用。


- transclude
  设置html页面中使用指令时指定的原始数据在指令内部是否使用，当该属性设置为true时，将指令中的内容渲染到 ng-transclude 指令指定的地方。

如下所示：

```javascript
var myApp = angular.module('myApp', [])
.directive('customTags', function () {
    return {
        restrict: 'ECAM',
        template:'<div>新数据 <span ng-transclude></span></div>',
        replace: true,
        transclude:false
    }
});
```
```html
<div ng-app="myApp">
    <custom-tags>原始数据</custom-tags>
</div>
```

渲染的结果：

```shell
新数据 原始数据
```


- require


填写当前指令需要依赖的其他指令的名称（或者指令名称的字符串数组），并将其控制器作为link函数的第四个参数传入。

| 选项             | 用法                                      |
| -------------- | --------------------------------------- |
| directiveName  | 通过驼峰法命名法指定指令名称，,Angular将会在自身所提供的指令查找控制器 |
| ^directiveName | 在上游的指令链中查找所指定的指令中的控制器                   |
| ?directiveName | 表示指令是可选的，如果当前指令中没有找到所需要的控制器，不需要抛出异常     |

#### 2. 指令编译

在angularJs应用启动之前，它们是以HTML文本形式存在文本编辑器当中。应用启动会进行编译和链接，作用域会同HTML进行绑定。

![](/blog/images/201709/2.png)

- 编译阶段（complie）： 

  在编译阶段，AngularJS会遍历整个HTML文档并根据JavaScript中的指令定义来处理页面上声明的指令。当AngularJS调用HTML文档根部的指令时，会遍历其中所有的模板，模板中也可能包含带有模板的指令，一旦对指令和其中的子模板进行遍历或编译，编译后的模板会返回一个叫做模板函数的函数。我们有机会在指令的模板函数被返回前，对编译后的DOM树进行修改。

  以下是AngularJS中complie函数的参数：

  `complie：function(tElement, tAttrs, transclude)`

  注意事项：

  - complie函数用来对模版自身进行转换，仅仅在编译阶段运行一次。
  - complie中直接返回的函数是postLink，表示link参数需要执行的函数，也可以返回一个对象里面包含preLink和postLink。
  - 当定义complie参数时，将无视link参数，因为complie里返回的就是该指令需要执行的link函数。

- 链接阶段（link）：

  链接函数来将模板与作用域链接起来;负责设置事件监听器，监视数据变化和实时的操作DOM.链接函数是可选的。如果定义了编译函数，它会返回链接函数，因此当两个函数都定义了时，编译函数会重载链接函数。

  以下是AngularJS中link函数的参数：

  `link(scope, iElement, iAttrs, controller)` 

  注意事项：

  - link参数代表的是complie返回的postLink。
  - preLink 表示在编译阶段之后，指令连接到子元素之前运行。
  - postLink 表示会在所有子元素指令都连接之后才运行。
  - link函数负责在模型和视图之间进行动态关联，对于每个指令的每个实例，link函数都会执行一次。

```javascript
    var myApp = angular.module('myApp', []);

    myApp.directive('customTags', function () {
        return {
            restrict: 'ECAM',
            template: '<div>{{user.name}}</div>',
            replace: true,
            compile: function (tElement, tAttrs, transclude) {

                tElement.append(angular.element('<div>{{user.name}}{{user.count}}</div>'));

                // 编译阶段...
                console.log('customTags compile 编译阶段...');

                return {
                    // 表示在编译阶段之后，指令连接到子元素之前运行
                    pre: function preLink(scope, iElement, iAttrs, controller) {
                        console.log('customTags preLink..')
                    },
                    // 表示在所有子元素指令都连接之后才运行
                    post: function postLink(scope, iElement, iAttrs, controller) {

                        iElement.on('click', function () {
                            scope.$apply(function () {
                                scope.user.name = 'click after';
                                scope.user.count = ++i;
                                // 进行一次 脏检查
                            });
                        })

                        console.log('customTags all child directive link..')
                    }
                }
                // 可以直接返回 postLink
                // return postLink function(){
                // console.log('compile return fun');
                //}
            },
            // 此link表示的就是 postLink
            link: function () {
//                iElement.on('click',function(){
//                    scope.$apply(function(){
//                        scope.user.name = 'click after';
//                        scope.user.count = ++i;
//                        // 进行一次 脏检查
//                    });
//                })
            }
        }
    });
```

```html
<div ng-app="myApp">
    <div ng-controller="firstController">
        <div ng-repeat="user in users" custom-tags="">
        </div>
    </div>
</div>

```

执行结果如下：

![](/blog/images/201709/3.png)

#### 3. 使用实例

以下通过AngularJS指令实现手风琴式的下拉菜单功能，代码如下：

```javascript
angular.module('myApp', [])
// 数据
    .factory('Data', function () {
        return [
            {
                title: 'no1',
                content: 'no1-content'
            },
            {
                title: 'no2',
                content: 'no2-content'
            },
            {
                title: 'no3',
                content: 'no3-content'
            }
        ];
    })
    // 控制器
    .controller('firstController', ['$scope', 'Data', function ($scope, Data) {
        $scope.data = Data;
    }])

    .directive('kittencupGroup', function () {
        return {
            restrict: 'E',
            replace: true,
            template: '<div class="panel-group" ng-transclude></div>',
            transclude: true,
            controllerAs: 'kittencupGroupContrller',
            controller: function () {
                this.groups = [];

                this.closeOtherCollapse = function (nowScope) {
                    angular.forEach(this.groups, function (scope) {
                        if (scope !== nowScope) {
                            scope.isOpen = false;
                        }
                    })
                }
            }
        }
    })

    .directive('kittencupCollapse', function () {
        return {
            restrict: 'E',
            replace: true,
            require: '^kittencupGroup',
            templateUrl: 'app/kittencupCollapse.html',
            scope: {
                heading: '@'
            },
            link: function (scope, element, attrs, kittencupGroupContrller) {
                scope.isOpen = false;

                scope.changeOpen = function () {
                    scope.isOpen = !scope.isOpen;

                    kittencupGroupContrller.closeOtherCollapse(scope);
                }


                kittencupGroupContrller.groups.push(scope);
            },
            transclude: true
        }
    });
```

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <link rel="stylesheet" href="../../vendor/bootstrap3/css/bootstrap.min.css"/>
</head>
<body>
<div ng-app="myApp">
    <div class="container">
        <div ng-controller="firstController">

            <kittencup-group>

                <kittencup-collapse ng-repeat="collapse in data" heading="{{collapse.title}}">
                    {{collapse.content}}
                </kittencup-collapse>

            </kittencup-group>

        </div>
    </div>
</div>
<script type="text/javascript" src="../../vendor/angular/angularjs.js"></script>
<script type="text/javascript" src="app/index.js"></script>
</body>
</html>
```

模板文件 `kittencupCollapse.html` 如下：

```html
<div class="panel panel-default">
    <div class="panel-heading" ng-click="changeOpen()">
        <h4 class="panel-title">
            <a href="#">
                {{heading}}
            </a>
        </h4>
    </div>
    <div class="panel-collapse" ng-class="{collapse:!isOpen}">
        <div class="panel-body" ng-transclude>
        </div>
    </div>
</div>
```

实现效果：

![](/blog/images/201709/2.gif)

### 五、参考博客

- 一招制敌 - 玩转 AngularJS 指令的 Scope (作用域) [https://segmentfault.com/a/1190000002773689](https://segmentfault.com/a/1190000002773689)
- kittencup 的AngularJS教学视频 [http://kittencup.com/](http://kittencup.com/)
- 菜鸟教程AngularJS教程指令章节 [http://www.runoob.com/angularjs/angularjs-directives.html](http://www.runoob.com/angularjs/angularjs-directives.html)