### Assets and Files

此示例将说明如何为特定目录中的 CSS，JavaScript 或图像等静态文件提供服务。

```go
// static-files.go
package main

import "net/http"

func main(){
    fs := http.FileServer(http.Dir("assets/"))
    http.Handle("/static/", http.StripPrefix("/static/", fs))
    http.ListenAndServe(":8080", nil)
}
```

```
$ tree assets/
assets/
└── css
    └── style.css
```

```shell
$ go run static-files.go
$ curl -s http://localhost:8080/static/css/style.css

body{
    background-color :black;
}
```
