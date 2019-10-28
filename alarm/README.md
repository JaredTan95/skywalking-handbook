# 告警模块

告警的核心由一组规则驱动，这些规则定义在 安装解压缩包里面的`config/alarm-settings.yml`文件中。

告警规则的定义分为两部分。
1. 告警规则。它们定义了应该如何触发度量警报，应该考虑什么条件。
2. [网络钩子](#Webhook}。当警告触发时，哪些服务终端需要被告知。

## 告警规则

告警规则主要有以下几点：
- **Rule name**。在告警信息中显示的唯一名称。必须以`_rule`结尾。
- **Metrics name**。 也是 OAL 脚本中的度量名。只支持long,double和int类型。详情见
[所有可能的度量名称列表](#所有可能的度量名称列表).
- **Include names**。其下的实体名称都在此规则中。比如服务名，终端名。
- **Threshold**。阈值。
- **OP**。 操作符, 支持 `>`, `<`, `=`。欢迎贡献所有的操作符。
- **Period**.。多久告警规则需要被核实一下。这是一个时间窗口，与后端部署环境时间相匹配。    
- **Count**。 在一个Period窗口中，如果**value**s超过Threshold值（按op），达到Count值，需要发送警报。
- **Silence period**。在时间N中触发报警后，在**TN -> TN + period**这个阶段不告警。 默认情况下，它和**Period**一样，这意味着相同的告警（在同一个Metrics name拥有相同的Id）在同一个Period内只会触发一次。

示例配置如下：


```yaml
rules:
  # Rule unique name, must be ended with `_rule`.
  endpoint_percent_rule:
    # Metrics value need to be long, double or int
    metrics-name: endpoint_percent
    threshold: 75
    op: <
    # The length of time to evaluate the metrics
    period: 10
    # How many times after the metrics match the condition, will trigger alarm
    count: 3
    # How many times of checks, the alarm keeps silence after alarm triggered, default as same as period.
    silence-period: 10
    
  service_percent_rule:
    metrics-name: service_percent
    # [Optional] Default, match all services in this metrics
    include-names:
      - service_a
      - service_b
    threshold: 85
    op: <
    period: 10
    count: 4
```

## 默认告警规则
为了方便，我们在发行版中提供了默认的`alarm setting.yml`文件，包括以下规则
1.过去3分钟内服务平均响应时间超过1秒。
1.服务成功率在过去2分钟内低于80%。
1.服务90%响应时间在过去3分钟内低于1000毫秒.
1.服务实例在过去2分钟内的平均响应时间超过1秒。
1.端点平均响应时间过去2分钟超过1秒。

## 所有可能的度量名称列表
这些度量名称定义在 [OAL 脚本](https://github.com/apache/skywalking/blob/master/oap-server/server-starter/src/main/resources/official_analysis.oal)中, 现在，来自**Service**, **Service Instance**, **Endpoint**的度量可以用于告警，我们会在后期版本中进行扩展。

## Webhook
SkyWalking 的告警 Webhook 要求接收方是一个 Web 容器。 需要在部署启动的时候在 `config/alarm-settings.yml`文件中配置接受请求的URL：

```yaml
# Sample alarm rules.
rules:
  service_resp_time_rule:
    metrics-name: service_resp_time
    # [Optional] Default, match all services in this metrics
    include-names:
      - dubbox-provider
      - dubbox-consumer
    threshold: 1000
    op: ">"
    period: 10
    count: 1

👇 配置webhook，接收告警规则的URL。
webhooks: 
  - http://127.0.0.1:8090/notify/
  - http://127.0.0.1:8888/go-wechat/
```

告警的消息会通过 HTTP 请求进行发送, 请求方法为 `POST`, `Content-Type` 为 `application/json`,
JSON 格式基于 `List<org.apache.skywalking.oap.server.core.alarm.AlarmMessage`, 包含以下信息.
- **scopeId**，**scope**. 所有可用的 Scope 请查阅 `org.apache.skywalking.oap.server.core.source.DefaultScopeDefine`.
- **name**. 目标 Scope 的实体名称.
- **id0**. Scope 实体的 ID.
- **id1**. 未使用.
- **ruleName**. 触发的告警规则名称.
- **alarmMessage**. 报警消息内容.
- **startTime**. 告警时间.

以下是一个样例
```json
[{
	"scopeId": 1, 
        "scope": "SERVICE",
        "name": "serviceA", 
	"id0": 12,  
	"id1": 0,  
        "ruleName": "service_resp_time_rule",
	"alarmMessage": "alarmMessage xxxx",
	"startTime": 1560524171000
}, {
	"scopeId": 1,
        "scope": "SERVICE",
        "name": "serviceB",
	"id0": 23,
	"id1": 0,
        "ruleName": "service_resp_time_rule",
	"alarmMessage": "alarmMessage yyy",
	"startTime": 1560524171000
}]
```
