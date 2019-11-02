# 分布式追踪

## 概述

分布式追踪的概念源自最早面对超大规模分布式场景的 Google 公司于2010年发表的论文——[《Dapper, a Large-Scale Distributed Systems Tracing Infrastructure》](https://ai.google/research/pubs/pub36356)。
论文中详细地介绍了 Google 的 Dapper 系统实现及原理，Dapper 的核心实现方法是在分布式请求的上下文中假如 span id 以及 parent id , 用于记录请求的上下级关系，如下如所示：

![dapper-01.png](../images/dapper-01.png)

阿里巴巴公司曾经分享过鹰眼系统的实现方案，也是国内早期的分布式追踪系统实现方案，鹰眼系统使用类似书签索引，简化了 parent id 和 span id 的表达方式, 一个浏览器请求可能触发的系统间调用如下图所示：

![ali-edge.png](../images/ali-edge.png)

鹰眼系统与 Dapper 在原理上没有本质区别。之所以在这里提及，是因为鹰眼系统的编码方案可以帮助读者理解 Trace 树形结构。但是，需要强调的是，Trace 并非只有树形结构，对于消息消费、异步处理等批量模型，会出现一条 Trace 关联多个 TraceId 的情况。

## 目前常见的开源解决方案

Apache Zipkin 是起步最早、社区生态最为完备的分布式追踪系统解决方案。她借助了稳定的 API 库以及广泛的集成，几乎覆盖了从开源到商业级别的分布式系统的各个角落。
其覆盖的语言包括 Java、C#、Go、JavaScript、Ruby、Scala、C++、PHP、Elixir、Lua等，甚至为每种语言都提供了不止一种 API 库，更好地适应了各种不同的应用场景。