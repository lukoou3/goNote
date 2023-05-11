# Golang配置解析神器goviper使用详解
摘抄自：`https://www.jb51.net/article/248627.htm`

更新时间：2022年05月19日 15:18:05  作者：程序员读书

viper是一个很完善的Go项目配置解决方案，很多著名的开源项目都在使用，比如Hugo,Docker都使用了该库，使用viper可以让我们专注于自己的项目代码，而不用自己写那些配置解析代码，本文给大家介绍Golang配置解析神器goviper使用，感兴趣的朋友一起看看吧


## 前言

对于现代应用程序，尤其大中型的项目来说，在程序启动和运行时，往往需要传入很多参数来控制程序的行为，这些参数可以通过以下几种方式传递给程序：

* 命令行参数    
* 环境变量    
* 配置文件

显然，对于Go项目而言，单个去读取命令行、环境变量、配置文件并不难，但一个个读取却是很麻烦，有没有一个第三方库可以帮我们一次性读取上面几种数据源的配置呢？

有的，这里推荐使用`viper`库，viper支持读取不同数据源和不同格式的配置文件，是Go项目读取配置的神器，今天跟着这篇文章，一起来探究一下吧！~


## viper简介

viper是一个很完善的Go项目配置解决方案，很多著名的开源项目都在使用，比如`Hugo`,`Docker`都使用了该库，使用`viper`可以让我们专注于自己的项目代码，而不用自己写那些配置解析代码。


### 功能

* 支持配置key默认值设置    
* 支持读取JSON,TOML,YAML,HCL,envfile和java properties等多种不同类型配置文件    
* 可以监听配置文件的变化，并重新加载配置文件    
* 读取系统环境变量的值    
* 读取存储在远程配置中心的配置数据，如ectd，Consul,firestore等系统，并监听配置的变化    
* 从命令行读取配置    
* 从buffer读取配置    
* 可以显示设置配置的值


### viper配置优先级

viper支持从多个数据源读取配置值，因此当同一个配置key在多个数据源有值时，viper读取的优先级如下：

* 显示使用Set函数设置值    
* flag：命令行参数    
* env：环境变量    
* config：配置文件    
* key/value store：key/value存储系统，如(etcd)    
* default:默认值

优先级示意图

![picture 1](assets/img-paste-20230511-232944.png)  



### 安装viper

viper的安装非常简单，如同其他Go第三方包一样，只需要`go get`命令即可安装，如：

安装
```go
go get github.com/spf13/viper
```

使用
```go
import "github.com/spf13/viper"
```


### 支持哪些文件格式

我们一直在说，viper支持多种不同格式的配置文件，到底是哪些格式呢？如下：

* json    
* toml    
* yaml    
* yml    
* properties    
* props    
* prop    
* hcl    
* tfvars    
* dotenv    
* env    
* ini


### key大小写问题

viper的配置的key值是不区分大小写，如：
```go
# 小写的key
viper.set("test","this is a test value")
# 大写的key，也可以读到值
fmt.Println(viper.get("TEST"))//输出"this is a test value"
```

## 使用指南

在了解了viper是什么之后，下面我们来看看要怎么使用viper去帮我们读取配置。


### 如何访问viper的功能

使用包名viper，如:
```go
viper.SetDefault("key1","value")//调用包级别下的函数
```


使用`viper.New()`函数创建一个Viper Struct，如：
```go
viper := viper.New()
viper.SetDefault("key2","value2")
```

其实，这就是Go包的编程惯例，对实现功能对象再进行封装，并通过包名来调用。

因此，下面所有示例中调用函数使用viper，可以是指包名viper,或者通过viper.New()返回的对象。


### 配置默认值
```go
viper.SetDefault("key1","value1")
viper.SetDefault("key2","value2")
```


### 读取配置文件

直接指定文件路径
```go
viper.SetConfigFile("./config.yaml")
viper.ReadInConfig()
fmt.Println(viper.Get("test"))
```


多路径查找
```go
viper.SetConfigName("config")     // 配置文件名，不需要后缀名
viper.SetConfigType("yml")            // 配置文件格式
viper.AddConfigPath("/etc/appname/")  // 查找配置文件的路径
viper.AddConfigPath("$HOME/.appname") // 查找配置文件的路径
viper.AddConfigPath(".")              // 查找配置文件的路径
err := viper.ReadInConfig()           // 查找并读取配置文件
if err != nil {                       // 处理错误
    panic(fmt.Errorf("Fatal error config file: %w \n", err))
}
```

读取配置文件时，可能会出现错误，如果我们想判断是否是因为找不到文件而报错的，可以判断err是否为`ConfigFileNotFoundError`。

```go
if err := viper.ReadInConfig(); err != nil {
	if _, ok := err.(viper.ConfigFileNotFoundError); ok {
		
	} else {
		
	}
}
```


### 写配置文件

除了读取配置文件外，viper也支持将配置值写入配置文件，viper提供了四个函数，用于将配置写回文件。


#### WriteConfig

WriteConfig函数会将配置写入预先设置好路径的配置文件中，如果配置文件存在，则覆盖，如果没有，则创建。


#### SafeWriteConfig

SafeWriterConfig与WriteConfig函数唯一的不同是如果配置文件存在，则会返回一个错误。


#### WriteConfigAs

WriteConfigAs与WriteConfig函数的不同是需要传入配置文件保存路径，viper会根据文件后缀判断写入格式。


#### SafeWriteConfigAs

SafeWriteConfigAs与WriteConfigAs的唯一不同是如果配置文件存在，则返回一个错误。


### 监听配置文件

viper支持监听配置文件，并会在配置文件发生变化，重新读取配置文件和回调函数，这样可以避免每次配置变化时，都需要重启启动应用的麻烦。

```go
viper.OnConfigChange(func(e fsnotify.Event) {
	fmt.Println("Config file changed:", e.Name)
})
viper.WatchConfig()
```

### 从io.Reader读取配置

除了支持从配置文件读取配置外，viper也支持从实现了io.Reader接口的实例中读取配置(其实配置文件也实现了io.Reader)，如：

```go
viper.SetConfigType("json") //设置格式

var yamlExample = []byte(`
{
	"name":"小明"
}
`)
viper.ReadConfig(bytes.NewBuffer(yamlExample))
fmt.Println(viper.Get("name")) //输出“小明”

```


### 显示设置配置项

也可以使用`Set`函数显示为某个key设置值，这种方式的优先级最高，会覆盖该key在其他地方的值，如：

```go
viper.SetConfigType("json") //设置格式
var yamlExample = []byte(`
{
	"name":"小明"
}
`)
viper.ReadConfig(bytes.NewBuffer(yamlExample))
fmt.Println(viper.Get("name")) //输出:小明
viper.Set("name","test")
fmt.Println(viper.Get("name"))//输出:test
```

### 注册和使用别名

为某个配置key设置别名，这样可以方便我们在不改变key的情况下，使用别的名称访问该配置。

```go
viper.Set("name", "test")
//为name设置一个username的别名
viper.RegisterAlias("username", "name")
//通过username可以读取到name的值
fmt.Println(viper.Get("username"))
//修改name的配置值，username的值也发生改变
viper.Set("name", "小明")
fmt.Println(viper.Get("username"))
//修改username的值，name的值也发生改变
viper.Set("username", "测试")
fmt.Println(viper.Get("name"))
```



### 读取环境变量

对于读取操作系统环境变量，viper提供了下面五个函数：

* AutomaticEnv()    
* BindEnv(string...) : error    
* SetEnvPrefix(string)    
* SetEnvKeyReplacer(string...) *strings.Replacer    
* AllowEmptyEnv(bool)

要让viper读取环境变量，有两种方式：

* 调用AutomaticEnv函数，开启环境变量读取

```go
fmt.Println(viper.Get("path"))
//开始读取环境变量，如果没有调用这个函数，则下面无法读取到path的值
viper.AutomaticEnv()
//会从环境变量读取到该值，注意不用区分大小写
fmt.Println(viper.Get("path"))
```


* 使用BindEnv绑定某个环境变量

```go
//将p绑定到环境变量PATH,注意这里第二个参数是环境变量，这里是区分大小写的
viper.BindEnv("p", "PATH")
//错误绑定方式，path为小写，无法读取到PATH的值
//viper.BindEnv("p","path")
fmt.Println(viper.Get("p"))//通过p可以读取PATH的值
```


使用函数SetEnvPrefix可以为所有环境变量设置一个前缀，这个前缀会影响`AutomaticEnv`和`BindEnv`函数

```go
os.Setenv("TEST_PATH","test")
viper.SetEnvPrefix("test")
viper.AutomaticEnv()
//无法读取path的值，因为此时加上前缀，viper会去读取TEST_PATH这个环境变量的值
fmt.Println(viper.Get("path"))//输出:nil
fmt.Println(viper.Get("test_path"))//输出：test
```


环境变量大多是使用下划号(_)作为分隔符的，如果想替换，可以使用`SetEnvKeyReplacer`函数，如：

```go
//设置一个环境变量
os.Setenv("USER_NAME", "test")
//将下线号替换为-和.
viper.SetEnvKeyReplacer(strings.NewReplacer("-", "_", ".", "_"))
//读取环境变量
viper.AutomaticEnv()
fmt.Println(viper.Get("user.name"))//通过.访问
fmt.Println(viper.Get("user-name"))//通过-访问
fmt.Println(viper.Get("user_name"))//原来的下划线也可以访问
```


默认的情况下，如果读取到的环境变量值为空(注意，不是环境变量不存在，而是其值为空)，会继续向优化级更低数据源去查找配置，如果想阻止这一行为，让空的环境变量值有效，则可以调用`AllowEmptyEnv`函数：


```go
viper.SetDefault("username", "admin")
viper.SetDefault("password", "123456")
//默认是AllowEmptyEnv(false)，这里设置为true
viper.AllowEmptyEnv(true)
viper.BindEnv("username")
os.Setenv("USERNAME", "")
fmt.Println(viper.Get("username"))//输出为空，因为环境变量USERNAME空
fmt.Println(viper.Get("password"))//输出：123456
```


### 与命令行参数搭配使用

viper可以和解析命令行库相关flag库一起工作，从命令行读取配置，其内置了对pflag库的支持，同时也留有接口让我们可以支持扩展其他的flag库


### pflag

```go
pflag.Int("port", 8080, "server http port")
pflag.Parse()
viper.BindPFlags(pflag.CommandLine)
fmt.Println(viper.GetInt("port"))//输出8080
```

### 扩展其他flag

如果我们没有使用pflag库，但又想让viper帮我们读取命令行参数呢？


```go
package main
import (
	"flag"
	"fmt"
	"github.com/spf13/viper"
)
type myFlag struct {
	f *flag.Flag
}
func (m *myFlag) HasChanged() bool {
	return false
}
func (m *myFlag) Name() string {
	return m.f.Name
}
func (m *myFlag) ValueString() string {
	return m.f.Value.String()
}
func (m *myFlag) ValueType() string {
	return "string"
}
func NewMyFlag(f *flag.Flag) *myFlag {
	return &myFlag{f: f}
}
func main() {
	flag.String("username", "defaultValue", "usage")
	m := NewMyFlag(flag.CommandLine.Lookup("username"))
	viper.BindFlagValue("myFlagValue", m)
	flag.Parse()
	fmt.Println(viper.Get("myFlagValue"))
}
```



### 远程key/value存储支持

viper支持存储或者读取远程配置存储中心和NoSQL(目前支持etcd,Consul,firestore)的配置，并可以实时监听配置的变化，不过需要在代码中引入下面的包：

```go
import _ "github.com/spf13/viper/remote"
```


现在远程配置中心存储着以下JSON的配置信息


```go
{
	"hostname":"localhost",
	"port":"8080"
}
```


那么我们可以通过下面的方面连接到系统，并读取配置，也可以单独开启一个Goroutine实时监听配置的变化。

连接Consul


```go
viper.AddRemoteProvider("consul", "localhost:8500", "MY_CONSUL_KEY")
```


连接etcd

```go
viper.AddRemoteProvider("etcd", "http://127.0.0.1:4001","/config/hugo.json")
```


连接firestore


```go
viper.AddRemoteProvider("firestore", "google-cloud-project-id", "collection/document")
```


连接上配置中心后，就可以像读取配置文件读取其中的配置了，如：

```go
viper.SetConfigType("json")
err := viper.ReadRemoteConfig()
fmt.Println(viper.Get("port")) // 输出:8080
fmt.Println(viper.Get("hostname")) // 输出:localhost
```


监听配置系统，如：


```go
go func(){
	for {
		time.Sleep(time.Second * 5) 
		err := viper.WatchRemoteConfig()
		if err != nil {
			log.Errorf("unable to read remote config: %v", err)
			continue
		}
	}
}()
```


另外，viper连接etcd,Consul,firestore进行配置传输时，也支持加解密，这样可以更加安全，如果想要实现加密传输可以把`AddRemoteProvider`函数换为`SecureRemoteProvider`。

```go
viper.SecureRemoteProvider("etcd", "http://127.0.0.1:4001","/config/hugo.json","/etc/secrets/mykeyring.gpg")
```

### 访问配置

viper可以帮我们读取各个地方的配置，那读到配置之后，要怎么用呢？


### 直接访问

```go
{
  "mysql":{
    "db":"test"
  },
  "host":{
	  "address":"localhost"
	  "ports":[
		  "8080",
		  "8081"
	  ]
  }
}
```


对于多层级配置key，可以用逗号隔号,如：

```go
viper.Get("mysql.db")
viper.GetString("user.db")
viper.Get("host.address")//输出：localhost
```


数组，可以用序列号访问，如：

```go
viper.Get("host.posts.1")//输出: 8081
```

也可以使用`sub`函数解析某个key的下级配置,如：

```go
hostViper := viper.Sub("host")
fmt.Println(hostViper.Get("address"))
fmt.Println(hostViper.Get("posts.1"))
```

viper提供了以下访问配置的的函数：

* Get(key string) : interface{}    
* GetBool(key string) : bool    
* GetFloat64(key string) : float64    
* GetInt(key string) : int    
* GetIntSlice(key string) : []int    
* GetString(key string) : string    
* GetStringMap(key string) : map[string]interface{}    
* GetStringMapString(key string) : map[string]string    
* GetStringSlice(key string) : []string    
* GetTime(key string) : time.Time    
* GetDuration(key string) : time.Duration


### 序列化

读取了配置之后，除了使用上面列举出来的函数访问配置，还可以将配置序列化到struct或map之中，这样可以更加方便访问配置。

示例代码

配置文件：config.yaml

```yaml
host: localhost
username: test
password: test
port: 3306
charset: utf8
dbName: test
```


解析代码：

```go
type MySQL struct {
	Host     string
	DbName   string
	Port     string
	Username string
	Password string
	Charset  string
}
func main() {

	viper.SetConfigName("config")
	viper.SetConfigType("yaml")
	viper.AddConfigPath(".")
	viper.ReadInConfig()
	var mysql MySQL
	viper.Unmarshal(&mysql)//序列化

	fmt.Println(mysql.Username)
	fmt.Println(mysql.Host)
}
```


对于多层级的配置，viper也支持序列化到一个复杂的struct中，如：

我们将config.yaml改为如下结构：

```yaml
mysql: 
  host: localhost
  username: test
  password: test
  port: 3306
  charset: utf8
  dbName: test
redis: 
  host: localhost
  port: 6379
```


示例程序

```go
type MySQL struct {
	Host     string
	DbName   string
	Username string
	Password string
	Charset  string
}
type Redis struct {
	Host string
	Port string
}
type Config struct {
	MySQL MySQL
	Redis Redis
}
func main() {
	viper.SetConfigName("config")
	viper.SetConfigType("yaml")
	viper.AddConfigPath(".")
	viper.ReadInConfig()
	var config Config
	viper.Unmarshal(&config)
	fmt.Println(config.MySQL.Username)
	fmt.Println(config.Redis.Host)
}
```

### 判断配置key是否存在
```go
if viper.IsSet("user"){
	fmt.Println("key user is not exists")
}
```

### 打印所有配置
```go
m := viper.AllSettings()
fmt.Println(m)
```

## 小结

到此这篇关于Golang配置解析神器goviper使用详解的文章就介绍到这了,更多相关goviper使用内容请搜索脚本之家以前的文章或继续浏览下面的相关文章希望大家以后多多支持脚本之家！


