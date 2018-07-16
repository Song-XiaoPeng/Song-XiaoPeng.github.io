### location directive

>Syntax: location \[ = \| ~ \| ~* \| ^~ \] uri { ... }  
location @name { ... }  
Default:	—  
Context: server, location  
  
语法 ： location [ = | ~ | ~* | ^~ ] uri { ... }    
location @name { ... } 命名location  
语境： server, location  

>Sets configuration depending on a request URI.

设置依赖于一个请求URI的配置项

>The matching is performed against a normalized URI, after decoding the text encoded in the “%XX” form, resolving references to relative path components “.” and “..”, and possible compression of two or more adjacent slashes into a single slash.

该匹配项是应用在一个标准格式的URI上，在将编码方式为'%xx'格式的text文本解码后，解析对相对路径组件'.'和'..'的引用，并且可能将2个或多个相邻的斜线压缩成一个斜线。

>A location can either be defined by a prefix string, or by a regular expression. Regular expressions are specified with the preceding “~*” modifier (for case-insensitive matching), or the “~” modifier (for case-sensitive matching). To find location matching a given request, nginx first checks locations defined using the prefix strings (prefix locations). Among them, the location with the longest matching prefix is selected and remembered. Then regular expressions are checked, in the order of their appearance in the configuration file. The search of regular expressions terminates on the first match, and the corresponding configuration is used. If no match with a regular expression is found then the configuration of the prefix location remembered earlier is used.

一个路径(的值)可以被定义为一个前缀的字符串，或者是一个正则表达式。正则表达式是通过在前面的'~*'修饰语被定义的（大小写不敏感），或者是'~'修饰语（大小写敏感）。为了找到和一个给予的请求相匹配的路径，nginx首先检查那些使用前缀字符串（前缀路径）定义的路径。在它们之中，有着最长匹配前缀的路径会被选择和被记下来。之后，正则表达式被选择，以它们在配置文件中出现的顺序。对于正则表达式的搜素在第一次匹配时会终止，并且相应的配置项会被应用。如果没有一个正则表达式的匹配被发现，那么之前被记住的那个前缀的路径的配置项会被应用。

>location blocks can be nested, with some exceptions mentioned below.

location路径块可以是嵌套的，通过使用一些其它的下面提及的格式。

>For case-insensitive operating systems such as macOS and Cygwin, matching with prefix strings ignores a case (0.7.7). However, comparison is limited to one-byte locales.

对于那些大小写敏感的运行系统，例如mac系统和Cygwin，前缀字符串的匹配会忽略一个情况（0.7.7）。然而，比较仅仅限于1字节的区域。

>Regular expressions can contain captures (0.7.40) that can later be used in other directives.

正则表达式可以包含捕获（子组）（0.7.40），之后可以被用于其他的指令中

>If the longest matching prefix location has the “^~” modifier then regular expressions are not checked.

如果一个最长的匹配的前缀路径具有'^~'修饰语，那么正则表达式就不会被选择了。

>Also, using the “=” modifier it is possible to define an exact match of URI and location. If an exact match is found, the search terminates. For example, if a “/” request happens frequently, defining “location = /” will speed up the processing of these requests, as search terminates right after the first comparison. Such a location cannot obviously contain nested locations.

同时，对一个URI和路径定义一个明确的匹配时使用'='修饰语是合适的。如果一个明确的匹配被找到了，搜索就会停止。举个例子，如果一个'/'请求经常地发生，定义'location = /'这个配置项将会加速这些请求的处理，因为在第一次比较之后，搜索就终止了。这种路径不能明显的包含一个嵌套的locations。

>In versions from 0.7.1 to 0.8.41, if a request matched the prefix location without the “=” and “^~” modifiers, the search also terminated and regular expressions were not checked.

在0.7.1到0.8.41版本中，如果一个请求匹配了没有‘=’和‘^~’修饰语的前缀的路径，搜索会终止并且正则表达式不会被选择。

>Let’s illustrate the above by an example:

让我们通过一个例子举例说明上述的情况：
```
location = / {
    [ configuration A ]
}

location / {
    [ configuration B ]
}

location /documents/ {
    [ configuration C ]
}

location ^~ /images/ {
    [ configuration D ]
}

location ~* \.(gif|jpg|jpeg)$ {
    [ configuration E ]
}
```
>The “/” request will match configuration A, the “/index.html” request will match configuration B, the “/documents/document.html” request will match configuration C, the “/images/1.gif” request will match configuration D, and the “/documents/1.jpg” request will match configuration E.

'/'请求将匹配配置项A，‘index.html’请求将匹配配置项B，“/documents/document.html”请求将匹配配置项C，“/images/1.gif”请求将匹配配置项D，“/documents/1.jpg”请求将匹配配置项E。

>The “@” prefix defines a named location. Such a location is not used for a regular request processing, but instead used for request redirection. They cannot be nested, and cannot contain nested locations.

‘@’前缀定义了一个命名的路径。这种路径不能被用于一个正则请求进程中，但是替代的，可以被用于请求重定向。它们不能是嵌套的，并且不能包含嵌套路径。

>If a location is defined by a prefix string that ends with the slash character, and requests are processed by one of proxy_pass, fastcgi_pass, uwsgi_pass, scgi_pass, memcached_pass, or grpc_pass, then the special processing is performed. In response to a request with URI equal to this string, but without the trailing slash, a permanent redirect with the code 301 will be returned to the requested URI with the slash appended. If this is not desired, an exact match of the URI and location could be defined like this:
```
location /user/ {
    proxy_pass http://user.example.com;
}

location = /user {
    proxy_pass http://login.example.com;
}
```

如果一个路径被定义为一个字符串前缀，它们以一个斜杠字符作为结束，请求会被proxy_pass, fastcgi_pass, uwsgi_pass, scgi_pass, memcached_pass, 或者 grpc_pass 这些配置项其中之一处理，之后这个特别的进程就会被执行。在一个URI请求的响应和字符串的响应是相同的，但是没有尾部的斜杠，一个使用301状态码的永久的重定向将（会和一个追加的斜杠一起被返回给被请求的这个URI）被返回给这个被请求的具有后缀斜杠的URI。如果这不是渴望的（想得到的），一个URI和路径的确切的匹配可以被这样定义：

### 参考资料
[nginx官方文档](http://nginx.org/en/docs/http/ngx_http_core_module.html#location)

##### 邮箱：<hellosonee@gmail.com>