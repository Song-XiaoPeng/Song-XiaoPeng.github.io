### return directive

>Syntax:	return code [text];  
return code URL;  
return URL;  
Default:	—  
Context:	server, location, if  

语法： return code [text];  
return code URL;  
return URL;  
默认：  
语境：  server, location, if  

>Stops processing and returns the specified code to a client. The non-standard code 444 closes a connection without sending a response header.

停止进程，返回指定的状态码给客户端。非标准的状态码444结束一个连接，不会发送一个响应头。

>Starting from version 0.8.42, it is possible to specify either a redirect URL (for codes 301, 302, 303, 307, and 308) or the response body text (for other codes). A response body text and redirect URL can contain variables. As a special case, a redirect URL can be specified as a URI local to this server, in which case the full redirect URL is formed according to the request scheme ($scheme) and the server_name_in_redirect and port_in_redirect directives.

从0.8.42版本开始，可以指定一个跳转的URL（为状态码301，302，303，304，和 308）或者相应实体文本（为其它的状态码）。一个响应实体内容和重定向URL可以包含变量。作为一个特殊的情况，一个重定向URL可以被指定为一个本地服务器的URI（可以将重定向URL指定为该服务器的本地URI），在这种情况下，这个完整的重定向URL依据请求协议和server_name_in_redirect和port_in_redirect指令被加工。

>In addition, a URL for temporary redirect with the code 302 can be specified as the sole parameter. Such a parameter should start with the “http://”, “https://”, or “$scheme” string. A URL can contain variables.

此外，一个状态码为302的临时重定向的URL可以被指定为唯一的参数。这个参数应该以“http://”, “https://”, 或者 “$scheme” 字符开始。一个URL可以包含变量。

>Only the following codes could be returned before version 0.7.51: 204, 400, 402 — 406, 408, 410, 411, 413, 416, and 500 — 504.

在版本0.7.51之前，只有如下的这些状态码可以被返回

>The code 307 was not treated as a redirect until versions 1.1.16 and 1.0.13.

307状态码直到1.1.16和1.0.13版本，一直没有被看做一个重定向的状态码。

>The code 308 was not treated as a redirect until version 1.13.0.

308直到1.13.0版本，一直没有被视为一个重定向。

### 参考资料
[nginx官方文档](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html#return)

##### 邮箱：<hellosonee@gmail.com>