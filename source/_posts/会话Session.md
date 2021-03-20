---
title: 会话Session
mathjax: false
date: 2021-02-27 20:45:27
tags: 后端
---

&nbsp;

<!-- more -->

<!-- toc -->

&nbsp;

[toc]

# HTTP会话

Web服务器需要跟踪客户状态，用户多步操作时要能够知道是哪个用户在进行后续操作。跟踪状态常由四种方法：表单加入隐藏字段、重写URL使包含数据、Cookie传送数据、会话机制。

## 1. 会话 Session

1. HTTP协议是无状态的，多次HTTP通信中，协议本身没有提供服务器连续跟踪特定浏览器端状态的规范。会话指在一段时间内，单个客户与Web应用的一连串相关的交互过程。在一个会话中，客户可能多次访问Web应用
2. Servlet规范指定了基于Java的会话运作机制。Servlet API中定义了`javax.servlet.http.HttpSession`接口，容器要实现该接口。会话开始时，容器创建一个HttpSession对象，其中可以存放表示客户状态的信息。容器为每个HttpSession对象分配一个唯一标识符，称为 Session ID
3. 会话运作流程：
	1. 一个浏览器进程，第一次请求访问服务器中的某一Web应用中的任一支持会话的网页，容器试图寻找请求中表示Session ID的Cookie。由于首次访问，并无Cookie，于是创建一个HttpSession对象，为其分配唯一的Session ID，并将此ID作为Cookie添加到HTTP响应结果中。浏览器接收到HTTP响应结果后，把其中表示Session ID的Cookie保存在客户端
	2. 浏览器进程继续访问此应用中的任一支持会话的网页，本次请求中就包含了表示Session ID的Cookie。容器试图寻找请求中表示Session ID的Cookie。由于请求包含该Cookie，因此认定此次请求处于一个会话中，容器不在创建新HttpSession对象，而是从Cookie中获取Session ID，根据ID找到内存中对应的HttpSession对象
	3. 当前会话被销毁，HttpSession对象结束生命周期
4. JSP默认支持会话，也可显式声明 `<%@ page session="true" %>`
5. 支持会话的组件才能访问HttpSession对象

&nbsp;

## 2. 生命周期和范围

1. 会话范围：浏览器端与一个Web应用进行一次会话的过程；实际与HttpSession对象的生命周期对应，故组件只需共享同一HttpSession对象即可共享会话范围数据
2. HttpSession接口方法：
	1. `getId()` 返回Session ID
	2. `invalidate()` 销毁当前对话，容器释放HttpSession对象占用资源
	3. `setAttribute(String name, Object value)`
	4. `getAttribute(String name)` 
	5. `getAttributeNames()` 返回属性名数组
	6. `removeAttribute(String name)` 
	7. `isNew()` 判断是否是新创建的会话
	8. `setMaxInactiveInterval(int interval)`  设定一个会话可以处于不活动状态的最长时间s，超时后容器自动销毁对话。若负数，表示不限制。Tomcat默认1800s
	9. `getMaxInactiveInterval()` 读取当前会话可以处于不活动状态的最长时间
	10. `getServletContext()` 返回对话所属的Web应用的ServletContext对象
3. 开始新会话：首次访问或销毁会话后再次访问。后者请求带有Session ID的Cookie，但容器无法找到对应HttpSession对象，会创建新对象，开始新会话
4. 会话终止/HttpSession对象结束生命周期：浏览器进程终止（实际认为超时过期）、服务器端执行HttpSession对象的`invalidate()`方法、会话过期（一段时间内客户一直没有和Web应用交互）
5. Tomcat中Web应用终止时，会话不销毁，而是持久化，应用重启后Tomcat重新加载会话
6. JSP默认支持会话；HttpServlet默认不支持。Servlet中要获得HttpSession对象，要通过req对象
	1. `getSession()` 使得当前Servlet支持会话。若已存在对象就返回，否则新建并返回
	2. `getSession(boolean create)` 会话对象不存在时是否新建并返回，false且不存在时返回null
7. JSP是否支持会话？
	1. 默认支持
	2. 声明`<%@ page session="false" %>`则不支持
	3. 调用`session.invalidate()`后的代码不支持

&nbsp;

## 3. 跟踪会话其他方法之重写URL

1. 如浏览器不可用Cookie，容器可以重写Web组件的URL，把Session ID添加到URL信息中

2. HttpServletResponse接口提供了重写URL的方法 `public String encodeURL(String url)`

3. 使用：

	```jsp
	// check.jsp 中 链接login.jsp
	// 修改前
	<a href="login.jsp" />
	// 修改后
	<a href="<%= response.encodeURL("login.jsp") %>" />
	```

4. 流程：

	1. 先判断当前页面check.jsp是否支持会话（实际是这个链接时是否支持），若不支持，直接返回参数指定的URL（login.jsp）
	2. 再判断浏览器是否支持Cookie，若支持，返回参数指定的URL（login.jsp）；若不支持，就在参数指定的URL中加入当前Session ID，然后返回修改后的URL，相当于`<a href="login.jsp;jsessionid=..."/>`
	3. 即，只有当Web组件支持会话、浏览器端不支持会话时，encodeURL方法才会重写URL，否则直接返回原始URL

&nbsp;

## 4. 会话持久化

会话开始时容器创建HttpSession对象，需要时容器会将其存入永久性存储设备（文件、数据库），需要访问时再加载到内存。

1. 好处：节约内存空间，把不活动状态的HttpSession对象移出内存，提高内存利用率；确保服务器重启或Web应用重启后能恢复会话
2. 持久化会话时，容器不仅持久化HttpSession对象，还会将其所有可以序列化（实现了Serializable接口）的属性进行持久化，确保存放在会话范围内的共享数据不会丢失
3. 何时持久化（搁置）：
	1. 服务器或Web应用终止时
	2. 会话处于不活动状态太久，达到限制值
	3. Web应用中运行时状态的会话太多，达到限制值，部分会话搁置
4. 何时激活（加载）：
	1. 服务器或Web应用重启
	2. 处于会话中的客户端向Web应用发出HTTP请求，相应会话激活
5. 会话的搁置和激活对客户端来说是透明的，客户端感觉上会话始终运行时状态
6. Servlet API没有为持久化提供标准接口，持久化完全依赖于Servlet容器的具体实现，Tomcat采用会话管理器来管理会话，包含以下两种：标准会话管理器、持久化会话管理器

### 4.1 标准会话管理器 StandardManager

​	`org.apache.catalina.session.StandardManager`类

1. 是默认的标准会话管理器
2. 实现机制：服务器终止或Web应用终止时，会对应用的HttpSession对象持久化，保存到文件系统中，默认文件`<CATALINA_HOME>/work/Catalina/[hostname]/[applicationname]/SESSIONS.ser)`，当服务器或应用重启时，激活

### 4.2 持久化会话管理器 PersistentManager

更为灵活

1. 本管理器把存放HttpSession对象的永久性存储设备称为会话Store
2. 本管理器功能；
	1. 服务器关闭或重启、Web应用重启时，对应用的HttpSession对象持久化，存入会话Store
	2. 容错功能：及时把HttpSession对象备份到会话Store中，当服务器意外关闭重启后能恢复会话
	3. 灵活控制内存中的HttpSession对象数目，将部分对象转移到会话Store中
3. Tomcat中会话Store的接口为`org.apache.Catalina.Store`，目前提供了两个实现了该接口的类：
	1. `org.apache.Catalina.FileStore` 把HttpSession对象保存在文件中
	2. `org.apache.Catalina.JDBCStore` 保存在数据库的一张表中

#### 4.2.1 配置FileStore

1. 保存的目的文件的默认目录 `<CATALINA_HOME>/work/Catalina/[hostname]/[applicationname]`，每个HttpSession对象对应一个文件，以Session ID作为文件名，扩展名为.session

2. 在`webapp/META-INF/context.xml`中加入`<Manager>`元素配置

	```xml
	<Context>
		<Manager className="org.apache.catalina.session.PersistentManager" 
	             saveOnRestart="true"
	             maxActiveSessions="10"
	             minIdleSwap="60"
	             maxIdleSwap="120"
	             maxIdleBackup="180"
	             maxInactiveInterval="300">
	    	<Store 
	               className="...session.FileStore"
	               directory="mydir" />
	    </Manager>
	</Context>
	```

3. `<Manager>`元素专用于配置会话管理器，如采用PersistentManager，还需配置`<Store>`子元素

4. 各属性：

	1. className：会话管理器类名
	2. saveOnRestart：Web应用终止时保存会话对象
	3. maxActiveSessions：运行时状态的会话的最大数目，超过时转移一部分会话对象；如值-1，表示不限制
	4. minIdleSwap：指定会话处于不活动状态的最短时间s，超过后就有可能转移到会话Store中；-1表示不限制最短时间
	5. maxIdleSwap：会话处于不活动状态的最长时间s，超时则Tomcat必须把HttpSession对象移到Store；-1表示不限制最长时间
	6. maxIdleBackup：会话处于不活动状态的最长时间s，超时则Tomcat在Store中备份HttpSession对象，此时HttpSession对象仍处于内存中
	7. maxInactiveInterval：会话处于不活动状态的最长时间s，超时则Tomcat使本会话过期

#### 4.2.2 配置JDBCStore：

1. 数据库表的字段：

	1. session_id：Session ID
	2. session_data：HttpSession对象的序列化数据
	3. app_name：会话所属Web应用的名字
	4. session_valid：会话是否有效
	5. session_inactive：会话可以处于不活动状态的最长时间
	6. last_access：最近一次访问会话的时间

2. 首先需要MySQL中创建表，如表名为tomcat_sessions，位于数据库tomcatsessionDB

	```mysql
	create table tomcat_sessions (
		session_id varchar(100) not null primary key,
	    session_valid cahr(1) not null,
	    max_inactive int not null,
	    last_access bigint not null,
	    app_name varchar(255),
	    session_data mediumblob,
	    KEY kapp_name(app_name)
	);
	```

3. 在`webapp/META-INF/context.xml`中加入`<Manager>`元素配置

	```xml
	<Context reloadable="true">
		<Manager 
	             className="org.apache.catalina.session.PersistentManager"
	             saveOnRestart="true"
	             maxActiveSessions="10"
	             minIdleSwap="60"
	             maxIdleSwap="120"
	             maxIdleBackup="180"
	             maxInactiveInterval="300">
	        <Store 
	               className="...session.JDBCStore"
	               driverName="com.mysql.jdbc.Driver"
	               connectionURL="jdbc:mysql://localhost/tomcatsessionDB?user=jsy&amp;password=1234&amp;useSSL=false"
	               sessionTable="tomcat_session"
	               sessionIdCol="session_id"
	               sessionDataCol="session_data"
	               sessionValidCol="session_valid"
	               sessionMaxInactiveCol="max_inactive"
	               sessionLastAccessedCol="last_access"
	               sessionAppCol="app_name"
	               checkInterval="60" />
		</Manager>
	</Context>
	```

4. 为确保Tomcat服务器能够访问MySQL数据库，应将驱动程序类库jar复制到`webapp/WEB-INF/lib`下

5. `<Store>`属性：

	1. className：会话Store类名    
	2. driverName：数据库驱动程序类名
	3. connectionURL：访问数据库的URL
	4. sessionTable：存放HttpSession对象的表的名字
	5. checkInterval：Tomcat定期检查会话状态的时间间隔

&nbsp;

## 5. 监听会话

Servlet API中定义了四个用于监听会话中各种事件的监听器接口

1. `HttpSessionListener`接口，监听创建会话以及销毁会话事件

	1. `sessionCreated(HttpSessionEvent event)`：当容器创建一个会话后，调用此方法
	2. `sessionDestroyed(HttpSessionEvent event)`：容器将要销毁一个会话前，调用此方法

2. `HttpSessionAttributeListener`接口，监听向会话中加入、替换、删除属性的事件

	1. `attributeAdded(HttpSessionBindingEvent event)`：Web应用向一个会话中加入新属性时，容器调用此方法
	2. `attributeRemoved(HttpSessionBindingEvent event)`
	3. `attributeReplaced(HttpSessionBindingEvent event)`：应用替换了会话中一个已存在的属性的值，容器调用此方法

3. `HttpSessionBindingListener`接口：监听会话与一个属性绑定或解除绑定的事件

	1. `valueBound(HttpSessionBindingEvent event)`：Web应用把一个属性与会话绑定后，容器调用此方法
	2. `valueUnbound(HttpSessionBindingEvent event)`：Web应用将要解除绑定前，调用此方法

4. `HttpSessionActivationListener`接口：监听会话被激活和被搁置的事件

	1. `sessionDidActivate(HttpSessionEvent event)`：容器把一个会话激活后，调用
	2. `sessionWillPassivate(HttpSessionEvent event)`：容器将要把一个会话搁置前，调用

5. 前两个监听器，必须在web.xml文件中通过`<listener>`元素向容器注册：首先定义类实现接口，如MySessionLifeListener，实现了两接口（方法），然后xml中配置

	```xml
	<web-app>
		<listener>
	    	<listener-calss>package.MySessionLifeListener</listener-calss>
	    </listener>
	</web-app>
	```

6. 后两个监听器，由会话的属性实现。如MyData类的对象会作为会话的属性与会话绑定，并希望监听MyData对象的绑定、解绑、会话激活、搁置的事件，就要让MyData类实现接口，类中实现各个方法