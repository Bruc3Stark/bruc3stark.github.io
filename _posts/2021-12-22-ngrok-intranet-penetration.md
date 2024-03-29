---
layout: mypost
title: ngrok搭建内网穿透服务
categories: [Linux]
---

ngrok 是一款内网穿透软件，可以是你的本地项目在公网上访问。使用最多的就是微信开发调试和向别人展示你的本地项目。

之所以要自己搭建就是因为网上的收费太贵。免费的太慢且不稳定。

这里推荐下[小米球 ngrok](http://ngrok.ciqiuwl.cn),有免费和付费服务，免费的也挺稳定的，唯一的缺点是和客户端天天更新。。。

### 准备域名

先把域名 A 记录解析到服务器，注意这里做泛域名解析，做两条解析，主机记录填写例如`ngrok`和`*.ngrok`

之后 ping 一下`ngrok.xxx.com`和`任意.ngrok.xxx.com`有没有解析好，大概要 5 分钟左右

如果能对`ngrok.xxx.com`申请到泛域名 SSL 证书是最好的，可惜腾讯云里免费的 SSL 证书不支持泛域名

这里我们自己生成证书，缺点是浏览器会进行提示

```
cd ~
mkdir ca
cd ~/ca
openssl genrsa -out base.key 2048
openssl req -new -x509 -nodes -key base.key -days 10000 -subj "/CN=ngrok.xxx.com" -out base.pem
openssl genrsa -out server.key 2048
openssl req -new -key server.key -subj "/CN=ngrok.xxx.com" -out server.csr
openssl x509 -req -in server.csr -CA base.pem -CAkey base.key -CAcreateserial -days 10000 -out server.crt
```

### 下载源码编译

这里是 github 上的官方地址[https://github.com/inconshreveable/ngrok](https://github.com/inconshreveable/ngrok)

先说明下，ngrok 是使用 go 语言写的，所以先要把 go 的环境搭好，GOPATH 等配置好，这里略过

```
cd ~
git clone https://github.com/inconshreveable/ngrok.git
```

对 SSL 证书做替换

```
cp ~/ca/base.pem ~/ngrok/assets/client/tls/ngrokroot.crt
cp ~/ca/server.crt ~/ngrok/assets/server/tls/snakeoil.crt
cp ~/ca/server.key ~/ngrok/assets/server/tls/snakeoil.key
```

编译过程中需要下载依赖，稍等片刻

```
# 编译客户端，windows系统上用
export GOOS=windows
export GOARCH=amd64
make release-client

# 编译服务端
export GOOS=linux
export GOARCH=amd64
make release-server
```

编译后的文件会在 bin 目录下生成

![build](build.png)

### 运行

- 服务端启动

```
cd bin
./ngrokd -domain='ngrok.xxx.com' -httpAddr=':7080' -httpsAddr=':7443' -tunnelAddr=':7000'
```

- 客户端启动

把 windows_amd64/ngrok.exe 文件夹下下载到本地，同级目录下新建 ngrok.cfg 配置文件

ngrok.cfg 内容如下。这里的 7000 端口是连接服务端用的，因此服务端一致

```
server_addr: "ngrok.xxx.com:7000"
trust_host_root_certs: false
```

启动命令，subdomain 用于设置访问域名，5500 为本地应用服务的端口

```
ngrok.exe -log=ngrok.log -config=ngrok.cfg -subdomain test 5500
```

![result](result.png)

现在通过`http://test.ngrok.xxx.com:7080`即可访问你在本地 5500 单口启动的项目了

### nginx 代理

使用带有端口的域名还是优点麻烦，可以使用 nginx 做下反向代理

```
server {
    listen 80;
    server_name *.ngrok.xxx.com;
    location / {
        proxy_pass http://127.0.0.1:7080;
        proxy_set_header Host $host:7080;
        proxy_redirect off;
        client_max_body_size 10m;
        client_body_buffer_size 128k;
        proxy_connect_timeout 90;
        proxy_read_timeout 90;
        proxy_buffer_size 4k;
        proxy_buffers 6 128k;
        proxy_busy_buffers_size 256k;
        proxy_temp_file_write_size 256k;
    }
}
```

这样就可以直接使用`http://test.ngrok.xxx.com`访问了
