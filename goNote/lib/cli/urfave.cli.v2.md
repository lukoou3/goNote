
# 1.urfave/cli.v2的简单使用
https://www.jianshu.com/p/390187fb7ed0


## cli v2


### 开始


使用cli需要导包


```go
import github.com/urfave/v2

```



App is the main structure of a cli application.  //App是cli应用的主要结构。


It is recommended that an app be created with the cli.NewApp() function


//建议使用cli.NewApp()函数创建应用程序


```go
package main

import (
  "os"

  "github.com/urfave/cli/v2"
)

func main() {
  (&cli.App{}).Run(os.Args)
}

```


### 一个简单的例子


```go
app := &cli.App{
    Name: "boom",   //name：命令的名字

    Usage: "make an explosive entrance", //详细描述该命令的用途

    Action: func(c *cli.Context) error {    //该命令的执行动作函数
      fmt.Println("boom! I say!")
      return nil
    },
  }

err := app.Run(os.Args)     //app.Run(os.Args)
    if err != nil {
      log.Fatal(err)
  }

```


### Arguments 参数：命令后边可以加上参数


go run main.go arguments  (argument 为参数，使用c.Args().Get(0)可以获得到)


```go
Get(n int) string  //First returns the first argument, or else a blank string

```


**首先返回第一个参数，或者是一个空字符串**


```go
 app := &cli.App{
    Action: func(c *cli.Context) error {
        fmt.Printf("Hello %q", c.Args().Get(0)) //c.Args().Get(0) 用于获取命令后边跟的参数
      return nil
    },
  }

```


### Flags 标记：使bash完成命令的布尔值


```go
Flags: []cli.Flag{
            &cli.StringFlag{
                Name:  "",
                Value: "",
                Usage: "",
            },
        },
        Action: func(c *cli.Context) error {
            if c.String("lang") == "spanish"
            return nil
        },

```


为Flag设置一个终止值：destination variable


```go
    var language string     //设置一个变量用来接收命令后跟的参数 - 具名参数
&cli.StringFlag{
        Name:        "",
        Value:       "",
        Usage:       "",
        Destination: &language,   //设置一个终止值
      },

      Action: func(c *cli.Context) error {
          if c.String("lang") == "spanish"  //旧
          if language == "spanish"          //新
          return nil
      },


```


### Alternate Names：用于简洁命令选项 --lang  &&  -l


```go
Aliases: []string{"l"},

```


```go
    app := &cli.App{
        Flags: []cli.Flag{
            &cli.StringFlag{
                Name:        "lang",
                Value:       "english",
                Usage:       "language for the greeting",
                Aliases:     []string{"l"},     //Aliases：别名
                Destination: &language,
            },
    },


原始输出：           --lang value  language for the greeting (default: "english")

加上Aliases后输出：  --lang value, -l value  language for the greeting (default: "english")

```


### Placeholder Values：占位值


```go
这两个的Usage值不同，所以输出也是不同的！

Usage:   "Load configuration from `FILE`",
//--lang FILE, -l FILE  Load configuration from FILE (default: "english")

Usage:   "Load configuration from FILE",
//--lang value, -l value  Load configuration from FILE (default: "english")

```


```go
 app := &cli.App{
    Flags: []cli.Flag{
      &cli.StringFlag{
        Name:    "config",
        Aliases: []string{"c"},
        Usage:   "Load configuration from `FILE`",
      },
    },
  }

```

# 2.urfave/cli.v2的Flag选项
https://www.jianshu.com/p/63e0c5e51712


## urfave - cli.v2的进阶使用


### Ordering：排序


```go
sort.Sort(cli.FlagsByName(app.Flags))
sort.Sort(cli.CommandsByName(app.Commands))

```


```shell
//命令将排序展示

COMMANDS:
   add, a       add a task to the list
   complete, c  complete a task on the list
   help, h      Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --config FILE, -c FILE  Load configuration from FILE
   --lang value, -l value  Language for the greeting (default: "english")
   --help, -h              show help (default: false)

```


Flags for the application and commands are shown in the order they are defined. However, it's possible to sort them from outside this library by using`FlagsByName`or`CommandsByName`with`sort`.


应用程序和命令的标志按定义的顺序显示。但是，可以通过使用来从这个库之外对它们进行排序`FlagsByName`or`CommandsByName`with`sort`.


For example this:


```go
Flags: []cli.Flag{
        &cli.StringFlag{
            Name:    "lang",
            Aliases: []string{"l"},
            Value:   "english",
            Usage:   "Language for the greeting",
        },
        &cli.StringFlag{
            Name:    "config",
            Aliases: []string{"c"},
            Usage:   "Load configuration from `FILE`",
        },
    },
    Commands: []*cli.Command{
      {
        Name:    "complete",
        Aliases: []string{"c"},
        Usage:   "complete a task on the list",
        Action:  func(c *cli.Context) error {
          return nil
        },
      },
      {
        Name:    "add",
        Aliases: []string{"a"},
        Usage:   "add a task to the list",
        Action:  func(c *cli.Context) error {
          return nil
        },
      },
    },


sort.Sort(cli.FlagsByName(app.Flags))
sort.Sort(cli.CommandsByName(app.Commands))

```


### Values from the Environment：设置默认值为环境变量值


```go
EnvVars: []string{"APP_LANG"},

```


如果EnvVars包含多于一个string，则由第一个值决定


```go
EnvVars: []string{"LEGACY_COMPAT_LANG", "APP_LANG", "LANG"},

```


### Values from files：设置默认值为文件值


```go
FilePath: "abc.txt",

```


示例代码


```go
app.Flags = []cli.Flag{
        &cli.StringFlag{
            Name:     "password",
            Aliases:  []string{"p"},
            Usage:    "password for the mysql database",
            FilePath: "abc.txt",        //abc.txt文件内容为：123
        },
    }

```


```shell
输出： default值为123
GLOBAL OPTIONS:
   --password value, -p value  password for the mysql database (default: "123")
   --help, -h                  show help (default: false)

```


### Required Flags：必须参数


```go
app.Flags = []cli.Flag {
    &cli.StringFlag{
      Name: "lang",
      Value: "english",
      Usage: "language for the greeting",
      Required: true,       //Required设置为true
    },
  }

```


将Required设置为true，如果在使用命令行的时候没有lang这个flag，则会返回一个错误


```shell
2020/11/13 17:42:54 Required flag "lang" not set
exit status 1

```


### Default Values for help output：帮助输出的默认值


```go
DefaultText:"random"

```


```shell
输出： --port value  Use a randomized port (default: random)

```


```go
app := &cli.App{
    Flags: []cli.Flag{
      &cli.IntFlag{
        Name:    "port",
        Usage:   "Use a randomized port",
        Value: 0,
        DefaultText: "random",
      },
    },
  }

```


### 默认值优先级 由高到低


```css
0.Command line flag value from user     由用户在命令行输入的值
1.Environment variable (if specified)   环境变量（如果指定）
2.Configuration file (if specified)     配置文件（如果指定）
3.Default defined on the flag           Flag上定义的默认值

```

# 3.urfave/cli.v2的进阶使用
https://www.jianshu.com/p/fb945bd3fd7e


## urfave - cli.v2的进阶使用


### Ordering：排序


```go
sort.Sort(cli.FlagsByName(app.Flags))
sort.Sort(cli.CommandsByName(app.Commands))

```


```shell
//命令将排序展示

COMMANDS:
   add, a       add a task to the list
   complete, c  complete a task on the list
   help, h      Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --config FILE, -c FILE  Load configuration from FILE
   --lang value, -l value  Language for the greeting (default: "english")
   --help, -h              show help (default: false)

```


Flags for the application and commands are shown in the order they are defined. However, it's possible to sort them from outside this library by using`FlagsByName`or`CommandsByName`with`sort`.


应用程序和命令的标志按定义的顺序显示。但是，可以通过使用来从这个库之外对它们进行排序`FlagsByName`or`CommandsByName`with`sort`.


For example this:


```go
Flags: []cli.Flag{
        &cli.StringFlag{
            Name:    "lang",
            Aliases: []string{"l"},
            Value:   "english",
            Usage:   "Language for the greeting",
        },
        &cli.StringFlag{
            Name:    "config",
            Aliases: []string{"c"},
            Usage:   "Load configuration from `FILE`",
        },
    },
    Commands: []*cli.Command{
      {
        Name:    "complete",
        Aliases: []string{"c"},
        Usage:   "complete a task on the list",
        Action:  func(c *cli.Context) error {
          return nil
        },
      },
      {
        Name:    "add",
        Aliases: []string{"a"},
        Usage:   "add a task to the list",
        Action:  func(c *cli.Context) error {
          return nil
        },
      },
    },


sort.Sort(cli.FlagsByName(app.Flags))
sort.Sort(cli.CommandsByName(app.Commands))

```


### Values from the Environment：设置默认值为环境变量值


```go
EnvVars: []string{"APP_LANG"},

```


如果EnvVars包含多于一个string，则由第一个值决定


```go
EnvVars: []string{"LEGACY_COMPAT_LANG", "APP_LANG", "LANG"},

```


### Values from files：设置默认值为文件值


```go
FilePath: "abc.txt",

```


示例代码


```go
app.Flags = []cli.Flag{
        &cli.StringFlag{
            Name:     "password",
            Aliases:  []string{"p"},
            Usage:    "password for the mysql database",
            FilePath: "abc.txt",        //abc.txt文件内容为：123
        },
    }

```


```shell
输出： default值为123
GLOBAL OPTIONS:
   --password value, -p value  password for the mysql database (default: "123")
   --help, -h                  show help (default: false)

```


### Required Flags：必须参数


```go
app.Flags = []cli.Flag {
    &cli.StringFlag{
      Name: "lang",
      Value: "english",
      Usage: "language for the greeting",
      Required: true,       //Required设置为true
    },
  }

```


将Required设置为true，如果在使用命令行的时候没有lang这个flag，则会返回一个错误


```shell
2020/11/13 17:42:54 Required flag "lang" not set
exit status 1

```


### Default Values for help output：帮助输出的默认值


```go
DefaultText:"random"

```


```shell
输出： --port value  Use a randomized port (default: random)

```


```go
app := &cli.App{
    Flags: []cli.Flag{
      &cli.IntFlag{
        Name:    "port",
        Usage:   "Use a randomized port",
        Value: 0,
        DefaultText: "random",
      },
    },
  }

```


### 默认值优先级 由高到低


```css
0.Command line flag value from user     由用户在命令行输入的值
1.Environment variable (if specified)   环境变量（如果指定）
2.Configuration file (if specified)     配置文件（如果指定）
3.Default defined on the flag           Flag上定义的默认值

```

# 4.urfave/cli.v2的子命令
https://www.jianshu.com/p/83ecffda02c2


## Subcommands：子命令


可以为更像git的命令行应用程序定义子命令。


```go
Subcommands: []*cli.Command{            //子命令设置

```


```go
{
                Name:    "template",
                Aliases: []string{"t"},
                Usage:   "options for task templates",
                Subcommands: []*cli.Command{            //子命令设置
                    {
                        Name:  "add",
                        Usage: "add a new template",
                        Action: func(c *cli.Context) error {
                            fmt.Println("new task template: ", c.Args().First())
                            return nil
                        },
                    },
                    {
                        Name:  "remove",
                        Usage: "remove an existing template",
                        Action: func(c *cli.Context) error {
                            fmt.Println("removed task template: ", c.Args().First())
                            return nil
                        },
                    },
                },
            },

```


### Subcommands categories：子命令分类


* For additional organization in apps that have many subcommands, you can associate a category for each command to group them together in the help output.    
* 对于有许多子命令的应用程序中的其他组织，您可以为每个命令关联一个类别，以便在帮助输出中将它们分组在一起。


```go
 Category: "template",      //归类为template模板中

```


```go
app := &cli.App{
    Commands: []*cli.Command{
      {
        Name: "noop",
      },
      {
        Name:     "add",
        Category: "template",       //归类为template模板中
      },
      {
        Name:     "remove",
        Category: "template",       //归类为template模板中
      },
    },

```


```go
命令行输出：
COMMANDS:
   noop
   help, h  Shows a list of commands or help for one command
   template:
     add        //add和remove为template的子命令
     remove

```


### Exit code：退出代码


* 调用App.Run不会自动调用os.Exit    
* 一个显式的退出码可以通过返回一个满足cli的非nil错误来设置。ExitCoder，或者cli.MultiError。包含一个满足cli.ExitCoder的错误。    
* Action:func(ctx*cli.Context)error{if!ctx.Bool("ginger-crouton"){returncli.Exit("Ginger croutons are not in the soup",86)}returnnil},


### Combining short options：结合短选项


假设您希望用户能够将选项与他们的短名称组合在一起。这可以通过在应用程序配置中使用`UseShortOptionHandling`bool来完成，或者通过将它附加到命令配置中来完成单个命令。例如:


```go
app := &cli.App{}
app.UseShortOptionHandling = true       //设置UseShortOptionHandling 参数为 true
app.Commands = []*cli.Command{
    {
      Name:  "short",
      Usage: "complete a task on the list",
      Flags: []cli.Flag{
        &cli.BoolFlag{Name: "serve", Aliases: []string{"s"}},
        &cli.BoolFlag{Name: "option", Aliases: []string{"o"}},
        &cli.StringFlag{Name: "message", Aliases: []string{"m"}},
      },
      Action: func(c *cli.Context) error {
        fmt.Println("serve:", c.Bool("serve"))
        fmt.Println("option:", c.Bool("option"))
        fmt.Println("message:", c.String("message"))
        return nil
      },
    },
  }

//他们还可以组合起来使用，例如：
short -som "Some message"



```


### Bash Completion：Bash tab complete


你可以通过设置App中flag EnableBashCompletion为true。 默认情况下，这个设置将允许应用程序的子命令自动完成。但是你也可以为应用程序或它的子命令编写你自己的完成方法。


```go
    app := cli.NewApp()
    app.EnableBashCompletion = true

```





