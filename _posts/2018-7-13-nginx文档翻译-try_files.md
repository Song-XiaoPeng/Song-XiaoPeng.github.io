### try_files directive
>Syntax:	try_files file ... uri;  
try_files file ... =code;  
Default:	—  
Context:	server, location  

语法： try_files file ... uri;  
try_files file ... =code;  
默认：  
语境：  

>Checks the existence of files in the specified order and uses the first found file for request processing; the processing is performed in the current context. The path to a file is constructed from the file parameter according to the root and alias directives. It is possible to check directory’s existence by specifying a slash at the end of a name, e.g. “$uri/”. If none of the files were found, an internal redirect to the uri specified in the last parameter is made. For example:

检查指定顺序的文件是否存在，使用第一个被找到的文件作为来处理请求；这个进程被运行在当前的上下文之中。文件的路径是通过参照根目录和指令别名的文件参数构建的。通过在一个名字后面定义一个斜杠，会去检查目录是否存在，比如‘$uri/’。如果没有文件（目录）被找到，就会产生一个最后一个参数中指定的uri的内部的重定向。例如：

```
location /images/ {
    try_files $uri /images/default.gif;
}

location = /images/default.gif {
    expires 30s;
}
```
>The last parameter can also point to a named location, as shown in examples below. Starting from version 0.7.51, the last parameter can also be a code:

最后一个参数也可以指定一个命名路径（named location），就像在下面例子中展示的那样。从0.7.51版本开始，最后一个参数也可以是一个状态码：

```
location / {
    try_files $uri $uri/index.html $uri.html =404;
}
```
>Example in proxying Mongrel:

代理Mongrel的例子：
```
location / {
    try_files /system/maintenance.html
              $uri $uri/index.html $uri.html
              @mongrel;
}

location @mongrel {
    proxy_pass http://mongrel;
}
```
>Example for Drupal/FastCGI:

Drupal/FastCGI例子：
```
location / {
    try_files $uri $uri/ @drupal;
}

location ~ \.php$ {
    try_files $uri @drupal;

    fastcgi_pass ...;

    fastcgi_param SCRIPT_FILENAME /path/to$fastcgi_script_name;
    fastcgi_param SCRIPT_NAME     $fastcgi_script_name;
    fastcgi_param QUERY_STRING    $args;

    ... other fastcgi_param's
}

location @drupal {
    fastcgi_pass ...;

    fastcgi_param SCRIPT_FILENAME /path/to/index.php;
    fastcgi_param SCRIPT_NAME     /index.php;
    fastcgi_param QUERY_STRING    q=$uri&$args;

    ... other fastcgi_param's
}
```
>In the following example,

在接下来的这些例子中，
```
location / {
    try_files $uri $uri/ @drupal;
}
```
>the try_files directive is equivalent to

`try_files`指令是和下面的这个例子相同的（等价的，同等意义的）
```
location / {
    error_page 404 = @drupal;
    log_not_found off;
}
```
>And here,

这里，
```
location ~ \.php$ {
    try_files $uri @drupal;

    fastcgi_pass ...;

    fastcgi_param SCRIPT_FILENAME /path/to$fastcgi_script_name;

    ...
}
```
>try_files checks the existence of the PHP file before passing the request to the FastCGI server.

>Example for Wordpress and Joomla:

`try_files`指令在将请求传递给`FastCGI`服务器之前会检查PHP文件是否存在。

`Wordpress`和`Joomla`的例子
```
location / {
    try_files $uri $uri/ @wordpress;
}

location ~ \.php$ {
    try_files $uri @wordpress;

    fastcgi_pass ...;

    fastcgi_param SCRIPT_FILENAME /path/to$fastcgi_script_name;
    ... other fastcgi_param's
}

location @wordpress {
    fastcgi_pass ...;

    fastcgi_param SCRIPT_FILENAME /path/to/index.php;
    ... other fastcgi_param's
}
```