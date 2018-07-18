#### charset DIRECTIVE
Syntax:	charset charset | off;  
Default:	  
charset off;  
Context:	http, server, location, if in location  

语法： charset charset | off;   
默认：  
语境：  http, server, location, if in location 

>Adds the specified charset to the “Content-Type” response header field. If this charset is different from the charset specified in the source_charset directive, a conversion is performed.

为响应头字段“Content-Type”添加指定的字符集。如果这个字符集和source_charset指令定义的字符集不同的话，将会进行一个转换（执行、完成、演奏、表演）。

>The parameter off cancels the addition of charset to the “Content-Type” response header field.

参数off会取消向 “Content-Type” 响应头字段添加的字符集。

>A charset can be defined with a variable:

一个字符集可以和一个变量一起被定义：
```
charset $charset;
```

>In such a case, all possible values of a variable need to be present in the configuration at least once in the form of the charset_map, charset, or source_charset directives. For utf-8, windows-1251, and koi8-r charsets, it is sufficient to include the files conf/koi-win, conf/koi-utf, and conf/win-utf into configuration. For other charsets, simply making a fictitious conversion table works, for example:

在这种情况下，一个变量的所有可能值需要至少一次出现在配置中，在charset_map, charset, 或 source_charset 这些指令的格式中（在这种情况下，一个变量的所有可能值至少需要以charset_map、charset或source_charset指令的形式出现一次）。对于utf-8，将这些文件 conf/koi-win, conf/koi-utf, and conf/win-utf 包含到配置中就足够了（对于utf-8、windows-1251和koi8-r字符集，将conf/koi-win、conf/koi-utf和conf/win-utf文件包含到配置中就足够了）。对于其他字符集，只需创建一个虚构的转换表，例如：

```
charset_map iso-8859-5 _ { }
```

>In addition, a charset can be set in the “X-Accel-Charset” response header field. This capability can be disabled using the proxy_ignore_headers, fastcgi_ignore_headers, uwsgi_ignore_headers, scgi_ignore_headers, and grpc_ignore_headers directives.

此外，一个字符集可以设置在“X-Accel-Charset”响应头字段中（可以在“X-Accel-Charset”响应头字段中设置字符集）。这一能力（才能，能力；性能，容量）可以通过使用proxy_ignore_headers, fastcgi_ignore_headers, uwsgi_ignore_headers, scgi_ignore_headers, 和 grpc_ignore_headers 指令禁用。

### root DIRECTIVE
Syntax:	root path;  
Default:	  
root html;  
Context:	http, server, location, if in location  

>Sets the root directory for requests. For example, with the following configuration  

为请求设置根目录，例如，使用如下的配置：

```
location /i/ {
    root /data/w3;
}
```

>The /data/w3/i/top.gif file will be sent in response to the “/i/top.gif” request.

在对 “/i/top.gif” 的请求中，/data/w3/i/top.gif文件将会被作为响应返回。（发送到响应中）

>The path value can contain variables, except $document_root and $realpath_root.

路径的值可以包含变量，除了$document_root 和 $realpath_root之外。

>A path to the file is constructed by merely adding a URI to the value of the root directive. If a URI has to be modified, the alias directive should be used.

一个文件的路径被构建，（仅仅，只不过，只是）通过为root指令添加一个URI的值（只需向根指令的值添加一个URI，就可以构造文件的路径）。如果一个URI不得不被定义，别名alias指令应该被使用。（如果必须修改URI，则应该使用别名指令。）

### access_log DIRECTIVE
Syntax:	access_log path [format [buffer=size] [gzip[=level]] [flush=time] [if=condition]];  
access_log off;  
Default:	  
access_log logs/access.log combined;  
Context:	http, server, location, if in location, limit_except  

>Sets the path, format, and configuration for a buffered log write. Several logs can be specified on the same level. Logging to syslog can be configured by specifying the “syslog:” prefix in the first parameter. The special value off cancels all access_log directives on the current level. If the format is not specified then the predefined “combined” format is used.

设置缓冲日志写入的路径、格式和配置。可以在同一级别上指定多个日志。可以通过在第一个参数中指定“syslog:”前缀来配置登录到syslog。特殊值off取消当前级别上的所有access_log指令。如果没有指定格式，则使用预定义的“组合”格式。

>If either the buffer or gzip (1.3.10, 1.2.7) parameter is used, writes to log will be buffered.

>The buffer size must not exceed the size of an atomic write to a disk file. For FreeBSD this size is unlimited.

>When buffering is enabled, the data will be written to the file:

>if the next log line does not fit into the buffer;

>if the buffered data is older than specified by the flush parameter (1.3.10, 1.2.7);
when a worker process is re-opening log files or is shutting down.

>If the gzip parameter is used, then the buffered data will be compressed before writing to the file. The compression level can be set between 1 (fastest, less compression) and 9 (slowest, best compression). By default, the buffer size is equal to 64K bytes, and the compression level is set to 1. Since the data is compressed in atomic blocks, the log file can be decompressed or read by “zcat” at any time.

>Example:
```
access_log /path/to/log.gz combined gzip flush=5m;
```
>For gzip compression to work, nginx must be built with the zlib library.

>The file path can contain variables (0.7.6+), but such logs have some constraints:

- the user whose credentials are used by worker processes should have permissions to create files in a directory with such logs;
- buffered writes do not work;
- the file is opened and closed for each log write. However, since the descriptors of frequently used files can be stored in a cache, writing to the old file can continue during the time specified by the open_log_file_cache directive’s valid parameter
- during each log write the existence of the request’s root directory is checked, and if it does not exist the log is not created. It is thus a good idea to specify both root and access_log on the same level:

    ```
    server {
        root       /spool/vhost/data/$host;
        access_log /spool/vhost/logs/$host;
        ...
    ```
>The if parameter (1.7.0) enables conditional logging. A request will not be logged if the condition evaluates to “0” or an empty string. In the following example, the requests with response codes 2xx and 3xx will not be logged:

```
map $status $loggable {
    ~^[23]  0;
    default 1;
}

access_log /path/to/access.log combined if=$loggable;
```
### 参考资料
[nginx官方文档](http://nginx.org/en/docs/http/ngx_http_charset_module.html#charset)

##### 邮箱：<hellosonee@gmail.com>