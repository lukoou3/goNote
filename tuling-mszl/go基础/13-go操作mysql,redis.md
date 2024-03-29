# 1. go 操作mysql

## 1.1 表准备

```sql
CREATE TABLE `user` (
    `user_id` int(11) NOT NULL AUTO_INCREMENT,
    `username` varchar(255) DEFAULT NULL,
    `sex` varchar(255) DEFAULT NULL,
    `email` varchar(255) DEFAULT NULL,
    PRIMARY KEY (`user_id`)
  ) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
```

> mysql的前置知识，我们这里就不讲了，可自行去学习mysql教程

## 1.2 insert操作

> 首先，需要引入mysql驱动
>
> ```go
> _ "github.com/go-sql-driver/mysql"
> ```

```go
package main

import (
    "database/sql"
    "fmt"
    _ "github.com/go-sql-driver/mysql"
    "log"
    "time"
)
var DB *sql.DB
func init()  {
    db,err := sql.Open("mysql","root:root@tcp(localhost:3306)/go_learn")
    if err != nil {
        panic(err)
    }
    //最大空闲连接数，默认不配置，是2个最大空闲连接
    db.SetMaxIdleConns(5)
    //最大连接数，默认不配置，是不限制最大连接数
    db.SetMaxOpenConns(100)
    // 连接最大存活时间
    db.SetConnMaxLifetime(time.Minute * 3)
    //空闲连接最大存活时间
    db.SetConnMaxIdleTime(time.Minute * 1)
    err = db.Ping()
    if err != nil {
        log.Println("数据库连接失败")
        db.Close()
        panic(err)
    }
    DB = db

}

func save()  {
    r,err := DB.Exec("insert into user (username,sex,email) values(?,?,?)","test001","man","001@test.com")
    if err != nil {
        log.Println("执行sql语句出错")
        panic(err)
    }
    id, err := r.LastInsertId()
    if err != nil {
        panic(err)
    }
    fmt.Println("插入成功:",id)
}
func main()  {
    defer DB.Close()
    save()
}

```

## 1.3 Select操作

```go
type User struct {
    UserId   int    `db:"user_id"`
    Username string `db:"username"`
    Sex      string `db:"sex"`
    Email    string `db:"email"`
}

func query(id int)  (*User,error) {
    rows, err := DB.Query("select * from user where user_id=? limit 1", id)
    if err != nil{
        log.Println("查询出现错误:",err)
        return nil,errors.New(err.Error())
    }
    user := new(User)
    for rows.Next() {
        if err := rows.Scan(&user.UserId,&user.Username,&user.Sex,&user.Email); err != nil{
            log.Println("scan error:",err)
            return nil,errors.New(err.Error())
        }
    }
    return user,nil
}
```

## 1.4 Update

```go
func update(username string, id int)  {
    ret, err := DB.Exec("update user set username=? where user_id=?", username, id)
    if err != nil {
        log.Println("更新出现问题:",err)
        return
    }
    affected, _ := ret.RowsAffected()
    fmt.Println("更新成功的行数:",affected)
}

```

## 1.5 Delete

```go
func delete(id int)  {
    ret, err := DB.Exec("delete from user where user_id=?", id)
    if err != nil {
        log.Println("删除出现问题:",err)
        return
    }
    affected, _ := ret.RowsAffected()
    fmt.Println("删除成功的行数:",affected)
}
```

## 1.6 事务

mysql事务特性：

1. 原子性
2. 一致性
3. 隔离性
4. 持久性

```go
func insertTx(username string)  {
    tx, err := DB.Begin()
    if err != nil {
        log.Println("开启事务错误:",err)
        return
    }
    ret, err := tx.Exec("insert into user (username,sex,email) values (?,?,?)", username, "man", "test@test.com")
    if err != nil {
        log.Println("事务sql执行出错:",err)
        return
    }
    id, _ := ret.LastInsertId()
    fmt.Println("插入成功:",id)
    if username == "lisi" {
        fmt.Println("回滚...")
        _ = tx.Rollback()
    }else {
        _ = tx.Commit()
    }

}
```

# 2. go操作Redis

> redis不另行介绍，默认会，如果不了解，先去学习redis教程

安装：go get github.com/go-redis/redis/v8

```go
package main

import (
    "context"
    "fmt"
    "github.com/go-redis/redis/v8"
)

func main()  {
    ctx := context.Background()

    rdb := redis.NewClient(&redis.Options{
        Addr:      "localhost:6379",
        Password: "", // no password set
        DB:          0,  // use default DB
    })

    err := rdb.Set(ctx, "key", "value", 0).Err()
    if err != nil {
        panic(err)
    }

    val, err := rdb.Get(ctx, "key").Result()
    if err != nil {
        panic(err)
    }
    fmt.Println("key", val)

    val2, err := rdb.Get(ctx, "key2").Result()
    if err == redis.Nil {
        fmt.Println("key2 does not exist")
    } else if err != nil {
        panic(err)
    } else {
        fmt.Println("key2", val2)
    }
}

```



