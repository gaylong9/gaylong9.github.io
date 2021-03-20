---
title: IDEA与Tomcat
mathjax: false
date: 2021-01-26 12:44:32
tags: 后端
---

<br/>

<!-- more -->

<!-- toc -->

<br/>



# IDEA与Tomcat

## 1. IDEA集成Tomcat

File | Settings | Build, Execution, Deployment | Application Servers | +

&nbsp;

## 2. 旧版IDEA新建Web项目

Java Enterprise | Web Application 

&nbsp;

## 3. 新版IDEA新建Web项目



## 4. 目录结构

/webapp：根目录，存放JSP和HTML

/webapp/WEB-INF：配置文件web.xml

/webapp/WEB-INF/classes：各种.class文件，Servlet的.class也存放于此，且包名从classes后开写

/webapp/WEB-INF/lib：各种JAR

容器优先加载classes下的类，再加载lib下的JAR中的类，故两处若有同名类，则classes中的类有优先权。

浏览器端不能直接访问WEB-INF下的文件，只能被服务器端组件访问。

另有/webapp/src目录，是开发阶段存放Java类源文件的目录，正式发布阶段，一般不希望公开源码，应转移。