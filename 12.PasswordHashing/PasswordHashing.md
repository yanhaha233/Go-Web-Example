### Password Hashing (bcrypt)

此示例将说明如何使用 bcrypt 哈希密码。为此，我们必须像这样获取 golang bcrypt 库：

```shell
$ go get golang.org/x/crypto/bcrypt
```

```go
// passwords.go
package main

import (
    "fmt"

    "golang.org/x/crypto/bcrypt"
)

func HashPassword(password string) (string, error) {
    bytes, err := bcrypt.GenerateFromPassword([]byte(password), 14)
    return string(bytes), err
}

func CheckPasswordHash(password, hash string) bool {
    err := bcrypt.CompareHashAndPassword([]byte(hash), []byte(password))
    return err == nil
}

func main() {
    password := "secret"
    hash, _ := HashPassword(password) // 为了简便，省略错误

    fmt.Println("Password:", password)
    fmt.Println("Hash:    ", hash)

    match := CheckPasswordHash(password, hash)
    fmt.Println("Match:   ", match)
}
```

[返回](../README.md)
