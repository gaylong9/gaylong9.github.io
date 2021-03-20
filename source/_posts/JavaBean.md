---
title: JavaBean
mathjax: false
date: 2021-02-27 20:48:30
tags: 后端
---

&nbsp;

<!-- more -->

<!-- toc -->

&nbsp;

[toc]



# JavaBean

## 1. 简介

1. 可重用、跨平台的软件组织，分为有UI的和没有UI主要负责业务数据或业务处理的。JSP通常访问后种
2. JSP与JavaBean搭配的好处：分离HTML和Java，便于编写和维护；二者各司其职，充分利用JavaBean可重用特点，提高开发效率
3. JavaBean特性：公共类、有一个不含参的构造方法、属性有get和set方法（且方法名中属性名首字母大写）、如需持久化可实现Serializable接口
4. JavaBean存放位置：`web-app/WEB-INF/classes/mypack/MyBean.class`

&nbsp;

## 2. JSP访问JavaBean

1. JSP中可通过代码访问JavaBean，也可通过特定JSP标签访问，后者能减少JSP中的程序代码

2. 标签使用：

	1. 导入JavaBean类 `<%@ page import="mypack.MyBean" %>`

	2. 声明JavaBean对象 `<jsp:useBean id="myBean" class="mypack.MyBean" scope="session" />`

		1. id：对象ID/引用JavaBean对象的局部变量名/存放在特定范围内的属性名，且JSP规范要求此ID不可重复（生效期间的JavaBean）

		2. class：JavaBean类名

		3. scope：JavaBean对象的存放范围，page、request、session、application，默认page

		4. 本标签处理流程：

			1. 定义一个名为myBean的局部变量
			2. 尝试从指定的范围内读取名为myBean的属性，并且使得myBean局部变量引用具体属性值，即MyBean的对象
			3. 若指定范围内没有myBean属性，就通过MyBean的默认构造方法创建一个对象并存入该范围，属性名为myBean，并用局部变量myBean引用该对象

		5. 本标签等价于以下Java代码

			```java
			mypack.MyBean myBean = null;
			myBean = (mypack.MyBean) session.getAttribute("myBean");
			if (myBean == null) {
			    myBean = new mypack.MyBean();
			    session.setAttribute("myBean", myBean);
			}
			```

	3. 访问JavaBean属性：`<jsp:getProperty name="myBean" property="count" />`

		1. 根据name找到对应ID的Bean对象，打印它的count属性，等价于`<%= myBean.getCount() %>`:
		2. Servlet容器运行本标签时，会根据指定属性名自动调用get方法

	4. 给JavaBean属性赋值：`<jsp:setProperty name="myBean" property="count" value="1" />`，等价于`<% myBean.setCount(1); %>`

&nbsp;

## 3. JavaBean范围

1. page：
	1. 从客户请求访问一个JSP文件开始，到这个文件执行结束
	2. 每当客户请求本JSP，标签都会创建一个JavaBean对象，存放到页面范围内
	3. 何时结束生命周期：
		1. 客户请求访问的页面执行完毕，然后通过`<jsp:forward>`转发请求给其他Web组件
		2. 客户请求访问的页面执行完毕冰箱客户端发回响应
	4. JSP中有pageContext默认对象
2. request：
	1. 客户请求访问一个JSP开始，知道本JSP返回响应结果，或请求转发后其他Web组件返回响应结果
	2. 每当客户请求本JSP，标签都会创建一个JavaBean对象，存放到请求范围内
	3. 何时结束生命周期：
		1. 请求访问的当前JSp执行完毕并向客户端发回结果
		2. 当前JSP请求转发给其他组件，其他组件执行完毕发回结果
	4. 谁可以访问本JavaBean：第一个JSP本身、该JSP包含和转发的组件
	5. 请求范围的JavaBean对象实际上作为属性保存在HttpServletRequest对象中，属性名是JavaBean的ID，属性值是JavaBean对象，故也可通过request.getAttribute()获取请求范围内的JavaBean对象
3. session：
	1. 处于同一个绘画中的Web组件共享会话范围内的JavaBean对象
	2. 实际上作为属性保存在HttpSession对象中，属性名是JavaBean的ID，属性值是JavaBean对象，故也可通过session.getAttribute()获取会话范围内的JavaBean对象
4. application：
	1. 实际上作为属性保存在ServletContext对象中，属性名是JavaBean的ID，属性值是JavaBean对象，故也可通过application.getAttribute()获取应用范围内的JavaBean对象