### listen DIRECTIVE
Syntax:	listen address[:port] [default_server] [ssl] [http2 | spdy] [proxy_protocol] [setfib=number] [fastopen=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [ipv6only=on|off] [reuseport] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];  
listen port [default_server] [ssl] [http2 | spdy] [proxy_protocol] [setfib=number] [fastopen=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [ipv6only=on|off] [reuseport] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];  
listen unix:path [default_server] [ssl] [http2 | spdy] [proxy_protocol] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];  
Default:	  
listen *:80 | *:8000;  
Context:	server  

语法： listen address[:port]  ...  
默认： listen *:80 | *:8080;  
语境：  

>Sets the address and port for IP, or the path for a UNIX-domain socket on which the server will accept requests. Both address and port, or only address or only port can be specified. An address may also be a hostname, for example:

为IP设置地址和端口，或者为将要接受请求的UNIX-domain(类unix域名)套接字路径设置。地址和端口，或者仅仅地址或仅仅端口可以被指定。一个地址也可以是一个主机名，例如：
```
listen 127.0.0.1:8000;
listen 127.0.0.1;
listen 8000;
listen *:8000;
listen localhost:8000;
```
>IPv6 addresses (0.7.36) are specified in square brackets:

IPv6地址在方括号中被指定：
```
listen [::]:8000;
listen [::1];
```
>UNIX-domain sockets (0.8.21) are specified with the “unix:” prefix:

UNIX-domain 类unix域名套接字用前缀“unix:”指定：
```
listen unix:/var/run/nginx.sock;
```
>If only address is given, the port 80 is used.

如果仅仅给出了地址，端口80被应用。

>If the directive is not present then either *:80 is used if nginx runs with the superuser privileges, or *:8000 otherwise.

如果指令不存在（提出、介绍、呈现、赠送、现在、礼物、瞄准、现在的、出席的），那么如果nginx以超级管理员权限运行的话*:80被应用，否则*:8080被应用。
>The default_server parameter, if present, will cause the server to become the default server for the specified address:port pair. If none of the directives have the default_server parameter then the first server with the address:port pair will be the default server for this pair.

default_server参数，如果出现，将使这个服务器变为指定地址:端口对的默认服务器。如果所有指令都没有default_server参数，那么第一个具有地址:端口对的服务器将会是这一对默认的服务器。

```
In versions prior to 0.8.21 this parameter is named simply default.
```
在0.8.21版本之前，这个参数被简单的命名为default。

>The ssl parameter (0.7.14) allows specifying that all connections accepted on this port should work in SSL mode. This allows for a more compact configuration for the server that handles both HTTP and HTTPS requests.

ssl参数允许指定在此端口上被接受的所有的连接应该在SSL模式下工作。这允许一个更紧凑的（合同、契约、紧凑的、紧密的、简洁的）服务器配置，同时处理HTTP和HTTPS请求。

>The http2 parameter (1.9.5) configures the port to accept HTTP/2 connections. Normally, for this to work the ssl parameter should be specified as well, but nginx can also be configured to accept HTTP/2 connections without SSL.

http2参数设置接受HTTP/2连接的端口。通常的，为了它能够工作，ssl参数也应该被指定，但是nginx也可以被设置为不使用SSL来接受HTTP/2连接。

>The spdy parameter (1.3.15-1.9.4) allows accepting SPDY connections on this port. Normally, for this to work the ssl parameter should be specified as well, but nginx can also be configured to accept SPDY connections without SSL.

spdy参数允许接受在此端口上的SPDY连接。通常地，为了使该参数正常工作，ssl参数也应该被指定，但是nginx也可以被设置为不使用SSL来接受SPDY连接。

>The proxy_protocol parameter (1.5.12) allows specifying that all connections accepted on this port should use the PROXY protocol.

proxy_protocol代理协议参数允许指定所有此端口上被接受的连接应该使用PROXY协议。

>The PROXY protocol version 2 is supported since version 1.13.11.  

PROXY代理协议2版本自nginx版本1.13.11以来被支持。  

>The listen directive can have several additional parameters specific to socket-related system calls. These parameters can be specified in any listen directive, but only once for a given address:port pair.

listen指令对套接字相关的系统可以有多个附加的参数指定(特殊的、明确的、特定的、详细的，特性，细节)(listen指令可以有多个附加参数，具体到与套接字相关的系统调用)。这些参数可以在任意一个（任何）listen指令中指定，但是对于一个给予的地址:端口对只能（仅仅）使用一次。

```
In versions prior to 0.8.21, they could only be specified in the listen directive together with the default parameter.
```

在0.8.21版本之前，在listen指令中，它们只能和默认的参数一起被指定。（它们只能与默认参数一起在listen指令中指定。）

- setfib=number

    >this parameter (0.8.44) sets the associated routing table, FIB (the SO_SETFIB option) for the listening socket. This currently works only on FreeBSD.
- fastopen=number

    >enables “TCP Fast Open” for the listening socket (1.5.8) and limits the maximum length for the queue of connections that have not yet completed the three-way handshake.

    >Do not enable this feature unless the server can handle receiving the same SYN packet with data more than once.
- backlog=number

    >sets the backlog parameter in the listen() call that limits the maximum length for the queue of pending connections. By default, backlog is set to -1 on FreeBSD, DragonFly BSD, and macOS, and to 511 on other platforms.
- rcvbuf=size

    >sets the receive buffer size (the SO_RCVBUF option) for the listening socket.
- sndbuf=size

    >sets the send buffer size (the SO_SNDBUF option) for the listening socket.
- accept_filter=filter

    >sets the name of accept filter (the SO_ACCEPTFILTER option) for the listening socket that filters incoming connections before passing them to accept(). This works only on FreeBSD and NetBSD 5.0+. Possible values are dataready and httpready.
- deferred

    >instructs to use a deferred accept() (the TCP_DEFER_ACCEPT socket option) on Linux.
- bind

    >instructs to make a separate bind() call for a given address:port pair. This is useful because if there are several listen directives with the same port but different addresses, and one of the listen directives listens on all addresses for the given port (*:port), nginx will bind() only to *:port. It should be noted that the getsockname() system call will be made in this case to determine the address that accepted the connection. If the setfib, backlog, rcvbuf, sndbuf, accept_filter, deferred, ipv6only, or so_keepalive parameters are used then for a given address:port pair a separate bind() call will always be made.
- ipv6only=on|off

    >this parameter (0.7.42) determines (via the IPV6_V6ONLY socket option) whether an IPv6 socket listening on a wildcard address [::] will accept only IPv6 connections or both IPv6 and IPv4 connections. This parameter is turned on by default. It can only be set once on start.

    >Prior to version 1.3.4, if this parameter was omitted then the operating system’s settings were in effect for the socket.
- reuseport
    >this parameter (1.9.1) instructs to create an individual listening socket for each worker process (using the SO_REUSEPORT socket option on Linux 3.9+ and DragonFly BSD, or SO_REUSEPORT_LB on FreeBSD 12+), allowing a kernel to distribute incoming connections between worker processes. This currently works only on Linux 3.9+, DragonFly BSD, and FreeBSD 12+ (1.15.1).
    
    >Inappropriate use of this option may have its security implications.
- so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]
 
    >this parameter (1.1.11) configures the “TCP keepalive” behavior for the listening socket. If this parameter is omitted then the operating system’s settings will be in effect for the socket. If it is set to the value “on”, the SO_KEEPALIVE option is turned on for the socket. If it is set to the value “off”, the SO_KEEPALIVE option is turned off for the socket. Some operating systems support setting of TCP keepalive parameters on a per-socket basis using the TCP_KEEPIDLE, TCP_KEEPINTVL, and TCP_KEEPCNT socket options. On such systems (currently, Linux 2.4+, NetBSD 5+, and FreeBSD 9.0-STABLE), they can be configured using the keepidle, keepintvl, and keepcnt parameters. One or two parameters may be omitted, in which case the system default setting for the corresponding socket option will be in effect. For example,

    ```
    so_keepalive=30m::10
    ```

    >will set the idle timeout (TCP_KEEPIDLE) to 30 minutes, leave the probe interval (TCP_KEEPINTVL) at its system default, and set the probes count (TCP_KEEPCNT) to 10 probes.

>Example:

```
listen 127.0.0.1 default_server accept_filter=dataready backlog=1024;
```

### 参考资料
[nginx官方文档](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen)

##### 邮箱：<hellosonee@gmail.com>