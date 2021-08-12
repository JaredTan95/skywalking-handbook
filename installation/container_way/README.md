# Docker-Compose 容器化部署

## 前置条件

通过容器部署，你可能需要对 Docker 有所了解，以及 Docker-Compose 工具的使用。
如果你有 Kubernetes 相关经验，推荐使用后续的 Kubernetes 部署方式。

与容器相关的官方资源有：

- [Skywalking-Docker](https://github.com/apache/skywalking-docker)
- [Skywalking-Kubernetes](https://github.com/apache/skywalking-kubernetes)

## Docker-Compose 快速部署

拉取部署包, 以 **8.7.0** 版本为例：

```bash
$ git clone git@github.com:apache/skywalking-docker.git

$ cd skywalking-docker/8/8.7.0/compose/

$ docker-compose up -d
```

部署完成之后你可以访问 **http://localhost:8080** 访问 UI。

### 容器运行 Skywalking-OAP

同样的，你也可以分开单独运行，以 **8.7.0** 版本为例，和前面通过脚本部署一样，OAP 默认使用 H2 存储。

```bash
$ docker run --name oap --restart always -d apache/skywalking-oap-server:8.7.0
```

如果你需要选择 Elasticsearch 存储，你可以通过环境变量进行配置：

```bash
$ docker run --name oap --restart always -d -e SW_STORAGE=elasticsearch -e SW_STORAGE_ES_CLUSTER_NODES=elasticsearch:9200 apache/skywalking-oap-server:8.7.0-es6
```

更过环境变量配置可以参考 `Skywalking-Docker` 文档：https://github.com/apache/skywalking-docker/blob/master/8.x/8.7.0/oap/README.md#configuration

### 容器运行 Skywalking-RocketBot-UI

以 **8.7.0** 版本为例，这里假设 OAP Server 的地址就是 `oap:12800`.

```bash
$ docker run --name oap --restart always -d -e SW_OAP_ADDRESS=oap:12800 apache/skywalking-ui:8.7.0
```

部署完成之后你可以访问 **http://localhost:8080** 访问 UI。

更多配置可以参考：https://github.com/apache/skywalking-docker/blob/master/8/8.0/ui/README.md#configuration

## 通过 Kubernetes 平台部署

目前推荐使用 `helm-chart` 部署，你也可以将其转换为原生的 Kubernetes 编排文件。

拉取部署包, 以 **8.7.0** 版本为例：

```bash
$ git clone git@github.com:apache/skywalking-kubernetes.git

$ git checkout v3.1.0

$ cd skywalking-kubernetes/chart/skywalking

$ helm dep up skywalking

# 可选操作，根据自身需求进行配置更改
$ vim values.yml 

# 也可以通过如下方式进行参数覆盖。
# 这里推荐将 Elasticsearch 和 Skywalking 分开部署。不建议使用 Skywalking Helm Chart 自含的 ES 安装方式。
$ helm install <release_name> skywalking -n <namespace> \
        --set elasticsearch.enabled=false \
        --set elasticsearch.config.host=<es_host> \
        --set elasticsearch.config.port.http=<es_port> \
        --set elasticsearch.config.user=<es_user> \
        --set elasticsearch.config.password=<es_password> \
```

更多详细的操作可以参考：https://github.com/apache/skywalking-kubernetes




