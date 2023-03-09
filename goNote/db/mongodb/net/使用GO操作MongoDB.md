
# 使用GO操作MongoDB
https://www.cnblogs.com/ithubb/p/16257722.html


## 安装[MongoDB](https://so.csdn.net/so/search?q=MongoDB&spm=1001.2101.3001.7020 "MongoDB")驱动程序


```go
mkdr mongodb 
cd mongodb 
go mod init  
go get go.mongodb.org/mongo-driver/mongo
```


## 连接MongoDB


创建一个main.go文件

将以下包导入main.go文件中





```go
package main

import (
   "context"
   "fmt"
   "log"
   "go.mongodb.org/mongo-driver/bson"
   "go.mongodb.org/mongo-driver/mongo"
   "go.mongodb.org/mongo-driver/mongo/options"
   "time"
)
```


连接MongoDB的URI格式为


```go
mongodb://[username:password@]host1[:port1][,...hostN[:portN]][/[defaultauthdb][?options]]
```


单机版


```go
mongodb://localhost:27017
```


副本集


```go
mongodb://mongodb0.example.com:27017,mongodb1.example.com：27017，mongodb2.example.com：27017 /?replicaSet = myRepl
```


分片集群


```go
mongodb://mongos0.example.com:27017,mongos1.example.com:27017,mongos2.example.com:27017
```


mongo.Connect()接受Context和options.ClientOptions对象，该对象用于设置连接字符串和其他驱动程序设置。

通过`context.TODO()`表示不确定现在使用哪种上下文，但是会在将来添加一个

使用Ping方法来检测是否已正常连接MongoDB





```go
func main() {
    clientOptions := options.Client().ApplyURI("mongodb://admin:password@localhost:27017")
    var ctx = context.TODO()
    // Connect to MongoDB
    client, err := mongo.Connect(ctx, clientOptions)
    if err != nil {
        log.Fatal(err)
    }
    // Check the connection
    err = client.Ping(ctx, nil)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println("Connected to MongoDB!")
    defer client.Disconnect(ctx)
```


### 列出所有数据库


```go
databases, err := client.ListDatabaseNames(ctx, bson.M{})
if err != nil {
   log.Fatal(err)
}
fmt.Println(databases)
```


在GO中使用BSON对象

MongoDB中的JSON文档以称为BSON（二进制编码的JSON）的二进制表示形式存储。与其他将JSON数据存储为简单字符串和数字的数据库不同，BSON编码扩展了JSON表示形式，例如int，long，date，float point和decimal128。这使应用程序更容易可靠地处理，排序和比较数据。Go Driver有两种系列用于表示BSON数据：D系列类型和Raw系列类型。

D系列包括四种类型：


D：BSON文档。此类型应用在顺序很重要的场景下，例如MongoDB命令。

M：无序map。除不保留顺序外，与D相同。

A：一个BSON数组。

E：D中的单个元素。

插入数据到MongoDB

插入单条文档





```go
//定义插入数据的结构体
type sunshareboy struct {
    Name string
    Age  int
    City string
}

//连接到test库的sunshare集合,集合不存在会自动创建
collection := client.Database("test").Collection("sunshare")
wanger:=sunshareboy{"wanger",24,"北京"}
insertOne,err :=collection.InsertOne(ctx,wanger)
if err != nil {
    log.Fatal(err)
}
fmt.Println("Inserted a Single Document: ", insertOne.InsertedID)
```


同时插入多条文档



```go
collection := client.Database("test").Collection("sunshare")
dongdong:=sunshareboy{"张冬冬",29,"成都"}
huazai:=sunshareboy{"华仔",28,"深圳"}
suxin:=sunshareboy{"素心",24,"甘肃"}
god:=sunshareboy{"刘大仙",24,"杭州"}
qiaoke:=sunshareboy{"乔克",29,"重庆"}
jiang:=sunshareboy{"姜总",24,"上海"}
//插入多条数据要用到切片
boys:=[]interface{}{dongdong,huazai,suxin,god,qiaoke,jiang}
insertMany,err:= collection.InsertMany(ctx,boys)
if err != nil {
    log.Fatal(err)
}
fmt.Println("Inserted multiple documents: ", insertMany.InsertedIDs)
```


## 从MongDB中查询数据


### 查询单个文档


查询单个文档使用collection.FindOne()函数，需要一个filter文档和一个可以将结果解码为其值的指针





```go
var result sunshareboy
filter := bson.D{{"name","wanger"}}
err = collection.FindOne(context.TODO(), filter).Decode(&result)
if err != nil {
    log.Fatal(err)
}
fmt.Printf("Found a single document: %+v\n", result)
```


返回结果如下


```go
Connected to MongoDB!
Found a single document: {Name:wanger Age:24 City:北京}
Connection to MongoDB closed.
```


查询多个文档

查询多个文档使用collection.Find()函数，这个函数会返回一个游标，可以通过他来迭代并解码文档，当迭代完成后，关闭游标


Find函数执行find命令并在集合中的匹配文档上返回Cursor。

filter参数必须是包含查询运算符的文档，并且可以用于选择结果中包括哪些文档。不能为零。空文档（例如bson.D {}）应用于包含所有文档。

opts参数可用于指定操作的选项，例如我们可以设置只返回五条文档的限制（https://godoc.org/go.mongodb.org/mongo-driver/mongo/options#Find）。





```go
    //定义返回文档数量
    findOptions := options.Find()
    findOptions.SetLimit(5)

    //定义一个切片存储结果
    var results []*sunshareboy

    //将bson.D{{}}作为一个filter来匹配所有文档
    cur, err := collection.Find(context.TODO(), bson.D{{}}, findOptions)
    if err != nil {
        log.Fatal(err)
    }
    //查找多个文档返回一个游标
    //遍历游标一次解码一个游标
    for cur.Next(context.TODO()) {
        //定义一个文档，将单个文档解码为result
        var result sunshareboy
        err := cur.Decode(&result)
        if err != nil {
            log.Fatal(err)
        }
        results = append(results, &result)
    }
    fmt.Println(result)
    if err := cur.Err(); err != nil {
        log.Fatal(err)
    }
    //遍历结束后关闭游标
    cur.Close(context.TODO())
    fmt.Printf("Found multiple documents (array of pointers): %+v\n", results)
```


返回结果如下





```go
Connected to MongoDB!
{wanger 24 北京}
{张冬冬 29 成都}
{华仔 28 深圳}
{素心 24 甘肃}
{刘大仙 24 杭州}
Found multiple documents (array of pointers): &[0xc000266450 0xc000266510 0xc000266570 0xc0002665d0 0xc000266630]
Connection to MongoDB closed.
```


更新MongoDB文档

更新单个文档

更新单个文档使用collection.UpdateOne()函数，需要一个filter来匹配数据库中的文档，还需要使用一个update文档来更新操作


filter参数必须是包含查询运算符的文档，并且可以用于选择要更新的文档。不能为零。如果过滤器不匹配任何文档，则操作将成功，并且将返回MatchCount为0的UpdateResult。如果过滤器匹配多个文档，将从匹配的集合中选择一个，并且MatchedCount等于1。

update参数必须是包含更新运算符的文档（https://docs.mongodb.com/manual/reference/operator/update/），并且可以用于指定要对所选文档进行的修改。它不能为nil或为空。

opts参数可用于指定操作的选项。





```go
filter := bson.D{{"name","张冬冬"}}
//如果过滤的文档不存在，则插入新的文档
opts := options.Update().SetUpsert(true)
update := bson.D{
    {"$set", bson.D{
        {"city", "北京"}},
    }}
result, err := collection.UpdateOne(context.TODO(), filter, update,opts)
if err != nil {
    log.Fatal(err)
}
if result.MatchedCount != 0 {
    fmt.Printf("Matched %v documents and updated %v documents.\n", result.MatchedCount, result.ModifiedCount)
}
if result.UpsertedCount != 0 {
    fmt.Printf("inserted a new document with ID %v\n", result.UpsertedID)
}
```


返回结果如下


```go
Connected to MongoDB!
Matched 1 documents and updated 1 documents.
Connection to MongoDB closed.
```


### 更新多个文档


更新多个文档使用collection.UpdateOne()函数，参数与collection.UpdateOne()函数相同





```go
filter := bson.D{{"city","北京"}}
//如果过滤的文档不存在，则插入新的文档
opts := options.Update().SetUpsert(true)
update := bson.D{
    {"$set", bson.D{
        {"city", "铁岭"}},
    }}
result, err := collection.UpdateMany(context.TODO(), filter, update,opts)
if err != nil {
    log.Fatal(err)
}
if result.MatchedCount != 0 {
    fmt.Printf("Matched %v documents and updated %v documents.\n", result.MatchedCount, result.ModifiedCount)
}
if result.UpsertedCount != 0 {
    fmt.Printf("inserted a new document with ID %v\n", result.UpsertedID)
}
```


返回结果如下


```go
Connected to MongoDB!
Matched 2 documents and updated 2 documents.
Connection to MongoDB closed.
```


## 删除MongoDB文档


可以使用`collection.DeleteOne()`或`collection.DeleteMany()`删除文档。如果你传递`bson.D{{}}`作为过滤器参数，它将匹配数据集中的所有文档。还可以使用`collection. drop()`删除整个数据集。





```go
filter := bson.D{{"city","铁岭"}}
deleteResult, err := collection.DeleteMany(context.TODO(), filter)
if err != nil {
    log.Fatal(err)
}
fmt.Printf("Deleted %v documents in the trainers collection\n", deleteResult.DeletedCount)
```


返回结果如下


```go
Connected to MongoDB!
Deleted 2 documents in the trainers collection
Connection to MongoDB closed.
```


## 获取MongoDB服务状态


上面我们介绍了对MongoDB的CRUD，其实还支持很多对mongoDB的操作，例如[聚合](https://godoc.org/go.mongodb.org/mongo-driver/mongo#Collection.Aggregate "聚合")、[事物](https://godoc.org/go.mongodb.org/mongo-driver/mongo#Session "事物")等，接下来介绍一下使用golang获取MongoDB服务状态，执行后会返回一个bson.Raw类型的数据





```go
ctx, _ = context.WithTimeout(context.Background(), 30*time.Second)
serverStatus, err := client.Database("admin").RunCommand(
    ctx,
    bsonx.Doc{{"serverStatus", bsonx.Int32(1)}},
).DecodeBytes()
if err != nil {
    fmt.Println(err)
}
fmt.Println(serverStatus)
fmt.Println(reflect.TypeOf(serverStatus))
version, err := serverStatus.LookupErr("version")
fmt.Println(version.StringValue())
if err != nil {
    fmt.Println(err)
}
```


参考链接

https://www.mongodb.com/blog/post/mongodb-go-driver-tutorial

https://godoc.org/go.mongodb.org/mongo-driver/mongo

从MongoDB中查询数据从MongoDB中查询数据从MongoDB中查询数据








