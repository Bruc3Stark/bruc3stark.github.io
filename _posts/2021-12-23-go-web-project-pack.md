---
layout: mypost
title: Go Web 项目打包为一个文件
categories: [Golang]
---

使用过 golang 之后，我最喜欢的就是它的打包和交叉编译

尤其是打包，对于一些后台服务，打包成一个文件部署起来很是方便

但是在进行 web 开发时，必然会有静态资源文件，部署时就要把静态目录和打包后的二进制文件都上传到服务器。一台服务器还好，要是有多台服务器上传静态文件也是一件很麻烦的事情

去网上搜了下竟然还真有工具和把静态文件打包到二进制文件的，原理就是把静态资源生成一个很大的 go 文件（把文件进行字符编码），然后再引入到项目中，最后实现文件接口在读取文件时候从代码中读取文件

### 安装工具

安装打包工具 go-bindata 到`GOPATH/bin`中

```sh
# ... 检查所有目录下的main包编译可执行文件
go get -u github.com/jteeuwen/go-bindata/...
```

如果使用 go mod 一定要在项目外执行安装，因为这不是项目的代码依赖，同时需要指定为 master 分支，默认是安装最新的 tag，但是最新的 tag 的代码太老了，缺少`AssetInfo()`方法

```sh
go get -u github.com/jteeuwen/go-bindata/...@master
```

### 安装依赖

`go-bindata`只是把静态文件转化成 go 文件，在代码中读取这些静态文件需要使用`go-bindata-assetfs`

```sh
go get -u github.com/elazarl/go-bindata-assetfs
```

### 使用

切换到项目路径下，这里一个 web 服务器为例，所有静态资源放在 www 目录下

执行`go-bindata -o=data/data.go -pkg=data www/...`

这句话的意思是把 www 目录下所有文件生成为一个 go 文件，放置到`data/data.go`，文件的包名为`data`

```go
package main

import (
	"github.com/elazarl/go-bindata-assetfs"
	"github.com/tmaize/bindata/data"
	"net/http"
)

func main() {

	// 重新实现文件接口
	files := assetfs.AssetFS{
		Asset:     data.Asset,
		AssetDir:  data.AssetDir,
		AssetInfo: data.AssetInfo,
		Prefix:    "www", // 访问文件1.html = > 访问文件 www/1.html
	}

	// http.Handle("/", http.FileServer(http.Dir("./www")))
	http.Handle("/", http.FileServer(&files))
	http.ListenAndServe(":8899", nil)
}
```

可以看到引入了生成的`data.go`，最终在`http.FileServer`中传入自己实现的文件系统，使得在访问文件时找到对应的字符编码，再转换为文件流

### 说明

`go-bindata`是一个工具，使用把静态文件生成 go 代码，文件以 byte 数组的形式存在，项目并不依赖改项目，而是依赖该工具生成的 go 文件

`go-bindata-assetfs`是文件系统接口的实现，从生成的 go 文件中拿文件数据

如果只需要简单地读文件,可以不使用`go-bindata-assetfs`，因为`go-bindata`生成的 go 文件本身提供的一些方法返回文件的`[]byte`内容，可以根据需求自已去拿

每次静态文件改动的话在打包前记得重新执行`go-bindata`命令
