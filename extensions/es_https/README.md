# Skywalking 通过 HTTPS SSL 认证连接 Elasticsearch

## 证书准备

需要注意的是，目前 ElasticSearch 支持的证书类型有: jceks、jks、dks、pkcs11、pkcs12.

如果证书类型不在上面类型中，可以通过`keytool`工具进行转换

例如将一个 密码为`changeit `的`ca.pem` 格式的证书转换为`jks`格式的证书，将其命名为`es_keystore.jks`:

```bash
keytool -import -v -trustcacerts -file ca.pem  -keystore es_keystore.jks -keypass changeit -storepass changeit
```

## 更改Skywalking 中 application.yml关于ES的配置：

- 将`protocol`协议更改为`https`
- 配置`keyStorePath `和`keyStorePass`
- 注意`clusterNodes`配置中Elasticsearch连接的端口

```yaml
storage:
  elasticsearch:
    # nameSpace: ${SW_NAMESPACE:""}
    user: ${SW_ES_USER:""} # User needs to be set when Http Basic authentication is enabled
    password: ${SW_ES_PASSWORD:""} # Password to be set when Http Basic authentication is enabled
    clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:localhost:443}
    keyStorePath: ${SW_SW_STORAGE_ES_SSL_JKS_PATH:"../es_keystore.jks"}
    keyStorePass: ${SW_SW_STORAGE_ES_SSL_JKS_PASS:"changeit"}
    protocol: ${SW_STORAGE_ES_HTTP_PROTOCOL:"https"}
    indexShardsNumber: ${SW_STORAGE_ES_INDEX_SHARDS_NUMBER:2}
    indexReplicasNumber: ${SW_STORAGE_ES_INDEX_REPLICAS_NUMBER:0}
    # Those data TTL settings will override the same settings in core module.
    recordDataTTL: ${SW_STORAGE_ES_RECORD_DATA_TTL:7} # Unit is day
    otherMetricsDataTTL: ${SW_STORAGE_ES_OTHER_METRIC_DATA_TTL:45} # Unit is day
    monthMetricsDataTTL: ${SW_STORAGE_ES_MONTH_METRIC_DATA_TTL:18} # Unit is month
    # Batch process setting, refer to https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.5/java-docs-bulk-processor.html
    bulkActions: ${SW_STORAGE_ES_BULK_ACTIONS:2000} # Execute the bulk every 2000 requests
    bulkSize: ${SW_STORAGE_ES_BULK_SIZE:20} # flush the bulk every 20mb
    flushInterval: ${SW_STORAGE_ES_FLUSH_INTERVAL:10} # flush the bulk every 10 seconds whatever the number of requests
    concurrentRequests: ${SW_STORAGE_ES_CONCURRENT_REQUESTS:2} # the number of concurrent requests
```