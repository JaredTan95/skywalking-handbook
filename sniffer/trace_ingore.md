# 自定义忽略指定端点（Trace Path/Endpoint）

此方式作为一个可选插件来帮助我们忽略指定的Trace，比如常见的Eureka心跳信息等。

此方式作为一个可选插件来帮助我们忽略指定的Trace，比如常见的Eureka心跳信息等。

## 前置条件

获取到Skywalking探针的压缩包或者探针的镜像（此处采用的方式）。解压过后的目录结构如下：

```

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
    		apm-spring-annotation-plugin.jar 
    		apm-trace-ignore-plugin.jar ➊
    skywalking-agent.jar
    
```

由于是可选插件，因此，默认情况下并没有激活，我们需要将➊中的jar包拷贝或剪切到上面的`plugins`目录下并添加相关配置来激活该插件,即：

```

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
         apm-trace-ignore-plugin.jar ➊ 注意此处所在文件夹为plugins。
         .....
    +-- optional-plugins
    		apm-spring-annotation-plugin.jar 
    skywalking-agent.jar
    
```

在`config`目录下创建`apm-trace-ignore-plugin.config`文件并添加如下配置激活插件：

```txt
trace.ignore_path=/your/path/1/**,/your/path/2/** ➊
```

➊ 中的路径配置规则支持`Ant Path`匹配风格，比如：
`/path/*`, `/path/**`, `/path/?`。匹配的Path路径将不会被记录。

## [基于容器Sidecar的方式接入](docker-sidecar.md)的最佳实践

主要讲解在Kubernetes Pod initContainer 中执行Shell指令，将 `apm-trace-ignore-plugin.jar` 插件拷贝至 `plugins` 文件夹，并配置需要忽略的路径。

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dmp-ns1
  name: daoshop-user-center
  labels:
    app: daoshop-user-center
spec:
  selector:
    matchLabels:
      app: daoshop-user-center
  template:
    metadata:
      labels:
        app: daoshop-user-center
    spec:
      # refs: https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-initialization/
      initContainers:
        - name: dx-monitor-agent-sidecar
          image: registry.dx.io/daocloud-dmp/dx-monitor-agent-sidecar:release-2.3.0-0b0cbd1
          imagePullPolicy: IfNotPresent
          command: 
            - "sh"
            - "-c"
            - > 
              mv /sidecar/skywalking/agent/optional-plugins/apm-trace-ignore-plugin-6.5.0-SNAPSHOT.jar /sidecar/skywalking/agent/plugins; ➊
              echo 'trace.ignore_path=${TRACE_IGNORE_PATH:/eureka/**,/api/sail/**}' >> /sidecar/skywalking/agent/config/apm-trace-ignore-plugin.config; ➋
              cp -r /sidecar /target; 
          volumeMounts:
            - name: sidecar
              mountPath: /target
      containers:
        - image: {{ daoshop-user-center.image }}
          name: daoshop-user-center
          resources:
            requests:
              memory: "2048Mi"
              cpu: "500m"
            limits:
              memory: "2048Mi"
              cpu: "500m"
          ports:
            - containerPort: 18081
          env:
            - name: JAVA_OPTS
              value: "-javaagent:/sidecar/sidecar/skywalking/agent/skywalking-agent.jar" 
            - name: SW_AGENT_NAME
              value: apm-demo
            - name: SW_AGENT_COLLECTOR_BACKEND_SERVICES  
              value: dx-skywalking-oap-ng.dx-pilot.svc:11800
          volumeMounts:
            - name: sidecar
              mountPath: /sidecar
      volumes:
        - name: sidecar  #共享agent文件夹
          emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: daoshop-user-center
spec:
  type: NodePort
  ports:
    - port: 18081
  selector:
    app: daoshop-user-center
```

- ➊ 将trace-ignore插件拷贝至`plugins`目录下。
- ➋ 添加相关配置,这里将忽略追踪 `/api/sail/**` 为前缀的URL。


