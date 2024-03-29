---
layout: mypost
title: HTTP包学习
categories: [Golang]
---

http 包学习的一些代码

### ServeMux

这是系统默认提供的一套 ServeMux，实现了路由注册匹配,404 等相关功能

当使用`http.ListenAndServe`启动一个服务时，第二个参数为空的时候，就会创建一个默认的`http.DefaultServeMux`

注册路由使用`http.HandleFunc`或者使用`http.Handle`

```go
type h struct{}

// 1.实现http.Handler
func (i *h) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "/path1")
}
func main() {
    http.Handle("/path1", &h{})
    http.HandleFunc("/path2", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintln(w, "/path2")
    })
    http.ListenAndServe(":8080", nil)
}
```

为了防止多个程序干扰`DefaultServeMux`，也可以自己从`ServeMux`建立，而不是使用默认建立的`DefaultServeMux`

```go
mux := http.NewServeMux()
mux.HandleFunc("/path2", func(w http.ResponseWriter, r *http.Request) {
    http.ServeFile(w, r, "./1.html")
})
http.ListenAndServe(":8080", mux)
```

### 快速实现静态文件服务器

提供了 Handler 的快速实现，会根据文件后缀设置头

```go
http.Handle("/", http.FileServer(http.Dir("./www")))
http.ListenAndServe(":8080", nil)
```

当需要在自己的 Handler 中根据后缀返回一个文件时候可以使用如下方法，它会自动设置一些头

```go
http.ServeFile(w, r, "./1.html")
```

### 自己实现 Mux

得益于灵活的接口实现，完全可以自己实现一套，自已进行路径的匹配处理等逻辑

```go
type MyMux struct {
    // 所有注册的路由
    Route map[string]http.HandlerFunc
}

// 返回一个Router实例
func NewMyMux() *MyMux {
    r := &MyMux{}
    r.Route = make(map[string]http.HandlerFunc)
    return r
}

// 根据路径将方法注册到路由
func (mux *MyMux) AddRouter(path string, f http.HandlerFunc) {
    if _, status := mux.Route[path]; status {
        log.Println("警告:路由被覆盖", path)
    }
    mux.Route[path] = f
}

// Mux必须实现Handler接口
func (mux *MyMux) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    reqPath := r.URL.Path
    // 这里你可以做任何的自定义处理
    if value, status := mux.Route[reqPath]; status {
        value(w, r)
    } else {
        log.Println("警告:未命中路由", reqPath)
        w.WriteHeader(404)
        w.Write([]byte("404 Not Found !"))
    }
}

func main() {
    mux := NewMyMux()
    mux.AddRouter("/123", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("/123"))
    })
    mux.AddRouter("/456", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("/456"))
    })
    http.ListenAndServe(":8080", mux)
}
```

### 第三方 httprouter

默认的 ServeMux 不好用，路由匹配不支持正则也不支持参数提取

自己实现又太麻烦，其实第三方已经有了完整的轮子

这里推荐使用[julienschmidt/httprouter](https://github.com/julienschmidt/httprouter)

功能很多，如区分method，Named parameters，Basic Authentication

```go
router := httprouter.New()
// /hello/      x
// /hello/zhang o
// /hello/zhang/ -> /hello/zhang o
// /hello/zhang/18 x
router.GET("/hello/:name", func(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
    fmt.Fprintf(w, "hello, %s!\n", ps.ByName("name"))
})
// *匹配所有
// /download - > /download/  path=/
// /download/1/2/3 - > /download/  path=/1/2/3
router.GET("/download/*filepath", func(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
    fmt.Fprintf(w, "filepath=%s", ps.ByName("filepath"))
})
log.Fatal(http.ListenAndServe(":8080", router))
```
