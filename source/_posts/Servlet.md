---
title: Servlet
mathjax: false
date: 2021-02-24 20:57:00
tags: 后端
---

&nbsp;

<!-- more -->

<!-- toc -->

&nbsp;



[toc]

# Servlet

## 1. HTTP协议

1. 超文本传输协议，客户端请求和响应的标准协议。
2. URL：协议+地址+端口+资源路径+参数，`http://localhost:8080/myweb/sr01?name=zhangsan`
3. 特点：
	1. C/S模式
	2. 简单快速
	3. 灵活：允许传输任意类型数据，传输类型用`Content-Type`标记
	4. 无连接、1.1版本支持有连接甚至流水线
	5. 无状态：协议对事务没有记忆能力，后续处理用到之前数据要重传
4. 请求：请求行（一行）+请求头（请求状态，多行）+请求正文（仅POST请求有）
5. 响应：状态行（一行）+响应头（响应状态，多行）+响应正文
6. 报头域：键值对，中以冒号+空格间隔，键名大小写无关

&nbsp;

## 2. Servlet API

一个Servlet样例

```java
@WebServlet("/location")
public class MyServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // actions
    }
}
```

1. 代码结构：

	1. WebServlet标注，内容为本Servlet的各种设置
	2. 继承HttpServlet类
	3. 复写service方法（内含doGet和doPost方法，自动处理）

2. 工作流程：

	1. 通过请求头确定要访问的主机
	2. 通过请求行确定要访问的应用
	3. 通过请求行的路径确定要访问的资源
	4. 通过4中资源路径，匹配项目的真实路径
	5. 若首次访问Servlet对象，服务器会创建实例，并init方法
	6. 调用service方法处理请求
	7. 方法完毕，服务器将response缓冲区数据取出，以HTTP响应的格式发送给浏览器


### 2.1 Servlet接口

本接口是API的核心，所有Servlet类都需实现，包含5个方法

1. `init(ServletConfig config)`：容器在创建好Servlet对象后调用本方法
2. `service(ServletRequest req, ServletResponse res)`：容器接收到客户端访问特定Servlet对象的请求时调用
3. `destroy()`：释放Servlet对象占用资源，结束生命周期时调用
4. 以上方法均由容器自行调用
5. `getSerletConfig()`：返回ServletConfig对象，包含了初始化参数信息
6. `getServletInfo()`：返回一个包含了Servlet创建者、版本、版权等信息的字符串

&nbsp;

### 2.2 GenericServlet抽象类

1. 本抽象类实现了Servlet、ServletConfig和Serializable接口，且HttpServlet抽象类是本类的子类。
2. 本类实现了init方法，传入的ServletConfig对象，由私有变量config引用，此时GenericServlet对象即于ServletConfig对象关联。
3. 同时本类还定义了不带参数的`init()`方法，带参init会调用不带参init，故若要覆盖父类的初始化，则可覆盖父类的不带参init，或覆盖带参init（此时若需关联config，则要`super.init(config)`先用父类init关联）。
4. 本类未实现service方法，这是唯一抽象方法。其他方法虽不是抽象，但多数无实际意义。
5. destory方法中，可以释放所占资源，如文件流关闭，数据库关闭等。
6. 本类实现了ServletConfig的所有方法，可直接调用。

&nbsp;

### 2.3 HttpServlet抽象类

1. 是GenericServlet的抽象子类，本类为Servlet接口提供了与HTTP协议相关的实现，即本类适合运行在与客户端采用HTTP协议通信的servlet容器或web服务器中。JavaWeb开发时定义类一般扩展本类。

2. 本类为HTTP的每种请求方式提供了响应的服务方法，如`doGet()`、`doPost()`、`doPut()`、`doDelete()`等。

3. 本类实现了`service(ServletRequest req, ServletResponse res)`方法，内部将ServletRequest和ServletResponse强转为HttpServletRequest和HttpServletResponse，之后调用重载方法`service(HttpServletRequest req, HttpServletResponse resp)`。重载方法中通过`res.getMethod()`获取请求方式，然后调用不同请求方式对应的不同do方法。

4. 本类的各种默认实现的do方法，都会客户端返回一个错误：

	1. 若HTTP1.1通信，那么返回`HttpServletResponse.SC_METHOD_NOT_ALLOWED`即错误405
	2. 若不是HTTP1.1，则返回``HttpServletResponse.SC_BAD_REQUEST`即错误400

	故子类中应覆盖各种do方法。？

&nbsp;

### 2.4 ServletRequest接口

1. Servlet接口的service方法中，有此类型的参数。容器接收到访问Servlet请求时，将原始请求数据包装成一个ServletRequest对象，容器调用service方法时就传入该对象。

2. 本接口提供了一系列获取请求数据的方法：

	1. `getContentLength()` 返回请求正文长度/-1
	2. `getContentType()` 获取请求正文的MIME类型/null
	3. `getInputStream()` 返回读取请求正文的输入流
	4. `getLocalAddr()` 服务器端IP
	5. `getLocalName()` 服务器端主机名
	6. `getLocalPort()` 服务器端FTP端口号
	7. `getParameter(String name)` 根据请求参数名获取参数值
	8. `getProtocol()` 通信协议名称和版本号
	9. `getReader()` 用于读取字符串形式的请求正文的BufferedReader对象
	10. `getRemoteAddr()` 客户端IP
	11. `getRemoteHost()` 客户端主机名
	12. `getRemotePort()` 客户端FTP端口号

	以下是定义了一组用于在请求范围内存取共享数据的方法

	1. `setAttribute(String name, Object)` 在请求范围内保存一个属性（即要共享的数据），参数是属性名与值
	2. `getAttribute(String name)` 根据name获取属性值
	3. `removeAttribute(String name)` 在请求范围中删除一个属性

&nbsp;

### 2.5 HttpServletRequest接口

1. 是2.4中ServletRequest的子接口

2. 本接口提供了读取HTTP请求中相关信息的方法

	1. `getContextPath()` 客户端请求访问的Web应用的URL入口，如客户端访问`http://localhost:8080/helloapp/info`，则本方法返回`/helloapp`，即Web应用名字
	2. `getCookies()` 返回HTTP请求中的所有Cookie
	3. `getHearder(String name)` 返回HTTP请求头部的特定项
	4. `getHearNames()` 返回一个Enumeration对象，包含HTTP请求头部的所有项目名
	5. `getMethod()` Http请求方式
	6. `getRequestURI()` HTTP请求头部第一行的URI，请求行中的资源名称部分（项目名称开始）
	7. `getQueryString()` HTTP请求中的查询字符串，即`?`后的内容，参数部分

3. `getRequestURL()` 完整URL

	9. `getProtocol()` HTTP版本号
	10. `getParameter(name)` 获取指定名称的参数值（参数均为键值对）
	11. `getParameterValues(name)` 获取指定名称参数的所有值

	由此可见，Servlet容器已经将HTTP请求进行了解析，只需get方法获取即可。

&nbsp;

### 2.6 ServletResponse接口

1. Servlet通过本接口对象生成响应结果。当容器接收到访问Servlet请求时，创建一个本接口对象，作为参数传给service方法。
2. 定义了一系列与生成响应结果相关的方法：
	1. `setHeader("content-type", "text/html");` 设置响应MIME类型
	2. `set/getCharacterEncoding(String charser)` 响应正文的字符编码，默认编码为ISO-8859-1不支持中文，GB2312支持简中，其扩展GBK支持简繁中
	3. `setcontentLength()` 响应正文长度
	4. `set/getContenttype(String type)` 响应正文的MIME类型（ServletResponse中默认MIME是`text/plain`纯文本，而HttpServletResponse中默认MIME是`text/html`HTML文档）
	5. `set/getBufferSize(int size)` 存放响应正文数据的缓冲区大小
	6. `reset()` 清空缓冲区内正文数据，并清空响应状态代码和响应头
	7. `resetBuffer()` 仅清空缓冲区正文数据
	8. `flushBuffer()` 强制把缓冲区响应正文数据发送到客户端
	9. `isCommitted()` 若true，表示缓冲区数据已发送到客户端
	10. `getOutputStream().write(string.getBytes())` 返回ServletOutputStream对象，用于输出二进制正文数据，字节流，可响应一切数据
	11. `getWriter().write(string)` 返回PrintWriter对象，用于输出字符串形式正文数据，字符流，响应字符
	12. 响应回的数据到客户端被解析
	13. 两种输出流不可同时使用
3. 本对象主要用于产生HTTP相应结果正文部分。
4. 缓冲区数据发送到客户端的时机：
	1. 缓冲区已满，自动发送，并清空缓冲区
	2. 调用本对象的`flushBuffer()`方法
	3. 调用ServletOutputStream对象或PrintWriter对象的flush或close方法
5. 为确保缓冲区数据完全发送，应输出数据完毕后调用ServletOutputStream对象或PrintWriter对象的`close()`方法。Tomcat中，若service未执行close，则自动调用。
6. MIME和字符编码的设置（其set方法），要先于输出（get输出方法）。

&nbsp;

### 2.7 HttpServletResponse接口

1. 是2.6的ServletResponse接口的子接口，提供了与HTTP协议相关的一些方法，可用于设置响应头或向客户端写Cookie：
	1. `addHeader(String name, String value)` 向响应头加入内容
	2. `sendError(int sc)` 向客户端发送HTTP错误状态响应码
	3. `sendError(int sc, String msg)` 向客户端发送响应码和具体信息
	4. `setHeader(String name, String value)` 设置响应头的一项内容，会覆盖已存在项
	5. `setStatus(int sc)` 设置响应状态码
	6. `addCookie(Cookie cookie)` 向相应中加入Cookie
	7. `sendRedirect(String path)` 重定向
2. 本接口中定义了一些响应码静态常量
	1. `HTTPServletResponse.SC_BAD_REQUEST` 400
	2. `HTTPServletResponse.SC_FOUND` 302
	3. `HTTPServletResponse.SC_METHOD_NOT_ALLOWED` 405
	4. `HTTPServletResponse.SC_NON_AUTHORITATIVE_INFORMATION` 203
	5. `HTTPServletResponse.SC_FORBIDDEN` 403
	6. `HTTPServletResponse.SC_NOT_FOUND` 404
	7. `HTTPServletResponse.SC_OK` 200

&nbsp;

### 2.8 ServletConfig接口

1. Servlet接口的init方法接收此参数。当Servlet容器初始化一个Servlet对象时，对为其创建一个ServletConfig对象，其中包含了Servlet初始化参数信息。init中会将Servlet对象与ServletConfig对象关联。
2. 此外，ServletConfig对象还与当前web应用的ServletContext对象关联。
3. 本接口有以下方法：
	1. `getInitParameter(String name)` 根据参数名获取参数值
	2. `getInitParameterNames` 返回一个Enumeration对象，包含所有参数名
	3. `getServletContext()` 返回一个ServletContext对象
	4. `getServletName()` 即web.xml中，`<servlet-name>`的值，若未指定则返回类名
4. 每个初始化参数包括一对名和值，在web.xml中配置Servlet时，可以通过`<servlet>`内的`<init_param>`来设置初始化参数，有`<param_name>`和`<param_value>`两个子元素。
5. GenericServlet类实现了本接口，HttpServlet类继承GenericServlet类，故在其及其子类中可调用本接口方法。

&nbsp;

### 2.9 ServletContext接口

1. ServletContext是Servlet和容器通信的接口。容器在启动一个web应用时，会为其创建一个ServletContext对象，即web应用有一个专属对象。本web应用下的所有Servlet对象都共享这个“总管家”，通过它访问容器的资源：
2. 在web应用范围内存取共享数据：
	1. `set/getAttribute(String name, Object)` 将一个对象与name绑定，存入ServletContext
	2. `getAttributeNames()` 返回Enumeration对象，包含所有存放于ServletContext的属性名
	3. `removeAttribute(String name)` 删除属性
3. 访问当前web应用的资源：
	1. `getContextPath()` 当前web应用的URL入口
	2. `getInitParameter(String name)` 根据参数名，返回web应用范围内的匹配的初始化参数值。在web.xml中，在`<web-app>`根元素下定义的`<context_param`表示应用范围内的初始化参数
	3. `getInitParameterNames()` Enumeration对象，包含web应用范围内的所有初始化参数名
	4. `getServletContextName()` web应用名字，即web.xml中`<display-name`的值
	5. `getRequestDispathcer(String path)` 返回一个用于向其他web组件转发请求的RequestDispathcer对象
4. 访问Servlet容器中的其他web应用
	1. `getContext(String uripath)` 根据uri，返回当前容器中其他web应用的ServletContext对象
5. 访问容器相关信息
	1. `getMajorVersion()` 容器支持的Java Servlet API的主版本号
	2. `getMinorVersion()` 容器支持的Java Servlet API的次版本号
	3. `getServerInfo()` 容器的名字和版本
6. 访问服务器端文件系统资源
	1. `getRealPath(String path)` 根据参数指定的虚拟路径，返回文件真实路径
	2. `getResource(String path)` 返回一个映射到参数指定路径的URL
	3. `getResourceAsStream(String path)` 返回一个用于读取指定文件的输入流，默认从当前Web应用根目录下取，故path也如此
	4. `getMimeType(String file)` 返回参数指定文件的MIME类型
7. 输出日志
	1. `log(String msg)` 向Servlet日志文件写日志
	2. `log(String msg, Throwable)` 写错误日志及异常堆栈信息
	3. 默认日志输出到`<CATALINA_HOME>/logs/localhost.YYYY-MM-DD.log`文件
8. ServletConfig接口中定义了`getServletContext()`方法，故其实现类、继承类均可直接调用本方法获得ServletContext对象。

&nbsp;

## 3. Java Web应用的生命周期

Java Web生命周期由Servlet容器控制，共三阶段：

1. 启动：加载Web应用数据，创建ServletContext对象，对filter和一些Servlet初始化
2. 运行时：提供服务
3. 终止：释放Web应用占用资源

### 3.1 启动阶段

1. web.xml加载到内存
2. 为Java Web应用创建一个ServletContext对象（此时有ServletContextListener监听，并调用方法）
3. 对所有Filter初始化
4. 对启动阶段需要初始化的Servlet初始化

### 3.2 运行时阶段

此时，所有Servlet待命，随时响应请求，提供服务。若请求的Servlet还不存在，容器就先加载并初始化Servlet，再调用service方法。

### 3.3 终止阶段

1. 销毁应用中所有运行时状态的Servlet
2. 销毁应用中所有运行时状态的Filter
3. 销毁所有与应用相关的对象，如ServletContext，并释放占用资源（此时有ServletContextListener监听，并调用方法）

### 3.4 Tomcat管理平台管理应用生命周期

1. `<CATALINA_HOME>/conf/tomcat-users.xml`中加入`<user username="name" password="password" roles="manager-gui"/>`
2. 启动Tomcat，访问`http://localhost:8080/manager/html`，即可进入管理平台（也是一个web应用，位于`webapps/manager`）
3. 在将web应用发布到Tomcat时，可为其配置`<Context>`元素，内有reloadable属性，若true，则Tomcat运行时监视应用的`WEB-INF/classes`和`WEB-INF/lib`目录下类文件的改动，若检测到更新，会自动重启应用。默认false，开发阶段可设为true。

&nbsp;

## 4. Servlet生命周期

也由容器控制，分为初始化、运行时、销毁阶段，Servlet接口的init、service。destroy方法分别在三个状态调用。

此处说明一点，为Servlet命名时，可以有多个名字，则代表有多个Servlet对象（使用同一类）。

### 4.1 初始化阶段

1. 初始化步骤：
	1. 容器加载Servlet类，把clss文件读入内存
	2. 创建ServletConfig对象，包含特定Servlet的初始化配置信息，如初始化参数；并且容器将ServletConfig对象与当前web应用的ServletContext对象关联
	3. 容器创建Servlet对象
	4. 容器调用Servlet对象的`init(ServletConfig)`方法，GenericServlet会将Servlet与ServletConfig关联
2. 以上初始化结束后，Servlet对象可通过`getServletContext()`直接得到应用的ServletContext对象
3. 以下情况，Servlet会进入初始化阶段：
	1. web应用运行时阶段，特定Servlet被首次请求访问
	2. web.xml中为某个Servlet设置了`<load-on-startup> num </load-on-startup>`元素，则容器启动应用时就初始化该Servlet，num表示在此阶段Servlet被初始化的顺序（0开始）（初始化：实例化并调用其init()方法，并非service方法）
	3. 当web应用重启时，所有Servlet都会在特定时间被重新初始化

### 4.2 运行时阶段

此时Servlet随时响应请求。

1. 容器接收到访问请求，为Servlet创建ServletRequest和ServletResponse对象，调用相应Servlet对象的service方法
2. 当容器将结果发送给客户端，就会销毁ServletRequest和ServletResponse对象

### 4.3 销毁阶段

当应用终止，容器先调用应用中所有Servlet对象的destroy方法，然后再销毁这些对象。在destroy方法中释放占用资源。

此外容器还会销毁与Servlet对象关联的ServletConfig对象，ServletRequest和ServletResponse不会销毁。

&nbsp;

## 5. ServletContext与Web应用范围

1. ServletContext与Web应用有同样的生命周期
2. 范围：时间段，与可共享数据的Web组件集合
3. Web应用范围：Wen应用生命周期构成的时间段 与 Web应用生命周期内所有Web组件的集合
4. 存放在Web应用范围内的共享数据：共享数据的生命周期位于Web生命周期的一段内 且 共享数据可被Web应用中的所有组件共享
5. 由于ServletContext与Web应用生命周期相同，故用其存放Web应用范围的共享数据
6. 由于Web终止时，销毁ServletContext对象，故此时存中的共享数据/属性一并销毁

### 5.1 ServletContextListener

1. Servlet API中有此接口，能够监听ServletContext对象的生命周期，即Web应用的生命周期

2. 当容器启动或终止Web应用时，会触发ServletContextEvent事件，由ServletContextListener处理。接口中定义了处理事件的两个方法：

	1. `contextInitialized(ServletContextEvent sce)` 当容器启动应用时调用此方法，之后再对Filter初始化，初始化必要Servlet
	2. `contextDestroyed(ServletContextEvent sce)` 当容器终止应用时调用，先销毁Servlet和Filter，再调用此法（此时甚至不能调用ServletContext中的共享数据）

3. 自定义的监听器也需web.xml中注册

	```xml
	<Listener>
	    <listener-class>MyServletContextListener</listener-class>
	</Listener>
	```

	也可用标注配置`@WebListener`

&nbsp;

## 6. Servlet.service()的异常

`public void service(ServletRequest req, res) throws ServletException, java.io.IOException`

ServletException表示常规操作时异常，IOException表示进行IO时异常。

1. ServletException有子类UnavailableException，表示无法访问当前Servlet的异常。若Servlet由于一些系统级别的原因（内存不足、无法访问第三方服务器如数据库服务器等）而不能响应请求，就可抛出此异常。
2. UnavailableException的两个构造方法：
	1. `UnavailableException(String msg)` 创建一个表示Servlet永远不能被访问的异常，此后除非重启Web应用，否则再也无法通过浏览器访问此Servlet
	2. `UnavailableException(String msg, int seconds)` 创建一个表示Servlet暂时不能被访问的异常，seconds表示暂时不能访问的时间，若为0或负数表示无法估计暂且不能被访问的时间
3. service方法抛出的异常由容器捕获，并向客户端发送错误信息。

&nbsp;

## 7. 防止页面被客户端缓存

1. 浏览器为了能够快速响应请求，会把来自服务端的页面存放至缓存。

2. 但缓存技术适用于静态网页，及不包含敏感数据的网页

3. 可通过一下方式禁止缓存：

	```java
	resp.addHeader("Pragma", "no-cache");	// HTTP/1.0
	resp.setHeader("Cache-Control", "no-cache");	// HTTP/1.1
	resp.setHeader("Expires", "0");	// 两版本协议均支持，表示网页过期时间，0即立即过期，需要从服务器获取最新数据
	```

&nbsp;

## 8. Annotation标注配置Servlet

从Servlet3开始，为简化开发，可不必在web.xml配置，直接在类中Annotation标注配置。

### 8.1 Servlet的标注配置

1. 各属性用法：

	| 属性           | 类型           | 描述                                                        |
	| -------------- | -------------- | ----------------------------------------------------------- |
	| name           | String         | 指定Servlet名字，`<servlet-name>`，未指定则默认类的全限定名 |
	| urlPatterns    | String[]       | 指定一组URL匹配模式，`<url-pattern>`                        |
	| loadOnStartup  | int            | 指定Servlet启动时加载顺序，`<load-on-startup>`              |
	| initParams     | WebInitParam[] | 指定一组初始化参数，`<init-param>`                          |
	| asyncSupported | boolean        | 声明Servlet是否支持异步处理模式，`<async-supported>`        |
	| desciption     | String         | 指定Servlet描述信息，`<description>`                        |
	| displayName    | String         | 指定Servlet显示名，通常配合工具使用，`<display-name>`       |

2. eg：

	```xml
	<servlet>
		<servlet-name>Font</servlet-name>
	    <servlet-class>mypack.FontServlet</servlet-class>
	    <init-param>
	    	<param-name>color</param-name>
	        <param-value>blue</param-value>
	    </init-param>
	    <init-param>
	    	<param-name>size</param-name>
	        <param-value>15</param-value>
	    </init-param>
	</servlet>
	<servlet-mapping>
		<servlet-name>Font</servlet-name>
	    <url-pattren>/font</url-pattren>
	</servlet-mapping>
	```

	```java
	@WebServlet(name="Font", 
	            urlPatterns={"\font"},
	           initParams={
	               @WebInitParam(name="color", value="blue"),
	               @WebInitParam(name="size", value="15")
	           })
	public class FontServlet extends HttpServlet {...}
	```

3. 仅指定一个urlPatterns时，可直接`@WebServlet("/hello")`

4. 配置方式二选一：若配置固定，可标注形式，否则web.xml更集中管理方便

### 8.2 ServletContextListener的标注配置

```xml
<Listener>
    <listener-class>MyServletContextListener</listener-class>
</Listener>
```

```java
@WebListener
public class MyServletContextListener implements ServletContextListener  {}
```

&nbsp;

## 9. 中文字符编码

请求和响应的编码设定，要在获取输入输出流之前

### 9.1 HTTP请求

接收客户端参数并解析时，部分浏览器默认使用ISO-8859-1编码，不支持中文，会乱码.

1. 单独对参数更改编码：`username = new String(username.getBytes(ISO8859-1), "GBK");` ，或UTF-8
2. POST：接收数据前 `req.setCharacterEncoding("UTF-8");`，或GBK
3. ServletContext也有`set/getRequestCharacterEncoding`方法，对整个Web应用修改，从Servlet 4开始
4. 以上方法都是对请求正文的设置编码，而GET的参数位于请求头。Tomcat8起，GET方式不会乱码
5. 在Tomcat的server.xml中对HTTP连接器更改配置，自动把URI中的请求参数转换编码，一劳永逸`<Connection ... URIEncoding="UTF-8" />`
6. 不同浏览器传递的请求参数采用字符编码不同，常用的四种GBK、GB2312、ISO-8859-1、UTF-8，可通过`str.equals(new String(str.getBytes(encodingType), encodingType))`判断str是否是encodingType类型编码

### 9.2 HTTP响应

`resp.setCharacterEncoding("GBK");`

`resp.setContentType("text/html;charset=GBK");`


&nbsp;

## 10. 下载文件

1. IDEA中，项目根目录下不可新增文件夹，要在“web”下新建“store”目录存放上传下载文件

2. 前台超链接方法：前台a标签跳转时，若资源无法被浏览器识别则下载，可识别资源也可以通过设置download属性实现下载。该属性为空使用原文件名，指定属性即为下载的文件名。

  `<a href="download/avatar.jpg" download="dragon.jpg">头像图片</a>`

3. 后台代码方法：

  4. 需要通过`response.setContentType`方法设置Content-type头字段的值为浏览器无法使用某种方式或激活某个程序来处理的MIME类型，例如`application/octet-stream`、`application/x-msdownload`、`application/force-download`等。
  5. 需要通过`response.setHeader`方法设置`Content-Disposition`头的值为`attachment;filename=文件名`
  6. 读取下载文件，调用`response.getOutputStream`方法向客户端写入附件内容。  

  ```java
  public class DownloadServlet extends HttpServlet {
      @Override
      protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          System.out.println("file download....");
          req.setCharacterEncoding("UTF-8");
          // 获取文件名并判空
          String fileName = req.getParameter("fileName");
          System.out.println("filename: " + fileName);
          if (fileName == null || "".equals(fileName.trim())) {
              resp.getWriter().write("input name again.");
              resp.getWriter().close();
              return;
          }
          // 确定文件存在，开始下载
          String path = req.getServletContext().getRealPath("/download/");
          File file = new File(path + fileName);
          if (file.exists() && file.isFile()) {
              // 输入流
              InputStream in = new FileInputStream(file);
              //in = getServletContext().getResourceAsStream(path);
              // 响应类型
              resp.setContentType("application/x-msdownload");
              // 响应头
              resp.setHeader("Content-Disposition", "attachment;filename=" + fileName);
              resp.setHeader("Content-Length", String.valueof(in.available()));
              // 字节输出流
              ServletOutputStream out = resp.getOutputStream();
              byte[] bytes = new byte[1024];
              int len = 0;
              while ((len = in.read(bytes)) != -1) {
                  out.write(bytes, 0, len);
              }
              // 关闭资源
              out.close();
              in.close();
          }else {
              resp.getWriter().write("file doesn`t exists.");
              resp.getWriter().close();
          }
      }
  }
  
  ```

&nbsp;

## 11. 上传文件

### 11.1 前台

前台：表单、post、指定的enctype数据类型`multipart/form-data`，表示复杂的包括多个子部分的复合表单。

```jsp
<form method="post" enctype="multipart/form-data" action="UpLoad">
    姓名：<input type="text" name="uname"> <br>
    文件：<input type="file" name="myfile"> <br>
    <button>提交</button>
</form>
```

此请求发至后台，请求会被以下两方法看作多个子部分：HTTP请求头不被考虑进内，主要是正文被分为多个子部分：

1. 普通表单域：
	1. 此子部分的请求头
		1. `Content-Disposition:`
			1. `form-data;`
			2. `name="uname"` 前端`<input>`标签指定的name
	2. 此子部分正文：`<input>`输入内容
2. 文件域：
	1. 子部分请求头
		1. `Content-Disposition:`
			1. `form-data;`
			2. `name="myfile"` 前端`<input>`标签指定的name
			3. `filename="C:\...\file1.rar"` 此处完整路径名
		2. `Content-Type:application/octet-stream` 此文件MIME类型根据文件不同自动形成
	2. 子部分正文：即上传文件的内容

&nbsp;

### 11.2 后台

容器会将HTTP请求包装为HttpServletRequest对象，当请求正文为`multipart/form-data`类型时，Servlet从HttpServletRequest中解析复合表单子部分仍然十分繁琐，为了简化，可使用Apache开源类库或Servlet API提供的Part接口（Servlet 3.0出现）。

#### 11.2.1 Apache

1. Apache开源类库：提供了两个与上传文件相关的包：

	1. fileupload（commons-fileupload-X.jar），[下载地址](http://commons.apache.org/fileupload)
	2. I/O（commons-io-X.jar），[下载地址](http://commons.apache.org/io)

	两个jar文件放于`app/WEB-INF/lib`下，Servlet利用fileupload包的类和接口完成上传，而此包依赖于I/O包。

2. fileupload包中主要类和接口：

	1. FileItem接口
	2. FileItemFactory接口，依赖于FileItem接口，是创建FileItem对象的工厂
	3. DiskFileItem类，实现了FileItem接口，表示基于硬盘的FileItem，能够把客户端上传的文件数据保存到硬盘
	4. DiskFileItemFactory类，实现了FileItemFactory接口，依赖于DiskFileItem类，是创建DiskFileItem对象的工厂
	5. ServletFileUpload类，与FileItemFactory接口关联，是文件上传处理器

3. fileupload软件包把`multipart/form-data`类型HTTP请求正文中，复合表单包含的每个子部分看作是一个FileItem对象，此类对象有两种类型：formFiled普通表单域如文本域及提交按钮，非formFiled即上传文件类型

4. 包实现中，为提高效率，会使用缓存和临时目录存放临时数据，而ServletFileUpload作为文件上传处理器，与工厂接口关联，能够解析请求对象，返回一组FileItem对象的List集合

	```java
	// 创建工厂
	DiskFileItemFactory factory = new DiskFileItemFactory();
	// 设置缓冲区，此处4k
	factory.setSizeThreshold(4 * 1024);
	// 设置临时目录
	factory.setRepository(new File(tempFilePath));
	
	// 创建文件上传处理器
	ServletFileUpload upload = new ServletFileUpload(factory);
	// 设置允许上传的文件最大尺寸，此处4MB
	upload.setSizeMax(4 * 1024 * 1024);
	// 解析HTTPServletRequest请求
	List<FileItem> items = upload.parseRequest(request);
	// 遍历集合，分别处理
	for (FileItem item : items) {
	    if (item.isFormFiled()) 
	        processFormFile(item, out);	// 处理普通表单域
	    else 
	        processUploadedFile(item, out);	// 处理上传文件
	}
	
	// processFormFile()
	void processFormFile(FileItem item, PrintWriter out) {
	    String name = item.getFieldName();
	    String value = item.getString();
	    out.println(...);
	}
	
	// processUploadedFile()
	void processUploadedFile(FileItem item, PrintWriter out) throws Exception {
		String fileName = item.getName();
	    int idx = fileName.lastIndexOf("\\");
	    fileName = fileName.substring(idx+1, fileName.length());
	    long fileSize = item.getSize();
	    if (fileName.equals("") && fileSize == 0)
	        return;
		File uploadFile = new File(filePath + "/" + fileName);
	    item.write(uploadFile);
	    out.println(fileName + " is saved. Size is " + fileSize);
	}
	```

5. 因要用到上传文件存储目录和临时目录，故可在web.xml或标注中，将两路径作为参数，Servlet.init()中直接从ServletConfig中获取即可

	```xml
	<servlet>
	    <init-param>
			<param-name>filePath</param-name>    	
	         <param-value>store</param-value>
	    </init-param> 
	    <init-param>
	    	<param-name>tempFilePath</param-name>
	    	<param-value>temp</param-value>
	    </init-param>
	</servlet>
	```

	```java
	public class UploadServlet extends HttpServlet {
	    private String filePath;
	    private String tempFilePath;
	    
	    public void init(ServletConfig config) throws ServletException {
	        super.init(config);
	        filePath = config.getInitParameter("filePath");
	        filePath = getServletContext().getRealPath(filePath);
	        tempFilePath = config.getInitParameter("tempFilePath");
	        tempFilePath = getServletContext.getRealPath(tempFilePath);
	    }
	}
	```

	&nbsp;

#### 11.2.2 Part

Servlet将`multipart/form-data`的POST请求封装成Part，通过Part对上传的文件操作。

1. 注解`@MultipartConfig`将一个Servlet标识为支持文件上传
2. 会将复合表单的每个子部分看作一个Part对象。`HttpServletRequest.getParts()`方法返回Part对象集合，每个对象代表请求中复合表单的一个子部分
3. Part对象方法：
	1. `getHeader(String name)` 读取子部分请求头中特定选项值
	2. `getContentType()` 读取子部分请求正文的数据类型
	3. `getSize()` 读取子部分的请求正文长度
	4. `getName()` 读取子部分名字，与HTML表单中`<input>`元素的name值对应
	5. `write(String filename)` 把子部分请求正文写到参数指定文件中



```java
@WebServlet("/UpLoad")
@MultipartConfig
public class UploadServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.setCharacterEncoding("UTF-8");
        System.out.println("file upload...");

        // 直接根据前端元素name处理
        String uname = req.getParameter("uname");
        System.out.println("uname: " + uname);

        Part part = req.getPart("myfile");
        String fileName = part.getSubmittedFileName();	// Tomcat8
        System.out.println("file name: " + fileName);

        String filePath = req.getServletContext().getRealPath("/");
        System.out.println("file path: " + filePath);
        part.write(filePath + "/" + fileName);
        
        // 遍历Part
        Collection<Part> parts = request.getParts();
        for (Part part : parts) {
            if (part.getContentType() == null) {	// 文本域
                String name = part.getName();
                String value = req.getParameter(name);
            } else if (part.getName().indexOf("file") != -1) {	
                // 文件域，此处“file”是前端指定的name
                String head = part.getHeader("Content-Disposition");
                // head : "form-data; name="myfile"; filename="uploadedfile.rar"
			   String filename = cd.substring(cd.lastIndexOf("=")+2, head.length()-1);
                // Tomcat8使用上述part.getSubmittedFileName()即可
            }
        }
    }
}
```

&nbsp;

## 12. 动态生成图像

1. 所需类：

	1. `java.awt.image.BufferedImage` 位于缓存中的图像
	2. `java.awt.Graphics` 画笔
	3. `javax.imageio.ImageIO` 把原始图像转换为特定格式，利用外界提供输出流来输出

2. ImageIO类以`write()`静态方法输出，会产生临时文件，需在Tomcat根目录下事先创建temp目录

	`public static boolean write (RenderedImage im, String formatName, File output) throws IOException`

3. 绘图实例

	```java
	resp.setContentType("image/jpeg");
	var out = resp.getOutputStream();
	// 创建一个缓存中的图像，长高11*16
	var image = new BufferedImage(11, 16, BufferedImage.TYPE_INT_RGB);
	// 画笔
	var g = image.getGraphics();	
	g.setColor(Color.white);
	var font = new Font("Courier", Font.BOLD, 12);
	g.setFont(font);
	g.drawString("..");
	g.drawLine(...);
	// 输出JPEG
	ImageIO.write(iamge, "jpeg", out);
	out.close();
	```

4. 前端`<img>`标签：`<img src='image?param=12'/>` image表示自定义绘图ImageServlet，param=12表示传给该Servlet的参数（动态生成img，相当于此元素需要再次发送请求给后台）

&nbsp;

## 13. Cookie

1. 是客户端访问服务器时，服务器存于客户端硬盘的信息，服务器可以根据Cookie跟踪客户状态，尤其便于区别客户端。但保存在客户端，安全性差。

2. Cookie建立过程：

	1. 客户端首次请求访问服务器时，发送一个不含Cookie的请求
	2. 服务器的响应中包含一个含有客户端信息的Cookie（Cookie位于协议头）
	3. 客户端解析响应，把Cookie保存到本地硬盘
	4. 之后的请求中，包含Cookie
	5. 浏览器收到含有Cookie的请求，就可获取客户端信息

3. 原始Cookie数据的解析也由Servlet容器完成。`javax.servlet.http.Cookie`类可操作Cookie。每个Cookie对象包含一个Cookie名和值。

4. 服务器端创建与发送：

  ```java
  Cookie cookie01 = new Cookie("name", "admin");
  resp.addCookie(cookie01);
  ```

5. 服务器端获取：仅有`getCookies`一个方法，返回数组，需要遍历并`getName() getValue()`，若无Cookie则返回null

  ```java
  Cookie[] cookies =  req.getCookies();
  if (cookies != null && cookies.length > 0) {
      for (Cookie temp :
           cookies) {
          String name = temp.getName();
          String value = temp.getValue();
          System.out.println("name: " + name + ", value: " + value);
      }
  }
  ```

6. 到期时间：默认关闭浏览器即失效。`setMaxAge(int)`设置到期时间，秒

  7. -1（或负数）：默认就是-1，表示不存储，只在内存存活，关闭浏览器就失效
  8. 正整数：存储到磁盘
  9. 0：删除cookie

10. 注意点：

  11. 保存在当前浏览器，换电脑或浏览器均不能使用
  12. 中文：默认不能存储中文，需要`URLEncoder.encode()`编码存储，`decode()`解码读取
  13. 发送重名cookie会覆盖
  14. 有数量上限

15. 路径：决定能否读取到cookie 

  16. `cookie.setPath("/")` 即为服务器根路径，表示当前服务器下所有项目都可访问
  17. `cookie.setPath("/pro1")` 当前项目下可访问，默认为此
  18. `cookie.setPath("/pro2")` 仅项目pro2下可访问，即使是pro1项目创建的cookie
  19. `cookie.setPath("/pro1/cook")` 仅项目pro1下的cook目录下可访问
  20. 当设置了路径且访问路径在设置的路径下，cookie就会加载到req对象中
  21. `cookie.setDomain(".cat.tom")` 表示可使其他服务器`www.cat.tom`的所有应用访问本Cookie

&nbsp;

## 14. 访问Web应用的工作目录

1. 每个Web应用下都有一个工作目录，容器将应用相关临时文件存于此。Tomcat默认工作目录是`<CATALINA_HOME>/work/[enginename]/[hostname]/[contextpath]`
2. 显式指定：配置Web应用的`<Context>`元素时，指定`workDir`属性
3. 访问：除容器，Servlet也可访问工作目录。Servlet规范规定容器初始化应用时，要向创建的ServletContext设置`javax.servlet.context.tempdir`的属性，属性值是一个`java.io.File`对象，代表当前应用的工作目录。故Servlet获取应用工作目录`File workDir = (File)context.getAttribute("javax.servlet.context.tempdir")`

&nbsp;

## 15. 转发和包含

1. 从Servlet2.1开始，Servlet内无法获取其他Servlet的引用，也就无法调用其他Servlet，为完成组件协作，Servlet规范提供了两种途径：请求转发（源组件处理一部分后将请求转发给目标组件）和包含（源组件把其他组件/目标组件生成的响应结果包含到自身的响应结果中）
2. 二者特点：
	1. 源组件与目标组件，处理的是同一请求，共享一份ServletRequest和ServletResponse对象
	2. 目标组件可以是Servlet、JSP、HTML
	3. 都依赖`javax.servlet.RequestDispatcher`接口
	4. 地址栏的URL不变，整个过程仅一个请求
3. RequestDispatcher请求分发器，有两个方法：
	1. `public void forward(ServletRequest request, ServletResponse response) throws ServletException, java.io.IOException` 把请求转发给目标组件
	2. `public void include(ServletRequest request, ServletResponse response) throws ServletException, java.io.IOException` 包含目标组件响应结果
4. 获取RequestDispatcher对象：
	1. `servletContext.getRequestDispatcher(String path)` 目标组件绝对路径
	2. `servletRequest.getRequestDispatcher(String path)` 绝对/相对路径
	3. url起始不加/

### 15.1 请求转发

1. 流程：清空存放响应正文的缓冲区；若目标组件是Servlet或JSP，就调用其service方法，把该方法结果发送到客户端；若目标组件是HTML，就把文档发送到客户端
2. 特点：
	1. 源组件不能提交响应结果，否则forward()时会报错，仅目标组件可响应，数据通信用`req.setAttribute()`
	2. 源组件转发后，后面的代码仍会执行

### 15.2 包含

1. 流程：目标组件Servlet或JSP，就将其service产生的响应正文添加到源组件相应结果中；若HTML就直接把文档内容添加到源组件响应结果
2. 特点：目标组件中对响应状态码和响应头所做的修改都会被忽略

### 15.3 请求范围

1. Web应用范围/ServletContext：整个Web应用的生命周期
2. 请求范围/ServletRequest：服务器响应一次客户请求的过程，从接收到返回；容器接收到请求，创建req和resp对象传给相应Servlet，待容器返回给客户结果后，俩对象销毁
3. 

&nbsp;

## 16. 重定向

1. 流程：
	1. 客户端输入URl访问服务器某个组件
	2. 服务器端组件返回302结果和另一个组件的URL，表示让浏览器在请求访问该组件。该组件不一定在同一服务器上
	3. 浏览器接收到响应结果后立即自动请求访问另一组件
	4. 浏览器接收到来自另一组件的响应结果
2. 方法：`HttpServletRequest.sendRedirect(String location)`
3. 特点：
	1. 源组件响应结果不会发送到客户端
	2. 源组件重定向前已提交响应结果会抛出异常
	3. 源组件与目标组件不共享ServletRequest对象，不能共享请求范围内的数据
	4. 参数location若以`/`开头，表示**相对于当前服务器根路径的URL**，要`/webapp/...`，若以`http://`开头表示完整URL
	5. 目标组件不必是同一服务器的同一Web应用的组件，可以是网络上任一有效网页

&nbsp;

## 17. 访问Servlet容器内其他Web应用

1. 一个容器进程内可以同时运行多个Web应用，各应用之间通过ServletContext通信
2. `ServletContext.getContext(String uripath)`用于获取其他Web应用的ServletContext对象，参数指定其他应用的URL入口，如`/app1`
3. 为了安全起见，多数容器允许用户设置是否允许应用得到其他应用的ServletContext对象。Tomcat中，`<Context>`元素的crossContext属性用于设置本项。默认false，调用上述方法返回null

&nbsp;

## 18. 避免并发问题

1. 并发访问同一Servlet，容器通常会为每个请求分配一个工作线程，线程们并发执行同一个Servlet.service()方法
2. 但多线程引入线程并发问题，处理原则：
	1. 根据实际需求，决定Servlet中变量作用域，是实例变量还是局部变量
	2. 多线程同时访问共享数据而导致并发问题时，采用Java同步机制对线程同步
	3. `javax.servlet.SingleThreadModel`接口已废弃
3. 合理决定变量作用域：
	1. 不同变量作用域：
		1. 局部变量：方法中定义，执行方法时线程堆栈中创建局部变量，线程执行完方法时局部变量销毁。多线程时，每个线程有独立的局部变量	
		2. 实例变量：即类的成员，每个对象（类的实例）有独立的实例变量。实例销毁，实例变量才销毁。若多线程执行同一实例的方法，则访问到同一组实例变量
	2. 由于每个请求对应一个线程，故如需每个请求对应单独变量时，使用局部变量；而需每个请求都对应同一变量时，使用实例变量
4. Java同步机制
	1. 当需每个请求都对应同一变量，使用实例变量时，要使用同步机制对多线程同步
	2. 可使用锁对象或synchronized关键字，锁方法或代码块
5. 废弃的SingleThreadModel接口：
	1. 如Servlet实现此接口，可采用以下方式之一运行Servlet：
		1. 任一时刻仅允许一个工作线程执行service方法，其余请求放入等待队列，依次响应，这实际上禁止了多客户端对同一Servlet的并发访问
		2. 容器为Servlet创建对象池，其中存放多个对象，每个请求的线程使用一个实例对象。此方法一来使得不同请求访问不同实例对象，二来请求过多时要创建过多实例对象，消耗内存资源
	2. 从Servlet API 2.4开始废弃，Tomcat采用上述方法2

&nbsp;

## 19. 对请求的异步处理

### 19.1 Servlet中的异步

1. Servlet API 3.0之前，容器对请求分配各自线程，这些线程来自主线程池的空闲线程，但请求过多，每个线程耗时过长时，会极大降低并发访问性能，故3.0开始引入异步处理机制，3.1加入非阻塞IO进一步增强异步处理的性能
2. Servlet异步处理机制：Servlet从HttpServletRequest对象中获得一个AsyncContext对象，表示异步处理的上下文。它把响应当前请求的任务传给另一个新线程，新线程完成处理请求并向客户端返回结果的任务。而最初由容器分配给请求的线程就可及时释放回主线程池，从而及时处理更多请求。故Servlet异步处理机制，就是把响应请求的任务传递给另一个线程
3. AsyncContext接口的方法：
	1. `setTimeout(long timeout)` 设置异步线程处理请求任务的超时时间（毫秒），异步线程必须在参数时间内完成任务
	2. `start(Runnable run)` 启动一个异步线程，执行参数run指定的任务
	3. `addListener(AsyncListener listener)` 添加一个异步监听器
	4. `complete()` 通知容器任务完成，返回响应结果
	5. `dispatch(String path)` 把请求派发给参数指定的Web组件
	6. `getRequest()` 获取当前上下文中的req对象
	7. `getResponse()`

### 19.2 异步流程

1. 开启asyncSupport属性，使Servlet支持异步处理

	```java
	@WebServlet(name="...",
	           urlPatterns="/...",
	           asyncSupported=true)
	```

	```xml
	<servlet>
		<servlet-name>...</servlet-name>
	    <servlet-class>...</servlet-class>
	    <async-supported>true</async-supported>
	</servlet>
	```

2. service方法中，获得AsyncContext对象

	```java
	AsyncContext asyncContext = req.startAsync();
	```

3. 调用AsyncContext对象的setTimeout方法，设置处理请求的异步线程超时时间（非必须）

4. 启动异步线程执行处理请求的任务，启动方法有三种：

  1. `aysncContext.start(Runnable)`，参数可自定义一个实现了Runnable接口的新类，为的是将AsyncContext对象传入该任务类的实例变量，能够在run方法中执行complete方法。AsyncContext对象的start方法的实现取决于具体容器，有的容器额外维护一个线程池，从中取出线程用于响应，有的容器从主线程池中取线程；后者并未改进性能

  2. `new Thread(Runnable).start()` 此法亲自创建新线程，并发访问量大时导致创建大量新线程，降低运行性能

  3. 利用Java API的线程池ThreadPoolExecutor类创建一个线程池，所有异步线程都放在此池中。此法可灵活设定池。
  	
  	```java
  	private static ThreadPoolExecutor executor = new ThreadPoolExecutor(100, 200, 50000L, TimeUnit.MILLISECONDS, new ArrayBlockingQueue<>(100));
  	// 代表池中最少100线程，最多200，允许的线程空闲时间，空闲时间单位，池使用的缓冲队列
  	
  	service() {
  	    ...
  	    executor.execute(Runnable);	// 从池中取出一个空闲线程执行任务
  	}
  	
  	destory() {
  	    executor.shutdownNow();	// 需显式关闭
  }	
  	```
  	

5. 调用AsyncContext对象的complete方法，通知容器已完成任务，或dispatch方法，把请求转发给其他Web组件

### 19.3 异步监听器AsyncListener

1. 可用AsyncListener捕获并处理异步线程运行中的特定事件

	1. `onStartAsync(AsyncEvent event)` 异步线程开始时调用
	2. `onError(event)` 异步线程出错时调用
	3. `onTimeout(event)` 异步线程执行超时时调用
	4. `onComplete(event)` 异步线程执行完毕时调用

2. 如例
	
	```java
	AsyncContext asyncContext = req.getAsyncContext();
	asyncContext.setTimeout(60 * 1000);
	asyncContext.addListener(new AsyncListener() {
	    // 复写四个监听方法
	});
	asyncContext.start(Runnable);
	```

### 19.4 非阻塞I/O

1. 阻塞I/O：

	1. 线程通过输入流执行读操作时，若输入流的可读数据还未准备好，则当前线程进入阻塞状态，当读到数据或到达数据末尾，线程才从读方法退出。如上传大文件，网络传输耗时，后台读数据阻塞
	2. 线程通过输出流写操作时，若因为某些原因，暂时不能向目的地写数据，就阻塞，只有完成了写数据的操作才从写方法中退出。如发送大文件。

2. 非阻塞I/O：

	1. 线程输入流读操作，数据未准备好，则线程立即退出读方法。只有输入流中有可读数据才进行读操作
	2. 线程输出流写操作，暂时不能写数据时立即退出写方法。只有可以写时才进行写操作

3. Java中传统读写都是阻塞I/O。当异步线程用阻塞I/O读写时，会使异步线程阻塞，还是削弱了并发性能。故从Servlet API 3.1开始，引入非阻塞I/O机制，建立在异步基础上

4. 实现方法：引入两个监听器

	1. ReadListener接口：监听ServletInputStream输入流行为
		1. `onDataAvailable()` 输入流中有可读数据时触发此方法
		2. `onAllDataRead()` 输入流中所有数据读完时触发此方法
		3. `onError(Throwable)` 输入操作出现错误时触发此方法
	2. WriteListener接口：监听ServletOutputStream输出流行为
		1. `onWritePossible()` 可以向输出流写数据时触发此方法
		2. `onError(Throwable)` 输出操作出现错误时触发此方法

5. 操作步骤：在支持异步处理的Servlet类中进行非阻塞I/O

	```java
	// 得到输入输出流
	ServletInputStream input = req.getInputStream();
	ServletOutputStream output = resp.getOutputStream();
	
	// 注册监听器
	input.setReadListener(new MyReadListener(input, asyncContext));
	output.setWriteListener(new MyWriteListener(output, asyncContext));
	
	// MyReadListener.class
	public class MyReadListener implements ReadListener {
	    private ServletInputStream imput;
	    private AsyncContext asyncContext;
	    private StringBUilder builder = new StringBuilder();
	    
	    public MyReadListener (ServletInputStream input, AsyncContext asyncContext) {
	        this.input = input;
	        this.asyncContext = asyncContext;
	    }
	    
	    public void onDataAvailable() {
			try {
	            int len = -1;
	            byte[] buff = new byte[1024];
	            // 读取客户端提交的数据
	            while (input.isReady() && (len = input.read(buff)) > 0) {
	                String data = new String(buff, 0, len);
	                builder.append(data);
	            }
	        } catch (Exception ex) {
	            ex.printStackTrace();
	        }
	    }
	    
	    public void onAllDataRead() {
	        // 将数据设为req范围属性，并派发给负责输出的Servlet组件
	        asyncContext.getRequest.setAttribute("msg", builder.toString());
	        asyncContext.dispatch("/output");
	    }
	    
	    public void OnError (Throwable t) {
	        t.printStackTrace();
	    }
	}
	```

6. 当主线程使用监听器时，service方法会先于读取结束，可见容器会提供一个异步线程来执行监听器中的非阻塞I/O操作

&nbsp;

## 20. 服务器端推送

1. 如最新消息推送、原请求包含其他链接时的主动发送等，都需主动推送。
2. HTTP/2引入服务器推送概念，Servlet API 4开始提供服务器推送支持，有PushBuilder接口实现
	1. `path(String path)` 指定待推送资源的URL
	2. `push()` 把path指定资源推送到客户端

&nbsp;

## 21. Session

1. 介绍：
	1. HttpSession对象是同名接口的实例，无父接口。session本身就是http协议的内容。
	2. session标识一次会话/确认一个用户（一个用户的多次请求），并在会话期间共享数据（更长的生命周期）
	3. 对于服务器，每个连接来的客户端都是一个session。Servlet容器用此接口创建客户端和服务器的会话，保留指定时长，跨多个连接或请求。一个会话通常对应一个用户，可用session查看和操作会话属性。
	4. session保留在浏览器，切换浏览器无法取得之前的会话。
2. `req.getSession()`获取session对象。
3. JSession标识符：
	1. 会话的唯一标志sessionId
	2. 名为JSESSIONID的cookie，请求中如果访问了session，则服务器会创建一个名为JSESSIONID，值为获取到的session（无论是获取到的还是新创建的）的sessionId的cookie对象，并添加到response对象中，响应给客户端，有效时间为关闭浏览器。所以Session的底层依赖Cookie来实现。
	3. 每当一次请求到达服务器，如果开启了会话（访问了session），服务器第一步会查看是否从客户端回传一个名为JSESSIONID的cookie，如果没有则认为这是一次新的会话，会创建一个新的session对象，并用唯一的sessionId为此次会话做一个标志。如果有JESSIONID 这个cookie回传，服务器则会根据JSESSIONID这个值去查看是否含有id为JSESSION值的session对象，如果没有则认为是一个新的会话，重新创建一个新的 session对象，并标志此次会话；如果找到了相应的session对象，则认为是之前标志过的一次会话，返回该session对象，数据达到共享。
4. session域对象：`session.setAttribute(key, value)` `req.getAttribute(key)` `session.removeAttribute(key)`。数据存储在session域对象中。
5. 生命周期：
	1. 默认时长：当客户端第一次请求 servlet 并且操作session时，session 对象生成，Tomcat 中 session 默认的存活时间为30min，即你不操作界面的时间，一旦有操作，session 会重新计时。  
	2. `session.setMaxInactiveInterval(int)`按秒设置最大不活动时间或get。
	3. `session.invalidate()` 立刻失效
	4. 关闭浏览器：jsession基于cookie，若cookie有效期默认，关闭浏览器后session也结束。
	5. 关闭服务器：session销毁。



## 22. 域对象总结

1. request域对象：一次请求中有效。请求转发有效，重定向无效。
2. session域对象：一次会话中有效。请求转发和重定向都有想，session销毁无效。
3. ServletContext域对象：整个应用程序有效，服务器关闭失效。



## 23. web.xml

为Servlet映射URL既可以通过xml，也可以用标注`@WebServlet(/serlvetName)`

```xml
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>Dispatcher</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/dispatcher</url-pattern>
</servlet-mapping>
```

1. `servlet为Servlet指定名字`
	1. `servlet-name`为Servlet指定名字
	2. `servlet-class`指定具体类的完整路径/类名
	3. `init-param`定义初始化参数（参数名和值），可有多个本元素
	4. `load-on-startup`指定Servlet容器启动web时加载Servlet的次序，小数字优先级高，0或未指定则表示访问时才加载
2. `servlet-mapping`为其映射一个URL
	1. `url-pattern`指定相对URL路径，可一个Servlet映射多个URL

3. `welcome-file-list`
	1. `welcome-file>`用于为web设置默认主页



## 24. URL总结

1. Servlet前的`@WebServlet("/servletName")`，此为Servlet的名字，并且在地址栏中显示。
2. HTML中的`action="servletName"`，同1中URL，但不加`/`，表示要调用的Servlet。
3. 请求转发时，url要加`/`。

&nbsp;

## cookie保存密码免登录的例子

```java
/**
 * Cookie04.java
 * 用例测试，保存密码
 */

@WebServlet("/cookie04")
public class Cookie04 extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        boolean hasName = false;
        boolean hasPwd = false;
        String uname = null;
        String upwd = null;

        // 先检测有无用户名和密码的cookie
        String name = null;
        String value = null;
        Cookie[] cookies = req.getCookies();
        if (cookies != null && cookies.length > 0) {
            for (Cookie cooike :
                    cookies) {
                name = cooike.getName();
                value = cooike.getValue();
                if (name.equals("uname")) {
                    hasName = true;
                    uname = value;
                    System.out.println("has uname: " + value);
                }
                if (name.equals("upwd")) {
                    hasPwd = true;
                    upwd = value;
                    System.out.println("has upwd: " + value);
                    // resp.getWriter().write("uname: " + uname + ", upwd: " + upwd);
                }
            }
        }

        if (hasName && hasPwd) {
            // 姓名密码都有，不用登陆，直接转发至首页
            System.out.println("have two, index");
            req.getRequestDispatcher("index.jsp").forward(req, resp);
        } else {
            // cookie不足，需要登录，转发至登录页面
            System.out.println("have no, login");
            req.getRequestDispatcher("login.jsp").forward(req, resp);
        }
    }
}
```

```java
/**
 * Cookie05.java
 * 用例测试，保存密码
 * 登录页面调用此Servlet，设置cookie后转发至首页
 */

@WebServlet("/cookie05")
public class Cookie05 extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String uname = req.getParameter("uname");
        String upwd = req.getParameter("upwd");
        Cookie nameCookie = new Cookie("uname", uname);
        Cookie pwdCookie = new Cookie("upwd", upwd);
        resp.addCookie(nameCookie);
        resp.addCookie(pwdCookie);

        req.getRequestDispatcher("index.jsp").forward(req, resp);
    }
}
```