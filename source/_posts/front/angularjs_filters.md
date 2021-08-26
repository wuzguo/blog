---
title: AngularJS的过滤器

date: 2017-09-09 14:23:54

author: Zak

avatar: /blog/images/avatar.png

authorLink: http://www.wuzguo.com

authorAbout: https://github.com/wuzguo

authorDesc: 一个追求进步的「十八线码农」

categories: 前端

tags:
	- AngularJS
        - 过滤器

	- Filter

keywords: 过滤器，Filter，AngularJS

photos:
	- /blog/images/avatar.png

description: AngularJS的过滤器和自定义过滤器
---

### 一、定义

过滤器用于对数据的格式化，或者筛选的函数，可以直接在模板中通过一种语法使用：

```html
// 单个过滤器
{{ expression | filter }}
// 多个过滤器共同使用
{{ expression | filter1 | filter2 }}
// 过滤器中传递参数
{{ expression | filter1:param,….}}
```

### 二、内置的过滤器

AngularJS中常用的内置过滤器有如下：

| 滤器        | 描述            |
| --------- | ------------- |
| currency  | 格式化数字为货币格式。   |
| filter    | 从数组项中选择一个子集。  |
| lowercase | 格式化字符串为小写。    |
| orderBy   | 根据某个表达式排列数组。  |
| uppercase | 格式化字符串为大写。    |
| number    | 格式化数字。        |
| json      | 格式化json对象。    |
| limitTo   | 限制数组长度或字符串长度。 |
| date      | 格式化日期。        |

使用实例：

```javascript
angular.module('myApp', [])
    .factory('DefaultData', function () {
        return {
            message: 'Hello World',
            city: [
                {
                    name: '上海11212',
                    py: 'shanghai'
                },
                {
                    name: '北京',
                    py: 'beijing'
                },
                {
                    name: '四川',
                    py: 'sichuan'
                }
            ]
        };
    })
    .controller('firstController', ['$scope', 'DefaultData', '$filter', function ($scope, data, $filter) {
        $scope.today = new Date;
        $scope.data = data;

        // 过滤器
        var number = $filter('number')(3000);

        var strJson = $filter('json')($scope.data);
        console.log(strJson);
        console.log($scope.data);

        $scope.checkName = function (obj) {
            if (obj.py.indexOf('h') === -1)
                return false;
            return true;
        }
    }]);
```

```html
   <div ng-controller="firstController">
        <p>---------------------------------------------------------</p>
        <p>{{123456789 | number}}</p>
        <p>{{12345.6789 | number:3}}</p>
        <p>---------------------------------------------------------</p>
        <p>{{999999 | currency}}</p>
        <p>{{999999 | currency:'rmb'}}</p>
        <p>---------------------------------------------------------</p>
        <p>default:{{ today }}</p>
        <p>medium: {{ today | date:'medium'}}</p>
        <p>short:{{ today | date:'short'}}</p>
        <p>fullDate:{{ today | date:'fullDate'}}</p>
        <p>longDate:{{ today | date:'longDate'}}</p>
        <p>mediumDate:{{ today | date:'mediumDate'}}</p>
        <p>shortDate:{{ today | date:'shortDate'}}</p>
        <p>mediumTime:{{ today | date:'mediumTime'}}</p>
        <p>shortTime:{{ today | date:'shortTime'}}</p>
        <p>---------------------------------------------------------</p>
        <p>
            year:
            {{today | date : 'y'}}
            {{today | date : 'yy'}}
            {{today | date : 'yyyy'}}
        </p>
        <p>
            month:
            {{today | date : 'M'}}
            {{today | date : 'MM'}}
            {{today | date : 'MMM'}}
            {{today | date : 'MMMM'}}
        </p>
        <p>
            day:
            {{today | date : 'd'}}
            Day in month {{today | date : 'dd'}}
            Day in week {{today | date : 'EEEE'}}
            {{today | date : 'EEE'}}
        </p>

        <p>
            hour:
            {{today | date : 'HH'}}
            {{today | date : 'H'}}
            {{today | date : 'hh'}}
            {{today | date : 'h'}}
        </p>

        <p>
            minute:
            {{today | date : 'mm'}}
            {{today | date : 'm'}}
        </p>

        <p>
            second:
            {{today | date : 'ss'}}
            {{today | date : 's'}}
            {{today | date : '.sss'}}
        </p>
        <p>{{today | date : 'y-MM-d H:m:s'}}</p>
        <p>---------------------------------------------------------</p>
        <p>{{[1,2,3,4,5,6,7] | limitTo:5}}</p>
        <p>{{[1,2,3,4,5,6,7] | limitTo:-5}}</p>
        <p>---------------------------------------------------------</p>
        <p>{{data.message | lowercase}}</p>
        <p>{{data.message | uppercase}}</p>
        <p>---------------------------------------------------------</p>
        <p>{{ data.city | filter : '上海'}}</p>
        <p>{{ data.city | filter : 'name'}}</p>
        <p>{{ data.city | filter : {py:'g'} }}</p>
        <p>{{ data.city | filter : checkName }}</p>
        <p>---------------------------------------------------------</p>
        <!-- 默认顺序是 正序 asc a~z -->
        <p>{{ data.city | orderBy : 'py'}} </p>
        <!-- 默认顺序是 反序 desc z~a  -->
        <p>{{ data.city | orderBy : '-py'}}</p>

        <p>---------------------------------------------------------</p>
    </div>
```

执行结果：

```html
---------------------------------------------------------

123,456,789

12,345.679

---------------------------------------------------------

$999,999.00

rmb999,999.00

---------------------------------------------------------

default:"2017-09-09T05:51:07.808Z"

medium: Sep 9, 2017 1:51:07 PM

short:9/9/17 1:51 PM

fullDate:Saturday, September 9, 2017

longDate:September 9, 2017

mediumDate:Sep 9, 2017

shortDate:9/9/17

mediumTime:1:51:07 PM

shortTime:1:51 PM

---------------------------------------------------------

year: 2017 17 2017

month: 9 09 Sep September

day: 9 Day in month 09 Day in week Saturday Sat

hour: 13 13 01 1

minute: 51 51

second: 07 7 .808

2017-09-9 13:51:7

---------------------------------------------------------

[1,2,3,4,5]

[3,4,5,6,7]

---------------------------------------------------------

hello world

HELLO WORLD

---------------------------------------------------------

[{"name":"上海11212","py":"shanghai"}]

[]

[{"name":"上海11212","py":"shanghai"},{"name":"北京","py":"beijing"}]

[{"name":"上海11212","py":"shanghai"},{"name":"四川","py":"sichuan"}]

---------------------------------------------------------

[{"name":"北京","py":"beijing"},{"name":"上海11212","py":"shanghai"},{"name":"四川","py":"sichuan"}]

[{"name":"四川","py":"sichuan"},{"name":"上海11212","py":"shanghai"},{"name":"北京","py":"beijing"}]

---------------------------------------------------------
```

### 三、自定义过滤器

在AngularJS中可以通过以下格式自定义过滤器：（**过滤器必须返回一个Function**）

```javascript
module.filter(name,filterFactory)
```

或者

```javascript
$filterProvider.register(name,filterFactory);
```

如下通过不同的方式定义两个过滤器`filterCity` 和 `filterAge` ：

```javascript
var myApp = angular.module('myApp', [], function ($filterProvider) {
    $filterProvider.register('filterAge', function () {
        return function (obj) {
            var newObj = [];

            angular.forEach(obj, function (o) {
                if (o.age > 20) {
                    newObj.push(o);
                }
            });
            return newObj;
        }
    });
})
// module.filter
.filter('filterCity',function(){
    return function(obj){
        var newObj = [];
        angular.forEach(obj, function (o) {
            if (o.city === '上海') {
                newObj.push(o);
            }
        });

        return newObj;
    }
});
```

### 四、参考博客

- kittencup 的AngularJS教学视频 [http://kittencup.com/](http://kittencup.com/)