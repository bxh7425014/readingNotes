JSP内置对象

![内置对象](http://upload-images.jianshu.io/upload_images/1124873-abaec88e6dbd86c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 内置对象及其应用场合
JSP内置对象是应用JSP进行Web开发时，通过它们可以对Web开发中的请求、响应、会话、应用、输出、配置信息和异常信息等内容进行控制。

## request对象
该对象封装了由客户端生成的HTTP请求的所有细节，主要包括HTTP头信息、请求方式和请求参数等。
> 在开发Web应用时经常应用request对象获取请求参数的值和获取Cooike数据等。

- 举例：

```
<a href="delete.jsp?id=1">删除</a>
```
在delete.jsp页面中可以通过request对象的getParameter()方法获取传递的参数值，代码如下：

```
<% String id=request.getParameter("id");%>
```
执行了getParameter()方法后变量id的值为1。

- request对象获取客户端信息的常用方法
  - getHeader(String name) 
获得HTTP协议定义的文件头信息
  - getHeaders(String name) 
返回指定名称的request Header的所有值，结果是一个枚举型的实例
  - getHeadersNames() 
返回所有request Header的名称，结果是一个枚举型的实例
  - getMethod()
获得客户端向服务器端传送数据的方法，如get、post、header和trace等
  - getProtocol()
获得客户端向服务器端传送数据所依据的协议名称。
  - getRequestURI()
获得发出请求字符串的客户端地址，不包括请求的参数。
  - getRequestURL()
获取发出请求字符串的客户端地址。
  - getRealPath()
返回当前请求文件的绝对路径
  - getRemoteAddr()
获取客户端的IP地址
  - getRemoteHost()
获取客户端的主机名
  - getServerName()
获取服务器的名字
  - getServerPath()
获取客户端所请求的脚本文件的文件路径
  - getServerPort()
获取服务器的端口号
  - request.getCookies()
获取客户端保存的Cookie数据

## response对象
该对象适用于响应客户端请求信息.
> 开发Web应用时经常应用response对象重定向网页、设置HTTP响应报头和缓冲区配置等。

## session对象
该对象适用于在同一个应用程序中每个客户端的各个页面中共享数据。
> session对象通常应用于保存用户/管理员信息和购物车信息等。

## application对象
该对象适用于在同一个应用程序中各个用户间共享数据。
> 通常应用在计数器或是聊天室中。

## out对象
该对象适用于向客户端输出各种类型的数据。
> 通常用来在JSP页面中输出文本。

## page对象
该对象适用于操作JSP页面自身。
> 在开发Web应用时很少应用。

## config对象
该对象适用于读取服务器的配置信息。

## exception对象
该对象适用于操作JSP文件执行时发生的异常信息。

## pageContext对象
该对象适用于获取JSP页面的request、re-sponse、session、application和out等对象。
> 由于这些对象均为JSP的内置对象，所以在Web应用开发时很少使用pageContext对象。
