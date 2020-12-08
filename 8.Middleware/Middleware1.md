### Middleware (Basic) 中间件

此示例将展示如何在 Go 中创建基本的日志记录中间件。 中间件只是将 http.HandlerFunc 作为其参数之一，将其包装并返回新的 http.HandlerFunc 供服务器调用。

```go
// basic-middleware.go
package main

import (
    "fmt"
    "log"
    "net/http"
)

func logging(f http.HandlerFunc)http.HandlerFunc{
    return func(w http.ResponseWriter,r *http.Request){
        log.Println(r.URL.Path)
        f(w,r)
    }
}

func foo(w http.ResponseWriter,r *http.Request){
    fmt.Fprintln(w,"foo")
}

func bar(w http.ResponseWriter,r *http.Request){
    fmt.Fprintln(w,"bar")
}

func main(){
    http.HandleFunc("/foo",logging(foo))
    http.HandleFunc("/bar",logging(bar))

    http.ListenAndServe(":8080",nil)
}
```

[返回](../README.md)
