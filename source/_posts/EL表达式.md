---
title: EL表达式
mathjax: false
date: 2021-02-27 20:53:04
tags: 后端
---

&nbsp;

<!-- more -->

<!-- toc -->

&nbsp;

[toc]



# EL表达式

Expression Language是JSP 2中引入的新特性，用于JSP文件中的数据访问，简化代码，替代传统`<%= %>`的Java表达式，以及部分`<% %>`Java程序片段。



## 1. 基本语法

1. 基本形式`${ }`，若在JSP模板文本中使用EL表达式，会输出到网页上
2. 和Java表达式一样，既能直接插入到JSP文件的模板文本中输出到网页，也能作为JSP标签的属性的值
3. `\${}` 转义输出
4. 主要操作域对象的数据，不可操作局部变量
5. 命名变量默认从小范围起找，若域对象全找完仍没有，返回空字符串

### 1.1 访问对象属性及数组元素

1. `.` 访问对象属性，如`${customer.name}`表示customer对象的name属性
2. 也可用`[]`访问对象属性，如`${customer["name"]}`；如属性中有特殊字符，如`-`则必须用`[]`访问
3. `[]`还可访问数组元素，如`${customers[0]}`

### 1.2 EL运算符

1. 支持以下运算符：$+-*\quad/div\quad\%mod\quad==eq\quad!=ne\quad<lt\quad>gt\quad \le le \quad \ge ge$

	$\&\&and\quad||or\quad!not\quad empty\quad ?:$

2. 特殊运算符`${empty obj}`，判断obj对象是否为空：

	1. obj不存在，没有被定义，返回true
	2. obj为null，返回true
	3. obj引用集合类型对象，并且集合中不包含元素，返回true
	4. 可于`!`连用，`${! empty obj}`，不空则true

### 1.3 隐含对象

1. EL定义了11个隐含对象，10个都是Map类型，用于访问Web应用中的特定数据

	1. header：Map，HTTP请求头部的项目名和项目值映射，如`${header.host}`等价于`<%=request.getHeader("host")%>`

	2. headerValues：Map，HTTP请求头部的项目名和所有匹配的项目值的数组映射，如`${headerValues["accept-language"]}`等价于`<%=request.getHeaders("accept-language")%>`

	3. param：Map，请求中的请求参数名和参数值映射

	4. paramValues：Map，请求中的请求参数名和所有匹配的参数值数组映射

	5. cookie：Map，请求中的Cookie名和Cookie对象映射，如`${cookie.username.value}`等价于调用名为username的Cookie对象的getValue()方法

		

	6. pageScope：Map，页面范围内的属性名和属性值映射，如`${pageScope.obj.total}`表示页面范围内的属性名为obj的对象的getTotal()方法

	7. requestScope：Map，请求范围内的属性名和属性值映射

	8. sessionScope：Map，会话范围内的属性名和属性值映射

	9. applicationScope：Map，Web应用范围内的属性名和属性值映射

		

	10. initParam：Map，Web应用的初始化参数名和参数值映射，如`${initParam.driver}`等价于`<%=application.getInitParameter("driver")%>`

		

	11. pageContext：PageContext对象，如`${pageContext.request.requestURL}`等价于`<%=request.getRequestURL()%>`

2. EL表达式中无法直接访问JSP的隐含对象，可用pageContext获取，如`${pageContext.request.requestURL}`

### 1.4 命名变量

EL表达式中的变量称为命名变量，不是JSP文件中的局部变量或实例变量，是存放在特定范围内的属性；命名变量的名字和特定范围内的属性名字对应，如`${username}`表示从page到application范围寻找username这一属性attribute。

&nbsp;

## 2. EL函数

1. EL表达式可以访问EL函数；**EL函数实际上与Java类中的方法对应，此Java类要为public，作为函数的方法要为public static**；Java类定义后应在标签库描述符TLD文件中把类的方法映射为函数

2. 步骤：

	1. 编译java文件，把编译生成的类文件放到`webapp/WEB-INF/classes/mypack/`下

	2. 在标签库描述符TLD文件中，把类的方法映射为函数，在`WEB-INF`目录下创建mytaglib.tld文件

		```xml
		<taglib>
			...
		    <function>
		    	<description>...</description>
		        <name>起名</name>
		        <function-class>mypack.XX</function-class>
		        <function-signature>
		        	int func(int a, int b)
		        </function-signature>
		    </function>
		</taglib>
		```

	3. 在web.xml中加入`<taglib>`元素

		```xml
		<web-app>
		    <jsp-config>
		        <taglib>
		            <taglib-uri>/mytaglib</taglib-uri>
		            <taglib-location>/WEB-INF/mytaglib.tld</taglib-location>
		        </taglib>
		    </jsp-config>
		</web-app>
		```

		