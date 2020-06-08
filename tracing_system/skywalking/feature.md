# Skywalking 功能介绍

本篇首先从 UI 使用层面上来介绍 Skywalkalking 的核心功能特性。

## 可配置的仪表盘

![skywalking-dashboard-01](http://cdn.jared-says.cn/skywalking-8-01.png)

仪表盘提供了针对全局（Global）、服务（Service）、端点（Endpoint）、实例（Instance）四个不同粒度的观测角度，方便我们从全局到具体的进行观测系统的健康并及时发现那些糟糕的信号量。

由于 Skywalking 提供了较多的指标数据，其中新版本中更是包括了从 Prometheus 接收的指标数据以及自身监控指标。这就导致之前的 UI 展示的指标信息可能并不是每个人都关心或者对用户更加直观的数据。

因此，在 8.0 版本中，新增了对仪表盘指标报表自定义展示，并支持保存自定义仪表盘为模版的个性化需求。

![skywalking-dashboard-template](http://cdn.jared-says.cn/WechatIMG154.jpeg)

## 新的拓扑关系

![](http://cdn.jared-says.cn/WX20200607-231520.png)

新的拓扑图中，除了支持我们为一组服务创建一个分组来帮我们梳理服务与服务、应用与应用的关系之外，还新增了直接从拓扑节点进行关联指标数据查询。省去了不同页面之间来回跳转的尴尬局面。

![](http://cdn.jared-says.cn/WX20200607-231826.png)

//TODO: client_server 指标

## Trace 展示多样化

链路展示与查询功能上，支持列表、树结构、表格三种不同的展示方式，满足了不同用户查看或者追踪链路的习惯。

![](http://cdn.jared-says.cn/WX20200607-232233.png)

## 性能剖析 Profile 

在系统性能监控方法上，新版 Skywalking 提出了代码级性能剖析这种在线诊断方法。这种方法基于一个高级语言编程模型共性，即使再复杂的系统，再复杂的业务逻辑，都是基于线程去进行执行的，而且多数逻辑是在单个线程状态下执行的。

代码级性能剖析就是利用方法栈快照，并对方法执行情况进行分析和汇总。并结合有限的分布式追踪 span 上下文，对代码执行速度进行估算。

![](http://cdn.jared-says.cn/WX20200607-232336.png)

> http://skywalking.apache.org/zh/blog/2020-03-23-using-profiling-to-fix-the-blind-spot-of-distributed-tracing.html

## 指标对比

![](http://cdn.jared-says.cn/WX20200607-232857.png)