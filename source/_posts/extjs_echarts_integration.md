---
title: ExtJs整合ECharts
date: 2017-08-23 21:27:25 
author: Zak
avatar: /blog/images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wuzguo
authorDesc: 一个自强不息，艰苦奋斗的「十八线码农」
categories: 前端
tags: 
	- ExtJs
	- ECharts
keywords: ECharts，ExtJs
photos:
	- /blog/images/201708_1/1.png
description: ExtJs整合ECharts，用ECharts的强大的图表展示各种报表
---

#### 一、背景

ECharts是百度开源的纯 Javascript 的图表库，可以流畅的运行在 PC 和移动设备上，兼容当前绝大部分浏览器。Ext JS主要用于跨浏览器功能构建Web应用程序提供了丰富的UI， 多用于企业级的管理系统和互联网后台管理系统。当我们在系统中展示各种图表的时候需要用到ECharts。

#### 二、集成

1. 首先定义一个Ext JS的组件，直接上代码。

```javascript
/**
 * 堆叠柱状图
 * Created by wuzguo on 2017/6/19
 */
Ext.ns('Ext.ux.chart');

Ext.ux.chart.stackedHistograms = Ext.extend(Ext.Component, {
    border: false,
    height: 100,//不能省略，虽然没用
    //修改/刷新数据
    update: function (exAxisData, eseries) {
        var me = this;
        // 清空数据
        me.exAxisdata.length = 0;
        me.eseries.length = 0;
        me.exAxisdata = exAxisData;
        me.eseries = eseries;
        me.echarts.clear();
        me.echeartOption(me);
    },
    //初始化box组件
    initComponent: function () {
        var me = this;
        if (!me.height) {
            throw new Error('图表组件需要设置高度!');
        }
        me.on("resize", function (me, mewidth, meheight) {
            me.getEl().dom.style.height = meheight + 'px';
            me.echarts = echarts.init(me.getEl().dom);
            me.echeartOption(me);
        });
    },
    //初始化echarts图表
    echeartOption: function (me) {

        var axisdata =
            ['0点', '1点', '2点',
                '3点', '4点', '5点',
                '6点', '7点', '8点',
                '9点', '10点', '11点',
                '12点', '13点', '14点',
                '15点', '16点', '17点',
                '18点', '19点', '20点',
                '21点', '22点', '23点'
            ];

        var seriesData = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0];

        var option = {
            title: {
                text: me.etext,
                subtext: me.esubtext
            },
            tooltip: {
                trigger: me.etrigger || 'axis',
                axisPointer: {            // 坐标轴指示器，坐标轴触发有效
                    type: me.eaxisPointer || 'line'  // 默认为直线，可选为：'line' | 'shadow'
                }
            },
            legend: {
                data: me.elegendata
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
                left: '3%',
                right: '4%',
                bottom: '3%',
                containLabel: true
            },
            xAxis: me.exAxis || [
                {
                    data: me.exAxisdata || axisdata,
                    type: me.exAxisType || 'category'
                }
            ],
            /*直角坐标系中纵轴数组*/
            yAxis: me.eyAxis || [
                {
                    type: 'value',
                    min: 'dataMin',
                    minInterval: 1
                }
            ],

            /*驱动图表生成的数据内容数组*/
            series: me.eseries || [
                {
                    name: 'PC',
                    type: 'bar',
                    stack: '注册',
                    barWidth: 60,
                    data: seriesData
                },
                {
                    name: '触屏',
                    type: 'bar',
                    stack: '注册',
                    data: seriesData
                },
                {
                    name: 'IOS',
                    type: 'bar',
                    stack: '注册',
                    data: seriesData
                },
                {
                    name: '微信',
                    type: 'bar',
                    stack: '注册',
                    data: seriesData
                },

                {
                    name: '安卓',
                    type: 'bar',
                    stack: '注册',
                    data: seriesData,
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

        // 刷新
        me.echarts.setOption(option);
    }
});

```

2. 创建显示面板，将上面定义的组件初始化进去。

```javascript
//图表显示面板
var stackedEchartspanel = new Ext.ux.chart.stackedHistograms({
    etext: '今日平台注册人数实时监控',
    esubtext: '实时数据',
    eaxisPointer: 'line',
    exAxisType: 'category',
    elegendata: ['PC', '触屏', '安卓', 'IOS', '微信'],
    exAxisdata: ['0点', '1点', '2点',
        '3点', '4点', '5点',
        '6点', '7点', '8点',
        '9点', '10点', '11点',
        '12点', '13点', '14点',
        '15点', '16点', '17点',
        '18点', '19点', '20点',
        '21点', '22点', '23点'
    ],
    eseriesname: '注册量（人）',
    eseries: [
        {
            name: 'PC',
            type: 'bar',
            stack: '注册',
            barWidth: 40,
            data: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
        },
        {
            name: '触屏',
            type: 'bar',
            stack: '注册',
            data: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
        },
        {
            name: 'IOS',
            type: 'bar',
            stack: '注册',
            data: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
        },
        {
            name: '微信',
            type: 'bar',
            stack: '注册',
            data: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
        },

        {
            name: '安卓',
            type: 'bar',
            stack: '注册',
            data: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
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
});
```

3. 创建一个容器，将上面的Panel放进去，并且调用后台接口定时更新数据。

```javascript
/**
 * Created by wuzguo on 2017/06/14 14:02:23
 * 日志监控 -> 注册人数监控
 */

Ext.onReady(function () {
    //主界面
    var mainPanel = Ext.create('Ext.panel.Panel', {
        width: '100%',
        height: '100%',
        layout: 'card',
        id: 'mainPanelId',
        bodyStyle: "background-color: white",
        border: false,
        autoScroll: true,
        plain: true,
        //  items: [echartspanel]
        items: [stackedEchartspanel]
    });

    // 容器
    var viewport = Ext.create('Ext.container.Viewport', {
        layout: 'fit',
        id: 'mainFrame',
        border: false,
        bodyStyle: "background-color: white",
        plain: true,
        autoScroll: true,
        items: [mainPanel]
    });


    var stackedTask = {
        run: function () {
            Ext.Ajax.request({
                url: basePath + '/admin/ucweb/statRegistUser',
                method: "GET",
                timeout: 30000,
                success: function (res) {
                    //    console.log("res: " + JSON.stringify(res));
                    if (res.status == 200) {
                        var result = res.responseText;
                        var exAxisdata = ['0点', '1点', '2点',
                            '3点', '4点', '5点',
                            '6点', '7点', '8点',
                            '9点', '10点', '11点',
                            '12点', '13点', '14点',
                            '15点', '16点', '17点',
                            '18点', '19点', '20点',
                            '21点', '22点', '23点'
                        ];

                        var eseriesdata = [];
                        var retJson = JSON.parse(result);
                        // console.log("retJson: " + retJson);
                        if (retJson.success && retJson.data) {
                            var rest = retJson.data;
                            // console.log("rest: " + rest);
                            for (var key in rest) {
                                // 将对象转换为数组
                                var indexData = rest[key];
                                // 给默认值，防止数据为空
                                var tempData = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0];
                                for (var k in indexData) {
                                    for (var i = 0; i < 24; i++) {
                                        if ((k + "点") == exAxisdata[i]) {
                                            // 由于数据量太小，这里加上随机数
                                            tempData[i] = indexData[k] + parseInt(Math.random() * 1000);
                                        }
                                    }
                                }

                                eseriesdata.push({
                                    name: key,
                                    type: 'bar',
                                    stack: '注册',
                                    barWidth: 40,
                                    data: tempData
                                });
                            }

                            // 最上的堆叠块显示最大值，最小值以及平均值
                            var index = eseriesdata.length - 1;
                            eseriesdata[index].markPoint = {
                                data: [
                                    {type: 'max', name: '最大值'},
                                    {type: 'min', name: '最小值'}
                                ]
                            };
                            eseriesdata[index].markLine = {
                                data: [
                                    {type: 'average', name: '平均值'}
                                ]
                            }

                            // console.log("eseriesdata: " + JSON.stringify(eseriesdata));
                            // 刷新表格中的数据
                            stackedEchartspanel.update(exAxisdata, eseriesdata);
                        }
                    }
                }
            });
        },

        // 刷新频率（10秒）
        interval: 10000
    }


    // 启动定时服务去后台查询数据
    Ext.TaskManager.start(stackedTask);
});

```

4. 以上就是关键代码，然后不管你是通过JSP还是HTML将Echarts的库文件引进来，然后就可以正常显示图表了。以下就是展示效果：

![](/blog/images/201708_1/1.png)

![](/blog/images/201708_1/2.png)