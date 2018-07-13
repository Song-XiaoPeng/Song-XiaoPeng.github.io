### 版本
`nginx -v`  1.10.0

### proxy_pass指令文档
>Syntax:	proxy_pass URL;  
Default:	—  
Context:	location, if in location, limit_except  

    语法：  proxy_pass URL;  
    默认值： 无;  
    语境：  

>Sets the protocol and address of a proxied server and an optional URI to which a location should be mapped. As a protocol, “http” or “https” can be specified. The address can be specified as a domain name or IP address, and an optional port:  

>proxy_pass http://localhost:8000/uri/;

>or as a UNIX-domain socket path specified after the word “unix” and 
enclosed in colons:
    
>proxy_pass http://unix:/tmp/backend.socket:/uri/;    

    为一个`代理服务器`设置协议和地址，为一个应该被映射的地址设置一个可选择的URI。作为一个协议，可以指定为'http'或'https'。这个地址可以使用一个`域名`或者一个`IP地址`来指定，此外还包括一个可选择的`端口号`:
 
    或者在以冒号结尾的单词`unix`后指定，作为一个`UNIX类型域名`的套接字路径；

>If a domain name resolves to several addresses, all of them will be used in a round-robin fashion. In addition, an address can be specified as a server group.

    如果一个域名包括了几个路径的话，它们将被用做一个循环的形式。此外，一个地址可以被指定为一个服务器组

>Parameter value can contain variables. In this case, if an address is specified as a domain name, the name is searched among the described server groups, and, if not found, is determined using a resolver.

    参数值可以包括变量。在这种情况下，如果一个地址是被指定为一个域名的话，这个域名会在这些被描述的服务器组中被搜索，同时，如果没有找到的话，会决定用一个`resolver解析`。

>A request URI is passed to the server as follows:

    一个请求的`URI`会通过以下几种方式传递给服务器：

* If the proxy_pass directive is specified with a URI, then when a request is passed to the server, the part of a normalized request URI matching the location is replaced by a URI specified in the directive:
```
location /name/ {
    proxy_pass http://127.0.0.1/remote/;
}  
```  
如果`proxy_pass`指令是和一个`URI`一起被定义的，那么当一个请求被提交给服务器时，这个匹配`location`地址的标准的请求`URI`的部分将会被这个指令中被定义的`URI`所替代：

>If proxy_pass is specified without a URI, the request URI is passed to the server in the same form as sent by a client when the original request is processed, or the full normalized request URI is passed when processing the changed URI:
```
location /some/path/ {
    proxy_pass http://127.0.0.1;
}
```
    如果这个`proxy_pass`指令被指定时不包括一个`URI`，当一个原始的请求被一个客户端运行时，这个请求的`URI`将会以相同的格式传递给服务器，或者当运行这个改变后的`URI`时，这个完整的标准化的请求的`URI`会被传递：

>Before version 1.1.12, if proxy_pass is specified without a URI, the original request URI might be passed instead of the changed URI in some cases.

    在1.1.12版本之前，如果一个`proxy_pass`指令没有和一个`URI`被指定时，在某些情况下，原始的请求的`URI`可能会代替改变后的`URI`被提交。

>In some cases, the part of a request URI to be replaced cannot be determined:

    在某些情况下，请求的URI中将要被替换的一部分是不能决定的：

* When location is specified using a regular expression, and also inside named locations.

* In these cases, proxy_pass should be specified without a URI.

    当`location`地址被一个正则表达式指定时，也包括命名的`named location`地址。

    在这些情况中，`proxy_pass`指令被指定时不应该包括一个URI。

* When the URI is changed inside a proxied location using the rewrite directive, and this same configuration will be used to process a request (break):
```
location /name/ {
    rewrite    /name/([^/]+) /users?name=$1 break;
    proxy_pass http://127.0.0.1;
}
```
>In this case, the URI specified in the directive is ignored and the full changed request URI is passed to the server.

    当URI被通过使用`rewrite`指令在内部改变为一个代理地址后，启动一个请求（break）时会使用相同的配置:

    这种情况下，在指令中指定的URI被忽略，完整的改变后的请求URI被传递给服务器

>When variables are used in proxy_pass:
```
location /name/ {
    proxy_pass http://127.0.0.1$request_uri;
}
```
>In this case, if URI is specified in the directive, it is passed to the server as is, replacing the original request URI.

    当变量被用于proxy_pass指令中时：

    在这种情况下，如果一个URI在一个指令中被指定时，它会以它指示的这种方式被传递给服务器，替代原始的请求的URI。

### 实践

```
server {

    #实际使用中，可以参考这些匹配规则定义，如下：
    #直接匹配网站根，通过域名访问网站首页比较频繁，使用这个会加速处理，官网如是说。
    #这里是直接转发给后端应用服务器了，也可以是一个静态首页
    # 第一个必选规则
    location = / {
        proxy_pass http://tomcat:8080/index
    }
    # 第二个必选规则是处理静态文件请求，这是nginx作为http服务器的强项
    # 有两种配置模式，目录匹配或后缀匹配,任选其一或搭配使用
    location ^~ /static/ {
        root /webroot/static/;
    }
    location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {
        root /webroot/res/;
    }
    #第三个规则就是通用规则，用来转发动态请求到后端应用服务器
    #非静态文件请求就默认是动态请求，自己根据实际把握
    #毕竟目前的一些框架的流行，带.php,.jsp后缀的情况很少了
    location / {
        proxy_pass http://tomcat:8080/
    }

}

```
### location指令设置规则

= 开头表示精确匹配  
^~ 开头表示uri以某个常规字符串开头，不是正则匹配  
~ 开头表示区分大小写的正则匹配;  
~* 开头表示不区分大小写的正则匹配  
/ 通用匹配, 如果没有其它匹配,任何请求都会匹配到  


### 匹配优先级
(location =) > (location 完整路径) > (location ^~ 路径) > (location ~,~* 正则顺序) > (location 部分起始路径) > (/)

### 参考资料
[nginx官方文档](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass)

[nginx配置location总结及rewrite规则写法](http://seanlook.com/2015/05/17/nginx-location-rewrite/)

##### 邮箱：<hellosonee@gmail.com>
