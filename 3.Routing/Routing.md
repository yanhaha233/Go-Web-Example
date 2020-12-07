### Introduction 介绍

Go 的 net/http 程序包为 HTTP 协议提供了许多功能。它做得不好的一件事是复杂的请求路由，例如将请求 url 分割成单个参数。
幸运的是，有一个非常流行的软件包，因 Go 社区中良好的代码质量而闻名。
在这里例子中，你将看到如何使用 gorilla/mux 包创建具有命名参数，GET/POST 处理程序和域限制的路由。

### Installing the gorilla/mux package 安装 gorilla/mux 包

gorilla/mux 是适用于 Go 的默认 HTTP 路由器的软件包。它具有许多功能，可以提高编写 Web 应用程序时的生产率。
它也符合 Go 的默认请求处理程序签名功能(w http.ResponseWriter, r \*http.Request)，因此该包可以与其他 HTTP 库(例如中间件或现有应用程序)混合使用。

终端安装：

```shell
$ go get -u github.com/gorilla/mux
```

### Creating a new Router 创建一个新路由

首先创建一个新的请求路由器。路由器是 Web 应用程序的主要路由器，以后将作为参数传递给服务器。它将接受所有 HTTP 连接，并将其传递给你将在其上注册的请求处理程序。

```go
r := mux.NewRouter()
```

### Registering a Request Handler 注册请求处理程序

一旦有了新的路由器，就可以像往常一样注册请求处理程序。唯一的区别是，它无需调用 http.HandleFunc(...)，你可以像这样在路由器上调用 HandleFunc:r.HandleFunc(...)

### URL Parameters URL 参数

gorilla/mux 路由器最大优势在于能够从请求 URL 中提取分段。
作为一个例子，这是你程序中的一个 URL：

```
/book/go-programming-blueprint/page/10
```

该网址有两个动态细分：
书名(go-programming-blueprint)
页数(10)
要使请求处理程序与上述 URL 匹配，可以使用 URL 模式中的占位符替换动态分段，例如：

```go
r.HandleFunc("/books/{title}/page/{page}",func(w http.ResponseWriter,r *http.Request){
    //得到书
    //导航页面
})
```

最后一件事是从这些段中获取数据。该软件包带有函数 mux.Vars(r)，该函数以 http.Request 为参数并返回各段的映射。

```go
func(w http.ResponseWriter, r *http.Request){
    vars := mux.Vars(r)
    vars["title"]
    vars["page"]
}
```

### 设置 HTTP 服务器的路由器

http.ListenAndServe(":80",nil)中的 nil 是什么？它是 HTTP 服务器的主路由器的参数，默认情况下为 nil，表示使用 net/http 软件包的默认路由器。
要使用自己的路由器，请将 nil 替换为路由器 r 的变量。

```go
http.ListenAndServe(":80",r)
```

### The Code

```go
package main

import (
    "fmt"
    "net/http"
    "github.com/gorilla/mux"
)

func main(){
    r := mux.NewRouter()

    r.HandleFunc("/books/{title}/page/{page}",func(w http.ResponseWriter,r *http.Request){
        vars := mux.Vars(r)
        title := vars["title"]
        page := vars["page"]

        fmt.Fprintf(w,"You've requested the book: %s on page %s\n",title,page)
    })
    http.ListenAndServe(":80",r)
}
```

[返回](../README.md)

### gorilla/mux 路由器的功能

##### Methods 方法

将请求处理程序限制为特定的 HTTP 方法

```go
r.HandleFunc("/books/{title}",CreateBook).Methods("POST")
r.HandleFunc("/books/{title}",ReadBook).Methods("GET")
r.HandleFunc("/books/{title}",UpdateBook).Methods("PUT")
r.HandleFunc("/books/{title}",DeleteBook).Methods("DELETE")
```

##### Hostnames & Subdomains 主机名和子域

将请求处理程序限制为特定的主机名或子域

```go
r.HandleFunc("/books/{title}",BookHandler).Host("www.mybookstore.com")
```

##### Schemes 方案

将请求处理程序限制为 http/https

```go
r.HandleFunc("/secure",SecureHandler).Schemes("https")
r.HandleFunc("/insecure",InsecureHandler).Schemes("http")
```

##### Path Prefixes & Subrouters 路径前缀和子路由器

将请求处理程序限制为特定的路径前缀

```go
bookrouter := r.PathPrefix("/books").Subrouter()
bookrouter.HandleFunc("/",AllBooks)
bookrouter.HandleFunc("/{title}",GetBook)
```

[返回](../README.md)