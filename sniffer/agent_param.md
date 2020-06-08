# 探针参数配置
以下内容详细解释了`agent/config/agent.config`配置文件中的参数：

配置Key | 描述 | 默认值 | 例如 |
----------- | ---------- | --------- | --------- |
`agent.namespace` | **默认设置为空字符`""`**。命名空间，可通过此参数实现隔离。具体效果则是共用同一套Es，并在索引Index前面带上前缀。命名空间隔离跨进程传播中的标头。`HEADER`名称将组合成： `HeaderName:Namespace`. | Not set | `agent.namespace=Team-A` |
`agent.service_name` | 在 DMP 链路追踪 UI 中展示的应用名，与 Eureka 中注册的应用名一致（在接入了 Eureka 的情况下）。 | `Your_ApplicationName` |`agent.application_code=Demo-App` |
`agent.sample_n_per_3_secs`|**默认全部**。每3秒采样的数量。设置为负数代表全样采取。|Not set|`agent.sample_n_per_3_secs=-1`|
`agent.authentication`|**默认未设置**。认证token，与后端application.yml中的设置保持一致。|Not set| `agent.authentication = dangrous` |
`agent.span_limit_per_segment`|**默认未设置**。在单个segment中最大的span数量，通过此设置，可以节省应用内存成本。|Not set |`agent.span_limit_per_segment=300`|
`agent.ignore_suffix`|**默认未设置**。忽略追踪过程中包含了此前缀的segment。|Not set|`agent.ignore_suffix=.jpg,.jpeg,.js,.css,.png,.bmp,.gif,.ico,.mp3,.mp4,.html,.svg`|
`agent.is_open_debugging_class`|`默认为false`。如果设置为`true `的话，探针会在 `/debugging` 目录中生成所有被增强的class文件。|Not set|`agent.is_open_debugging_class = true`|
`agent.active_v2_header`|**默认激活**`V2`版本。|`true`||
`agent.active_v1_header `|**默认关闭**`V1`版本。|`false`||
`collector.grpc_channel_check_interval`|grpc通道状态检测间隔时间。|`30`|`collector.grpc_channel_check_interval=40`|
`collector.backend_service`|默认为127.0.0.1:10800。后端Collector收集器的地址，通过逗号分割集群地址。|`127.0.0.1:11800`|
`logging.level`|`默认为DEBUG`。探针日志级别设置。|`DEBUG`|`INFO`|
`logging.file_name`|日志文件名称。|`skywalking-api.log`|`logging.file_name=skywalking-collector.log`|
`logging.dir`|日志文件目录名称。默认使用`system.out`输出日志。|`""`|
`logging.max_file_size`|日志文件大小最大值。如果日志超出此阈值，将会切分到新的日志文件中。|`300 * 1024 * 1024`|
`jvm.buffer_size`|收集Jvm信息的缓存大小。|`60 * 10`|
`buffer.channel_size`|Channel缓冲大小。|`5`|
`buffer.buffer_size`|缓冲大小。|`300`|
`plugin.elasticsearch.trace_dsl`|`**默认关闭**`,如果打开，将追踪Elasticsearch中的DSL语句。|`false`|