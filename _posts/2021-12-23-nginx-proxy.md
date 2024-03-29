---
layout: mypost
title: 使用Nginx代理上网
categories: [Linux]
---

我一般都是使用 nginx 做反向代理 tomcat 和其他应用的，其实 nginx 也是支持正向代理的

所谓正向代理就是内网用户通过网关访问外部资源，就是电脑上网时浏览器设置下 http 代理地址访问互联网

而反向代理就是外部用户通过网关访问内网资源，通俗讲就是，你的网站跑在内网的 8080 端口，别人能够通过 80 端口来访问它

### http 代理配置

```
# 正向代理上网
server {
    listen       38080;

    # 解析域名
    resolver     8.8.8.8;

    location / {
        proxy_pass $scheme://$http_host$request_uri;
    }
}
```

浏览器配置下代理 IP 和端口，然后访问[http://www.ip138.com](http://www.ip138.com),可以发现 IP 已经变化了，说明生效了

然而访问 https 网站却打不开，这是由于原生 nginx 只支持 http 正向代理，为了 nginx 支持 https 正向代理，可以打 ngx_http_proxy_connect_module 补丁+ ssl 模块支持

### 添加 https 代理模块

这里需要重新编译 nginx，需要查看当前 nginx 的版本和编译选项，然后去官网下载同版本的 nginx 源码进行重新编译

```
/usr/local/nginx/sbin/nginx -V
```

```
wget http://nginx.org/download/nginx-1.15.12.tar.gz
tar -zxvf nginx-1.15.12.tar.gz
```

下载模块 ngx_http_proxy_connect_module

```
git clone https://github.com/chobits/ngx_http_proxy_connect_module
```

打补丁，对 nginx 源码修改，这一步很重要，不然后面的 make 过不去

```
patch -d /root/nginx-1.15.12/ -p 1 < /root/ngx_http_proxy_connect_module/patch/proxy_connect_rewrite
```

在原有配置后追加模块,make 后注意不要 install

```
cd /root/nginx-1.15.12/
./configure --with-http_stub_status_module --with-http_ssl_module --with-file-aio --with-http_realip_module --add-module=/root/ngx_http_proxy_connect_module/
make
mv /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.bak
cp /root/nginx-1.15.12/objs/nginx /usr/local/nginx/sbin/
```

更改配置文件如下，然后启动服务

```
# 正向代理上网
server {
    listen       38080;

    # 解析域名
    resolver     8.8.8.8;

    # ngx_http_proxy_connect_module
    proxy_connect;
    proxy_connect_allow            443 563;
    proxy_connect_connect_timeout  10s;
    proxy_connect_read_timeout     10s;
    proxy_connect_send_timeout     10s;

    location / {
        proxy_pass $scheme://$http_host$request_uri;
    }
}
```

### 总结

代理感觉不是很稳定，有时候会打不开，尤其是 https 网站。访问国外网站千万不要这样搞，这里只是为了熟悉下 nginx 的正向代理功能
