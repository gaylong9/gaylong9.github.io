---
title: Servlet
mathjax: false
date: 2020-08-28 22:56:53
tags: JavaWeb
---

<br/>

<!-- more -->

<!-- toc -->

<br/>



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

<br/>

## 2. Servlet基础

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

	1. 指定WebServlet注解，内容为URL中项目后的位置（将类定义为Servlet组件，标注为可以处理用户请求的Servlet）
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

3. 生命周期：

	无main方法，运行完全由服务器控制

	1. 实例化和初始化时机：请求到达，查找对象是否存在，若无，则如此
	2. 就绪、调用、服务阶段：请求到达，调用service方法；可以多次调用；
	3. 销毁时机：容器关闭，自动销毁

4. 时序：

	1. HTTP请求到达
	2. Servlet容器接收请求
	3. 容器实例化请求对象和响应对象
	4. 调用service方法
	5. Servlet从请求对象获取信息，并设置响应对象中的信息
	6. 容器响应浏览器

<br/>

## 3. HttpServlet Request对象

1. 接收客户端发送请求的信息
2. 是ServletRequest接口的子接口HttpServletRequest的实例化对象

### 3.1 常用方法

1. `getRequestURL()` 完整URL
2. `getRequestURI()` 请求行中的资源名称部分（项目名称开始）
3. `getQueryString()` 请求行中的参数部分
4. `getMethod()` 客户端请求方式
5. `getProtocol()` HTTP版本号
6. `getContextPath()` webapp名字
7. `getParameter(name)` 获取指定名称的参数值（参数均为键值对）
8. `getParameterValues(name)` 获取指定名称参数的所有值

### 3.2 请求乱码问题

接收客户端参数并解析时默认使用ISO-8859-1，不支持中文，会乱码。

1. Tomcat8起，GET方式不会乱码。
2. POST：接收数据前 `req.setCharacterEncoding("UTF-8");`

### 3.3 请求转发

1. 服务器行为，将请求对象保存并传递，地址栏的URL不变，整个过程仅一个请求。
2. 多个资源协同响应。
3. `req.getRequestDispatcher(url).forward(req, resp);` url起始不加/

### 3.4 request作用域

1. 在一次请求中有效，即请求转发不影响。
2. `req.setAttribute(name, value)`
3. `req.getAttribute(name)`
4. `req.removeAttribute(name)` 
5. 可在请求转发中用request对象传递数据。

<br/>

## 4. HttpServlet Response对象

1. 服务器对于每个请求，分别创建一个代表请求的request对象和response对象
2. 获取客户端数据用request，向客户端输出数据用response

### 4.1 响应数据

1. `resp.setHeader("content-type", "text/html");` 设置响应MIME类型，默认字符串
2. `getWriter().write(string)` 获取字符流，响应回字符
3. `getOutputStream().write(string.getBytes())` 字节流，响应回一切数据
4. 响应回的数据到客户端被解析
5. 两种流不可同时使用。

### 4.2 响应乱码问题

1. 客户端与服务器编码不一致会乱码。	
2. getWriter字符乱码
	1. 响应回中文会乱码，因为服务器端用ISO-8859-1，不支持中文
	2. 告知服务器使用其他编码 `resp.setCharacterEncoding("UTF-8")`
	3. 告知客户端 `resp.setHeader("content-type", "text/html;charset=UTF-8")`
	4. 合并为一句 `resp.setContentType("text/html;charset=UTF-8")` 指定相应类型和编码
3. getOutputStream字节乱码：传输字节，中文不一定乱码，但应指定，仍使用上述方法即可。

### 4.3 重定向

1. 服务器指导，客户端行为。第一个请求被接收处理后，服务器响应并返回一个新地址，客户端收到响应立刻自动给新地址发起第二次请求，服务器接收并处理。
2. 两次请求，客户端行为。
3. `resp.sendRedirect(String uri)` 无/
4. 请求转发与重定向的区别：
	1. 一次请求，req域数据共享/两次请求，req无法共享
	2. 服务端行为/客户端行为
	3. 地址栏不变/变
	4. 仅在本站内跳转/可跳转至站外，甚至可`http://`开头

<br/>

## 5. Cookie对象

1. 保存在客户端，安全性差。

2. `javax.servlet.http.Cookie`类操作。随着服务器端的响应，将cookie内容返回给客户端并保存在浏览器，当下次访问时把cookie带回服务器。

3. 服务器端创建与发送：

	```java
	Cookie cookie01 = new Cookie("name", "admin");
	resp.addCookie(cookie01);
	```

4. 服务器端获取：仅有`getCookies`一个方法，返回数组，需要遍历并`getName() getValue()`

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

5. 到期时间：默认关闭浏览器即失效。`setMaxAge(int)`设置到期时间，秒

	1. -1：默认就是-1，表示不存储，只在内存存活，关闭浏览器就失效
	2. 正整数：存储到磁盘
	3. 0：删除cookie

6. 注意点：

	1. 保存在当前浏览器，换电脑或浏览器均不能使用
	2. 中文：默认不能存储中文，需要`URLEncoder.encode()`编码存储，`decode()`解码读取
	3. 发送重名cookie会覆盖
	4. 有数量上限

7. 路径：决定能否读取到cookie `cookie.setPath()`

	1. `("/")` 当前服务器下所有项目都可访问
	2. `("/pro1")` 当前项目下可访问，默认为此
	3. `("/pro2")` 仅项目pro2下可访问，即使是pro1项目创建的cookie
	4. `("/pro1/cook")` 仅项目pro1下的cook目录下可访问
	5. 当设置了路径且访问路径在设置的路径下，cookie就会加载到req对象中



## 6. Session

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



## 7. ServletContext

1. 介绍：每个web应用有一个ServletContext对象，又称application对象。在WEB容器启动的时候，会为每一个WEB应用程序创建一个对应的ServletContext对象。
2. 作用：作为域对象共享数据、保存应用程序信息
	1. 常用信息：项目信息`getServerInfo()` 真实路径`getRealPath(path)`
3. 获取对象：
	1. `req.getServletContext()`
	2. `req.getSession().getServletContext()`
	3. `getServletConfig().getServletContext()`
	4. `getServletContext()`
4. 作为域对象分享数据：但因不可手动删除，不易存入过多 `setAttribute(key, value)` `getAttribute(key)` `removeAttribute(key)`



## 8. 域对象总结

1. request域对象：一次请求中有效。请求转发有效，重定向无效。
2. session域对象：一次会话中有效。请求转发和重定向都有想，session销毁无效。
3. ServletContext域对象：整个应用程序有效，服务器关闭失效。



## 9. 文件上传

1. 前台：表单、post、指定的enctype

	```jsp
	<form method="post" enctype="multipart/form-data" action="UpLoad">
	    姓名：<input type="text" name="uname"> <br>
	    文件：<input type="file" name="myfile"> <br>
	    <button>提交</button>
	</form>
	```

2. 后台：注解`@MultipartConfig`将一个Servlet标识为支持文件上传。Servlet将multipart/form-data的POST请求封装成Part，通过Part对上传的文件操作。  

	```java
	@WebServlet("/UpLoad")
	@MultipartConfig
	public class UploadServlet extends HttpServlet {
	    @Override
	    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
	        req.setCharacterEncoding("UTF-8");
	        System.out.println("file upload...");
	
	        String uname = req.getParameter("uname");
	        System.out.println("uname: " + uname);
	
	        Part part = req.getPart("myfile");
	        String fileName = part.getSubmittedFileName();
	        System.out.println("file name: " + fileName);
	
	        String filePath = req.getServletContext().getRealPath("/");
	        System.out.println("file path: " + filePath);
	        part.write(filePath + "/" + fileName);
	    }
	}
	```



## 10. 文件下载

1. 超链接方法：前台a标签跳转时，若资源无法被浏览器识别则下载，可识别资源也可以通过设置download属性实现下载。该属性为空使用原文件名，指定属性即为下载的文件名。

	`<a href="download/avatar.jpg" download="dragon.jpg">头像图片</a>`

2. 后台代码方法：

	1. 需要通过`response.setContentType`方法设置Content-type头字段的值为浏览器无法使用某种方式或激活某个程序来处理的MIME类型，例如`application/octet-stream`或`application/x-msdownload`等。
	2. 需要通过`response.setHeader`方法设置`Content-Disposition`头的值为`attachment;filename=文件名`
	3. 读取下载文件，调用`response.getOutputStream`方法向客户端写入附件内容。  

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
	            resp.getWriter().write("input name");
	            resp.getWriter().close();
	            return;
	        }
	        // 确定文件存在，开始下载
	        String path = req.getServletContext().getRealPath("/download/");
	        File file = new File(path + fileName);
	        if (file.exists() && file.isFile()) {
	            // 响应类型
	            resp.setContentType("application/x-msdownload");
	            // 响应头
	            resp.setHeader("Content-Disposition", "attachment;filename=" + fileName);
	            // 输入流
	            InputStream in = new FileInputStream(file);
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





