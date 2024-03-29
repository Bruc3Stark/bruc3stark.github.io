---
layout: mypost
title: HTTP包默认路由匹配规则
categories: [Golang]
---

最近看到 http 包的相关内容，写了几个路由发现规则好像不是正则匹配，下面从源码触发分析下路由匹配和执行的过程

### 问题引入

```go
//路由1
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "/")
})
//路由2
http.HandleFunc("/path/", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "/path/")
})
//路由3
http.HandleFunc("/path/subpath", func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "/path/subpath")
})
// 传入nil，使用默认的DefaultServeMux中间件
http.ListenAndServe(":8080", nil)
```

上面的代码的执行情况如下，对于`/path`和`/path/subpath/`的结果是不是很意外

```
127.0.0.1:8080               输出 /
127.0.0.1:8080/abc/def       输出 /
127.0.0.1:8080/path          输出 /path/
127.0.0.1:8080/path/         输出 /path/
127.0.0.1:8080/path/subpath  输出 /path/subpath
127.0.0.1:8080/path/subpath/ 输出 /path/
```

### 源码解读

一般中间件的结构如下

```go
type ServeMux struct {
	mu    sync.RWMutex
	m     map[string]muxEntry // 所有注册的路由
	es    []muxEntry // 所有以/结尾的路由规则，有长到短排序
	hosts bool
}
```

先从`http.HandleFunc`路由注册追踪，一层一层往下点，找到如下代码

```go
func (mux *ServeMux) Handle(pattern string, handler Handler) {
    mux.mu.Lock()
    defer mux.mu.Unlock()

    if pattern == "" {
        panic("http: invalid pattern")
    }
    if handler == nil {
        panic("http: nil handler")
    }
    if _, exist := mux.m[pattern]; exist {
        panic("http: multiple registrations for " + pattern)
    }
    // 懒加载
    if mux.m == nil {
        mux.m = make(map[string]muxEntry)
    }
    e := muxEntry{h: handler, pattern: pattern}
    // 所有注册的路由存放到mux.m里面
    mux.m[pattern] = e
    // 如果路由以/做结尾，则在mux.es中存放一份规则，同时做好从长到短的排序，便于以后
    if pattern[len(pattern)-1] == '/' {
        mux.es = appendSorted(mux.es, e)
    }
    if pattern[0] != '/' {
        mux.hosts = true
    }
}
```

路由注册完后进入到`ListenAndServe`中，层层往下找，找到如下循环等待请求的代码

`func (c *conn) serve(ctx context.Context)`中又执行了`serverHandler{c.server}.ServeHTTP(w, w.req)`

```go
func (srv *Server) Serve(l net.Listener) error {
    // ...
    for {
        // ...
        // 启动一个goroutine处理请求
        go c.serve(ctx)
    }
}
```

找到接口 ServeHTTP，ServeMux 的实现`func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request)`继续往下追踪发现`redirectToPathSlash`方法,进而发现 shouldRedirectRLocked 方法。

这就解释了访问`/path`会被重定向到`/path/`的原因

```go
// mux.m 存在全匹配的的 fasle
// mux.m 不存在，若mux.m[c+"/"]存在返回true
func (mux *ServeMux) shouldRedirectRLocked(host, path string) bool {
    p := []string{path, host + path}

    for _, c := range p {
        if _, exist := mux.m[c]; exist {
            return false
        }
    }

    n := len(path)
    if n == 0 {
        return false
    }
    for _, c := range p {
        if _, exist := mux.m[c+"/"]; exist {
            return path[n-1] != '/'
        }
    }

    return false
}
```

在`func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request)`继续往下追踪，查看后继处理，找到 match 规则

```go
// 优先匹配mux.m
// 如果mux.m，然后最长匹配 mux.es
func (mux *ServeMux) match(path string) (h Handler, pattern string) {
    // Check for exact match first.
    v, ok := mux.m[path]
    if ok {
        return v.h, v.pattern
    }

    // Check for longest valid match.  mux.es contains all patterns
    // that end in / sorted from longest to shortest.
    for _, e := range mux.es {
        if strings.HasPrefix(path, e.pattern) {
            return e.h, e.pattern
        }
    }
    return nil, ""
}
```

### 总结

综上源码，匹配规则如下

- 先判断完全匹配

- 再判断重定向，如果你定义了`/p/`,而没有定义`/p`，访问`host+/p`会发现直接匹配`/p`匹配不到，而`/p/`的规则是存在的，会重定向到`host+/p/`

- 最后在判断最长匹配，最长匹配的数组来自所有以`/`结尾的规则

其实可以打印出`http.DefaultServeMux`看下 m 和 es,本例中的如下

```go
// map[/:{0x6402b0 /} /path/:{0x640370 /path/} /path/subpath:{0x640430 /path/subpath}]
// [{0x640370 /path/} {0x6402b0 /}]
fmt.Println(http.DefaultServeMux)
```

### 参考

年代久远，文章中的源码都过时了

[GOLANG 中 HTTP 包默认路由匹配规则阅读笔记](https://studygolang.com/resources/4657)
