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

OpenTracing 是 CNCF 托管的分布式追踪项目，它的官方定位是针对分布式追踪的 API 标准库，与厂商无关，旨在为不同的分布式追踪系统提供统一的对外 API 接入层。 因此，OpenTracing 并不包含任何实现，可以将它理解为接口协议，类似于 Java 的数据库访问接口 JDBC。

需要强调的是，OpenTracing 只是一套可选的接口 API 库，并非分布式追踪的实现标准，其原生支持者同样是 CNCF 托管的项目——Jaeger。另外，前面提到的运用最为广泛的分布式追踪系统 Zipkin 并不是 OpenTracing 的主要支持，Zipkin 具有完全独立的规范、协议和 API 接口。所以到目前为止，还不能承认 OpenTracing 是分布式追踪领域的 API 标准。

OpenCensus 来自于 Google，是2017年才崭露头角的新兴项目。它的定位介于 OpenTracing 和 Zipkin 之间，提供了统一的 API 层，同时提供了部分实现逻辑。
工程师只需在 OpenTracing 的基础上自定义实现最小范围的逻辑（比如上下文传递和数据格式上行）即可。 OpenCensus 目前支持发送 Zipin、Jaeger、 Stackdriver 和 SignalFx 格式的数据。 OpenCensus 在 Google 的大力支持下，已经成为 OpenTracing 的有力竞争者。

TODO：
OpenTelementry