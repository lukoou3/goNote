
# go之 urfave/cli 命令行
https://segmentfault.com/a/1190000042579640


## 引言

urfave/cli是一个简单、快速、有趣的包，用于在 Go 中构建命令行应用程序。目标是使开发人员能够以富有表现力的方式编写快速且可分发的命令行应用程序。

## 项目地址

> 项目地址： [](https://github.com/urfave/cli "")[https://github.com/urfave/cli](https://link.segmentfault.com/?enc=yX2DQ0XRelR/SvmsdJUxWA==.+qgZHuvxs7qH5U6Z6BgaY6a6J9GVhKLNCCtT6KJdTGo= "https://github.com/urfave/cli") [star:19k]


## 使用场景

* 构建命令行应用程序

## 安装

> go get github.com/urfave/cli/v2


## 常用方法

* Run 运行

## 例子

作为一个命令行工具，urfave/cli 足够好用，你可以自定义参数、版本、说明......

我们来个 🌰 ，参数说明可看备注。





```go
package main

import (
    "fmt"
    "log"
    "os"

    "github.com/urfave/cli/v2"
)

func main() {
    var language string

    app := &cli.App{
        Name:    "godaily day003",      // cli name
        Version: "v13",                 // cli version
        Usage:   "godaily day003 test", // usage
        Flags: []cli.Flag{ // 接受的 flag
            &cli.StringFlag{ // string
                Name:        "lang",        // flag 名称
                Aliases:     []string{"l"}, // 别名
                Value:       "english",     // 默认值
                Usage:       "language for the greeting",
                Destination: &language, // 指定地址，如果没有可以通过 *cli.Context 的 GetString 获取
                Required:    true,      // flag 必须设置
            },
        },
        Action: func(c *cli.Context) error {
            name := "who"
            if c.NArg() > 0 {
                name = c.Args().Get(0)
            }
            if language == "chinese" {
                fmt.Println("你好啊", name)
            } else {
                fmt.Println("Hello", name)
            }
            return nil
        },
    }

    if err := app.Run(os.Args); err != nil {
        log.Fatal(err)
    }
}
```


运行一下

> go run main.go --lang chinese xiaoou


除此之外，还支持sub 命令等等。小欧工作中很多项目以此为源起的。

更多实例可见:[](https://cli.urfave.org/v2/getting-started/ "")[https://cli.urfave.org/v2/get...](https://link.segmentfault.com/?enc=yuK3iIUEBCJIHQNShs5M+A==.fbpTHA3Ow0hXH6/LK9Qn8PxmHGWfMH1d4Nfv/DhCyr+JgrT+Woj5I/a0hoZhvulr "https://cli.urfave.org/v2/get...")

### 实例源码

[](https://github.com/oscome/godaily/tree/main/day003 "")[https://github.com/oscome/god...](https://link.segmentfault.com/?enc=q24EAoY8YxTQ/Ov52WympA==.RJO9jB2ufQ6x9KuEwp6xBAAC+XGVA9/3zPPT/A35N4ruvGZMOU50CtL9IN2Zs8FoGBtWhz75SL87zSo9V3wQPg== "https://github.com/oscome/god...")

## tips

* 参数较多时需要注意，例如配合 go-micro 的时候，参数会有重合，可能会产生影响。



