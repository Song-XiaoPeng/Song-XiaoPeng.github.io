### 问题记录
```
location ^~ /admin-api/ 
{
    proxy_pass http://127.0.0.1:7777/;
    proxy_set_header    REMOTE-HOST $remote_addr;
    proxy_set_header   Host $host;
    proxy_set_header   X-Real-IP $remote_addr;
    proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

上述配置项的proxy_pass的值设置时应该注意：

1. 当值为http://127.0.0.1:7777/,url后边`带斜杠`，此时转发的路由为
2. 当值为http://127.0.0.1:7777,url后边`不带斜杠`，此时转发的路由为

### 软件版本
`nginx -v`  1.10.0

[nginx官网文档](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass)


