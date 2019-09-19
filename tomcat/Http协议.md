
## Http请求
### HTTP请求的三个组成部分

+ 方法-统一资源标识符（URI）-协议/版本
+ 请求的头部
+ 主题内容

~~~http request
POST /examples/default.jsp HTTP/1.1 第一部分
Accept: text/plain; text/html       第二部分
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


## Http响应