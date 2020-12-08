### Middleware (Advanced) 中间件

此示例将展示如何在 Go 中创建中间件的更高级版本。
中间件本身只是将 http.HandlerFunc 作为其参数之一，将其包装并返回新的 http.HandlerFunc 供服务器调用。
在这里，我们定义了一种新型的中间件，最终使将多个中间件链接在一起变得更加容易。这个想法的灵感来自 Mat Ryers 关于建筑 API 的演讲。您可以在此处找到更详细的解释，包括演讲。
该片段详细说明了如何创建新的中间件。在下面的完整示例中，我们通过一些样板代码来简化此版本。

```go
func createNewMiddleware() Middleware {

    // 创建一个新的中间件
    middleware := func(next http.HandlerFunc) http.HandlerFunc {

        // 定义服务器最终调用的http.HandlerFunc
        handler := func(w http.ResponseWriter, r *http.Request) {

            // ... 中间件

            // 调用链中的下一个中间件/处理程序
            next(w, r)
        }

        // 返回新创建的处理程序
        return handler
    }

    // 返回新创建的中间件
    return middleware
}
```

整体例子

```go
// advanced-middleware.go
package main

import (
    "fmt"
    "log"
    "net/http"
    "time"
)

type Middleware func(http.HandlerFunc) http.HandlerFunc

// 日志记录所有请求及其路径和处理时间
func Logging() Middleware {

    // 创建一个新的中间件
    return func(f http.HandlerFunc) http.HandlerFunc {

        // 定义http.HandlerFunc
        return func(w http.ResponseWriter, r *http.Request) {

            // 中间件
            start := time.Now()
            defer func() { log.Println(r.URL.Path, time.Since(start)) }()

            // 调用链中的下一个中间件/处理程序
            f(w, r)
        }
    }
}

// 方法确保只能使用特定方法来请求url，否则返回400 Bad Request
func Method(m string) Middleware {

    // 创建一个新的中间件
    return func(f http.HandlerFunc) http.HandlerFunc {

        // 定义http.HandlerFunc
        return func(w http.ResponseWriter, r *http.Request) {

            // 中间件
            if r.Method != m {
                http.Error(w, http.StatusText(http.StatusBadRequest), http.StatusBadRequest)
                return
            }

            // 调用链中的下一个中间件/处理程序
            f(w, r)
        }
    }
}

// 链将中间件应用于http.HandlerFunc
func Chain(f http.HandlerFunc, middlewares ...Middleware) http.HandlerFunc {
    for _, m := range middlewares {
        f = m(f)
    }
    return f
}

func Hello(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "hello world")
}

func main() {
    http.HandleFunc("/", Chain(Hello, Method("GET"), Logging()))
    http.ListenAndServe(":8080", nil)
}
```

[返回](../README.md)
