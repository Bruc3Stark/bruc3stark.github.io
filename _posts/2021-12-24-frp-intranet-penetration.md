---
layout: mypost
title: FRP快速搭建内网穿透服务
categories: [其他]
---

最近开发小程序又需要用到内网穿透服务了，之前有写过使用 ngrok 内网穿透的文章。由于换了服务器需要重新搭建，看了下之前的搭建过程感觉有点麻烦，网上搜了下发现了 FRP 这款工具，搭建起来非常的方便。

另外网上也有现成的免费服务，这里推荐[Sakura Frp](https://www.natfrp.org)，当然了如果自己有服务器的话推荐自己搭建。

### 下载使用

- [项目地址](https://github.com/fatedier/frp)

- [编译好的](https://github.com/fatedier/frp/releases)

frp 是用 Go 写的，不嫌麻烦自己也可以下载到本地编译，不过项目上已经提供了各个平台下编译好的二进制文件，直接使用就成了，windows 平台下载 windows_amd64，一般 linux 都是下载 linux_amd64

下载解压后会后两个二进制文件 frpc 和 frps，分别是客户端服务端，以及他们的 ini 配置文件。配置文件有两个。一个基础版本，只包含了必须的配置项；一个是完整版，包含所有配置项的文件；可以根据自己的需要在基础配置中添加配置

启动命令也比较简单，就一个参数，注意启动是不会使用默认配置文件的，需要自己指定

客户端 `./frpc -c ./frpc.ini`

服务端 `./frps -c ./frps.ini`

### 简单配置

服务端配置

```ini
[common]
# 客户端连接地址
bind_addr = 0.0.0.0
bind_port = 9000

# 通过这个端口访问客户端的HTTP服务
vhost_http_port = 9001

# 客户端链接凭证
token = 987654321

# web管理界面，查看连接情况的，可以不用
# dashboard_addr = 0.0.0.0
# dashboard_port = 9002
# dashboard_user = admin
# dashboard_pwd = 123456
```

客户端配置

```ini
[common]
# 服务端地址ip，域名均可
server_addr = 123.123.123.123
# 服务端frp配置的连接端口
server_port = 9000
token = 987654321

[web1]
type = http
local_ip = 127.0.0.1
# 内网APP端口
local_port = 9000
# 所以这里的域名必须是真实的，而且要解析到frps服务端的IP上
custom_domains = frp.demo.com
```

### 无域名配置

和 nginx 一样，为了复用端口，frp 会根据 host 来区分不用应用，所以当`type = http`时候，你必须要配置 custom_domains 或 subdomain

当没有域名或不想配置域名时候可以使用`type = tcp`，毕竟 http 也是属于 tcp 的

每次客户端连接后，服务端都会根据`remote_port`来启动一个 tcp 服务，所以要保证端口在服务端是空闲的

服务端配置

```ini
[common]
bind_addr = 0.0.0.0
bind_port = 9000
token = 987654321
```

客户端配置

```ini
[common]
server_addr = 123.123.123.123
server_port = 9000
token = 987654321

[web1]
# 类型是tcp
type = tcp
local_ip = 127.0.0.1
local_port = 9000
remote_port = 9001
```

### system 启动

下载的 frp 压缩包内也提供了一份 systemd 服务的写法，可以修改下 ExecStart 的内容，然后直接拷贝到`/usr/lib/systemd/system`就行了。之后就可以通过 systemct 来管理

```sh
systemctl start frps
systemctl stop frps
systemctl restart frps
systemctl status frps
systemctl enable frps
systemctl disable frps
```

### 总结

关于 frp 的配置有很多，这里值列举了我自己用到的，有兴趣可以看下`frpc_full.ini`文件

一般来说都是把 80 端口给 nginx，所以我个人使用的是上面的无域名配置。然后使用 nginx 做反向代理，这样一来配置 https 也比较方便，直接在 nginx 上配置，不用去配置 frp 的 https
