---
    title: flink配置log4j
    categories: 
    tags:
    creator: cjq
    create_time: 2021/01/27

---

因为本地测试中，本地构建的数据源并非预想中的单线程运行，所以想看下线程id，以确定是否每次都新起了线程。

推荐使用xml，结构清晰

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Appenders>
        <!-- 控制台输出 -->
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>

        <!-- 文件输出（可选） -->
        <File name="File" fileName="logs/flink-job.log">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </File>
    </Appenders>

    <Loggers>
        <!-- Flink 相关日志级别 -->
        <Logger name="org.apache.flink" level="INFO"/>

        <!-- 你的项目包名 -->
        <Logger name="com.jingqicao.code" level="DEBUG"/>

        <Root level="INFO">
            <AppenderRef ref="Console"/>
            <AppenderRef ref="File"/>
        </Root>
    </Loggers>
</Configuration>
```



### 解析

如果Logger不指定additivity=false，默认为true。此时当前Logger及Root都会处理日志，如果Logger与Root配了相同的Appender，就会重复输出。



Root的level，表示的是Root使用的日志级别，如果有一个Logger没有具体指定，就会交由Root处理，即日志级别由Root决定。

