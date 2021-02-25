---
title: JSP
mathjax: true
date: 2021-02-25 22:45:54
tags: 后端
---

&nbsp;

<!-- more -->

<!-- toc -->

&nbsp;

[toc]

# JSP

Java service page

2. 可以嵌套Java代码，动态页面，还能进行排版
3. jsp对应一个Servlet：第一次访问jsp时，jsp引擎将jsp翻译为Servlet源文件，编译为Servlet类，存放在tomcat的work目录中，初始化并运行该类
4. 多数Servlet容器能在Web应用运行中检测到JSP的更新，自动生成新的源文件并编译，运行新的Servlet
5. JSP对应的Servlet继承了Tomcat提供的HttpJSPBase类，该类实现了HttpJspPage接口，该接口继承了Servlet API中的Servlet接口

&nbsp;

## 1. 语法

### 1.1 JSP指令

`<%@ 指令名 属性=值 %>` 设置和整个JSP网页相关的属性，常用page、include、taglib三种指令

1. page指令：`<%@ page 属性1=“值1” 属性2=“值2” %>`
	1. `language = "java"` 指定程序代码所用编程语言，目前仅支持Java；多次使用仅第一次生效
	2. `method = "doGet"` 指定Java程序片段所属的方法的名称，程序片段会成为指定方法的主体，默认service，支持service、doGet、doPost；多次使用仅第一次生效
	3. `import = "java.io.*, java.util.Hashtale"` 逗号分隔，导入包或类，可多次使用
	4. `content_type="text/html; charset=GBK"` 指定MIME和字符编码，默认`text/html`和ISO-8859-1，多次调用首次生效
	5. `session="true|false"` 指定本页面是否使用Session，默认true
	6. `errorPage="error_url"` 指定发生异常时请求转发目的
	7. `inErrorPage="true|false"` 表示此页是否为处理异常的页面
2. include指令：`<%@ include file="目标组件绝对/相对URL" %>` 包含其他JSP或HTML文件，代码重用，静态包含

&nbsp;

### 1.2 JSP声明

`<%! declaration;declaration; ... %>`

用于声明对应Servlet类的成员变量和方法

eg：

```jsp
<%! int v1 = 0; %>
<%! int v2, v3, v4; %>
<%! String v5 = "hello";
	static int v6;
%>
<%! 
	public String func (int i) {
    	...
	}    
%>
```

此方法声明变量只在当前JSP有效，若希望多个JSP文件都包含这些声明，就要写去一个JSP页面并include。

&nbsp;

### 1.3 Java程序片段

Scriplet `<% ... %>`，嵌入Java代码，默认这些代码位于对应Servlet的service代码中。

eg：

```jsp
<% 
	String gender = "female";
	if (gender.equals("female")) {
%>
She is a girl.	<%-- 模板文本 --%>
<% } else { %>
He is a boy.	<%-- 模板文本 --%>
<% } %>
```

JSP翻译、编译为Servlet的过程中，是将各部分转换为对应Java代码，不在`<>`内的模板文本，会转换为`out.print(模板文本)`。故编写Scriplet、JSP时，相当于是在拼接一个Servlet代码。

&nbsp;

### 1.4 Java表达式

`<%= ... %>` 将表达式的值输出到网页，基本数据类型的数字会自动转换为字符串。

除了输出值，也可当做JSP标签属性的值。

&nbsp;

### 1.5 JSP隐含对象

JSP中，ServletContext、ServletRequest等对象，均有固定的引用变量在引用。JSP中不需进行声明就可以直接使用引用变量。

| 引用变量    | 隐含对象                   |
| ----------- | -------------------------- |
| request     | HttpServletRequest         |
| response    | HttpServletResponse        |
| pageContext | PageContext                |
| application | ServletContext             |
| out         | JspWriter                  |
| config      | ServletConfig              |
| page        | Object（相当于this关键字） |
| session     | HttpSession                |
| exception   | Exception                  |

out即为对应Servlet中的JspWriter对象，`out.print(...)`等价于JSP中`<%= ... %>`。

request和response是对应Servlet中方法的参数，其余引用则是方法的局部变量。

&nbsp;

## 2. 生命周期

1. JSP需要先经容器编译成类才能运行。

2. 生命周期：

	1. 解析阶段：容器解析JSP代码，如有错误代码则向客户端返回错误信息
	2. 翻译阶段：容器把JSP文件翻译成Servlet源文件
	3. 编译阶段：容器编译Servlet源文件，生成Servlet类
	4. 初始化阶段：加载Servlet类，创建实例，调用init方法
	5. 运行时阶段：调用service方法
	6. 销毁阶段：调用destory方法，然后销毁实例

3. 解析、翻译、执行是JSP特有阶段，仅发生于：

	1. JSP文件首次被客户端请求访问
	2. JSP文件被更新
	3. 对应Servlet类文件被手动删除

4. JSP对应Servlet实现JspPage接口，该接口继承Servlet接口。JspPage接口中有`jspInit()`和`jspDestroy()`方法，与Servlet的`init()`和`destroy()`作用相同。编写JSP文件时可以实现`jspInit()`和`jspDestroy()`方法；而对应Servlet类的`_jspService()_`方法由容器自动根据JSP源文件生成（此三方法是JSP翻译后生成Servlet的方法）

	```jsp
	<%!
		public void jspInit() {}  
		public void jspDestroy() {}
	%>
	
	```

5. application引用对象是在`_jspService()`中定义，故`jspInit()`和`jspDestroy()`方法中要使用ServletContext对象时要通过在Servlet接口中定义的 `getServletConfig().getServletContext()`

&nbsp;

## 3. 请求转发

`<jsp:forward page="绝对/相对URL" />`

1. 转发目标组件可以是HTML、JSP、Servlet

2. 特性与Servlet中转发大体相同，如共享req和resp对象，JSP源组件中的输出数据不会发送到客户端，但唯一区别是JSP源组件中转发标签后的代码不会执行

3. 相对路径：不以/开头

	绝对路径：以/开头

4. 还可以在`<jsp:forward page="..."> ... </jsp:forward>`之间插入`<jsp:param name="..." value="..."/>`标签，以给request对象添加请求参数，目标组件调用`request.getParameter(name)`获取

&nbsp;

## 4. 包含

使用包含指令/标签，引入其他JSP、HTML页面，实现页面的局部变化/局部不变。

1. include指令，静态包含：`<%@include file="绝对/相对路径" %>` 
2. include标签，动态包含：`<jsp:include page="file.jsp"/>` 
3. 源组件与目标组件在请求范围内共享数据
4. 静态包含：
	1. 发生在解析JSP源组件阶段，对内容原封不动地进行直接替换，然后Servlet容器翻译编译。即两JSP代码整合，全放到`_jspService()`方法中，生成一份Servlet
	2. 不可include Servlet
	3. 两JSP对应一个Servlet，目标组件可以访问源组件中定义的局部变量，故不能有同名变量
	4. 运行效率高一点，但耦合高，不灵活，常用于不会发生变化的网页内容
	5. 有些开发习惯中，将用于被静态包含的文件，用`.jspf`、`.htmf`、或均`.inc`作为扩展名
	6. 客户端非首次访问源JSP时，若源组件与目标组件均未更新，则直接调用对应Servlet，若目标组件更新，以Tomcat为代表的容器能够检测到并重新include、翻译、编译、初始化、运行，但部分容器不能检测到，就需要手动删除源组件对应Servlet源文件和类文件，迫使容器重新操作。原生Tomcat路径`<CATALINA_HOME>/work/Catalina/localhost/helloapp/org/apache/jsp`
5. 动态包含：
	1. 源组件与目标组件各自独立，访问源组件JSP时，其中include标签被翻译为`or.apache.jsper.runtime.JspRuntimeLibrary.include(request, response, "des.jsp", out, false);`，初始化后调用服务方法，运行到include方法时，开始des.jsp的解析、翻译、初始化、调用服务方法
	2. page可以是jsp文件，也可以是<%= %>
	3. 可以include HTML、JSP和Servlet
	4. include标签含有flush属性，默认false，开启后源组件在include前先把生成的响应正文提交给客户
	5. 编译阶段文件独立，运行时才动态包含进来，常用于会发生变化的网页内容
	6. 动态包含还可在标签之间插入`<jsp:param name=".." value=".."/>`，用于向req添加请求参数，实现页面间传参
6. 静态、动态包含混用：如侧边栏、标题栏等每个页面都显示相同的内容，（分别）放在单独页面中静态包含，各页面不同内容分别放在单独页面包含；但如此后，仍会有一些重复代码（如每个页面都要有对侧边栏、标题栏的包含），可以继续将页面提炼为模板，此时每个页面仅需定义指示文件的局部变量，并包含模板页面，模板中包含侧边栏等通用页面，并通过局部变量包含专属页面
7. 指令中`<%@ page include="<%=desfile %/>" %>`不能识别为变量，会简单识别为不存在的URL，标签中`<jsp:include page="<%=desfile %/>" />` 可以识别为变量
8. `<jsp:include page=" "> </jsp:include>`中也可添加`<jsp:param name="" value="" />`标签，用于向包含目标组件传递请求参数

&nbsp;

## 5. JSP异常处理

1. 出现异常时可以 `<%@ page errorPage="errorpage.jsp" %>`，将请求转发给专门处理异常的页面

2. 应声明本页面是专门异常处理的页面 `<%@ page isErrorPage="true" %>`

3. 异常处理页面可以直接访问exception隐含对象，获取当前异常的详细信息，如

	`错误原因为：<% exception.printStackTrace(new PrintWriter(out)); %>`

4. 还可在web.xml中配置整个应用的通用错误处理页面：

	```xml
	<!-- 403错误时跳转至此 -->
	<error-page>
		<error-code>403</error-code>
	    <location>/403.html</location>
	</error-page>
	<!-- JSP或Servlet运行出现异常也可跳转 -->
	<error-page>
		<exception-type>java.lang.Exception</exception-type>
	    <location>/error.jsp</location>
	</error-page>
	```

	此法仍需在错误处理页面通过指令指定本页面为错误处理页面，且普通页面中若指定了所使用的错误处理页面，会遮盖xml中的全局设定

&nbsp;

## 6. JSP发布

JSP也可在web.xml中进行设置

```xml
<servlet>
	<servlet-name>hi</servlet-name>
    <jsp-file>/hello.jsp</jsp-file>
</servlet>
<servlet-mapping>
	<servlet-name>hi</servlet-name>
    <url-pattern>/hi</url-pattern>
</servlet-mapping>
```

&nbsp;

## 7. JSP预编译

首次访问JSP时需要容器进行JSP的翻译、编译、初始化、执行等操作，比较耗时，可以采用预编译。

1. JsP规范规定了一个特殊的请求参数`jsp_precompile`，若为true，则容器仅将该JSP转化为Servlet类并存放，但不运行

2. 预编译步骤：

	1. 访问URL指定参数为true，此时容器生成了Servlet源文件和类文件，存于`<CATALINMA_HOME>/work/Catalina/localhost/app_name/org/apache/jsp/`

	2. 手动将生成的类文件复制到应用的`WEB_INF/classes`的对应子目录下，实际为`app_name/WEB_INF/classes/org/apache/jsp/`

	3. web.xml配置

		```xml
		<servlet>
			<servlet-name>hi</servlet-name>
		    <servlet-class>org.apache.jsp.jsp_name</jsp-file>
		</servlet>
		<servlet-mapping>
			<servlet-name>hi</servlet-name>
		    <url-pattern>/hi</url-pattern>
		</servlet-mapping>
		```

	4. 删除原始jsp文件。此时将原始的jsp访问变成了Servlet访问。

3. 特点：提高JSP响应速度，防止JSP源码泄露，但依赖当前容器，兼容性差，其他版本或其他容器就不一定支持

&nbsp;

## 8. PageContext类

JSP API提供了PageContext类，继承自JSPContext，主要在Java程序段、JSP自定义标签的处理类中使用。

1. 类中关于各种范围的静态常量
	1. `PAGE_SCOPE` 1，页面范围
	2. `REQUEST_SCOPE` 2，请求范围
	3. `SESSION_SCOPE` 3，会话范围
	4. `APPLICATION_SCOPE` 4，Web应用范围
2. 本类特点：由容器创建，JSP文件内置pageContext引用对象变量
3. 向各种范围内存取属性的方法
	1. `getAttribute(String name)` 页面范围内属性
	2. `getAttribute(String name, int scope)` 指定范围内获取属性
	3. `setAttribute(String name, Object value, int scope)` 指定范围存放属性
	4. `removeAttribute(name, scope)`
	5. `findAttribute(name)` 从所有范围（从小到大）寻找属性，若无则null
	6. `int getAttributeScope(name)` 返回指定属性所属范围，失败则0
4. 获得由容器提供给其他对象的引用的方法（自定义标签类中无法使用默认的引用变量，就需要一下方法获取）
	1. `getPage()` 返回JSP对应Servlet的实例
	2. `getRequest()` 返回ServletRequest对象
	3. `getServletConfig()`
	4. `getServletContext`
	5. `getSession()`
	6. `getOut()` 返回一个用于输出响应正文的JspWriter对象
5. 用于请求转发和包含的方法（自定义标签类中无法用JSP标记实现，故需下法）
	1. `forward(path)` 请求转发
	2. `include(path)` 包含

&nbsp;

## 9. web.xml配置JSP

可用`<jsp-config>`元素对一组JSP文件配置，包括`<taglib>`和`<jsp-property-group>`两个子元素。

1. `<taglib>`

2. `<jsp-property-group>`子元素

	1. `<url-pattern>` 设定此配置所影响的JSP
	2. `<description>` 描述
	3. `<display-name>` JSP显示的名称
	4. `<el-ignored>` true为不支持EL语法
	5. `<scripting-invalid>` true为不支持`<% %>`Java程序段
	6. `<page-encoding>` 设定JSP文件字符编码
	7. `<include-prelude>` 设置自动包含的JSP头部文件
	8. `<include-coda>` 设置自动包含的JSP尾部文件

3. eg

	```xml
	<jsp-config>
		<jsp-property-group>
	    	<description>
	            Special property group for JSP configuration
	        </description>
	        <display-name>HomePage</display-name>
	        <url-pattern>*.jsp</url-pattern>
	        <el-ignored>true</el-ignored>
	        <page-encoding>GBK</page-encoding>
	        <scripting-invalid>true</scripting-invalid>
	    </jsp-property-group>
	
	    <jsp-property-group>
	    	<url-pattern>/mypath</url-pattern>
	        <el-ignored>true</el-ignored>
	        <include-prelude>/include/head.jsp</include-prelude>
	        <include-coda>/include/foot.jsp</include-coda>
	    </jsp-property-group>
	</jsp-config>
	```

	第一个`<jsp-property-group>`为所有.jsp结尾的JSP文件有效。第二个仅对应用根目录下path子目录下的JSP文件有效

&nbsp;

## 10. 注释

1. 显式注释：客户端可以看到。HTML风格：`<!-- ... -->`
2. 隐式注释：Java风格`// /**/`、JSP独有`<%-- --%>`

&nbsp;

## 11. 四大域对象

1. page范围：`pageContext`，只在页面中保存属性，服务端跳转后无效
2. request范围：`request`，只在一次请求有效，服务器跳转后有效，客户端跳转无效
3. session范围：`session`，只在一次会话范围，跳转也可使用，客户端服务端都可获取，但重开浏览器丢失，因session保存在浏览器中
4. application范围：`application`，在整个服务器保存，每个用户/session都可访问，若服务器重启则丢失
5. 方法：
	1. `setAttribute(String name, Object obj)`
	2. `getAttribute(String name)`
	3. `removeAttribute(String name)`

&nbsp;

## 12. 发展趋势

JSP为了简便编写动态页面而生，但HTML标记和Java程序段大量混合，使得开发维护都十分复杂，故后续产生各种方法将Java分离出去，只留HTML和JSP标签。

1. 分离Java：
	1. Java代码放在JavaBean中，JSP通过专门标签访问JavaBean
	2. 用EL表达式语言替换原本的`<%= %>`
	3. 使用自定义JSP标签
	4. 使用JSP的标准标签库JSTL
	5. Web应用采用基于MVC设计模式的框架，使JSP谓语视图层，仅展示数据，不负责流程控制和业务逻辑
2. JSP 2是对1.2的升级，引入了EL和自定义标签

