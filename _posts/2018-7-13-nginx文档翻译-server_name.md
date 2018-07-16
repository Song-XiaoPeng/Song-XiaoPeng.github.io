### Server names
>Wildcard names  
Regular expressions names  
Miscellaneous names  
Internationalized names  
Optimization  
Compatibility  

通配符名称  
正则表达式名称  
混杂的名称  
国际化的名称  
优化  
兼容性  

>Server names are defined using the server_name directive and determine which [server](http://nginx.org/en/docs/http/ngx_http_core_module.html#server) block is used for a given request. See also “[How nginx processes a request](http://nginx.org/en/docs/http/request_processing.html)”. They may be defined using exact names, wildcard names, or regular expressions:  

服务器名称通过使用server_name指令被定义，决定为给与的请求使用哪一个server配置块。也可以看这个网站。它们可以使用确切的名字，通配符名称，或者正则表达式来定义。

```
server {
    listen       80;
    server_name  example.org  www.example.org;
    ...
}

server {
    listen       80;
    server_name  *.example.org;
    ...
}

server {
    listen       80;
    server_name  mail.*;
    ...
}

server {
    listen       80;
    server_name  ~^(?<user>.+)\.example\.net$;
    ...
}
```
>When searching for a virtual server by name, if name matches more than one of the specified variants, e.g. both wildcard name and regular expression match, the first matching variant will be chosen, in the following order of precedence:

当使用名称来查找一个虚拟主机时，如果名字匹配了不止一个定义的变体（多样化，不同的），例如：通配符名称和正则表达式匹配，第一个匹配的变体将被选择，按照如下的优先顺序：

1. exact name  
2. longest wildcard name starting with an asterisk, e.g. “*.example.org”  
3. longest wildcard name ending with an asterisk, e.g. “mail.*”  
4. first matching regular expression (in order of appearance in a configuration file)  

#### Wildcard names  

通配符名称

>A wildcard name may contain an asterisk only on the name’s start or end, and only on a dot border. The names “www.*.example.org” and “w*.example.org” are invalid. However, these names can be specified using regular expressions, for example, “~^www\..+\.example\.org$” and “~^w.*\.example\.org$”. An asterisk can match several name parts. The name “*.example.org” matches not only www.example.org but www.sub.example.org as well.

>A special wildcard name in the form “.example.org” can be used to match both the exact name “example.org” and the wildcard name “*.example.org”.

#### Regular expressions names

正则表达式名称
>The regular expressions used by nginx are compatible with those used by the Perl programming language (PCRE). To use a regular expression, the server name must start with the tilde character:

nginx使用的正则表达式是和那些使用Perl编程语言（PCRE）使用的正则表达式是兼容的。为了使用正则表达式，域名必须以~（波浪号）字符开始：
```
server_name  ~^www\d+\.example\.net$;
```

>otherwise it will be treated as an exact name, or if the expression contains an asterisk, as a wildcard name (and most likely as an invalid one). Do not forget to set “^” and “$” anchors. They are not required syntactically, but logically. Also note that domain name dots should be escaped with a backslash. A regular expression containing the characters “{” and “}” should be quoted:

否则，它将被看作一个明确的名字，或者如果该表达式包含一个星号，将被看作为一个通配符名字（最可能作为一个无效的名字）。不要忘记设置“^”和“$”锚点（转折点，集合，靠山）。它们不是语法上的需要，而是逻辑上的需要。同时注意，域名点应该被一个反斜杠转义（逃避，避开，被忽视）。一个包含字符串“{}”的正则表达式应该被引号包裹：
```
server_name  "~^(?<name>\w\d{1,3}+)\.example\.net$";
```
>otherwise nginx will fail to start and display the error message:

否则nginx将启动失败，显示错误消息：

```
directive "server_name" is not terminated by ";" in ...
```
指令server_name没有被分号结尾...

>A named regular expression capture can be used later as a variable:

一个命名的正则表达式捕获可以作为一个变量被稍后使用：
```
server {
    server_name   ~^(www\.)?(?<domain>.+)$;

    location / {
        root   /sites/$domain;
    }
}
```
>The PCRE library supports named captures using the following syntax:

PCRE库支持使用如下的语法的命名捕获：

1. ?<name>	Perl 5.10 compatible syntax, supported since PCRE-7.0
2. ?'name'	Perl 5.10 compatible syntax, supported since PCRE-7.0
3. ?P<name>	Python compatible syntax, supported since PCRE-4.0
>If nginx fails to start and displays the error message:
pcre_compile() failed: unrecognized character after (?< in ...
this means that the PCRE library is old and the syntax “?P<name>” should be tried instead. The captures can also be used in digital form:
```
server {
    server_name   ~^(www\.)?(.+)$;

    location / {
        root   /sites/$2;
    }
}
```
>However, such usage should be limited to simple cases (like the above), since the digital references can easily be overwritten.

然而，这些用法应该被限制在简单的情况（比如上述的），因为数字引用可以被轻易地覆盖掉。
#### Miscellaneous names
>There are some server names that are treated specially.

有些服务器名称被特殊的对待。

>If it is required to process requests without the “Host” header field in a server block which is not the default, an empty name should be specified:

如果在一个不是默认的server配置块中需要不使用‘Host’头字段处理请求时，一个空的名称应该被指定：
```
server {
    listen       80;
    server_name  example.org  www.example.org  "";
    ...
}
```
>If no server_name is defined in a server block then nginx uses the empty name as the server name.

如果没有server_name在server配置块中被定义，那么nginx会使用空的名字作为服务器的名字。

>nginx versions up to 0.8.48 used the machine’s hostname as the server name in this case.

在这种情况下，nginx版本号高达0.9.48的版本，使用机器的hostname作为服务器名称。

>If a server name is defined as “$hostname” (0.9.4), the machine’s hostname is used.

如果一个主机名被定义为“$hostname”（0.9.4），主机的hostname被使用。

>If someone makes a request using an IP address instead of a server name, the “Host” request header field will contain the IP address and the request can be handled using the IP address as the server name:

如果一个人通过使用一个IP地址而不是服务器名称提交一个请求，“Host”请求头字段将包含IP地址，并且该请求可以通过使用IP地址作为服务器名称被处理。
```
server {
    listen       80;
    server_name  example.org
                 www.example.org
                 ""
                 192.168.1.1
                 ;
    ...
}
```
>In catch-all server examples the strange name “_” can be seen:

在所有的服务器实例中，可以看到一个陌生的名字“_”：

```
server {
    listen       80  default_server;
    server_name  _;
    return       444;
}
```
>There is nothing special about this name, it is just one of a myriad of invalid domain names which never intersect with any real name. Other invalid names like “--” and “!@#” may equally be used.

这个名字没有什么特殊的地方，它仅仅是种种的无效的域名中的其一，它从不会和任意一个真实的名字相交。其它无效的名字，比如“--”和“!@#”同样可以使用。

>nginx versions up to 0.6.25 supported the special name “*” which was erroneously interpreted to be a catch-all name. It never functioned as a catch-all or wildcard server name. Instead, it supplied the functionality that is now provided by the server_name_in_redirect directive. The special name “*” is now deprecated and the server_name_in_redirect directive should be used. Note that there is no way to specify the catch-all name or the default server using the server_name directive. This is a property of the listen directive and not of the server_name directive. See also “How nginx processes a request”. It is possible to define servers listening on ports *:80 and *:8080, and direct that one will be the default server for port *:8080, while the other will be the default for port *:80:

nginx版本一直到0.6.25可以支持这个特殊的名字“*”，它被错误的解析为一个所有（全方位）的名字。它从来不被应用为一个全部的或通配符域名。相反，它提供一个目前被server_name_in_redirect指令提供的功能（它提供了server_name_in_redirect指令现在提供的功能）。现在这个特殊的名字“*”是过期的，server_name_in_redirect指令应该被使用。注意，没有方法使用server_name指令去指定一个全方位的名字或者默认的服务器。这是listen指令的一个属性而不是server_name指令。可以查看...。可以去定义服务器监听端口80和8080，指示其中一个将是端口*:8080的默认的服务器，另一个将是端口*:80默认的服务器。
```
server {
    listen       80;
    listen       8080  default_server;
    server_name  example.net;
    ...
}

server {
    listen       80  default_server;
    listen       8080;
    server_name  example.org;
    ...
}
```
#### Internationalized names
>Internationalized domain names (IDNs) should be specified using an ASCII (Punycode) representation in the server_name directive:

国际化域名(IDNs)应该在server_name指令中使用ASCII (Punycode)表示来指定:
```
server {
    listen       80;
    server_name  xn--e1afmkfd.xn--80akhbyknj4f;  # пример.испытание
    ...
}
```
#### Optimization
>Exact names, wildcard names starting with an asterisk, and wildcard names ending with an asterisk are stored in three hash tables bound to the listen ports. The sizes of hash tables are optimized at the configuration phase so that a name can be found with the fewest CPU cache misses. The details of setting up hash tables are provided in a separate [document](http://nginx.org/en/docs/hash.html).

确切的名字，以一个星号开始的通配符名字，以星号结尾的通配符名字被存储在3个被绑定了相应端口的哈希表中（以星号开头的精确名称、通配符名称和以星号结尾的通配符名称存储在绑定到侦听端口的三个散列表中）。哈希表的大小在配置阶段被优化，这样可以用最少的CPU缓存丢失找到啊一个名字。（哈希表的大小在配置阶段进行了优化，这样可以用最少的CPU缓存丢失找到一个名称。）设置哈希表的详细说明在一个分离的文档中被提供。

>The exact names hash table is searched first. If a name is not found, the hash table with wildcard names starting with an asterisk is searched. If the name is not found there, the hash table with wildcard names ending with an asterisk is searched.

精确名称的哈希表首先被搜索。如果一个名字没有被发现，以星号开头的通配符名字的哈希表被搜索。如果名字没有在这儿被找到，以星号结尾的通配符名字将被搜索。

>Searching wildcard names hash table is slower than searching exact names hash table because names are searched by domain parts. Note that the special wildcard form “.example.org” is stored in a wildcard names hash table and not in an exact names hash table.

搜索通配符名称的哈希表比精确名字的哈希表慢，因为名字是通过域名部分来进行搜索的。注意，特殊的统配符名格式“.example.org”是被存储在一个通配符哈希表中，而不是在一个精确的哈希表中。

>Regular expressions are tested sequentially and therefore are the slowest method and are non-scalable.

正则表达式被顺序的测试，因此是最慢的方式，并且是不可扩展（可攀登，可称量的，可去鳞的）的。

>For these reasons, it is better to use exact names where possible. For example, if the most frequently requested names of a server are example.org and www.example.org, it is more efficient to define them explicitly:

三条原因，在尽可能的情况下使用精确的名字是比较好的。例如，如果被请求最频繁地的域名是example.org 和 www.example.org，明确地定义它们是更有效的。
```
server {
    listen       80;
    server_name  example.org  www.example.org  *.example.org;
    ...
}
```
>than to use the simplified form:

优于使用这种简单的方式：
```
server {
    listen       80;
    server_name  .example.org;
    ...
}
```
>If a large number of server names are defined, or unusually long server names are defined, tuning the server_names_hash_max_size and server_names_hash_bucket_size directives at the http level may become necessary. The default value of the server_names_hash_bucket_size directive may be equal to 32, or 64, or another value, depending on CPU cache line size. If the default value is 32 and server name is defined as “too.long.server.name.example.org” then nginx will fail to start and display the error message:

如果大量的域名被定义，或者不寻常的长的域名被定义了，在http等级中调节server_names_hash_max_size 和 server_names_hash_bucket_size指令可能是必须的。默认的server_names_hash_max_size 和 server_names_hash_bucket_size的值可能和32，或64，或其它的值相等，取决于（依赖与）CPU缓存行大小。如果默认的值是32，服务器名称被定义为“too.long.server.name.example.org”（长度为32bytes），那么nginx将启动失败，并且显示如下的错误消息：
```
could not build the server_names_hash, you should increase server_names_hash_bucket_size: 32  
```
不能创建server_names_hashg服务器名称哈希，你应该增加server_names_hash_bucket_size：32  

>In this case, the directive value should be increased to the next power of two:

在这种情况下，指令的值应该被增长到2的下一个幂（功率，力量，能力、势力、政权、电力，激励）：

```
http {
    server_names_hash_bucket_size  64;
    ...
```    
>If a large number of server names are defined, another error message will appear:

如果很多服务器名称被定义，另一个错误消息将出现

```
could not build the server_names_hash,
you should increase either server_names_hash_max_size: 512
or server_names_hash_bucket_size: 32
```

不能创建server_names_hash，  
你应该增加server_names_hash_max_size为512，  
或者server_names_hash_bucket_size为32

>In such a case, first try to set server_names_hash_max_size to a number close to the number of server names. Only if this does not help, or if nginx’s start time is unacceptably long, try to increase server_names_hash_bucket_size.

在这种情况下，首先尝试将server_names_hash_max_size设置为一个接近于服务器名字的数字。仅仅当这没有帮助时，或者如果nginx的启动时间长的不能接受时，试着去增加server_names_hash_bucket_size。

>If a server is the only server for a listen port, then nginx will not test server names at all (and will not build the hash tables for the listen port). However, there is one exception. If a server name is a regular expression with captures, then nginx has to execute the expression to get the captures.

如果一个服务器是一个监听端口的唯一一个服务器时，nginx将不会测试服务器名字（不会为监听的端口创建哈希表）。然而，有一个例外。如果一个服务器名字是一个有捕获的正则表达式，那么nginx不得不执行表达式去得到这些捕获。

#### Compatibility

兼容性

- The special server name “$hostname” has been supported since 0.9.4.
- A default server name value is an empty name “” since 0.8.48.
- Named regular expression server name captures have been supported since 0.8.25.
- Regular expression server name captures have been supported since 0.7.40.
- An empty server name “” has been supported since 0.7.12.
- A wildcard server name or regular expression has been supported for use as the first server name since 0.6.25.
- Regular expression server names have been supported since 0.6.7.
- Wildcard form example.* has been supported since 0.6.0.
- The special form .example.org has been supported since 0.3.18.
- Wildcard form *.example.org has been supported since 0.1.13.


- 特殊的服务器名字“$hostname”自0.9.4版本以来被支持。
- 一个默认的服务器名称自0.8.48以来是一个空的名称。
- 命名的正则表达式服务器名称捕获子组自0.8.25以来一直被支持。
- 正则表达式服务器名称捕获自版本0.7.40以来一直被支持。
- 一个空的服务器名称自版本0.7.12以来一直被支持。
- 一个通配符服务器名称或正则表达式自0.6.25版本以来一直被支持作为第一个服务器名称使用。
- 正则表达式名称自0.6.0版本以来一直被支持。
- 特殊的格式“.example.org”自0.3.18以来一直被支持。
- 通配符格式“*.example.org”自0.1.13以来一直被支持。

### Setting up hashes

设置哈希
>To quickly process static sets of data such as server names, map directive’s values, MIME types, names of request header strings, nginx uses hash tables. During the start and each re-configuration nginx selects the minimum possible sizes of hash tables such that the bucket size that stores keys with identical hash values does not exceed the configured parameter (hash bucket size). The size of a table is expressed in buckets. The adjustment is continued until the table size exceeds the hash max size parameter. Most hashes have the corresponding directives that allow changing these parameters, for example, for the server names hash they are server_names_hash_max_size and server_names_hash_bucket_size.

为了快速的运行数据的静态配置，比如：域名，map指令的值，MIME类型，请求头字符串的名字，nginx使用hash表。在启动和每个重新配置过程中，nginx选择哈希表的最小可能大小，以便存储具有相同（同一的，完全相同的）哈希值的键的bucket大小不会超过配置的参数(哈希桶大小)。哈希表的大小用buckets表示。直到哈希表大写超过了哈希最大大小参数，调整会一直进行。大多数的哈希有相应的指令允许改变这些参数，例如，对于域名哈希表，它们是server_names_hash_max_size 和 server_names_hash_bucket_size指令

>The hash bucket size parameter is aligned to the size that is a multiple of the processor’s cache line size. This speeds up key search in a hash on modern processors by reducing the number of memory accesses. If hash bucket size is equal to one processor’s cache line size then the number of memory accesses during the key search will be two in the worst case — first to compute the bucket address, and second during the key search inside the bucket. Therefore, if nginx emits the message requesting to increase either hash max size or hash bucket size then the first parameter should first be increased.

哈希桶大小参数是和大小对齐的，是处理器缓存行大小的倍数（散列桶大小参数与处理器缓存行大小的倍数的大小对齐。）。通过减少内存访问数量来加速一个现代处理器上的哈希中的键的查询。如果哈希桶大小和一个处理器的缓冲行大小相等，那么在最坏情况下，在键搜索过程中的内存访问数量将会是2次，首先是计算哈希桶的地址，其次是在桶中的键搜索过程。因此，如果nginx发送消息请求去增加哈希最大值或者哈希桶的最大值，那么第一个参数应该首先被增加。

### 参考资料
[nginx官方文档](http://nginx.org/en/docs/http/server_names.html)

##### 邮箱：<hellosonee@gmail.com>