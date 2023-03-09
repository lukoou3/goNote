
# go 操作mongoDB
http://events.jianshu.io/p/ffd75fb3b687


## 1、MongoDB的驱动包


```cpp
"go.mongodb.org/mongo-driver/bson"    //BOSN解析包
"go.mongodb.org/mongo-driver/mongo"    //MongoDB的Go驱动包
"go.mongodb.org/mongo-driver/mongo/options"

```


## 2、BSON简介


BSON是一种类json的一种二进制形式的存储格式，简称Binary JSON。MongoDB使用了BSON这种结构来存储数据和网络数据交换。


BSON对应Document这个概念，因为BSON是schema-free的，所以在MongoDB中所对应的Document也有这个特征，这里的一个Document也可以理解成关系数据库中的一条Record，只是Document的变化更丰富一些，Document可以嵌套。


MongoDB以BSON做为其存储结构的一个重要原因是它的可遍历性。


BSON编码扩展了JSON表示，使其包含额外的类型，如int、long、date、浮点数和decimal128。


## 3、BSON类型


BSON数据的主要类型有：`A`，`D`，`E`，`M`和`Raw`。其中，`A`是数组，`D`是切片，`M`是映射，`D`和`M`是Go原生类型。


`A`类型表示有序的BSON数组。


```bash
bson.A{"bar", "world", 3.14159, bson.D{{"qux", 12345}}}

```


`D`类型表示包含有序元素的BSON文档。这种类型应该在顺序重要的情况下使用。如果元素的顺序无关紧要，则应使用M代替。


```bash
bson.D{{"foo", "bar"}, {"hello", "world"}, {"pi", 3.14159}}

```


`M`类型表示无序的映射。


```bash
bson.M{"foo": "bar", "hello": "world", "pi": 3.14159}

```


`E`类型表示D里面的一个BSON元素。


`Raw`类型代表未处理的原始BSON文档和元素，Raw系列类型用于验证和检索字节切片中的元素。当要查找BSON字节而不将其解编为另一种类型时，此类型最有用。


```go
// MongoFilter go 操作mongo时常用的查询条件
func MongoFilter(uin int64, showID string) {
    filter := bson.M{"uin": uin}               // 指定filter为uin=uin
    filter = bson.M{"uin": bson.M{"$eq": uin}} // 和上面一样

    var arr = [2]string{"762513400", "762513401"}
    filter = bson.M{"uin": bson.M{"$in": arr}, "showID": "my_showID"} // uin在arr中存在，且showID同时等于my_showID

    filter = bson.M{"uin": uin, "room.roomID": 111} // 对象中嵌套子对象

    filter2 := bson.D{} // 查询条件是数组，需要同时满足数组内的全部条件
    if uin > 0 {
        filter2 = append(filter2, bson.E{Key: "uin", Value: uin})
    }
    if showID != "" {
        filter2 = append(filter2, bson.E{Key: "showID", Value: showID})
    }

    fmt.Println(filter)
}

```


## 4、常用的MongoDB客户端操作命令


```swift
选择切换数据库：use articledb
插入数据：db.comment.insert({bson数据})
查询所有数据：db.comment.find();
条件查询数据：db.comment.find({条件})
查询符合条件的第一条记录：db.comment.findOne({条件})
查询符合条件的前几条记录：db.comment.find({条件}).limit(条数)
查询符合条件的跳过的记录：db.comment.find({条件}).skip(条数)
修改数据：db.comment.update({条件},{修改后的数据}) 或db.comment.update({条件},{$set:{要修改部分的字段:数据})
修改数据并自增某字段值：db.comment.update({条件},{$inc:{自增的字段:步进值}})
删除数据：db.comment.remove({条件})
统计查询：db.comment.count({条件})
模糊查询：db.comment.find({字段名:/正则表达式/})
条件比较运算：db.comment.find({字段名:{$gt:值}})
包含查询：db.comment.find({字段名:{$in:[值1，值2]}})或db.comment.find({字段名:{$nin:[值1，值2]}})
条件连接查询：db.comment.find({$and:[{条件1},{条件2}]})或db.comment.find({$or:[{条件1},{条件2}]})

```


## 5、go操作mongoDB


util.go


```go
package util

import (
    "context"
    "fmt"

    "go.mongodb.org/mongo-driver/mongo"
    "go.mongodb.org/mongo-driver/mongo/options"
)

var mgoCli *mongo.Client
func initEngine() {
    var err error
    clientOptions:=options.Client().ApplyURI("mongodb://localhost:27017")

    // 连接到mongoDB
    mgoCli,err=mongo.Connect(context.TODO(),clientOptions)
    if err != nil {
        fmt.Println("连接失败")
    }
    // 检查连接
    err=mgoCli.Ping(context.TODO(),nil)
    if err != nil {
        fmt.Println("ping 失败")
    }

}

//
func GetMgoCli() *mongo.Client{
    if mgoCli==nil {
        initEngine()
    }
    return mgoCli
}


```


main.go


```go
package main

import (
    "context"
    "fmt"
    "time"

    "go.mongodb.org/mongo-driver/bson"
    "go.mongodb.org/mongo-driver/bson/primitive"
    "go.mongodb.org/mongo-driver/mongo"
    "go.mongodb.org/mongo-driver/mongo/options"

    "smallzhoutest/models"
    "smallzhoutest/mongoDBtest/util"
)

var (
    client     *mongo.Client
    db         *mongo.Database
    collection *mongo.Collection
)

func init() {
    // 创建连接
    client = util.GetMgoCli()

    //2.选择数据库 my_db
    db = client.Database("my_db")

    //3.选择表 my_collection
    collection = db.Collection("my_collection")

}

func main() {
    //findPerson()
    //findAll()
    //incr()
    //findGte()
    //updateSet()
    //gtegle()
    //findLimit()
    //aggregate()
    //findIn()
}

// findAll 根据条件查找全部符合的数据
func findAll() {
    filter := bson.M{"age": 18}
    res, err := collection.Find(context.TODO(), filter)
    if err != nil {
        panic(err)
    }
    var persons []models.Person
    err = res.All(context.TODO(), &persons)
    if err != nil {
        panic(err)
    }
    fmt.Println(persons)
}

// findPerson 普通条件查询一条符合的数据，返回所有符合条件的第一条
func findPerson() {
    filter := bson.M{"age": 18}
    var person models.Person
    err := collection.FindOne(context.Background(), filter).Decode(&person)
    if err != nil {
        panic(err)
    }
    fmt.Println("res==", person)

}

// findIn find操作的In操作，类似于MySQL的in操作
func findIn() {
    arr := []int{22, 30}
    filter := bson.D{
        {"age", bson.D{
            {"$in", arr},
        }},
    }
    res, err := collection.Find(context.TODO(), filter)

    if err != nil {
        panic(err)
    }
    var persons []models.Person
    err = res.All(context.TODO(), &persons)
    if err != nil {
        panic(err)
    }
    fmt.Println(persons)
}

// aggregate 查询操作的聚合操作，这里只是简单的分组然后统计整数 类似MySQL的group by
func aggregate() {
    aggregate := mongo.Pipeline{bson.D{
        {"$group", bson.D{
            {"_id", "$age"},
            {"count", bson.D{
                {"$sum", 1},
            }},
        }},
    }}
    res, err := collection.Aggregate(context.TODO(), aggregate)
    if err != nil {
        panic(err)
    }
    var results []bson.M
    err = res.All(context.TODO(), &results)
    if err != nil {
        panic(err)
    }
    fmt.Println(results)
}

// findLimit 查询操作的分页操作 批量查询全部符合条件的数据
func findLimit() {
    var result []models.Person
    var limit int64 = 1
    var index int64 = 0
    var findOptions = options.FindOptions{}
    filter := bson.M{"age": bson.M{"$gte": 18}}
    for {
        findOptions.SetLimit(limit)
        findOptions.SetSkip(index * limit)
        cur, err := collection.Find(context.TODO(), filter, &findOptions)
        if err != nil {
            panic(err)
        }
        per := make([]models.Person, 0)
        err = cur.All(context.TODO(), &per)
        if len(per) > 0 {
            result = append(result, per...)
        }
        if len(per) < int(limit) {
            fmt.Println("查询结束")
            break
        }
        index++
    }
    //
    fmt.Printf("len==%d  value==%+v\n", len(result), result)
}

// findAnd find查询的and查询，类似MySQL的and
func findAnd() {
    filter := bson.D{
        {"$and", bson.A{
            bson.M{"age": bson.M{"$gte": 22}},
            bson.M{"age": bson.M{"$lte": 30}},
        }},
    }
    res, err := collection.Find(context.TODO(), filter)

    if err != nil {
        panic(err)
    }
    var persons []models.Person
    err = res.All(context.TODO(), &persons)
    if err != nil {
        panic(err)
    }
    fmt.Println(persons)
}

// findGte gte大于  lte小于
func findGte() {
    //filter:=bson.M{"$gte":bson.M{"age":18}}
    filter := bson.D{
        {"age", bson.M{"$gte": 18}},
    }
    res, err := collection.Find(context.TODO(), filter)
    if err != nil {
        panic(err)
    }
    var persons []models.Person
    err = res.All(context.TODO(), &persons)
    if err != nil {
        panic(err)
    }
    fmt.Println(persons)
}

// updateSet 修改某个字段
func updateSet() {
    filter := bson.M{"age": 18}
    update := bson.M{"$set": bson.M{"age": 30}}
    _, err := collection.UpdateOne(context.TODO(), filter, update)
    if err != nil {
        panic(err)
    }
    findGte()
}

// incr 修改某个字段，操作是让某个字段增加count,类似redis的Incr
func inc(count int) {
    filter := bson.M{"age": 18}
    update := bson.M{"$inc": bson.M{"age": count}}
    _, err := collection.UpdateOne(context.TODO(), filter, update)
    if err != nil {
        panic(err)
    }
    findAll()
}

// InsertPersonData 插入一条person数据
func InsertPersonData() {
    person := models.Person{
        Card: models.Card{
            ID:   1,
            Time: time.Now().Format("2006-01-02 15:04:05"),
        },
        Age:   18,
        Hobby: []string{"basketball", "football"},
    }
    // 将结构体插入数据库中
    insertResult, err := collection.InsertOne(context.TODO(), person)
    if err != nil {
        fmt.Println("插入数据失败")
        return
    }
    // 查看插入数据后生成的默认id
    id := insertResult.InsertedID.(primitive.ObjectID)
    fmt.Println("自增ID：", id.Hex())
}

// findOneAndDelete 找到一条符合的数据并删除
func findOneAndDelete() {
    filter := bson.M{"key": "a_b"}
    doc := models.Person{}
    res := collection.FindOneAndDelete(context.TODO(), filter)
    if res.Err() != nil {
        panic(res.Err())
    }
    err := res.Decode(&doc)
    if err != nil {
        panic(err)
    }
    fmt.Println(doc)

}

// Remove 删除匹配的所有数据
func Remove() {
    filter := bson.M{"age": 18}
    res, err := collection.DeleteMany(context.TODO(), filter)
    if err != nil {
        panic(err)
    }
    fmt.Println("deleteCount==", res.DeletedCount)
}

// findRegex 通过正则表达式查找数据
func findRegex() {
    filter := bson.M{
        "name": bson.M{"$regex":"small1.*"},
    }
    res,err:=mongoCli.Find(context.TODO(),filter)
    if err != nil {
        panic(err)
    }
    var persons []model.Person
    err=res.All(context.TODO(),&persons)
    if err!=nil {
        panic(err)
    }
    fmt.Printf("res==%+v\n",persons)
}

// 对totalcardiacvalue进行从大到小的排序（1是从小到大的排序） 跳过10个，取10个值
func sort() {
    filter := bson.M{"uin": 1}
    var findOptions = options.FindOptions{}
    findOptions.SetSort(bson.M{"totalcardiacvalue": -1}).SetSkip(10).SetLimit(10)
    var fans []FansRank
    res, err := collection.Find(context.TODO(), filter, &findOptions)
    if err != nil {
        fmt.Println(err)
        return
    }
    err = res.All(context.TODO(), &fans)
    fmt.Println(fans)
}


```


