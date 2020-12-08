### Websockets

此示例将会显示在 Go 中 websockets 是如何起作用的。
我们将构建一个简单的服务器，该服务器回显我们发送给它的所有内容。为此，我们必须去获得流行的 gorilla/ websocket 库，如下所示：

```shell
$ go get github.com/gorilla/websocket
```

从现在开始，我们编写的每个应用程序都将可以使用此库。

```go
// websockets.go
package main

import (
    "fmt"
    "net/http"
    "github.com/gorilla/websocket"
)

var upgrader = websocket.Upgrader{
    ReadBufferSize:1024,
    WriteBufferSize:1024,
}

func main(){
    http.HandleFunc("/echo",func(w http.ResponseWriter, r *http.Request){
        conn,_ := upgrader.Upgrade(w,r,nil) //为简单起见，忽略了错误

        for {
            // 从浏览器中读取信息
            msgType, msg, err := conn.ReadMessage()
            if err != nil {
                return
            }

            // 在控制台打印信息
            fmt.Printf("%s sent: %s\n", conn.RemoteAddr(), string(msg))

            // 写个信息返回给浏览器
            if err = conn.WriteMessage(msgType, msg); err != nil {
                return
            }
        }
    })
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        http.ServeFile(w, r, "websockets.html")
    })

    http.ListenAndServe(":8080", nil)
}

```

```html
<!-- websockets.html-->
<input id="input" type="text" />
<button onclick="send()">Send</button>
<pre id="output"></pre>
<script>
  var input = document.getElementById('input');
  var output = document.getElementById('output');
  var socket = new WebSocket('ws://localhost:8080/echo');

  socket.onopen = function () {
    output.innerHTML += 'Status: Connected\n';
  };

  socket.onmessage = function (e) {
    output.innerHTML += 'Server: ' + e.data + '\n';
  };

  function send() {
    socket.send(input.value);
    input.value = '';
  }
</script>
```

[返回](../README.md)
