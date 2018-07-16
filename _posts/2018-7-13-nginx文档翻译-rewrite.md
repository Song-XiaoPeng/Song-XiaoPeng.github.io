### rewrite directive
  
>Syntax:	rewrite regex replacement [flag];  
Default:	—  
Context:	server, location, if  

语法：  rewrite regex(正则表达式) replacement(替换)     [flag]   
默认：  
语境：  

>If the specified regular expression matches a request URI, URI is changed as specified in the replacement string. The rewrite directives are executed sequentially in order of their appearance in the configuration file. It is possible to terminate further processing of the directives using flags. If a replacement string starts with “http://”, “https://”, or “$scheme”, the processing stops and the redirect is returned to a client.

如果指定的正则表达式匹配了一个请求URI，URI会被改变为指令中指定的替代字符串。rewrite(重写)指令按照在配置文件中出现的顺序被有序的执行。通过使用flags指令，可以终止进一步的进程。如果一个替代的字符串是以“http://”, “https://”, 或 “$scheme”开头的，进程会停止，并且一个重定向将被返回给客户端。

>An optional flag parameter can be one of:

一个可选择的flag(标志)参数可以是下面中的一个：

>last  
stops processing the current set of ngx_http_rewrite_module directives and starts a search for a new location matching the changed URI;

last  最后  
停止运行当前设置的ngx_http_rewrite_module指令，并且为一个新的匹配这个被改变的URI的location(路径)开始一个搜索。

>break  
stops processing the current set of ngx_http_rewrite_module directives as with the break directive;

break  打断  
通过使用break指令，停止处理当前ngx_http_rewrite_module指令的设置（停止使用break指令处理当前的ngx_http_rewrite_module指令集）；

>redirect  
returns a temporary redirect with the 302 code; used if a replacement string does not start with “http://”, “https://”, or “$scheme”;

redirect 重定向，跳转  
返回一个暂时的状态码为302的临时重定向；当替换的字符串不是以“http://”, “https://”, 或者 “$scheme”开始的可以使用该指令；

>permanent  
returns a permanent redirect with the 301 code.
The full redirect URL is formed according to the request scheme ($scheme) and the server_name_in_redirect and port_in_redirect directives.

permanent  永久的  
返回一个状态码为301的永久的重定向。  
这个完整的重定向URL依据请求依据请求的scheme(协议)和server_name_in_redirect(重定向的server_name)和port_in_redirect(重定向的端口)指令被定义。

>Example:

例子

```
server {
    ...
    rewrite ^(/download/.*)/media/(.*)\..*$ $1/mp3/$2.mp3 last;
    rewrite ^(/download/.*)/audio/(.*)\..*$ $1/mp3/$2.ra  last;
    return  403;
    ...
}
```
>But if these directives are put inside the “/download/” location, the last flag should be replaced by break, or otherwise nginx will make 10 cycles and return the 500 error:

但是，如果这些指令是放在'/download/'（location）指令块内的，最后的flag（标志）应该被break替换，否则nginx将产生10个循环并返回一个状态码为500 的error（错误）：

```
location /download/ {
    rewrite ^(/download/.*)/media/(.*)\..*$ $1/mp3/$2.mp3 break;
    rewrite ^(/download/.*)/audio/(.*)\..*$ $1/mp3/$2.ra  break;
    return  403;
}
```
>If a replacement string includes the new request arguments, the previous request arguments are appended after them. If this is undesired, putting a question mark at the end of a replacement string avoids having them appended, for example:

如果一个替换的字符串包括新的请求参数，之前的请求参数会追加到新的参数后面。如果这是不被期望的，在替换字符串的最后放置一个question（问号？）标签来避免追加它们，比如：

```
rewrite ^/users/(.*)$ /show?user=$1? last;
```

>If a regular expression includes the “}” or “;” characters, the whole expressions should be enclosed in single or double quotes.

如果一个正则表达式包括 “}” 或者 “;”字符，那么该表达式应该被一个单引号或者双引号封闭。

### break directive
>Syntax:	break;  
Default:	—  
Context:	server, location, if  

语法：  
默认：  
语境：  

>Stops processing the current set of ngx_http_rewrite_module directives.

停止处理当前的ngx_http_rewrite_module指令配置集。

>If a directive is specified inside the location, further processing of the request continues in this location.

如果一个指令是在location里面指定的，在这个location（路径）中的这个请求之后的（进一步的，更远的）进程会继续。

>Example:

```
if ($slow) {
    limit_rate 10k;
    break;
}
```

### if directive
>Syntax:	if (condition) { ... }    
Default:	—    
Context:	server, location    

语法：  
默认：  
语境：  

>The specified condition is evaluated. If true, this module directives specified inside the braces are executed, and the request is assigned the configuration inside the if directive. Configurations inside the if directives are inherited from the previous configuration level.

对指定的条件进行评估。如果值为真，被指定在这个大括号里面的这个模块指令将被执行，并且这个请求被分配到if指令内部的配置。if指令内部的配置继承于之前的配置层级。

>A condition may be any of the following:

一个条件可能是以下的任意一个：

- a variable name; false if the value of a variable is an empty string or “0”;
    >Before version 1.0.1, any string starting with “0” was considered a false value.
- comparison of a variable with a string using the “=” and “!=” operators;
- matching of a variable against a regular expression using the “~” (for case-sensitive matching) and “~*” (for case-insensitive matching) operators. Regular expressions can contain captures that are made available for later reuse in the $1..$9 variables. Negative operators “!~” and “!~*” are also available. If a regular expression includes the “}” or “;” characters, the whole expressions should be enclosed in single or double quotes.
- checking of a file existence with the “-f” and “!-f” operators;
- checking of a directory existence with the “-d” and “!-d” operators;
- checking of a file, directory, or symbolic link existence with the “-e” and “!-e” operators;
- checking for an executable file with the “-x” and “!-x” operators.
>Examples:

- 一个变量名；如果该变量的值是一个空字符串或者为0，那么该条件的值为false；
    >在版本1.0.1之前，任意一个以0开始的字符串都被看做一个false的值。
- 使用运算符“=” 和 “!=” 来对一个变量和一个字符串进行比较；
- 使用“~”（大小写敏感匹配）和“~*”（大小写不敏感匹配）运算符来匹配一个变量和正则表达式。正则表达式可以包含捕获子组，这些子组被制成$1到$9的变量，以用于之后的重用。否定的运算符“!~” 和 “!~*”也是可利用的。如果一个正则表达式包括“}” 或者 “;” 字符，整个正则表达式应该被一组单引号或双引号封闭。
- 使用“-f” 和 “!-f”运算符检查一个文件是否存在；
- 使用“-d” 和 “!-d”运算符检查一个目录是否存在；
- 使用“-e” 和 “!-e”运算符检查一个文件，目录，或者符号链接是否存在；
- 使用“-x” 和 “!-x”运算符检查一个可执行文件是否存在；

例如：

```
if ($http_user_agent ~ MSIE) {
    rewrite ^(.*)$ /msie/$1 break;
}

if ($http_cookie ~* "id=([^;]+)(?:;|$)") {
    set $id $1;
}

if ($request_method = POST) {
    return 405;
}

if ($slow) {
    limit_rate 10k;
}

if ($invalid_referer) {
    return 403;
}
```
>A value of the $invalid_referer embedded variable is set by the valid_referers directive.

内置变量$invalid_referer(有效的参照页，引用页)的值被valid_referers指令设置。

### 实例
```
location / {
    # 情况一
    try_files $uri $uri/ @rewrites;
}

location @rewrites {
    rewrite ^.*$ /index.html break|last;
}

location / {
    # 情况二
    try_files /index.html =404;
}

location / {
    # 情况三
    rewrite ^.*$ /index.html break;
}

location / {
    # 情况四
    rewrite ^.*$ /index.html last;
}

location / {
    # 情况五
    try_files $uri $uri/;
}

location /bd6/ {
    # 情况七
    rewrite ^.*$ http://www.baidu.com;
    rewrite .* http://www.baidu.com;
}

location /bd6 {
    # 情况八
    rewrite ^.*$ http://www.baidu.com;
    rewrite .* http://www.baidu.com;
}

location /bd7 {
    # 情况九
    rewrite ^.*$ http://www.google.com;
}       

```
#### 结果：
情况一二三，正常，  
情况三报错：
>*2450 rewrite or internal redirection cycle while processing "/index.html", client: 120.0.0.1, server: backend-shop.ss1s.com, request: "GET /goodsShelf/list HTTP/1.1", host: "backend-shop.ss1s.com"

情况五报错:
>*2538 rewrite or internal redirection cycle while internally redirecting to "/distributeSchool/list///////////", client: 113.57.70.119, server: backend-shop.hellobirds.top, request: "GET /distributeSchool/list HTTP/1.1", host: "backend-shop.hellobirds.top"

情况7：  
当访问http://foobar.com/bd6时，第一个rewrite规则匹配不到，但第二个规则可以匹配

情况八：  
当访问http://foobar.com/bd6或者http://foobar.com/bd6/时，两个规则分别都可以匹配到

### 参考资料
[nginx官方文档](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html#rewrite)  
[JS-MDN：HTTP 的重定向](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Redirections)  
[转发和重定向的区别](https://blog.csdn.net/powerfulK/article/details/71069888)  
[内部跳转(请求转发)和外部跳转(重定向)的区别?](https://blog.csdn.net/pozmckaoddb/article/details/49617003)  

##### 邮箱：<hellosonee@gmail.com>
