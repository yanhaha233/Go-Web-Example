### Introduction 介绍

Go 是一个充满能量的语言，在其内置了 Web 服务器。
标准库中的 net/http 包包含了所有的 HTTP 协议。（HTTP 客户端和 HTTP 服务器）
在这个例子中你会明白创建一个可在浏览器中查看的网络服务器是非常简单的。

### Registering a Request Handler 注册请求处理程序

首先，创建一个处理程序从浏览器，HTTP 客户端或 API 请求接收所有传入的 HTTP 连接。
Go 中的处理程序是具有此签名的函数。

```go
func (w http.ResponseWriter,r * http.Request)
```

此函数具有两个参数：
<b>http.ResponseWriter</b> 是 编写 text/html 响应的。
<b>http.Request</b> 包含了 HTTP 请求中的所有信息比如 URL、头文件等内容。

将请求处理程序注册到默认的 HTTP Server 中：

```go
http.HandleFunc("/",func(w http.ResponseWriter,r *http.Request){
    fmt.Fprintf(w,"Hello,you've requester:%s\n",r.URL.Path)
})
```

### Listen for HTTP Connections 侦听 HTTP 连接

Request Handler(请求处理程序)不能接收来自外部的任何 HTTP 连接。
HTTP 服务器必须侦听端口以将连接传递到请求处理程序。
因为端口 80 在大多数情况下是 HTTP 流量的默认端口，所以该服务器将继续侦听它。
以下代码将启动默认 HTTP 服务器并侦听 80 端口，浏览器导航到 localhost:/,然后可以看到服务器处理你的请求。

```go
http.ListenAndServe(":80",nil)
```

### The Code

```go
package main

func main(){

    http.HandleFunc("/"，func(w http.ResponseWriter,r *http.Request){
        fmt.Fprintf(w,"Hello,you've requested: %s\n",r.URL.Path)
    })

    http.ListenAndServe(":80",nil)
}
```

[返回](../README.md)
