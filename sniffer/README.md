# 探针接入

## 前置条件

Skywalking OAP后端收集器器已经安装部署好，假设暴露的地址为: localhost:11800

## 步骤

- 获取探针
- 安装探针
- 启动时配置相关参数

### 获取探针

不同版本下载地址: http://skywalking.apache.org/downloads/

此处以 8.1.0 为例:
首先点击右边地址下载 Skywalking 安装包，⾥面包含了探针 Agent 压缩包:https://www-us.apache.org/dist/skywalking/8.1.0/apache-skywalking-apm-8.1.0.tar.gz;也可以查看该⽂档同级⽬目录下的压缩包。
或者在Linux系统中执⾏行行如下命令:

- 下载压缩包

```bash
$ wget https://www-us.apache.org/dist/skywalking/6.4.0/apache-skywalking-apm-8.1.0.tar.gz
```

- 解压缩之后

```bash
$ tar -zxvf apache-skywalking-apm-8.1.0.tar.gz

  +-- agent
     +-- activations
          apm-toolkit-log4j-1.x-activation.jar
          apm-toolkit-log4j-2.x-activation.jar
          apm-toolkit-logback-1.x-activation.jar
          ...
     +-- config
          agent.config
     +-- plugins
          apm-dubbo-plugin.jar
          apm-feign-default-http-9.x.jar
          apm-httpClient-4.x-plugin.jar
          .....
     +-- optional-plugins
          apm-gson-2.x-plugin.jar
          .....
     +-- bootstrap-plugins
          jdk-http-plugin.jar
          .....
     +-- logs
     skywalking-agent.jar
```

### 安装探针

主要分为两种情况:

- 如果你的业务服务通过war包部署
    - Linux Tomcat 7, Tomcat 8

    需要在 `tomcat/bin/catalina.sh` 添加如下配置:
    ```bash
    CATALINA_OPTS="$CATALINA_OPTS -javaagent:/path/to/skywalking-agent/skywalking-
     agent.jar"; export CATALINA_OPTS
     ```

    - Windows Tomcat 7, Tomcat 8
    需要在 `tomcat/bin/catalina.sh` 添加如下配置:
    ```bash
    set "CATALINA_OPTS=-javaagent:/path/to/skywalking-agent/skywalking-agent.jar"
    ```

另外，关于war包容器器化, 需要将  `tomcat/bin/catalina.sh`  更改后重新打包或者挂载至 Docker 镜像。
 - `catalina.sh` 文件:

 ```bash
······
 # SkyWalking Configuration
 CATALINA_OPTS="$CATALINA_OPTS -javaagent:/skywalking-agent/skywalking-agent.jar";
 export CATALINA_OPTS
 ·······
```

- `Dockerfile`:

```bash
FROM tomcat:7.0.94-jre7
ENV TZ=Asia/Shanghai \
     DIST_NAME=query-service \
     AGENT_REPO_URL="http://nexus.mschina.io/nexus/service/local/repositories/labs/
 content/io/daocloud/mircoservice/skywalking/agent/2.0.1/agent-2.0.1.gz"
ADD $AGENT_REPO_URL /
RUN set -ex; \
     tar -zxf /agent-2.0.1.gz; \
     rm -rf agent-2.0.1.gz;
RUN ln -sf /usr/share/zoneinfo/$TZ /etc/localtime \
     && echo $TZ > /etc/timezone
ADD catalina.sh /usr/local/tomcat/bin/
ADD target/struts-spring-demo.war /usr/local/tomcat/webapps/
WORKDIR /usr/local/tomcat/webapps/
CMD ["catalina.sh","run"]
```

- 如果你的业务服务通过Jar包部署:

```bash
$ java -javaagent:/path/to/skywalking-agent/skywalking-agent.jar -jar yourApp.jar
```

### 【推荐】通过 Kubernetes Sidecar 方式接⼊：

这种方式则将探针单独的打包成了一个镜像，通过 initContainers 和业务容器共享⽂件夹，以此加载探针。

```yml
    # refs: https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-initialization/
    spec:
      initContainers:
        - name: sidecar
          image: your-skywalking-agent-sidecar:6.4.0 # 容器镜像，包含静态资源文件
          imagePullPolicy: IfNotPresent
          command: ["cp", "-r", "/data/agent", "/sidecar"]
          volumeMounts:
            - name: sidecar
              mountPath: /sidecar
      containers:
        - name: apm-item
          image: your-business-apm-demo
          imagePullPolicy: IfNotPresent
          env:
            - name: JAVA_OPTS
              value: "-javaagent:/sidecar/agent/skywalking-agent.jar"
            - name: SW_AGENT_NAME
              value: apm-demo
            - name: SW_AGENT_COLLECTOR_BACKEND_SERVICES
              value: oap:11800
          ports:
            - name: http
              containerPort: 8082
          volumeMounts:
            - name: sidecar
              mountPath: /sidecar
      volumes:
        - name: sidecar  #共享agent文件夹
          emptyDir: {}
      restartPolicy: Always
```

## 启动时配置相关参数

前面安装好探针之后，我们需要配置探针将数据发送到后端的地址。⼀般推荐通过环境变量参数传入配置值:

- `SW_AGENT_NAME` : 探针名字, 一般和业务服务名一致
- `SW_AGENT_COLLECTOR_BACKEND_SERVICES`: 后端收集器器地址。默认为 `127.0.0.1:11800` .

更多参数可以参考 `agent/config/agent.config` 文件。或者可以浏览官方配置说明：https://github.com/apache/skywalking/blob/master/docs/en/setup/service-agent/java-agent/README.md

参考文档:

- [Skywalking Agent](https://github.com/apache/skywalking/blob/master/docs/en/setup/service-agent/java-agent/README.md)
- [探针接入](https://github.com/SkyAPM/document-cn-translation-of-skywalking/blob/master/docs/zh/master/setup/service-agent/java-agent/README.md)

