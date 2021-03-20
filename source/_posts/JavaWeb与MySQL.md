---
title:  JavaWeb与MySQL
mathjax: false
date: 2021-02-27 20:47:01
tags: 后端
---

&nbsp;

<!-- more -->

<!-- toc -->

&nbsp;

[toc]



# JavaWeb与MySQL

## 1. JDBC

Java Database Connectivity，连接Java程序和数据库服务器的纽带。

### 1.1 简介

1. JDBC API主要位于java.sql包，此外javax.sql中包含了一些高级特性
2. 实现包括三部分：
	1. 驱动管理器：java.sql.DriverManager类，由Oracle公司实现，负责注册特定JDBC驱动器，以及根据特定驱动器建立与数据库的连接
	2. 驱动器API：由Oracle公司制定，最主要的接口是java.sql.Driver接口
	3. 驱动器：由数据库供应商或其他第三方工具提供商创建，实现了JDBC驱动器API，负责与特定数据库连接，以及处理通信细节；驱动器可以注册到驱动管理器中
3. API共两组：JDBC API负责Java应用程序与数据库连接，JDBC驱动器API则是第三方创建驱动器时要实现的API
4. 实际上JDBC驱动器才是真正连接Java程序与特定数据库的纽带；程序若要访问某种数据库，需要先获得相应JDBC驱动器的类库，注册到JDBC驱动管理器中
5. JDBC驱动器分为四类：
	1. JDBC-ODBC驱动器：Open Database Connectivity是微软为应用程序提供的访问任何一种数据库的标准API；本驱动器使Java程序能间接访问ODBC API，是唯一由Oracle公司实现的驱动器，属于JDK一部分，默认已在驱动管理器注册；但速度慢，JDK8开始已删除
	2. 部分Java程序代码和部分本地代码组成的驱动器：用于与数据库客户端API通信，使用时不仅需要Java类库，还要安装平台相关的本地代码
	3. 完全由Java语言编写的类库：用与具体数据库服务器无关的协议将请求发送给服务器的特定组件，由该组件按照特定数据库协议对请求进行翻译，发送给数据库服务器
	4. 完全由Java语言编写的类库：直接按照特定数据协议，把请求发送给数据库服务器
	5. 以上速度由慢到快

&nbsp;

### 1.2 java.sql中的接口和类

JDBC API主要位于本包。

1. Driver接口和DriverManager类：

	1. 所有JDBC驱动器都要实现Driver接口，编写访问数据库的Java程序时，要先将特定数据库的JDBC驱动器的类库加入到clsspath中
	2. DriverManager类用来建立和数据库的联机以及管理JDBC驱动器，方法均静态：
	3. `registerDriver(Driver)` 在DriverManager中注册JDBC驱动器
	4. `deregisterDriver(Driver)` 注销驱动器
	5. `getConnection(String url, String user, String pwd)` 建立和数据库的连接，返回表示连接的Connection对象
	6. `setLoginTimeOut(int sec)` 等待连接超时时间
	7. `setLogWriter(PrintWriter out)` 设定输出JDBC日志的对象

2. Connection接口：

	1. 代表程序和数据库的连接
	2. `getMetaData()` 返回表示数据库原数据的DatabaseMetaData对象；原数据包含了数据库的相关信息
	3. `createStatement()` 创建返回Statement对象
	4. `prepareStatement(String sql)` 创建返回PreparedStatement对象

3. Statement接口：

	1. 执行SQL语句的方法
	2. `execute(String sql)` 执行各种SQL，返回boolean，true表示有查询结果，通过Statement的`getResultSet()`获得结果
	3. `executeUpdata(String sql)` 执行insert、update、delete，返回int表示库中受影响的记录的数目
	4. `executeQuery(String sql)` 执行select，返回表示查询结果的ResultSet对象

4. PreparedStatement接口

	1. 继承自Statement接口，用于执行预准备的SQL语句（相同格式，参数不同的SQL语句），简化代码，提高访问性能，只需编译一次SQL语句，多次执行，而Statement需多次编译

	2. 用法：

		```java
		String selectStatement = "select * from BOOKS where NAME = ? and PRICE = ?";
		PreparedStatement predStmt = connection.prepaedStatement(selectStatement);
		prepStmt.setString(1, "Tom");	// 替换第一个?
		prepStmt.setFloat(2, 50);		// 替换第二个?
		ResultSet rs = perpStmt.executeQuery();
		```

5. ResultSet接口

	1. 表示select结果集，next()方法使游标定位到结果集的下一条记录，get方法获取一条记录中某个字段的值

	2. `getString(int columnIndex)` 返回参数指定字段的String类型的值

	3. `getString(String columnName)` 返回参数指定字段的String类型的值

	4. `getInt(int / String)` 返回指定字段int类型的值

	5. `getFloat(int / String)` 返回指定字段float类型的值

	6. 索引从1开始

	7. next方法：结果集中游标初始在第一个记录之前，需next才能指向第一个记录；最终在最后一个记录之后，返回false

	8. 访问所有结果：

	  ```java
	  while (rs.next()) {
	      String col1 = rs.getString(1);
	      String col2 = rs.getString(2);
	      String col3 = rs.getString(3);
	      float col4 = rs.getFloat(4);
	      System.out.println("ID:" + col1 + " NAME:" + col2 + " TITLE:" + col3 + " PRICE:" + col4);
	  }
	  ```

&nbsp;

### 1.3 访问数据库程序

1. ~~获得数据库JDBC驱动器类库，放在classpath下（IDEA可根据MySQL版本选择驱动器下载放置，或手动下载后从IDEA打开驱动器位置更换），但操作中发现新建lib放入jar包，右键add as library更稳定（普通Java程序）~~ jar放在`webapp/WEB-INF/lib`下，或放在`<CATALINA_HOME>/lib`下（Web项目）
2. 程序中加载注册驱动器，部分驱动器类加载时自动注册，MySQL即是
3. connection的url `jdbc:mysql://parameters` 后可接`?useUnicode=true&characterEncoding=GB2312`等参数
  4. `useUnicode`默认false
  5. `characterEncoding`默认false
  6. `autoReconnect` 数据库连接异常中断时是否自动重连，默认false
  7. `autoReconnectForPools` 针对数据库连接池的重连策略，默认false
  8. `failOverReadOnly` 自动重连后是否设置为只读，默认true
  9. `maxReconnetx` 自动重连尝试次数，默认3
  10. `initialTimeout` 自动重连间隔，默认2s
  11. `connectTimeout` 和库服务器建立socket连接时的超时时间ms，0表示永不超时，默认0
  12. `useSSL` 是否进行SSL安全验证，默认true
  13. 参数各版本或有改动
14. MySQL驱动程序会自动运行“Abandoned Connection Cleanup Thread”线程，当Web应用终止时，本线程可能未终止，为防止内存泄漏，可设置一个MyServletContextListener，它的`contextDestroyed()`方法调用`AbadonedConnectionCleanupThread.checkedShutdown()`方法，确保线程关闭。（下有例子）

```java
// 一个简单的访问数据库例子

// 加载MySQL Driver类
Class.forName("com.mysql.jdbc.Driver");
// java.sql.DriverManager.registerDriver(new com.mysql.jdbc.Driver());
// 建立与数据库的连接
Connection connection = DriverManager.getConnection(
        "jdbc:mysql://localhost:3306/bookdb", "jsy", "jiumima1870353-");
// 创建Statement对象，准备执行SQL语句
Statement statement = connection.createStatement();
// 执行SQL语句
String sql = "select ID, NAME, TITILE, PRICE from books";
ResultSet resultSet = statement.executeQuery(sql);
// 访问记录集
while (resultSet.next()) {
    String id = resultSet.getString(1);
    String name = resultSet.getString(2);
    String title = resultSet.getString(3);
    float price = resultSet.getFloat(4);

    System.out.println("ID:" + id + ", name:" + name + ", title:"
            + title + ", price:" + price);
    req.setAttribute("id", id);
    req.setAttribute("name", name);
    req.setAttribute("title", title);
    req.setAttribute("price", price);
}
// 依次关闭对象
resultSet.close();
statement.close();
connection.close();
```

```java
// 用于关闭线程的监听器
public class MyServletContextListener implements ServletContextListener {
    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println("bookstore app is Destroyed.");

        try {
            // 关闭MySQL的AbandonedConnectionCleanupThread
            com.mysql.cj.jdbc.AbandonedConnectionCleanupThread.checkedShutdown();
        } catch (Exception e) {
            e.printStackTrace();
        }

        try {
            Enumeration<Driver> drivers = DriverManager.getDrivers();
            while (drivers.hasMoreElements()) {
                // 注销jdbc驱动程序
                DriverManager.deregisterDriver(drivers.nextElement());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

&nbsp;

### 1.4 事务处理

事务：由一条或多条SQL语句组成的不可分割的工作单元。

1. 只有当一个事务中的所有操作都正常完成，整个事务才能被提交到数据库，否则必须撤销整个事务
2. Connection接口中三个相关方法：
	1. `setAutoCommit(boolean)` 是否自动提交事务，默认开启
	2. `commit()` 提交事务
	3. `rollback()` 撤销事务
3. JDBC中默认自动提交，将每个SQL语句看作一个事务
4. 可先set方法关掉自动提交，事务操作完成后手动commit；若某条SQL操作失败，会产生相应SQLException，catch中rollback即可

&nbsp;

## 2. 数据源 DataSource

### 2.1 简介

JDBC API提供了`javax.sql.DataSource`接口，负责建立与数据库的连接；在应用程序中连接数据库不必编写连接库的代码，可直接从数据源获得数据库连接。

1. 数据源和数据库连接池：数据源中事先建立了多个数据库连接，保存在连接池中。Java程序访问数据库时，只需从连接池中取出空闲状态的数据库连接；访问结束，再将数据库连接放回连接池，可以提高访问库效率；如每次收到前台请求，都与库新建连接，访问结束断开连接，则耗费大量资源（连接时，库验证用户名和密码，为连接分配资源，Java要把Connection对象加载入内存等）
2. 数据源和JNDI资源：DataSource对象通常由Servlet容器提供，程序无需自己创建，获取容器创建的对象的引用，要依赖Java的JNDI（Java Naming and Directory Interface）技术，将对象和名字绑定，对象工厂负责生产和唯一名字绑定的对象，外部程序通过名字获得某个对象的引用。`javax.naming`包中有Context接口，提供了将对象和名字绑定，通过名字检索对象的方法
	1. `bind(String name, Object obj)` 
	2. `lookup(String name)` 返回名字绑定的对象
3. Tomcat把DataSource作为一种可配置的JNDI资源来处理。生成DataSource对象的工厂是`org.apache.commons.BasicDataSourceFactory`
4. 数据源容器维护，故用此方法连接数据库时，驱动器jar包应放在Tomcat的lib下，不能放在应用的lib下

&nbsp;

### 2.2 配置数据源

修改`context.xml`和`web.xml`

1. context.xml中加入`<Resource>`元素，用来定义JNDI资源。Tomcat中数据源是JNDI资源的一种。context.xml在META-INF目录下创建。本元素具有以下属性：

	1. `name`：指定Resource的JNDI名字
	2. `auth`：指定管理Resource的Manager，可选值Container表示有容器创建和管理，Application表示有Web应用创建和管理
	3. `type`：指定Resource所属Java类名
	4. `maxActive`：指定数据库连接池中处于活动状态的数据库连接的最大数目，0表示不受限制
	5. `maxIdle`：处于空闲状态的最大数目，0表示不受限制
	6. `maxWait`：处于空闲状态的最长时间ms，超时抛出异常，-1表示无限期等待
	7. `username`：连接数据库的用户名
	8. `password`
	9. `driverClassName`：连接数据库的JDBC驱动器的Driver实现类的名字
	10. `url`：连接数据库的URL，如有多个参数要用&连接时，xml中的&用转义字符`&amp;`表示，如下例

	```xml
	<Context reloadable="true">
		<Resource name="jdbc/BookDB" 
	              auth="Container"
	              type="javax.sql.DataSource"
	              maxActive="100" maxIdle="30" maxWait="10000"
	              username="dbuser" password="1234"
	              driverClassName="com.mysql.jdbc.Driver"
	              url="jdbc:mysql://localhost:3306/BookDB?autoReconnect=true&amp;useUnicode=true&amp;characterEncoding=GB2312&amp;useSSL=false" />
	</Context>
	```

2. 若需数据源被容器内一个虚拟主机中的多个Web应用访问，可以在`<CATALINA_HOME>/conf/server.xml`的`<Host>`元素中配置`<Resource>`子元素

3. web.xml中加入`<resource-ref>`元素：

	1. 若Web应用访问了由容器管理的JNDi资源，就要在web.xml中声明对这个JNDI资源的引用
	2. 子元素：
		1. `description`：所引用资源的说明
		2. `res-ref-name`：所引用资源的JNDI名字，与`<Resource>`中的name属性对应
		3. `res-type`：所引用资源的类名字，与`<Resource>`中的type属性对应
		4. `res-auth`：所引用资源的Manager，与`<Resource>`中的auth属性对应

	```xml
	<webapp>
		<resource-ref>
	    	<description>DB connection</description>
	        <res-ref-name>jdbc/BookDB</res-ref-name>
	        <res-type>javax.sql.DataSource</res-type>
	        <res-auth>Container</res-auth>
	    </resource-ref>
	</webapp>
	```

	&nbsp;

	### 2.3 程序中访问数据源

	1. 获得数据源的引用：`javax.naming.Context` 

	2. 通过DataSource对象的getConnection方法获得数据库连接

	3. 访问结束后，调用Connection对象的close方法，将Connection对象返回连接池，并恢复到空闲状态

	4. DataSource接口的多数实现类，其getConnection方法返回的是Connection代理对象，也实现了`java.sql.Connection`接口，为真正的代表数据库连接的Connection对象提供代理。代理对象的close方法并不会关闭连接，而是把真正连接对象放回数据库连接池，回到空闲状态

		```java
		Context context = new InitialContext();
		DataSouce ds = (DataSource) context.lookup("java:comp/env/jdbc/BookDB");
		// jdbc/BookDB是数据源的名字
		Connection connection = ds.getConnection();
		// 访问库
		connection.close();
		```

	

&nbsp;
	

## 3. 数据库编码

数据库字符编码与网页编码不一致时出现乱码，可以在连接数据库时URL内就指定编码方式，或对取到的数据进行编码转换`col = new String(col.getBytes("ISO-8859-1"), "GB2313");`。

不同数据库/不同版本，编码都可能不同，若后法，需要提前了解当前库的编码。

&nbsp;

## 4. 分页显示批量数据

页面显示数据过多时，用户操作繁琐，并且后台读取数据库时大量数据加载到内存，耗时。

```jsp
<%-- dbaccess.jsp 网页公共部分，连接数据库 --%>

<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>database</title>
</head>
<body>
    <%
        try {
            Connection connection;
            Statement statement;
            ResultSet resultSet;
            // 连接数据库
            Context context = new InitialContext();
            DataSource ds = (DataSource) context.lookup("java:comp/env/jdbc/BOOKDB");
            connection = ds.getConnection();
            // 创建SQL
            statement = connection.createStatement();
    %>
    <%@ include file="pages.jsp" %>
    <%
        statement.close();
        connection.close();
        } catch (Exception exception){
            out.println(exception.getMessage());
        }
    %>
</body>
</html>
```

```jsp
<%-- pages.jsp 具体显示数据与分页链接 --%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
    //分页用变量
    final int e = 3;    // 每页显示记录数
    int totalPages = 0;
    int currentPage = 1;
    int totalCount = 0; // 库中总记录数
    int p = 1;          // 当前页面所显示的第一条记录的游标

    // 读取当前待显示的页面编号
    String tempStr = request.getParameter("currentPage");
    if (tempStr != null && !tempStr.equals(""))
        currentPage = Integer.parseInt(tempStr);

    // 计算总记录数
    resultSet = statement.executeQuery("select count(*) from books;")
    if (resultSet.next())
        totalCount = resultSet.getInt(1);

    // 计算总页数
    totalPages = (totalCount % e == 0)? totalCount / e: totalCount / e + 1;
    if (totalPages == 0)
        totalPages = 1;

    // 修正当前页面编号，确保 1 <= currentPage <= totalPage
    if (currentPage > totalPages)
        currentPage = totalPages;
    else if (currentPage < 1)
        currentPage = 1;

    // 计算当前页面所显示的第一条记录的游标
    p = (currentPage - 1) * e + 1;
    String sql = "select ID, NAME, TITLE, PRICE from books order by ID limit " + (p - 1) + "," + e;
    resultSet = statement.executeQuery(sql);
%>

<%-- 显示数据 --%>
<table border="1" width="400">
    <tr>
        <td bgcolor="#008b8b"><b>编号</b></td>
        <td bgcolor="#008b8b"><b>作者</b></td>
        <td bgcolor="#008b8b"><b>书名</b></td>
        <td bgcolor="#008b8b"><b>价格</b></td>
    </tr>

    <%
        while (resultSet.next()) {
            String id = resultSet.getString(1);
            String name = resultSet.getString(2);
            String title = resultSet.getString(3);
            float price = resultSet.getFloat(4);
    %>
    <tr>
        <td><%=id%></td>
        <td><%=name%></td>
        <td><%=title%></td>
        <td><%=price%></td>
    </tr>

    <%
        }
    %>

</table>


<%-- 显示页标签 --%>
页码：
<%
    for (int i = 1; i <= totalPages; i++) {
        if (i == currentPage) {
%>
            <%=i%>
<%      } else { %>
            <a href="dbaccess.jsp?currentPage=<%=i%>"><%=i%> </a>
        <%}%>   // #if else
<%  } %>    // #for
```

&nbsp;

## 5.可滚动结果集分页显示批量数据

默认的结果集只能顺序读，不能更新内容。

1. 获得可移动游标的结果集

	`createStatement(int type, int concurrency)`

	`createPreparedStatement(String sql, int type, int concurrency)`

2. type：

	1. `ResultSet.TYPE_FORWARD_ONLY`：游标只能从上往下移动，默认值
	2. `ResultSet.TYPE_SCROLL_INSENSITIVE`：可以上下滚动；程序对结果集做了修改，游标不变
	3. `ResultSet.TYPE_SCROLL_SENSITIVE`：可以滚动；修改结果集后游标位置相应变化（如插入删除，游标前后移动）

3. concurrency：

	1. `CONCUR_READ_ONLY`：结果集不能被更新
	2. `CONCUR_UPDATABLE`：结果集可以被更新

4. 结果集可以滚动时，移动游标方法：

	1. `first()`：游标定位到第一条记录
	2. `last()`：定位到末尾记录
	3. `next()`：移动到下一条记录
	4. `previous()`：移动到上一条记录
	5. `absolute(int row)`：移动到row行的记录

&nbsp;








​	
​	
​	

