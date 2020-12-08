### Session

本示例将展示如何使用 Go 中流行的 gorilla / sessions 包将数据存储在 session cookie 中。
Cookies 是存储在用户浏览器中的一小部分数据，并根据每个请求发送到我们的服务器。
在其中，我们可以存储例如用户是否登录到我们的网站并弄清楚他是谁（在我们的系统中）。
在此示例中，我们仅允许经过身份验证的用户在/ secret 页面上查看我们的秘密消息。
要访问它，将首先必须访问/ login 以获取有效的会话 cookie，该会话 cookie 会将他登录。
此外，他可以访问/logout 来撤消对我们秘密消息的访问。

```go
// sessions.go
package main

import (
    "fmt"
    "net/http"

    "github.com/gorilla/sessions"
)

var (
    // key must be 16, 24 or 32 bytes long (AES-128, AES-192 or AES-256)

    key = []byte("super-secret-key")
    store = sessions.NewCookieStore(key)
)

func secret(w http.ResponseWriter,r *http.Request){
    session,_ := store.Get(r,"cookie-name")

    // 检查用户是否通过身份验证

    if auth,ok := session.Values["authenticated"].(bool);!ok||!auth{
        http.Error(w,"Forbidden",http.StatusForbidden)
        return
    }

    // 打印秘密消息
    fmt.Fprintln(w, "The cake is a lie!")
}

func login(w http.ResponseWriter, r *http.Request) {
    session, _ := store.Get(r, "cookie-name")

    // 身份验证在这里
    // ...

    // 将用户设置为已认证
    session.Values["authenticated"] = true
    session.Save(r, w)
}

func logout(w http.ResponseWriter, r *http.Request) {
    session, _ := store.Get(r, "cookie-name")

    // 撤消用户身份验证
    session.Values["authenticated"] = false
    session.Save(r, w)
}

func main() {
    http.HandleFunc("/secret", secret)
    http.HandleFunc("/login", login)
    http.HandleFunc("/logout", logout)

    http.ListenAndServe(":8080", nil)
}
```

[返回](../README.md)
