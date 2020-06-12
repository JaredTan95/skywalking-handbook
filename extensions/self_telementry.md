# Skywalking 自监控

为了能够接收 Prometheus 相关的指标数据，Skywalking 新增了 Prometheus Fetcher 的插件。
同时，基于该插件，Skywalking 也可以将自身 Skywalking OAP Server 的指标接收并进行监控。
其中包括了 CPU、内存、GC 消耗时间、数据持久化的延时以及延时热力图等指标。

8.x 版本中，当我们部署完 Skywalking OAP Server 之后，Skywalking UI 提供了默认的指标监控面板。

![](http://cdn.jared-says.cn/WechatIMG157.jpeg)

我们可以对默认的面板进行调整，以此满足对 Skywalking 自监控的需求：

//TODO： 面板调整

Skywalking 默认提供了常见的监控指标，如果我们还需要其他更多的指标信息，可以在 `oap-server/server-bootstrap/src/main/resources/fetcher-prom-rules/self.yml`中进行自定义配置：

```yml
......
fetcherInterval: PT15S
fetcherTimeout: PT10S
metricsPath: /metrics
staticConfig:
  # targets will be labeled as "instance"
  targets:
    - localhost:1234
  labels:
    service: oap-server
metricsRules:
  - name: instance_cpu_percentage
    scope: SERVICE_INSTANCE
    operation: avg
    sources:
      process_cpu_seconds_total:
        counterFunction: RATE
        range: PT1M
        scale: 2
        relabel:
          service:
            - service
          instance:
            - instance
  - name: instance_jvm_memory_bytes_used
    scope: SERVICE_INSTANCE
    operation: avg
    sources:
      jvm_memory_bytes_used:
        relabel:
          service:
            - service
          instance:
            - instance
......
```