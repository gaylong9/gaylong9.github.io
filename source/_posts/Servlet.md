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







