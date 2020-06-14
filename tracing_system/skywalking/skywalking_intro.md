# Apache Skywalking 介绍

Apache Skywalking(http://skywalking.apache.org) 定位于一款专为微服务、云原生架构和基于容器（Docker、K8s、Nesos）架构而设计的分布式系统应用程序性能监控工具。

SkyWalking 是观察性分析平台和应用性能管理系统。提供分布式追踪、服务网格遥测分析、度量聚合和可视化一体化解决方案。

Skywalking 从以大数据平台为主的分析模式，进化为轻量化、低延迟的分析模式，同时支持分布式追踪和非侵入式的语言探针这两种模式，是能够提供一致性解决方案的开源项目。

自[项目](https://github.com/apache/skywalking) 创建至今，大家会发现，Skywalking 在不停地进行大规模的重构、迭代和演进，强大的开源社区力量是项目能够得到良好发展的根基。

目前项目有两条主线分支：

- 5.x 版本分支：稳定版本，处于维护状态，主要用于修复版本Bug以及扩展语言探针插件。
- 6.x 版本分支：目前也处于稳定版本，目的是替代 5.x 版本成为主推的生产分支，提供了基于语言探针和 Service Mesh 探针的数据收集，具有统一可定制化的数据分析、数据展现能力。
- 8.x 版本分支：目前最新分支。基于 6.x 与 7.x 版本，去掉了探针注册、OAP 缓存逻辑，完全不兼容以前的版本。支持 Prometheus 指标并提供了 UI 可修改功能。同时增加了 Python 探针。在 Service Mesh 场景有更多的提升。

## Skywalking 可观察性分析平台

Skywalking 不局限于以往的基于多语言探针的思想，它面向多数据源，可提供统一、高效、可定制化的可观察性分析平台解决方案，如下图所示：

![Skywalking  可观察性分析平台](http://cdn.jared-says.cn/687474703a2f2f736b7977616c6b696e672e6170616368652e6f72672f6173736574732f6672616d652d76382e6a70673f753d3230323030343233.jpeg)

可观察性分析平台（Observability Analysis Platform, OAP） 是 Skywalking 社区 PMC 团队提出的新概念，OAP 将可观察性分为两个维度和三个层面：

### 两个维度

- Tracing 追踪链路数据
- Metrics 指标数据

### 三个层次

- Receiver 接收器，针对不同协议提供解析和适配能力。Skywalking 除了接收原生的 Trace、Metrics 数据，同时兼容了 Zipkin、Jaeger 的 Trace 数据。8.x 支持了 Prometheus 指标的接收。
- Analysis Core 分析内核，提供面向 Source 分析源的流式分析方法，并派生出 OAL(Observability Analysis Language，可观察性分析语言) 来描述流式分析。支持文件配置级别的指标自定义。
- Query Core 查询内核，基于拓扑结构、基础数据和指标数据等多个维度提供基于 GraphQL 的查询，为页面和第三方系统集成提供支持。

### Skywalking 生态

除了以上提到的内容之外，Skywalking 活跃的社区为 Skywalking 提供了丰富的生态产品，其中包括辅助运维部署与监控的 GUI/CLI 、Skywalking Kubernetes 部署方案等等。

