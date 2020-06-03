# Tomcat
> servlet容器，其内部实现了servlet规范。当前的web应用主要是基于http协议进行的用户-浏览器-服务器之间的交互模式。
~~~java
class Tomcat{
    Connector connector;//处理请求--Request-> RequestFacade -> Servlet.service(HttpServletRequest)
    List<Servlet> servlets;
}
~~~


> 我们经常说tomcat是一个servlet容器，那么它到底是如何运行的呢?
+ 在servlet中有一个核心方法service(Request, Response),主要由它来处理请求，这个方法是由谁来执行的？
    + tomcat内部调用doGet方法
    + 在以war包部署项目时，tomcat会读取class文件中的所有请求路径，在tomcat接收到请求后，根据路径来匹配tomcat存在的路径，并开始生成HttpServletRequest对象，并调用doGet传入HttpServletRequest参数

> 部署方式,为什么可以部署war包，而不能部署jar包？
+ 在webapps下面部署war包，其核心部分就是class文件以及web.xml文件。在servlet3.1规范中，可以通过注解的方式去除web.xml文件，注解为@WebServlet。
+ 同样可以部署文件夹，只是war包是压缩的，文件夹是解压的
+ 还以在配置文件中配置<Context path="*" docBase="*"/>,提供项目所在路径，其实与war部署一样
+ jar包方式部署，tomcat无法确定是依赖包，还是web项目包，所以干脆用war包

>那么接下来，就来理解何为容器？
+ http://localhost:8080/demo/helloWorld,这个请求路径能够映射出host,context,servlet
    + host: http://localhost:8080
    + context: demo
    + servlet: helloWorld
+ 容器结构为：Engine->Host->Context->Servlet,Engine是为了项目的集群部署需要的容器属性
+ 而最终的容器结构为：Engine->Host->Context->Wrapper->Servlet,Wrapper是为了实现Servlet的并发访问衍生出来了的，一个Wrapper对应一个Servlet类，每一个Wrapper下有一个List<Servlet>集合。

~~~java
Engine:
    Pipeline    -> 管道valve定义管道属性
    List<Host> hosts;
Host:
    Pipeline
    List<Context> contexts;
Context:
    Pipeline
    List<Wrapper> wrappers;
Wrapper:
    Pipeline    ->最后一个管道阀门
    List<Servlet>   servlets;
~~~
> 当接收到请求之后，从Engine开始，经过管道处理到host到context到wrapper的最后一个管道阀门，然后将请求交给Servlet。

## Http协议
### Http请求
#### HTTP请求的三个组成部分

+ 方法-统一资源标识符（URI）-协议/版本
+ 请求的头部
+ 主题内容

~~~http request
POST /examples/default.jsp HTTP/1.1
Accept: text/plain; text/html
Accept-Language: en-gb
Connection: Keep-Alive
Host: localhost
User-Agent: Mozilla/4.0 (compatible; MSIE 4.01; Windows 98)
Content-Length: 33
Content-Type: application/x-www-form-urlencoded
Accept-Encoding: gzip, deflate

lastName=Franks&firstName=Michael   第三部分
~~~
> 对于请求头和主题内容之间有一个回车换行符（CRLF）是相当重要的。


### Http响应
#### HTTP请求的三个组成部分

+ 方法-统一资源标识符（URI）-协议/版本
+ 请求的头部
+ 主题内容
~~~http request
HTTP/1.1 200 OK
Server: Microsoft-IIS/4.0
Date: Mon, 5 Jan 2004 13:13:33 GMT
Content-Type: text/html
Last-Modified: Mon, 5 Jan 2004 13:13:12 GMT
Content-Length: 112

<html>
<head>
<title>HTTP Response Example</title>
</head>
<body>
Welcome to Brainy Software
</body>
</html>
~~~


+ tomcat是一个Http服务器+Servlet容器，简称Web容器
> web容器负责将接收用户的请求，转发给对应的服务器，并接收服务器的响应结果，最终发送给用户。
> + 接收用户的网络请求-Socket套接字。
> + 向服务器转发网络请求-Servlet容器管理。服务器的网络资源被定义成Servlet，多个资源的情况下，\
> 需要对多个Servlet进行管理。
>
> 因此，tomcat需要实现的两个核心功能
> + 处理 Socket 连接，负责网络字节流与 Request 和 Response 对象的转化。
> + 加载和管理 Servlet，以及具体处理 Request 请求。 \
> 
> Tomcat设计了两个核心组件：连接器（Connector）和容器（Container）,Connector负责外部的网络请求，\
> Container服务内部的请求转发。\
><img src="http://p1.pstatp.com/large/pgc-image/e907a8ce9f774d0aac6f1dcda9baf6a6">

连接器与容器的内容可以参考， <a href="https://www.toutiao.com/a6701177258800841219/">连接器与容器详解</a>

## Tomcat的运行流程
假设来自客户的请求为：http://localhost:8080/test/index.jsp 请求被发送到本机端口8080，被在那里侦听的Coyote HTTP/1.1 Connector,然后
* Connector把该请求交给它所在的Service的Engine来处理，并等待Engine的回应
* Engine获得请求localhost:8080/test/index.jsp，匹配它所有虚拟主机Host
* Engine匹配到名为localhost的Host(即使匹配不到也把请求交给该Host处理，因为该Host被定义为该Engine的默认主机)
* localhost Host获得请求/test/index.jsp，匹配它所拥有的所有Context
* Host匹配到路径为/test的Context(如果匹配不到就把该请求交给路径名为""的Context去处理)
* path="/test"的Context获得请求/index.jsp，在它的mapping table中寻找对应的servlet
* Context匹配到URL PATTERN为*.jsp的servlet，对应于JspServlet类，构造HttpServletRequest对象和HttpServletResponse对象，作为参数调用JspServlet的doGet或doPost方法
* Context把执行完了之后的HttpServletResponse对象返回给Host
* Host把HttpServletResponse对象返回给Engine
* Engine把HttpServletResponse对象返回给Connector
* Connector把HttpServletResponse对象返回给客户browser

+ Tomcat架构图
<img src="https://img.mukewang.com/5a26687d0001ca2712300718.png"></img>

> 通过此图可以得出以下信息：
> + 一个Tomcat对应一个Server, 即对应/config/server.xml
> + 一个Server下可以有多个Service
> + 一个Service对应一个容器Engine
> + 一个Service下可以有多个Connector，且每一个Connector都有一个线程池

### server.xml的整理结构
~~~xml
<Server>
    <Service>
        <Connector />
        <Connector />
        <Engine>
            <Host>
                <Context /><!-- 现在常常使用自动部署，不推荐配置Context元素，Context小节有详细说明 -->
            </Host>
        </Engine>
    </Service>
</Server>
~~~
更多内容，<a href="https://www.cnblogs.com/kismetv/p/7228274.html">详解server.xml</a>