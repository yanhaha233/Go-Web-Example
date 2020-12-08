### Introduction 介绍

Go 的 html / template 包为 HTML 模板提供了丰富的模板语言。
它主要用于 Web 应用程序中，以结构化的方式在客户端的浏览器中显示数据。
Go 的模板语言的一大优势是自动转义数据。
无需担心 XSS 攻击，因为 Go 会解析 HTML 模板并在将其显示给浏览器之前转义所有输入。

### First Template 第一个模板

用 Go 编写模板非常简单。本示例显示了一个 TODO 列表，在 HTML 中以无序列表（ul）的形式编写。
呈现模板时，传入的数据可以是 Go 的任何数据结构。
它可以是简单的字符串或数字，甚至可以是嵌套的数据结构，如下例所示。
要访问模板中的数据，最重要的变量是通过{{。}}访问。花括号内的点称为流水线和数据的根元素。

```go
data := TodoPageData{
    PageTitle: "My TODO list",
    Todos:[]Todo{
        {Title:"Task 1",Done:false},
        {Title:"Task 2",Done:true},
        {Title:"Task 3",Done:true},
    },
}
```

```html
<h1>{{.PageTitle}}</h1>
<ul>
  {{range.Todos}} {{if .Done}}
  <li class="done">{{.Title}}</li>
  {{else}}
  <li>{{.Title}}</li>
  {{end}} {{end}}
</ul>
```

### Control Structures 控制结构

模板语言包含一组丰富的控件结构以呈现 HTML。在这里，您将获得最常用的概述。要获取所有可能结构的详细列表，请访问：[text/template](https://golang.org/pkg/text/template/#hdr-Actions)

|   Control Structure 控制结构   |           Definition 定义           |
| :----------------------------: | :---------------------------------: |
|      {{/* a comment */}}       |              定义评论               |
|             {{.}}              |             渲染根元素              |
|           {{.Title}}           |     在嵌套元素中渲染“标题”字段      |
| {{if .Done}} {{else}} {{end}}  |          定义一个 if 语句           |
| {{range .Todos}} {{.}} {{end}} | 遍历所有“ Todos”并使用{{.}}进行渲染 |
| {{block "content" .}} {{end}}  |    定义一个名称为“ content”的块     |

### Parsing Templates from Files 从文件解析模板

可以从字符串或磁盘上的文件解析模板。 通常情况下，模板是从磁盘中删除的，此示例说明了如何执行此操作。 在此示例中，与 Go 程序位于同一目录中的模板文件名为 layout.html。

```go
tmpl,err := template.ParseFiles("layout.html")
// or
tmpl := template.Must(template.ParseFiles("layout.html"))
```

### Execute a Template in a Request Handler 在请求处理程序中执行模板

从磁盘上解析模板后，就可以在请求处理程序中使用它了。 Execute 函数接受用于写出模板的 io.Writer 和用于将数据传递到模板的 interface {}。 在 http.ResponseWriter 上调用该函数时，将在 HTTP 响应中自动将 Content-Type is 标头设置为 Content-Type：text

```go
func (w http.ResponseWriter,r *http.Request){
    tmpl.Execute(w,"data goes here")
}
```

### The Code

```go
package main

import (
    "html/template"
    "net/http"
)

type Todo struct {
    Title string
    Done bool
}

type TodoPageData struct{
    PageTitle string
    Todos []Todo
}

func main(){
    tmpl := template.Must(template.ParseFiles("layout.html"))
    http.HandleFunc("/",func(w http.ResponseWriter,r *http.Request){
        data := TodoPageData{
            PageTitle : "My TODO list",
            Todos: []Todo{
                {Title: "Task 1", Done: false},
                {Title: "Task 2", Done: true},
                {Title: "Task 3", Done: true},
            },
        }
        tmpl.Execute(w,data)
    })
    http.ListenAndServe(":80",nil)
}
```

```html
<h1>{{.PageTitle}}</h1>
<ul>
  {{range .Todos}} {{if .Done}}
  <li class="done">{{.Title}}</li>
  {{else}}
  <li>{{.Title}}</li>
  {{end}} {{end}}
</ul>
```

[返回](../README.md)
