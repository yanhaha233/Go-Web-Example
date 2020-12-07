### Introduction 介绍

在某个时间点，你希望 Web 应用程序存储和检索数据库中的数据。
当您处理动态内容，提供表单供用户输入数据或存储登录名和密码凭据以供用户进行身份验证时，几乎都是这样。为此，我们使用数据库。
数据库种类形式很多，MySQL 数据库是整个网络上最常用的数据库。它已经存在了很长时间，并且已经证明了它的位置和稳定性比你想象的要多的多。
在此示例中，我们将深入研究 Go 中数据库访问的基础知识，创建数据库表，存储数据并再次取回它。

### Installing the go-sql-driver/mysql package 安装 go-sql-driver/mysql 包

Go 编程语言附带一个名为`database / sql`的便捷软件包，用于查询各种 SQL 数据库。
这很有用，因为它将所有常见的 SQL 功能抽象到一个 API 中供您使用。
Go 不包括的是数据库驱动程序。
在 Go 中，数据库驱动程序是一个软件包，用于实现特定数据库（在我们的情况下为 MySQL）的低级详细信息。
您可能已经猜到了，这对于保持向前兼容很有用。由于在创建所有 Go 软件包时，作者无法预见每个数据库将来都会投入使用，而要支持每个可能的数据库将需要进行大量维护工作。

要安装 MySQL 数据库驱动程序，请转到您选择的终端并运行：

```shell
$  go get -u github.com/go-sql-driver/mysql
```

### Connecting to a MySQL database 连接 MySQL 数据库

安装后必要的第一件事就是检测能不能正确连接到 MySQL 数据库。
如果你尚未运行 MySQL 数据库服务器，则可以使用 Docker 启动新实例。
这是 Docker MySQL 映像的官方文档：https://hub.docker.com/_/mysql
要检查我们是否可以连接到数据库，请导入数据库/ sql 和 go-sql-driver / mysql 程序包，并按如下所示打开连接：

```go
import "database/sql"
import _ "go-sql-driver/mysql"

// 配置数据库连接
db,err := sql.Open("mysql","username:password@(127.0.0.1:3306)/dbname?parseTime=true")

// 初始化与数据库的第一个连接，以查看一切是否正常
// 确认检查错误

err := db.Ping()
```

### Creating our first database table 创建第一个数据库表格

我们数据库中的每个数据条目都存储在一个特定的表中。
数据库表由列和行组成。这些列为每个数据条目提供一个标签并指定其类型。这些行是插入的数据值。
创建一个表格如下:
|id|username|password|created_at|
|:--:|:--:|:--:|:--:|
|1|johndoe|secret|2019-08-10 12:30:00|
转换为 SQL 后，用于创建表的命令将如下所示：

```sql
CREATE TABLE users(
    id INT AUTO_INCREMENT,
    username TEXT NOT NULL,
    password TEXT NOT NULL,
    created_at DATETIME,
    PRIMARY KEY (id)
);
```

现在我们有了 SQL 命令，我们可以使用 database / sql 包在我们的 MySQL 数据库中创建表：

```go
query := `
    CREATE TABLE users(
    id INT AUTO_INCREMENT,
    username TEXT NOT NULL,
    password TEXT NOT NULL,
    created_at DATETIME,
    PRIMARY KEY (id)
);`
// 在我们的数据库中执行SQL查询。检查err以确保没有错误。
_,err := db.Exec(query)
```

### Inserting our first user 插入我们的第一个用户

如果您熟悉 SQL，那么向表中插入新数据就像创建表一样容易。
需要注意的是：在默认情况下，Go 使用准备好的语句将动态数据插入到我们的 SQL 查询中，这是一种不会造成任何破坏且将用户提供的数据安全地传递到我们数据库的方式。
在 Web 编程的早期，程序员将带有查询的数据直接传递到数据库，这导致了巨大的漏洞，并可能破坏整个 Web 应用程序。
要将我们的第一个用户插入数据库表，我们创建一个如下的 SQL 查询。
如你所见，我们省略了 id 列，因为它由 MySQL 自动设置。
问号告诉 SQL 驱动程序，它们是实际数据的占位符。在这里，您可以看到我们讨论的准备好的语句。

```sql
INSERT INTO users (username,password,created_at) VALUES (?,?,?)
```

现在，我们可以在 Go 中使用此 SQL 查询，并将新行插入到表中：

```go
import "time"
username := "johndoe"
password := "secret"
createdAt := time.Now()

// 将我们的数据插入到users表中，并返回结果和可能的错误。
// 结果包含有关最后插入的ID（为我们自动生成）的信息，以及此查询影响的行数。
result,err := db.Exec(`INSERT INTO users (username,password,created_at) VALUES (?,?,?)`,username,password,createdAt)
```

要为您的用户获取新创建的 ID，只需像这样获取它

```go
userID,err := result.LastInsertId()
```

### Querying our users table 查询我们的用户表

现在我们的表中有一个用户，我们想查询它并获取其所有信息。
在 Go 中，我们有两种查询表的可能性。
有 db.Query 可以查询多行，以便我们进行迭代；还有 db.QueryRow，以防我们只想查询特定的行。
查询特定行的工作原理基本上与我们之前介绍的所有其他 SQL 命令一样。
我们的 SQL 命令通过其 ID 查询单个用户，如下所示：

```sql
SELECT id,username,password,created_at FROM users WHERE id = ?
```

在 Go 中，我们首先声明一些变量来存储数据，然后查询单个数据库行，如下所示：

```go
var (
    id int
    username string
    password string
    createdAt time.Time
)

// 查询数据库并将值扫描到变量中。不要忘记检查错误。
query := `SELECT id,username,password,created_at FROM users WHERE id = ?`
err := db.QueryRow(query,1).Scan(&id,&username,&password,&createdAt)
```

### Querying all users

在前面的部分中，我们介绍了如何查询单个用户行。查询所有现有用户在许多应用程序中都有用例。虽然这与上面的示例相似，但是涉及到更多编码。

```sql
SELECT id,username,password,created_at FROM users
```

在 Go 中，我们首先声明一些变量来存储数据，然后查询单个数据库行，如下所示：

```go
type user struct{
    id int
    username string
    password string
    createdAt time.Time
}

rows,err := db.Query(`SELECT id,username,password,created_at FROM users`) //check err
defer rows.Close()

var users []user
for rows.Next(){
    var u user
    err := rows.Scan(&u.id,&u.username,&u.password,&u.createdAt)  //check err
    users = append(users,u)
}
err := rows.Err() // check err
```

用户切片现在可能包含以下内容：

```go
users{
    user{
        id:        1,
        username:  "johndoe",
        password:  "secret",
        createdAt: time.Time{wall: 0x0, ext: 63701044325, loc: (*time.Location)(nil)},
    },
    user{
        id:        2,
        username:  "alice",
        password:  "bob",
        createdAt: time.Time{wall: 0x0, ext: 63701044622, loc: (*time.Location)(nil)},
    }
}
```

### Deleting a user from our table 从表中删除一个用户

最后，从我们的表中删除用户与上述各节中的.Exec 一样简单：

```go
_,err := db.Exec(`DELETE FROM users WHERE id = ?`,1) // check err
```

### The Code

```go
package main
import (
    "database/sql"
    "fmt"
    "log"
    "time"

    _ "github.com/go-sql-driver/mysql"
)

func main(){
    db,err := sql.Open("mysql","root:root@(127.0.0.1:3306)/root?parseTime=true")
    if err != nil{
        log.Fatal(err)
    }
    if err := db.Ping();err != nil{
        log.Fatal(err)
    }

    {
        //Create a new table
        query := `
                CREATE TABLE users (
                id INT AUTO_INCREMENT,
                username TEXT NOT NULL,
                password TEXT NOT NULL,
                created_at DATETIME,
                PRIMARY KEY (id));`
        if _,err := db.Exec(query);err != nil{
            log.Fatal(err)
        }
    }

    {
        //Insert a new user
        username := "johnode"
        password := "secret"
        createdAt := time.Now()

        result, err := db.Exec(`INSERT INTO users (username, password, created_at) VALUES (?, ?, ?)`, username, password, createdAt)
        if err != nil {
            log.Fatal(err)
        }

        id, err := result.LastInsertId()
        fmt.Println(id)
    }

    {
        // Query a single user
        var(
            id int
            username string
            password string
            createdAt time.Time
        )

        query := "SELECT id, username, password, created_at FROM users WHERE id = ?"
        if err := db.QueryRow(query, 1).Scan(&id, &username, &password, &createdAt); err != nil {
            log.Fatal(err)
        }

        fmt.Println(id, username, password, createdAt)
    }

    {
        // Query all users
        type user struct {
            id        int
            username  string
            password  string
            createdAt time.Time
        }

        rows, err := db.Query(`SELECT id, username, password, created_at FROM users`)
        if err != nil {
            log.Fatal(err)
        }
        defer rows.Close()

        var users []user
        for rows.Next() {
            var u user

            err := rows.Scan(&u.id, &u.username, &u.password, &u.createdAt)
            if err != nil {
                log.Fatal(err)
            }
            users = append(users, u)
        }
        if err := rows.Err(); err != nil {
            log.Fatal(err)
        }

        fmt.Printf("%#v", users)
    }

    {
        // Delete a user
        _,err := db.Exec(`DELETE FROM users WHERE id = ?`,1)
        if err != nil{
            log.Fatal(err)
        }
    }
}
```

[返回](../README.md)
