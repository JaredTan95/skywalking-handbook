# 脚本部署

首先需要从官方下载压缩包并解压，不同版本下载地址：http://skywalking.apache.org/downloads/ 或者 http://skywalking.apache.org/downloads/

此处以 **6.4.0** 为例:

⾸先点击右边地址下载 Skywalking 安装包: https://www-us.apache.org/dist/skywalking/6.4.0/apache-skywalking-apm-6.4.0.tar.gz ;

或者在Linux系统中执⾏行行如下命令:

- 下载压缩包

```bash
$ wget https://www-us.apache.org/dist/skywalking/6.4.0/apache-skywalking-apm-6.4.0.tar.gz
```

- 解压缩之后

```bash
$ tar -zxvf apache-skywalking-apm-6.4.0.tar.gz

$ ls apache-skywalking-apm-bin

LICENSE    README.txt bin        licenses   webapp
NOTICE     agent      config     oap-libs
```

## 配置文件预览

在正式部署之前，建议你大致浏览一下 `config` 目录下的 `application.yml` 文件。该文件涉及 Skywalking 部署模式的选择，

- 比如 **单机** 还是**集群**模式部署，以下则是单机模式：

```yml
······
cluster:
  standalone:
  # Please check your ZooKeeper is 3.5+, However, it is also compatible with ZooKeeper 3.4.x. Replace the ZooKeeper 3.5+
  # library the oap-libs folder with your ZooKeeper 3.4.x library.
#  zookeeper:
#    nameSpace: ${SW_NAMESPACE:""}
#    hostPort: ${SW_CLUSTER_ZK_HOST_PORT:localhost:2181}
#    #Retry Policy
#    baseSleepTimeMs: ${SW_CLUSTER_ZK_SLEEP_TIME:1000} # initial amount of time to wait between retries
#    maxRetries: ${SW_CLUSTER_ZK_MAX_RETRIES:3} # max number of times to retry
#    # Enable ACL
#    enableACL: ${SW_ZK_ENABLE_ACL:false} # disable ACL in default
#    schema: ${SW_ZK_SCHEMA:digest} # only support digest schema
#    expression: ${SW_ZK_EXPRESSION:skywalking:skywalking}
#  kubernetes:
#    watchTimeoutSeconds: ${SW_CLUSTER_K8S_WATCH_TIMEOUT:60}
#    namespace: ${SW_CLUSTER_K8S_NAMESPACE:default}
#    labelSelector: ${SW_CLUSTER_K8S_LABEL:app=collector,release=skywalking}
#    uidEnvName: ${SW_CLUSTER_K8S_UID:SKYWALKING_COLLECTOR_UID}
#  consul:
#    serviceName: ${SW_SERVICE_NAME:"SkyWalking_OAP_Cluster"}
#     Consul cluster nodes, example: 10.0.0.1:8500,10.0.0.2:8500,10.0.0.3:8500
#    hostPort: ${SW_CLUSTER_CONSUL_HOST_PORT:localhost:8500}
······
```

- 以及链路追踪、指标数据存储的配置，默认是H2存储，不过这只适合在做演示的适合使用。因此你需要选择其他存储，推荐Elasticsearch集群，就像如下配置：

```yml
storage:
  elasticsearch:
    nameSpace: ${SW_NAMESPACE:""}
    clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:localhost:9200}
    protocol: ${SW_STORAGE_ES_HTTP_PROTOCOL:"http"}
    trustStorePath: ${SW_SW_STORAGE_ES_SSL_JKS_PATH:"../es_keystore.jks"}
    trustStorePass: ${SW_SW_STORAGE_ES_SSL_JKS_PASS:""}
    user: ${SW_ES_USER:""}
    password: ${SW_ES_PASSWORD:""}
    indexShardsNumber: ${SW_STORAGE_ES_INDEX_SHARDS_NUMBER:2}
    indexReplicasNumber: ${SW_STORAGE_ES_INDEX_REPLICAS_NUMBER:0}
    # Those data TTL settings will override the same settings in core module.
    recordDataTTL: ${SW_STORAGE_ES_RECORD_DATA_TTL:7} # Unit is day
    otherMetricsDataTTL: ${SW_STORAGE_ES_OTHER_METRIC_DATA_TTL:45} # Unit is day
    monthMetricsDataTTL: ${SW_STORAGE_ES_MONTH_METRIC_DATA_TTL:18} # Unit is month
    # Batch process setting, refer to https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.5/java-docs-bulk-processor.html
    bulkActions: ${SW_STORAGE_ES_BULK_ACTIONS:1000} # Execute the bulk every 1000 requests
    flushInterval: ${SW_STORAGE_ES_FLUSH_INTERVAL:10} # flush the bulk every 10 seconds whatever the number of requests
    concurrentRequests: ${SW_STORAGE_ES_CONCURRENT_REQUESTS:2} # the number of concurrent requests
    metadataQueryMaxSize: ${SW_STORAGE_ES_QUERY_MAX_SIZE:5000}
    segmentQueryMaxSize: ${SW_STORAGE_ES_QUERY_SEGMENT_SIZE:200}
```

关于其它配置，暂时选择默认的即可，如果你有特别需求，参见后面的**配置详解**章节。

## Skywalking 部署

关于 Skywalking 的所有启动脚本都在 `bin` 目录下面：

```bash
oapService.bat       oapServiceInit.sh    startup.bat          webappService.sh
oapService.sh        oapServiceNoInit.bat startup.sh
oapServiceInit.bat   oapServiceNoInit.sh  webappService.bat
```

如你所见，你可以选择执行 **./startup.sh** 脚本在 Linux/Unix  系统上面部署 Skywalking ，你也可以选择通过 **startup.bat** 脚本在 Windows 平台上面部署 Skywalking。

除此之外，以 Linux/Unix系统为例，你还可以通过 **./oapService.sh** 和 **./webappService.sh** 脚本将 Skywalking OAP 后端和 UI 分开部署。不过得注意 UI 与 OAP 后端之间的地址配置正确。

部署完成之后你可以访问 **http://localhost:8080** 访问 UI。