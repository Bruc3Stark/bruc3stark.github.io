---
layout: mypost
title: Nginx 常用配置
categories: [Linux]
---

记录一些 Nginx 的常用配置及其说明

### 反向代理

配置及其注释如下，可以看到 proxy_pass 后面加`/`与不加`/`的区别

总结起来可以把 `/proxy/` 理解为`~ ^/proxy/(.*)`。如果 `proxy_pass` 以 `/`结尾，则在`proxy_pass`后拼接上 `$1`,否则拼拼接上 `$uri`

```conf
server {
  listen       80;
  server_name  localhost;

  # http://127.0.0.1/proxy1/api/test.json
  # => http://127.0.0.1:5500/proxy1/api/test.json
  location /proxy1/ {
    proxy_pass http://127.0.0.1:5500;
  }

  # http://127.0.0.1/proxy2/api/test.json
  # => http://127.0.0.1:5500/api/test.json
  location /proxy2/ {
    proxy_pass http://127.0.0.1:5500/;
  }

  # http://127.0.0.1/proxy3/api/test.json
  # => http://127.0.0.1:5500/apiapi/test.json
  location /proxy3/ {
    proxy_pass http://127.0.0.1:5500/api;
  }

  # http://127.0.0.1/proxy4/api/test.json
  # => http://127.0.0.1:5500/api/api/test.json
  location /proxy4/ {
    proxy_pass http://127.0.0.1:5500/api/;
  }
}
```

### Tomcat 反向代理配置

所以对于 tomcat 下 context path 为 `/app01/`的应用，如果为期分配一个域名单独访问，最佳写法如下

```conf
location / {
    proxy_set_header Host $host;
    proxy_set_header X-Real-Ip $remote_addr;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_pass http://127.0.0.1:8080/app01/;
}
# 某些接口使用绝对路径时，请求`/app01/api/`经过上述代理实际上在内部请求的是`:8080/app01/app01/api/`
# 所以需要加上如下配置
location /app01/ {
    proxy_set_header Host $host;
    proxy_set_header X-Real-Ip $remote_addr;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_pass http://127.0.0.1:8080;
}
```

### 使用 try_files 修改静态资源路径

```conf
server {
  root            /apps/abc.com/;

  location ~ ^/coverage/s([1|3|6|7])/res/product/([0-9a-z]+)/(.+)\.html$ {
    add_header    Cache-Control  no-cache;
    try_files     "/product/$2/$3.html" =404; # 相对于root
  }

  location ~ ^/coverage/s([1|3|6|7])/res/product/([0-9a-z]+)/(.*) {
      # 把正则结果起来，后面再次使用正则后 $2 $3 就变了
      set $target  /product/$2/$3;

      if ($request_uri ~ /$) {
          rewrite /$ "${request_uri}index.html" last;
      }

      add_header    Cache-Control  max-age=3600;
      try_files     $target =404;
  }
}
```

### 配置 gzip

```conf
gzip             on;
gzip_comp_level  5;
gzip_min_length  4k;
gzip_types       text/plain text/css text/javascript text/xml application/javascript application/json;
```
