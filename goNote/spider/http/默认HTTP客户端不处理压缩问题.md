
## 背景
我自己在用go实现之前Python实现的爬虫时，同一个url的get请求发现go返回的数据是源码，在python和浏览器中都没有问题，应该不是服务端返回数据的问题，之后发现headers设置了Accept-Encoding为gzip，去掉这个设置后返回的数据就正常了。

http竟然不自动处理压缩！

百度了一下，发现别人也有这个问题，把网上搜索到的结果记录下。

## Golang：默认HTTP客户端不处理压缩
https://www.5axxw.com/questions/content/ftofgy

我正在尝试做一个HTTP请求，让golang添加accept-encoding头，并在响应被压缩时自动解压缩。我的印象是默认的HTTP客户端应该透明地处理它？然而，它似乎没有：
```go
    req, _ := http.NewRequest("Get", "https://stackoverflow.com", nil)
    // req.Header.Add("Accept-Encoding", "gzip")
    client := &http.Client{}
    resp, _ := client.Do(req)
    println(resp.Header.Get("Content-Encoding"))
```
如果我手动添加Accept-Encoding，它会发送头，但我必须手动解压缩响应。

最佳回答：

如果传输本身请求gzip并获得gzipped响应，则该响应将被透明地解压缩。在本例中，传输从响应中删除Content-Encoding头。检查回复。Uncompressed字段，以确定响应是否未压缩。
```go
req, _ := http.NewRequest("Get", "https://stackoverflow.com", nil)
resp, _ := http.DefaultClient.Do(req)
fmt.Println(resp.Uncompressed) // prints true
fmt.Println(resp.Header.Get("Content-Encoding"))  // prints blank line
```

如果应用程序显式请求gzip，那么响应将按原样返回。
```go
req, _ := http.NewRequest("Get", "https://stackoverflow.com", nil)
req.Header.Add("Accept-Encoding", "gzip")  
client := &http.Client{}
resp, _ := client.Do(req)
fmt.Println(resp.Uncompressed)  // prints false
fmt.Println(resp.Header.Get("Content-Encoding")) prints gzip
```

## golang http请求Accept-Encoding:gzip未解压问题
https://blog.csdn.net/shumoyin/article/details/120989832

问题描述:
golang在使用net/http发送http请求时，如果请求头中包含Accept-Encoding: gzip，且请求服务支持gzip压缩，那么在解析请求返回值时，需要手动解压

代码示例:
```go
package test

import (
    "bytes"
    "compress/gzip"
    "fmt"
    "github.com/pkg/errors"
    "io"
    "io/ioutil"
    "net/http"
    "testing"
    "time"
)

func TestPost(t *testing.T) {
    url := "http://127.0.0.1/test/gzip"
    param := "{\"gzip\": true}"

    headers := http.Header{}
    headers.Add("content-type", "application/json")
    headers.Add("Accept-Encoding", "gzip")
    entity, err := execute("POST", url, bytes.NewBufferString(param), headers)
    if err != nil {
        panic(err)
    }
    var body []byte
    if !entity.Uncompressed {
        // 如果返回有GZIP压缩，则需要解压
        defer entity.Body.Close()
        reader, err := gzip.NewReader(entity.Body)
        if err != nil {
            panic(err)
        }
        defer reader.Close()
        body, err = ioutil.ReadAll(reader)
        if err != nil {
            panic(err)
        }
    } else {
        defer entity.Body.Close()
        body, err = ioutil.ReadAll(entity.Body)
        if err != nil {
            panic(err)
        }
    }

    t.Logf("response is %s", string(body))
}

func execute(method string, url string, body io.Reader, header http.Header) (*http.Response, error) {
    // 构建请求参数
    request, err := http.NewRequest(method, url, body)
    if err != nil {
        return nil, errors.Wrap(err, fmt.Sprintf("New http request error: [%s] %s", method, url))
    }
    request.Header = header

    // 发送请求
    client := http.Client{Timeout: 60 * time.Second}
    resp, err := client.Do(request)
    if err != nil {
        return nil, errors.Wrap(err, fmt.Sprintf("Send http request error"))
    }
    return resp, nil
}
```
