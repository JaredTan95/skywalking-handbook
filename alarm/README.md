# 告警模块

Skywalking 发送告警的基本原理是每隔一段时间轮询 skywalking-oap 收集到的链路追踪的数据，再根据所配置的告警规则（如服务响应时间、服务响应时间百分比）等，如果达到阈值则发送响应的告警信息。
发送告警信息是以线程池异步的方式调用 webhook 接口完成，（具体的webhook接口可以使用者自行定义），从而开发者可以在指定的webhook接口中自行编写各种告警方式，钉钉告警、邮件告警等等。

告警的核心由一组规则驱动，这些规则定义在 安装解压缩包里面的`config/alarm-settings.yml`文件中, 打开之后如下所示：rules 即为需要配置的告警规则的列表；第一个规则 'endpoint_percent_rule'，是规则名，不能重复且必须以 '_rule' 为结尾.

```yaml
rules:
  # Rule unique name, must be ended with `_rule`.（告警规则名称应该具有唯一性，且必须以 `_rule` 结尾。）
  endpoint_percent_rule:
    # Metrics value need to be long, double or int. （指标值需要是long，double 或者 int）
    metrics-name: endpoint_percent
    threshold: 75
    op: <
    # The length of time to evaluate the metrics（告警检查周期：多久检查一次当前的指标数据是否符合告警规则，单位是分钟）
    period: 10
    # How many times after the metrics match the condition, will trigger alarm（累计达到多少次告警值后触发告警）
    count: 3
    # How many times of checks, the alarm keeps silence after alarm triggered, default as same as period.（忽略相同告警 message 的周期，默认与告警检查周期一致）
    silence-period: 10
    
  service_percent_rule:
    metrics-name: service_percent
    # [Optional] Default, match all services in this metrics（可选项，默认匹配所有服务）
    include-names:
      - service_a
      - service_b
    threshold: 85
    op: <
    period: 10
    count: 4
```

告警规则的定义分为两部分。
1. 告警规则。它们定义了应该如何触发度量警报，应该考虑什么条件。
2. 网络钩子。当警告触发时，哪些服务终端需要被告知，你可以通过钩子接收告警信息，然后按需将告警信息推送到其他告警平台。

## 告警规则

告警规则主要有以下几点：
- **Rule name**。在告警信息中显示的唯一名称。必须以`_rule`结尾。指定的规则（与规则名不同，这里是对应的告警中的规则map，具体可查看 [backend-alarm.md](https://github.com/apache/skywalking/blob/master/docs/en/setup/backend/backend-alarm.md#list-of-all-potential-metrics-name)，其中一些常见的，endpoint_percent_rule——端点相应半分比告警，service_percent_rule——服务相应百分比告警）
- **Metrics name**。 也是 OAL 脚本中的度量名。只支持long,double和int类型。详情见[所有可能的度量名称列表](#所有可能的度量名称列表).
- **Include names**。使用本规则告警的服务列表。比如服务名，端点名。
- **Threshold**。阈值,与metrics-name和下面的比较符号相匹配
- **OP**。 操作符, 支持 `>`, `<`, `=`。欢迎贡献所有的操作符。如 metrics-name: endpoint_percent, threshold: 75，op: < ,表示如果相应时长小于平均75%则发送告警
- **Period**.。多久告警规则需要被核实一下。这是一个时间窗口，与后端部署环境时间相匹配。    
- **Count**。 在一个Period窗口中，如果**value**s超过Threshold值（按op），达到Count值，需要发送警报。
- **Silence period**。在时间N中触发报警后，在**TN -> TN + period**这个阶段不告警。 默认情况下，它和**Period**一样，这意味着相同的告警（在同一个Metrics name拥有相同的Id）在同一个Period内只会触发一次。


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
    # [Optional] Default, match all services in this metrics（可选项，默认匹配所有服务）
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
JSON 格式可以参考 `List<org.apache.skywalking.oap.server.core.alarm.AlarmMessage`, 包含以下信息.
- **scopeId**，**scope**. 所有可用的 Scope 请查阅 `org.apache.skywalking.oap.server.core.source.DefaultScopeDefine`.
- **name**. 目标 Scope 的实体名称.
- **id0**. Scope 实体的 ID.
- **id1**. 未使用.
- **ruleName**. 触发的告警规则名称.
- **alarmMessage**. 报警消息内容.
- **startTime**. 告警时间.

以下是一个 POST 方式推送出去的告警信息样例：
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

以下是一个可参考的告警配置：

```yml
# Sample alarm rules.
rules:
  # Rule unique name, must be ended with `_rule`.
  service_resp_time_rule:
    metrics-name: service_resp_time
    op: ">"
    threshold: 500
    period: 10
    count: 1
    silence-period: 5
    message: Response time of service {name} is more than 1000ms in last 10 minutes.
  service_sla_rule:
    # Indicator value need to be long, double or int
    metrics-name: service_sla
    op: "<"
    threshold: 8000
    # The length of time to evaluate the metric
    period: 10
    # How many times after the metric match the condition, will trigger alarm
    count: 2
    # How many times of checks, the alarm keeps silence after alarm triggered, default as same as period.
    silence-period: 3
    message: Successful rate of service {name} is lower than 80% in last 10 minutes.
  service_p90_sla_rule:
    # Indicator value need to be long, double or int
    metrics-name: service_p90
    op: ">"
    threshold: 500
    period: 10
    count: 1
    silence-period: 5
    message: 90% response time of service {name} is lower than 1000ms in last 10 minutes
  service_instance_resp_time_rule:
    metrics-name: service_instance_resp_time
    op: ">"
    threshold: 500
    period: 10
    count: 1
    silence-period: 5
    message: Response time of service instance {name} is more than 1000ms in last 10 minutes.
  endpoint_avg_rule:
    metrics-name: endpoint_avg
    op: ">"
    threshold: 500
    period: 10
    count: 1
    silence-period: 5
    message: Response time of endpoint {name} is more than 1000ms in last 10 minutes.

webhooks:
#  - http://127.0.0.1/notify/
#  - http://127.0.0.1/go-wechat/

```