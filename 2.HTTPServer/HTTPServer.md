### Introduction 介绍

在这里例子中你将会学会怎么在 Go 中创建一个基础的 HTTP 服务。首先，让我们谈谈我们的 HTTP 服务器应该具备的功能。基本的 HTTP 服务器需要处理一些关键的工作。

<b>处理动态请求</b>：处理浏览网站，登录账户或发布图像的用户的传入请求。
<b>提供静态资产</b>：为浏览器提供 JavaScript，CSS 和图像，为用户创造动态体验。
<b>接收连接</b>：HTTP Server 必须在特定端口上侦听才能接收来自 Internet 的连接。

### Process dynamic requests 处理动态请求

net/http 包包含接受请求并动态处理请求所需的所有实用程序。我们可以使用 http.HandleFunc 函数注册一个新的处理程序。它的第一个参数是需要匹配的路径，第二个是要执行的功能。在以下示例中，当某人浏览您的网站时，将会给他（她）打招呼。

```go
http.HandleFunc("/",func(w http.ResponseWriter,r *http.Request){
    fmt.Fprint(w,"Welcome to my website!")
})
```

对于动态方面，http.Request 包含有关请求及其参数的所有信息。你可以使用 r.URL.Query()。Get("token")读取 GET 参数，或者使用 r.FormValue("email")读取 POST 参数(来自 HTML 表单的字段)

### Serving static assets 提供静态资产

为了提供 js,css 和图像等静态资产，我们使用内置的 http.FileServer 并将其指向 url 路径。
为了使文件服务器正常工作，需要知道从何处提供文件。

```go
fs := http.FileServer(http.Dir("static/"))
```

一旦我们的文件服务器到位，我们只需要指向它的 url 路径，就像处理动态请求一样。
需要注意的一件事：为了正确提供文件，我们需要剥离一部分 url 路径。通常，这是我们文件所在目录的名称。

```go
http.Handle("/static/",http.StripPrefix("/static/",fs))
```

### Accept connections 接收连接

完成基本 HTTP 服务器的最后一件事是，侦听端口以接受来自 Internet 的连接。 如您所料，Go 还具有内置的 HTTP 服务器，我们可以快速启动失败的服务器。 启动后，您可以在浏览器中查看 HTTP 服务器。

```go
http.ListenAndServe(":80",nil)
```

### The Code

```go
package main

import (
    "fmt"
    "net/http"
)

func main(){
    http.HandleFunc("/",func(w http.ResponseWriter,r *http.Request){
        fmt.Fprintf(w,"Welcome to my website!")
    })

    fs := http.FileServer(http.Dir("static/"))
    http.Handle("/static/",http.StripPrefix("/static/",fs))

    http.ListenAndServe(":80",nil)
}

```

[返回](../README.md)
