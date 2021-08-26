---
title: AngularJS 整合 ECharts
date: 2017-08-23 21:58:12 
author: Zak
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wuzguo
authorDesc: 一个追求进步的「十八线码农」
categories: 前端
tags: 
	- AngularJS
	- ECharts
keywords: ECharts，AngularJS
photos:
	- /blog/images/201708_1/1.png
description: AngularJS整合ECharts，用ECharts的强大的图表展示各种报表
---


#### 一、背景
AngularJS 是Google开发的前端技术框架，为了弥补了HTML在构建应用方面的不足，构建单页面应用程序（Single Page Application）非常有优势。

#### 二、整合

1. 定义AngularJS 的指令（`directive`）

```javascript
'use strict';

define([
    'workspace/workspace.module'
], function (workspaceModule) {

    // 定义指令
    workspaceModule.directive('stackedEchart', function () {
        //初始化echarts图表
        var echeartOption = function (me) {
            var axisData = [['0点', 0, 0, 0, 0, 0], ['1点', 0, 0, 0, 0, 0], ['2点', 0, 0, 0, 0, 0],
                ['3点', 0, 0, 0, 0, 0], ['4点', 0, 0, 0, 0, 0], ['5点', 0, 0, 0, 0, 0],
                ['6点', 0, 0, 0, 0, 0], ['7点', 0, 0, 0, 0, 0], ['8点', 0, 0, 0, 0, 0],
                ['9点', 0, 0, 0, 0, 0], ['10点', 0, 0, 0, 0, 0], ['11点', 0, 0, 0, 0, 0],
                ['12点', 0, 0, 0, 0, 0], ['13点', 0, 0, 0, 0, 0], ['14点', 0, 0, 0, 0, 0],
                ['15点', 0, 0, 0, 0, 0], ['16点', 0, 0, 0, 0, 0], ['17点', 0, 0, 0, 0, 0],
                ['18点', 0, 0, 0, 0, 0], ['19点', 0, 0, 0, 0, 0], ['20点', 0, 0, 0, 0, 0],
                ['21点', 0, 0, 0, 0, 0], ['22点', 0, 0, 0, 0, 0], ['23点', 0, 0, 0, 0, 0]];

            var stackedOption = {
                /*标题*/
                title: {
                    text: me.etext || '平台注册用户量数据统计',
                    subtext: me.esubtext || '实时数据',
                    x: 'left'
                },

                /*提示框，鼠标悬浮交互时的信息提示*/
                color: ['#3398DB'],
                tooltip: {
                    trigger: me.etrigger || 'axis',
                    axisPointer: { // 坐标轴指示器，坐标轴触发有效
                        type: 'shadow' // 默认为直线，可选为：'line' | 'shadow'
                    }
                },
                legend: {
                    data: me.elegendata || ['注册量（人）']
                },
                /*工具箱，每个图表最多仅有一个工具箱*/
                toolbox: {
                    show: true,
                    feature: me.efeature || {
                        mark: {
                            show: false
                        },
                        dataView: {
                            show: false,
                            readOnly: false
                        },
                        magicType: {
                            show: true,
                            type: ['line', 'bar']
                        },
                        restore: {
                            show: false
                        }
                    }
                },
                // 坐标网格设置
                grid: {
                    left: '1%',
                    right: '3%',
                    bottom: '2%',
                    containLabel: true
                },
                xAxis: me.exAxis || [
                    {
                        data: (me.exAxisData || axisData).map(function (item) {
                            return item[0];
                        }),

                        type: me.exAxisType || 'category'
                    }
                ],
                /*直角坐标系中纵轴数组*/
                yAxis: me.eyAxis || [
                    {
                        type: 'value',
                        min: 0,
                        minInterval: 1
                    }
                ],

                /*驱动图表生成的数据内容数组*/
                series: me.eseries || [
                    {
                        name: 'PC',
                        type: 'bar',
                        stack: '注册',
                        barWidth: 40,
                        data: (me.exAxisData || axisData ).map(function (item) {
                            return item[1];
                        })
                    },
                    {
                        name: '触屏',
                        type: 'bar',
                        stack: '注册',
                        data: (me.exAxisData || axisData ).map(function (item) {
                            return item[2];
                        })
                    },
                    {
                        name: 'IOS',
                        type: 'bar',
                        stack: '注册',
                        data: (me.exAxisData || axisData ).map(function (item) {
                            return item[3];
                        })
                    },
                    {
                        name: '微信',
                        type: 'bar',
                        stack: '注册',
                        data: (me.exAxisData || axisData ).map(function (item) {
                            return item[4];
                        })
                    },

                    {
                        name: '安卓',
                        type: 'bar',
                        stack: '注册',
                        data: (me.exAxisData || axisData ).map(function (item) {
                            return item[5];
                        }),
                        markPoint: {
                            data: [
                                {type: 'max', name: '最大值'},
                                {type: 'min', name: '最小值'}
                            ]
                        },
                        markLine: {
                            data: [
                                {type: 'average', name: '平均值'}
                            ]
                        }
                    }
                ]
            };

            // 重置图表
            me.echarts.clear();
            me.echarts.setOption(stackedOption);
        };

        return {
            restrict: 'AE',

            replace: false,

            // 定义指令和模板中使用的作用域的关系：
            // false: 复用组件具体使用位置所在的作用域
            // true: 创建子作用域，该作用域继承组件具体使用位置所在的作用域
            // 对象：创建独立作用域，该作用域没有继承原型：插入（@）、数据绑定（=）、表达式（&），在指令定义中会以名-值对的方式来定义这些接口：
            // scope: {
            //     isolated1: '@attribute1',
            //     isolated2: '=attribute2',
            //     isolated3: '&attribute3',
            // }
            // 如果在定义字段值时省略了属性名称，那么就表示该映射属性与对应的独立作用域字段名称完全一致
            scope: {
                source: '='
            },
            template: '<div>柱状图</div>',

            controller: ['$scope', '$element', '$attrs', function ($scope, $element, $attrs) {
                // 指令间数据共享的时候用到
            }],
            // 其他指令需要调用这个指令的时候需要用到 controller 的名称
            controllerAs: 'stackedEchartController',

            compile: function ($scope, element, attr) {
                // 获取图表的渲染位置
                var stacked = document.getElementById('echarts-stacked');
                // 初始化
                var me = me || {};
                // 初始化图表
                me.echarts = echarts.init(stacked);

                // 直接返回 link 函数
                return function ($scope, element, attr) {
                    $scope.$watch(
                        function ($scope) {
                            return $scope.source;
                        },
                        function () {
                            var data = $scope.source;
                            if (data && angular.isArray(data)) {
                                var self = me;
                                self.exAxisData = data;
                                echeartOption(self);
                            }
                        }, true);

                    $scope.resizeDom = function () {
                        return {
                            'height': stacked.offsetHeight,
                            'width': stacked.offsetWidth
                        };
                    };
                    $scope.$watch($scope.resizeDom, function () {
                        me.echarts.resize();
                    }, true);

                    // 渲染图表
                    echeartOption(me);
                }
            }
        };
    });
});

```

2. 定义AngularJS 的组件（`component`）

```javascript
'use strict';

define([
    'workspace/workspace.module',
    'text!echarts/stacked/stacked-histograms-template.html',
    'echarts/common/http-common-service',
    'echarts/stacked/stacked-histograms-directive',
    'css!echarts/stacked/stacked-histograms.css'
], function (workspaceModule, echartsHistogramsTemplate) {

    workspaceModule.component('stackedHistogramsComp', {
        template: echartsHistogramsTemplate,
        controller: function ($scope) {

        }
    });

    // 控制器
    workspaceModule.controller('stackedHistogramsCtrl',
        ['$scope', '$interval', '$timeout', 'httpCommonService', function ($scope, $interval, $timeout, httpCommonService) {
            // 获取数据的函数
            var registUser = function () {
                // 请求服务端数据
                httpCommonService.httpRequest("/echart/statRegistUser", 3000).then(function (res) {
                    if (res.success && res.data) {
                        // 给默认值，避免画图时候报错
                        var exAxisdata = [['0点', 0, 0, 0, 0, 0], ['1点', 0, 0, 0, 0, 0], ['2点', 0, 0, 0, 0, 0],
                            ['3点', 0, 0, 0, 0, 0], ['4点', 0, 0, 0, 0, 0], ['5点', 0, 0, 0, 0, 0],
                            ['6点', 0, 0, 0, 0, 0], ['7点', 0, 0, 0, 0, 0], ['8点', 0, 0, 0, 0, 0],
                            ['9点', 0, 0, 0, 0, 0], ['10点', 0, 0, 0, 0, 0], ['11点', 0, 0, 0, 0, 0],
                            ['12点', 0, 0, 0, 0, 0], ['13点', 0, 0, 0, 0, 0], ['14点', 0, 0, 0, 0, 0],
                            ['15点', 0, 0, 0, 0, 0], ['16点', 0, 0, 0, 0, 0], ['17点', 0, 0, 0, 0, 0],
                            ['18点', 0, 0, 0, 0, 0], ['19点', 0, 0, 0, 0, 0], ['20点', 0, 0, 0, 0, 0],
                            ['21点', 0, 0, 0, 0, 0], ['22点', 0, 0, 0, 0, 0], ['23点', 0, 0, 0, 0, 0]];
                        // 字典，这里要跟图表中的各个设备的展示顺序一致，否则数据将不正确
                        var indexDev = {'PC': 1, '触屏': 2, 'IOS': 3, '微信': 4, '安卓': 5};
                        var retJson = res.data;
                        // console.log("retJson: " + JSON.stringify(retJson));
                        for (var key in retJson) {
                            // 将对象转换为数组
                            var indexData = retJson[key];
                            for (var k in indexData) {
                                for (var i = 0; i < 24; i++) {
                                    if ((k + "点") == exAxisdata[i][0]) {
                                        // 由于数据量太小，这里加上随机数
                                        exAxisdata[k][indexDev[key]] = indexData[k] + parseInt(Math.random() * 1000);
                                    }
                                }
                            }
                        }

                        // console.log(JSON.stringify("exAxisdata: " + exAxisdata));
                        // 更新数据会触发指令的监听事件
                        $scope.dataGroup = exAxisdata;
                    }
                });
            };

            registUser();
            // 每个十秒请求一次
            $interval(function () {
                registUser();
            }, 10 * 1000);
        }]);
});

```

3. 定义AngularJS 的服务（service）

```javascript
'use strict';

define([
    'workspace/workspace.module'
], function (workspaceModule) {
    // 这里不要注入 $scope，否则会报错: [$injector:unpr] Unknown provider: $scopeProvider <- $scope <- httpCommonService
    workspaceModule.service('httpCommonService', ['$http', function ($http) {
        var doRequest = function (url, timeout) {
            var req = {
                method: 'GET',
                url: url,
                cache: false,
                timeout: timeout,
                headers: {
                    'Content-Type': 'application/json'
                }
            }

            return $http(req).then(function (response) {
                //  console.log("success: " + JSON.stringify(response));
                return response.data;
            }, function (response) {
                console.log("error: " + JSON.stringify(response));
                return response.data;
            });
        }
        return {
            httpRequest: function (url, timeout) {
                return doRequest(url, timeout);
            }
        };
    }]);
});


```

4. 在对应页面中添加指令

```html
<div class="echarts-mng-container">
    <div class="row">
        <div class="col-sm-12" ng-controller="stackedHistogramsCtrl">
            <div id="echarts-stacked" stacked-echart data-source='dataGroup' style="width: 100%; height:856px;"></div>
        </div>
    </div>
</div>
```

5. 运行效果

![](/blog/images/201708_1/1.png)

![](/blog/images/201708_1/2.png)



#### 三、源码

请移步到我的GitHUb，地址：[https://github.com/wuzguo/angular-echarts-examples](https://github.com/wuzguo/angular-echarts-examples) 。