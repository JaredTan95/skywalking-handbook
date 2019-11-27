# 支持自定义方法/函数增强

本节主要展示如何通过探针以及配置追踪到方法级别。

主要分为两种方式： 
- 手动在Java Method中加入 `@Trace` 注解.
- 通过 `apm-customize-enhance-plugin` 插件配置需要追踪的方法名.

## 手动在Java Method中加入 `@Trace` 注解

使用maven或gradle引入toolkit依赖。

```xml
 <dependency>
      <groupId>org.apache.skywalking</groupId>
      <artifactId>apm-toolkit-trace</artifactId>
      <version>${skywalking.version}</version>
   </dependency>
```

使用 TraceContext.traceId() API 得到 traceId 代码示例：

```java
import TraceContext;
...

modelAndView.addObject("traceId", TraceContext.traceId());
```

- 在你想追踪的方法上添加@Trace注解。添加后，你就可以在方法调用栈中查看到span的信息。
- 在被追踪的方法的上下文周期内添加自定义tag。
- ActiveSpan.error() 将当前 Span 标记为出错状态.
- ActiveSpan.error(String errorMsg) 将当前 Span 标记为出错状态, 并带上错误信息.
- ActiveSpan.error(Throwable throwable) 将当前 Span 标记为出错状态, 并带上 Throwable.
- ActiveSpan.debug(String debugMsg) 在当前 Span 添加一个 debug 级别的日志信息.
- ActiveSpan.info(String infoMsg) 在当前 Span 添加一个 info 级别的日志信息.

例如：

```java
ActiveSpan.tag("my_tag", "my_value");
ActiveSpan.error();
ActiveSpan.error("Test-Error-Reason");

ActiveSpan.error(new RuntimeException("Test-Error-Throwable"));
ActiveSpan.info("Test-Info-Msg");
ActiveSpan.debug("Test-debug-Msg");
```

## 通过 `apm-customize-enhance-plugin` 插件配置需要追踪的方法名

实现对类的自定义增强需要2步:
- 激活插件，将插件从 `optional-plugins/apm-customize-enhance-plugin.jar` 移动到 `plugin/apm-customize-enhance-plugin.jar`.
- 在 `agent.config` 中配置 `plugin.customize.enhance_file`，指明增强规则文件，比如 `/absolute/path/to/customize_enhance.xml`.

例如下`customize_enhance.xml` 配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<enhanced>
    <class class_name="test.apache.skywalking.testcase.customize.service.TestService1">
        <method method="staticMethod()" operation_name="/is_static_method" static="true"/>
        <method method="staticMethod(java.lang.String,int.class,java.util.Map,java.util.List,[Ljava.lang.Object;)" operation_name="/is_static_method_args" static="true">
            <operation_name_suffix>arg[0]</operation_name_suffix>
            <operation_name_suffix>arg[1]</operation_name_suffix>
            <operation_name_suffix>arg[3].[0]</operation_name_suffix>
            <tag key="tag_1">arg[2].['k1']</tag>
            <tag key="tag_2">arg[4].[1]</tag>
            <log key="log_1">arg[4].[2]</log>
        </method>
        <method method="method()" static="false"/>
        <method method="method(java.lang.String,int.class)" operation_name="/method_2" static="false">
            <operation_name_suffix>arg[0]</operation_name_suffix>
            <tag key="tag_1">arg[0]</tag>
            <log key="log_1">arg[1]</log>
        </method>
        <method method="method(test.apache.skywalking.testcase.customize.model.Model0,java.lang.String,int.class)" operation_name="/method_3" static="false">
            <operation_name_suffix>arg[0].id</operation_name_suffix>
            <operation_name_suffix>arg[0].model1.name</operation_name_suffix>
            <operation_name_suffix>arg[0].model1.getId()</operation_name_suffix>
            <tag key="tag_os">arg[0].os.[1]</tag>
            <log key="log_map">arg[0].getM().['k1']</log>
        </method>
    </class>
    <class class_name="test.apache.skywalking.testcase.customize.service.TestService2">
        <method method="staticMethod(java.lang.String,int.class)" operation_name="/is_2_static_method" static="true">
            <tag key="tag_2_1">arg[0]</tag>
            <log key="log_1_1">arg[1]</log>
        </method>
        <method method="method([Ljava.lang.Object;)" operation_name="/method_4" static="false">
            <tag key="tag_4_1">arg[0].[0]</tag>
        </method>
        <method method="method(java.util.List,int.class)" operation_name="/method_5" static="false">
            <tag key="tag_5_1">arg[0].[0]</tag>
            <log key="log_5_1">arg[1]</log>
        </method>
    </class>
</enhanced>
```

上述文件中的配置说明：

	| 配置  | 说明 |
	|:----------------- |:---------------|
	| class_name | 要被增强的类 |
	| method | 类的拦截器方法 |
	| operation_name | 如果进行了配置，将用它替代默认的operation_name |
	| operation_name\_suffix | 表示在operation_name后添加动态数据 |
	| static | 方法是否为静态方法 |
	| tag | 将在local span中添加一个tag。key的值需要在XML节点上表示。|
	| log | 将在local span中添加一个log。key的值需要在XML节点上表示。|
	| arg[x]   | 表示输入的参数值。比如args[0]表示第一个参数。 |
	| .[x]     | 当正在被解析的对象是Array或List，你可以用这个表达式得到对应index上的对象。|
	| .['key'] | 当正在被解析的对象是Map, 你可以用这个表达式得到map的key。|


