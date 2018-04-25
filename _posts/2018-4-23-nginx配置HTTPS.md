## 简介
1. nginx 将http协议升级为https
2. 将网站升级为https协议

## nginx升级https
1. 阿里云申请免费ssh证书 => 需先配置自己想要设置为https协议的域名
2. nginx配置`https listen 443`，并且开启防火墙的443端口 `修改/etc/iptables/rules.v4`
3. 根据不同的接口配置不同端口的虚拟主机
4. 后台接口通过127.0.0.1:端口 反向代理来实现api的调用

## 网站升级https
1. nginx配置
 - try_files 最核心的功能是可以替代rewrite。

## 遇到问题
1. nginx配置时，在server_name后面少填写了一个分号，导致配置的域名不生效
2. 升级为https后，需要将防火墙开启443端口，不然浏览器也访问不了 `iptables-restore < /etc/iptables/rule.v4`
3. 在当前目录下查找存在某个字符串的文件 `grep -rn "www.hellobir" ./`
4. 配置完成后，访问站点报错：

```
Mixed Content: The page at 'https://www.hellobirds.top/login' was loaded over HTTPS, but requested an insecure XMLHttpRequest endpoint 'http://sone.timeline.hellobirds.top/time_line/doLoginBackend'. This request has been blocked; the content must be served over HTTPS.
```

  - 原因： https站点，不能使用http协议，因为之前网站用的是http协议，调用的接口跨域，用的也是http协议，因此需要将接口的请求地址也改为https协议

5. composer安装时使用了php7.2 ，因为系统安装了两个版本的php-fpm，php7.0（用于lnmp）和php7.2（安装了swoole扩展），因此想要使用php7.0安装composer资源
  - 首先在项目目录下载composer.phar 
  - 然后直接使用php7.0版本安装依赖包
  - php7.0 composer.phar update
```
php7.0 -r"copy('https://getcomposer.org/installer','composer-setup.php');"
php7.0 -r"if(hash_file('SHA384','composer-setup.php')==='544e09ee996cdf60ece3804abc52599c22b1f40f4323403c44d44fdfdd586475ca9813a858088ffbc1f233e9b180f061'){echo'Installer verified';} else {echo'Installer corrupt'; unlink('composer-setup.PHP');} echo PHP_EOL;"
php7.0 composer-setup.php
php7.0 -r"unlink('composer-setup.php');"
```
6. php7.0安装gd扩展
  - apt-get install php7.0-gd
  - apt-get install php7.2-gd
  - service shadowsocks restart
  - /usr/sbin/php-fpm7.0 restart
    - 重启php-fpm报错,提示不能加载gd库，但是查看phpinfo，以及php7.0 -m时发现已经加载了gd2扩展（还没弄清楚原因）

      ```
         NOTICE: PHP message: PHP Warning:  PHP Startup: Unable to load dynamic library '/usr/lib/php/20151012/php_gd2.dll' - /usr/lib/php/20151012/php_gd2.dll: cannot open shared object file: No such file or directory in Unknown on line 0
      ```
  - php7.0 --ini 查看php加载的配置文件

    ```
      Configuration File (php.ini) Path: /etc/php/7.0/cli
      Loaded Configuration File:         /etc/php/7.0/cli/php.ini
      Scan for additional .ini files in: /etc/php/7.0/cli/conf.d
      Additional .ini files parsed:      /etc/php/7.0/cli/conf.d/10-mysqlnd.ini,
    ```
7. nginx配置完成后，但是访问vue站点，页面不会自动跳转到index.html，并且刷新页面也会访问不到
  - 解决方法：nginx 配置文件增加 `index index.html;`配置项 [手册](http://nginx.org/en/docs/http/ngx_http_index_module.html)
  - 将vue站点目录权限修改为655（可读4，可执行1）`chmod 655 -R ./`，或者`chown www-data ./`
  - 修改为644时也不可以，后来又增加了可执行权限，解决（还没弄明白原因） [linux文件权限](https://help.ubuntu.com/community/FilePermissions)
8. fillzilla连接不上vsftp 
  - ps -e | grep vsftp 查看vsftp进程是否运行
  - netstat -anp | grep vsftp 查看vsftp监听了哪个端口
  - 防火墙开启21端口，重试解决
9. vue站点axios请求拦截器access_token不生效
  - 原因：写在了axios.create里面，导致每次请求都用的是最开始axios.create时的设置，并没有重新获取token。
  - 解决：将access-token写在请求拦截器里面，每次请求都会重新获取token
10. 本地使用composer安装项目依赖时报错 ` failed to open stream: php_network_getaddresses: getaddrinfo failed: ` 
  - 可能原因：Apache/PHP主机连不上dns服务器，ping 远程主机查看是否通
  - 解决：cmd > ipconfig/flushdns 刷新dns缓存

## 附上nginx配置
```
server {
    listen 443;
    server_name www.yourwebsite.com;
    ssl on;
    root /usr/share/nginx/html/timeline/public;
    index index.html index.htm index.php;
    charset utf-8,gbk;

    ssl_certificate     cert/1525160726792.pem;
    ssl_certificate_key  cert/1525160726792.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on; 

    location / { 
      #try_files $uri $uri/ /index.php?$query_string;
      if (!-e $request_filename) {
        rewrite  ^(.*)$  /index.php?s=/$1  last;
        break;
      }   
    }  

    location / {
      try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ .*\.php$ {
      fastcgi_pass 127.0.0.1:9000;
      fastcgi_index index.php;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      include fastcgi_params;
    }   

    ## 静态资源缓存
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|js|css|svg|ttf|otf|eot|woff|woff2)$
    {   
        expires      30d;
    }   
}

server {
    listen 443;
    server_name www.yourwebsite.com;
    ssl on;
    root /usr/share/nginx/html/backend;
    index index.html index.htm index.php;
    charset utf-8,gbk;

    ssl_certificate     cert/1525863436900.pem;
    ssl_certificate_key  cert/1525863436900.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on; 

    location ^~ /tba/ {
      proxy_pass http://127.0.0.1:82/;
      proxy_set_header    REMOTE-HOST $remote_addr;
      proxy_set_header   Host $host;
      proxy_set_header   X-Real-IP $remote_addr;
      proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
    }   


    ## vuejs 系统配置
    location / { 
        try_files $uri $uri/ /index.html;
    }   

#    location @rewrites {
#        rewrite ^(.+)$ /index.html last;
#    }   

    location ~ .*\.php$ {
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
    }   

        ## 静态资源缓存
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|js|css|svg|ttf|otf|eot|woff|woff2)$
    {   
        expires      30d;
    }   
}

# enable-php-cors.conf
location ~ [^/]\.php(/|$)
{
    try_files $uri =404;
    fastcgi_pass  127.0.0.1:9000;
    fastcgi_index index.php;
    include fastcgi.conf;

location ~ [^/]\.php(/|$)
{
    try_files $uri =404;
    fastcgi_pass  127.0.0.1:9000;
    fastcgi_index index.php;
    include fastcgi.conf;

    # CORS settings
    # http://enable-cors.org/server_nginx.html
    # http://10.10.0.64 - It's my front end application
     if ($request_method = 'OPTIONS') {
        add_header 'Access-Control-Allow-Origin' '*' always;
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        #
        # Custom headers and headers various browsers *should* be OK with but aren't
        #
        add_header 'Access-Control-Allow-Headers' 'X_Requested_With,DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Content-Range,Range,Authorization';
        #
        # Tell client that this pre-flight info is valid for 20 days
        #
        add_header 'Access-Control-Max-Age' 1728000;
        add_header 'Content-Type' 'text/plain; charset=utf-8';
        add_header 'Content-Length' 0;
        return 204;
     }
     if ($request_method = 'POST') {
        add_header 'Access-Control-Allow-Origin' '*' always;
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Content-Range,Range';
        add_header 'Access-Control-Expose-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Content-Range,Range,Authorization';
     }
     if ($request_method = 'GET') {
        add_header 'Access-Control-Allow-Origin' '*' always;
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Content-Range,Range';
        add_header 'Access-Control-Expose-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Content-Range,Range,Authorization';
     }
}

# fastcgi.conf
fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
fastcgi_param  QUERY_STRING       $query_string;
fastcgi_param  REQUEST_METHOD     $request_method;
fastcgi_param  CONTENT_TYPE       $content_type;
fastcgi_param  CONTENT_LENGTH     $content_length;

fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
fastcgi_param  REQUEST_URI        $request_uri;
fastcgi_param  DOCUMENT_URI       $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  REQUEST_SCHEME     $scheme;
fastcgi_param  HTTPS              $https if_not_empty;

fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;

fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;

# PHP only, required if PHP was built with --enable-force-cgi-redirect
fastcgi_param  REDIRECT_STATUS    200;


```

```
<IfModule mod_rewrite.c>
    <IfModule mod_negotiation.c>
        Options -MultiViews
    </IfModule>

    RewriteEngine On

    # Redirect Trailing Slashes If Not A Folder...
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_URI} (.+)/$
    RewriteRule ^ %1 [L,R=301]

    # Handle Front Controller...
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]

    # Handle Authorization Header
    RewriteCond %{HTTP:Authorization} .
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
</IfModule>


#apache vhost 文件已经设置了
#api全部支持跨域
#在httpd.conf中开启 mod_headers 模块
<IfModule mod_headers.c>
    Header set Access-Control-Allow-Origin "*"
    Header set Access-Control-Allow-Credentials true
    Header set Access-Control-Allow-Methods "GET,PUT,POST,DELETE,PATCH,OPTIONS"
    Header set Access-Control-Allow-Headers "X-XSRF-TOKEN,DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Content-Range,Range,Authorization"
</IfModule>
```