---
title: chrome浏览器控制台性能监控
abbrlink: 20df0b93
date: 2020-02-28 16:43:45
tags: 性能优化
categories: 性能检测与优化
---
要实现对性能的优化，首先就要做到能合理有效地监控性能，发现哪里的性能出现问题，才能更好地去优化性能，这里我们说说如何通过chrome进行性能监控
<!-- more -->
# 1. 通过使用performance monitor来监控  
在控制台使用ctrl+shift+p打开command menu，输入performance monitor来监听

![](https://user-gold-cdn.xitu.io/2020/2/28/170879e92192e22b?w=726&h=118&f=png&s=11841)
可以监听如图的内容
![](https://user-gold-cdn.xitu.io/2020/2/28/170879f03e41404e?w=1443&h=447&f=png&s=59157)
这里的数据比较直观，也就不多说了

# 2. performance

performance是chrome控制台用来监控性能指标的一个利器

点击左上角的圆形按钮可以开始录制，按stop停止录制，就会把这段时间的性能监控反映出来，也可以通过刷新按钮重载页面，可以监控页面初始化时的性能  

![](https://user-gold-cdn.xitu.io/2020/2/28/1708a9f872f9e626?w=1605&h=977&f=png&s=144332)
拿到performance的性能报告图，这里我分成四个部分来说，首先看到第一部分
## 第一部分
![](https://user-gold-cdn.xitu.io/2020/2/28/1708a10d879704f1?w=1439&h=170&f=png&s=123338)
从上到下四个数据
* FPS：主要和动画性能有关，表示每秒的帧数。上图中出现了红色的长条，说明这部分动画性能有卡帧，需要进行优化，而像下图这样，绿色长条，就说明性能还可以，而绿色长条越高，说明性能越好  
![](https://user-gold-cdn.xitu.io/2020/2/28/1708a17f64a1d71d?w=422&h=98&f=png&s=24352)
* CPU：和底部的summary对应，显示了页面加载过程中，各阶段对CPU的占用时间，占用时间越多，表示该部分越需要被优化，其中不同的颜色就对应着不同的部分，相应部分对应的内容可以在底部的summary里面看到，不同颜色含义为  
![](https://user-gold-cdn.xitu.io/2020/2/28/1708a20ad8a838c6?w=163&h=204&f=png&s=4575)
    * 蓝色（Loading）：网络通信和html解析
    * 黄色（Scripting）：JavaScript执行
    * 紫色（Rendering）：样式计算和布局，对css的解析，即重排
    * 绿色（Painting）：重绘
    * 灰色（System）：其他事件花费的时间
    * 白色（Idle）：空闲时间
* NET：主要显示了请求的顺序和花费的时间，深蓝色的优先级比浅蓝色高，我们也可以在下面的network查看更具体的顺序和耗时  
![](https://user-gold-cdn.xitu.io/2020/2/28/1708a4ac612ab4c1?w=1311&h=380&f=png&s=106537)
点击network里面的内容在summary中会显示出具体的内容  
![](https://user-gold-cdn.xitu.io/2020/2/28/1708a4d2a6278cd6?w=644&h=363&f=png&s=33537)
* HEAP：表示js堆内存的消耗  
线的上升就表示这个时间点js执行消耗了一些内存
## 第二部分
我们刚刚说第一部分的内容时实际上也说到了下面三个部分的内容，接下来看看第二个部分里面内容最多的火焰图，main

![](https://user-gold-cdn.xitu.io/2020/2/28/1708a8c4ed8d050d?w=1567&h=468&f=png&s=75070)
火焰图的第一行表示发生的事件，下面的各行是上面事件的子项

当我们使用左上角的刷新去监控页面时，在这里会出现三条虚线，这里我们放大看看

![](https://user-gold-cdn.xitu.io/2020/2/28/1708a8e10dd57526?w=431&h=439&f=png&s=31690)
这三条虚线各有含义
* 蓝色：DOM加载完成
* 绿色：开始绘制页面
* 红色：页面加载完成
## 第三部分
看一下第三部分，这部分内容直观地显示了一些性能的详细信息，包括js堆内存，文档，节点，监听器和GPU

![](https://user-gold-cdn.xitu.io/2020/2/28/1708aa2dc620c679?w=1598&h=288&f=png&s=19476)
不同内容对应的颜色也已经在图中标明了，这里就不再多说

## 第四部分
接下来看看最后一部分
### Summary
我们看下图，这里是刚才已经说过了的CPU占用时间的总结，与上面不同的是，上面展示的是一个各个部分在不同时间的消耗，而这里采取的是一个扇形图的方式，可以让我们直观地看到各个部分CPU消耗的占比，我们也就能更好地找到需要优化的内容
![](https://user-gold-cdn.xitu.io/2020/2/28/1708aa8999b66c25?w=859&h=364&f=png&s=30909)

### Bottom-Up
这里显示的是各个部分具体的内存消耗对应的一些内容，根据时间消耗的长短，反向列出事件列表，
![](https://user-gold-cdn.xitu.io/2020/2/28/1708aaffe8db0ff6?w=1055&h=606&f=png&s=63637)
这里也可以通过filter和分组group来更精确地找到内容
![](https://user-gold-cdn.xitu.io/2020/2/28/1708aafbb9440c1a?w=388&h=57&f=png&s=2706)  
对于html的解析和css的解析我们可能做不了太多优化，更多地是看看时间消耗，把时间消耗多的那段解析减少，这里更多的是看看黄色的scripting的内容  
如下图，可以看到，这里的script可以通过细分，去准确地找到消耗这些CPU的是代码中的哪一行，方便我们进行一个性能上的优化
![](https://user-gold-cdn.xitu.io/2020/2/28/1708aacf3492ca78?w=1050&h=174&f=png&s=22881)

### Call Tree
以树形图的形式展示各项事件，这样事件调用跟踪会显得更为详细和直观，其他和bottom-up基本是一样的
![](https://user-gold-cdn.xitu.io/2020/2/28/1708ab22f3740e18?w=1073&h=447&f=png&s=48829)

### Event Log
事件日志，可以让我们更好地看到事件的详细信息，如图
![](https://user-gold-cdn.xitu.io/2020/2/28/1708ab3f5df0cf43?w=1071&h=619&f=png&s=66928)
比Bottom-Up和Call Tree多了右边的内容，更详细地展示事件信息