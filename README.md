# spring-boot-elk-efk-7.2.0
## 1、引入依赖

```java
<!-- logstash 依赖 -->
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>6.1</version>
</dependency>
```

## 2、application.yml配置

```java
spring:
  profiles:
    active: test
  application:
    name: app_name
```

## 3、logback-spring.xml配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/base.xml" />

    <springProperty scope="context" name="springProfile" source="spring.profiles.active" defaultValue="test"/>
    <springProperty scope="context" name="app_name" source="spring.application.name" defaultValue="app_name"/>

    <!--定义logstash 传输方式 以及地址-->
    <appender name="stash" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <destination>192.168.0.134:9000</destination>
        <encoder charset="UTF-8" class="net.logstash.logback.encoder.LogstashEncoder">
            <fieldNames>
                <timestamp>time</timestamp>
                <message>msg</message>
            </fieldNames>
<!-- 使用springProperty后，字段自动添加进去了 -->
<!--            <customFields>{"app_name":"${app_name}","springProfile":"${springProfile}"}</customFields>-->
            <timestampPattern>HH:mm:ss.SSS</timestampPattern>
        </encoder>
    </appender>

    <springProfile name="test, dev">
        <logger name="包名" level="debug">
            <appender-ref ref="stash"/>
        </logger>
    </springProfile>

    <springProfile name="pro">
        <logger name="包名" level="warn">
            <appender-ref ref="stash"/>
        </logger>
    </springProfile>

</configuration>
```

## 4、记录日志

> 根据springProfile激活的环境，记录不同级别的日志

```java
log.info("--- log 汉字 -> {}", name);
log.warn("--- log 汉字 -> {}", name);
log.error("--- log 汉字 -> {}", name);
log.error("***Exception", e);
```

## 5、定时删除日志

> DELETE

```java
http://192.168.0.134:9200/*-2019.07.22
```

## 6、查看日志

```java
http://192.168.0.134:5601
```

> 1.点击右下角菜单->管理;

> 2.点击Kibana下的索引模式，创建对应项目名的索引模式;

> 3.点击右上角菜单->Discover;

> 4.在Filters下面可以选择对应的索引模式，切换项目，可选择想要关注的字段进行查看.

## 注意

> 192.168.0.134:9000是filebeat部署机IP与监听端口号;

> 192.168.0.134:5601是kibana部署机IP与端口号;

> logback-spring.xml需要修改包名
