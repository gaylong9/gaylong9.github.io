---
title: Log4J
mathjax: false
date: 2021-04-11 11:44:50
tags: 后端
---

&nbsp;

<!-- more -->

<!-- toc -->

&nbsp;

Log4J是Apache的一个开源项目，日志操作软件包。能够制定日志信息输出格式和目的地。通过配置文件灵活配置，不需修改程序代码。

&nbsp;

# 1. Log4J简介

1. 目的：监视变量变化情况、跟踪代码运行轨迹、调试器
2. 构成组件：
	1. Logger：生成日志，能对日志信息分类筛选
	2. Appender：输出目的地
	3. Layout：输出格式
	4. 每个Logger可以对应多个Appender，每个Appender对应一个Layout
	5. 输出到console时用PatternLayout布局；输出到文件时用XMLLayout布局

## 1.1 Logger组件

核心组件，日志记录器，对日志信息分类筛选，决定信息是被输出还是被忽略。

1. `org.apache.logging.log4j.Logger`，提供的方法：

	1. 打印各种级别的日志：大于等于指定级别的日志被输出
		1. trace(Object msg)
		2. debug(Object msg)
		3. info(Object msg)
		4. warn(Object msg)
		5. error(Object msg)
		6. fatal(Object msg)
	2. 打印日志通用方法：
		1. log(Level, Object msg)

2. `org.apache.logging.log4j.LogManager`提供了获得Logger实例的静态方法：

	1. getRootLogger() 返回根对象
	2. getLogger(String name) 返回特定对象

3. 可以在XML配置文件中配置Logger组件，如下例配置了helloappLogger：

	```xml
	<Logger name="helloappLogger" level="warn" additivity="false">
	    <AppenderRef ref="console"/>
	</Logger>
	```

4. 继承性：

	1. rootLogger是所有Logger组件的祖先

		```xml
		<Root level="info">
		    <AppenderRef ref="console"/>
		</Root>
		```

	2. 配置文件中即可指定继承关系：本例中childLogger就是helloappLogger的子类

		```xml
		<Logger name="helloappLogger.childLogger" level="INFO" additivity="false">
		    <AppenderRef ref="console"/>
		</Logger>
		```

	3. 继承特点：

		1. 子类没有配置日志级别就继承父类级别
		2. 默认继承父类所有Appender，除非指明additivity为false，默认true

&nbsp;

## 1.2 Appender组件

决定日志输出目的地，当前支持以下地址：

* Console 控制台
* File 文件
* Remote socket server 远程套接字服务器
* NT的事件记录器
* Remote UNIX Syslog daemon 远程UNIX Syslog守护进程

一个Logger可以对应多个Appender：

```xml
<Logger name="helloappLogger" level="warn" additivity="false">
    <AppenderRef ref="console"/>
    <AppenderRef ref="file"/>
</Logger>
```

&nbsp;

## 1.3 Layout组件

决定日志输出格式，有以下类型：

* org.apache.log4j.PatternLayout 灵活地指定布局模式
* HTMLLayout 以HTML形式布局
* XMLLayout 以XML形式布局
* SerializedLayout 可序列化的信息

其中PatternLayout，依照Conversion Pattern转换模式定义输出格式，可以通过预定义符号指定内容和格式：

| 符号    | 描述                                       |
| ------- | ------------------------------------------ |
| %r      | 自程序开始运行到输出当前日志所消耗的毫秒数 |
| %t      | 输出当前日志的线程名                       |
| %level  | 日志级别                                   |
| %d      | 当前日志的日期和时间                       |
| %logger | 当前日志的Logger的名字                     |
| %msg%n  | 日志信息的内容                             |

eg：为file的Appender配置PatternLayout布局：

```xml
<File name="file" fileName="app.log">
    <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n" />
</File>
```

&nbsp;

# 2. Log4J基本使用方法

## 2.1 创建Log4J配置文件

支持代码或XML配置。XML配置文件默认名log4j2.xml，默认路径classpath根路径：

```xml
<Configuration status="WARN">
    <Appenders>
        <Console name="console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n" />
        </Console>
        <File name="file" fileName="app.log">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n" />
        </File>
    </Appenders>
    
    <Loggers>
        <Root level="info">
            <AppenderRef ref="console"/>
        </Root>
        <Logger name="helloappLogger" level="warn" additivity="false">
            <AppenderRef ref="console"/>
            <AppenderRef ref="file"/>
        </Logger>
        <Logger name="helloappLogger.childLogger" level="INFO" additivity="false">
    		<AppenderRef ref="console"/>
		</Logger>
    </Loggers>
</Configuration>
```

&nbsp;

## 2.2 程序中使用Log4J

[JAR下载地址](https://logging.apache.org/log4j) 主要是core和api两个包。

```java
Logger rootLogger = LoggerManager.getRootLogger();
Logger helloappLogger = LoggerManager.getLogger("helloappLogger");
var childLogger = LoggerManager.getLogger("helloappLogger.childLogger");
helloappLogger.warn("xxx");
```

若不存在指定名字的Logger，则创建并返回，所有属性继承父类。

若不存在配置文件，则均采用默认配置：WARN、console、SYSTEM_OUT、`%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n` 。

&nbsp;

# 3. 在Web应用中使用Log4J

加入JAR包、在WEB-INF/classes下创建配置文件后即可在组件中使用。

若将配置文件放在其他位置，还需在web.xml中通过`<context-param>`配置：

```xml
<context-param>
    <param-name>log4jConfiguration</param-name>
    <param-value>/WEB-INF/conf/log4j2.xml</param-value>
</context-param>
```

JSP使用时先引入包 `<%@ page import="org.apache.logging.log4j.*" %>`，随后就可在`<% ... %>`中取得Logger对象输出日志。

&nbsp;

