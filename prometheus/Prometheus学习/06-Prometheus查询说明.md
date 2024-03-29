## Prometheus学习（六）之Prometheus查询说明
https://www.cnblogs.com/even160941/p/15109453.html


### 0.前言


本文来自[Prometheus官网手册](https://prometheus.io/docs/introduction/first_steps/ "Prometheus官网手册")和[Prometheus简介](https://prometheus.fuckcloudnative.io/ "Prometheus简介")


### 1.Prothetheus查询


Prometheus提供一个函数式的表达式语言PromQL (Prometheus Query Language)，可以使用户实时地查找和聚合时间序列数据。表达式计算结果可以在图表中展示，也可以在表达式浏览器中以表格形式展示，或者作为数据源, 以[HTTP API](https://prometheus.io/docs/prometheus/latest/querying/api/ "HTTP API")的方式提供给外部系统使用。


#### 1.1例子


本文档仅供参考， 对于学习，从几个[例子](https://prometheus.io/docs/prometheus/latest/querying/examples/ "例子")开始可能更容易。


#### 1.2表达式语言数据类型


在Prometheus的表达式语言中，任何表达式或者子表达式都可以归为四种类型：


* instant vector瞬时向量：它是指在同一时刻，抓取的所有度量指标数据。这些度量指标数据的key都是相同的，也即相同的时间戳    
* range vector范围向量：它是指在任何一个时间范围内，抓取的所有度量指标数据    
* scalar 标量：一个简单的浮点值    
* string 字符串：一个当前没有被使用的简单字符串

根据用例（例如在绘制图形或显示表达式的输出时），由于用户指定的表达式的结果，其中只有某些类型是合法的。 例如，返回瞬时向量的表达式是唯一可以直接绘制图形的类型。


#### 1.3字面量


##### 1.3.1字符串字面量


字符串可以用单引号，双引号或反引号指定为文字。PromQL遵循与Go相同的[转义规则](https://golang.org/ref/spec#String_literals "转义规则")。在单引号，双引号中，反斜杠成为了转义字符，后面可以跟着`a,b, f, n, r, t, v或者\`。 可以使用八进制`(\nnn)`或者十六进制`(\xnn, \unnnn和\Unnnnnnnn)`提供特定字符。在反引号内不处理转义字符。与Go不同，Prometheus不会丢弃反引号中的换行符。例如：


```sh
"this is a string"
'these are unescaped: \n \\ \t'
`these are not unescaped: \n ' " \t"'`

```


##### 1.3.2浮点数字面量


标量浮点值可以直接写成形式[-](digits "-")[.(digits)]。


```sh
-2.43

```


#### 1.4时间序列选择器


##### 1.4.1瞬时向量选择器


瞬时向量选择器允许在给定时间戳（即时）为每个选择一组时间序列和单个样本值：在最简单的形式中，仅指定度量名称。 这会生成包含具有此度量标准名称的所有时间序列的元素的即时向量。


下面这个例子选择所有时间序列度量名称为http_requests_total的样本数据：


```sh
http_requests_total

```


通过在度量指标后面增加{}一组标签可以进一步地过滤这些时间序列数据。


此示例仅选择具有`http_requests_total`度量标准名称的时间系列，该名称也将job标签设置为`prometheus`，并将其`group`标签设置为`canary`：


```sh
http_requests_total{job="prometheus",group="canary"}

```


可以采用不匹配的标签值也是可以的，或者用正则表达式不匹配标签。标签匹配操作如下所示：


```sh
=  : 精确地匹配标签给定的值
!= : 不等于给定的标签值
=~ : 正则表达匹配给定的标签值
!~ : 给定的标签值不符合正则表达式

```


例如：度量指标名称为`http_requests_total`，正则表达式匹配标签`environment`为`staging`,`testing`,`development`的值，且http请求方法不等于`GET`。


```sh
http_requests_total{environment=~"staging|testing|development",method!="GET"}

```


匹配空标签值的标签匹配器也可以选择没有设置任何标签的所有时间序列数据。正则表达式完全匹配。 可以为同一标签名称提供多个匹配器。


向量选择器必须指定一个名称或至少一个与空字符串不匹配的标签匹配器。 以下表达式是非法的：


```sh
{job=~".*"} # Bad!

```


相反，这些表达式是有效的，因为它们都有一个与空标签值不匹配的选择器。


```sh
{job=~".+"}              # Good!
{job=~".*",method="get"} # Good!

```


标签匹配器能够被应用到度量指标名称，使用`__name__`标签筛选度量指标名称。例如：表达式`http_requests_total`等价于`{__name__="http_requests_total"}`。 其他的匹配器，如：`= ( !=, =~, !~)`都可以使用。下面的表达式选择了度量指标名称以job:开头的时间序列数据：


```sh
{__name__=~"job:.*"}

```


Prometheus中的所有正则表达式都使用[RE2语法](https://github.com/google/re2/wiki/Syntax "RE2语法")。


##### 1.4.2范围向量选择器


范围向量的工作方式与即时向量相同，不同之处在于它们从当前即时选择回采样范围。 在语法上，范围持续时间附加在向量选择器末尾的方括号（`[]`）中，指定为每个结果范围向量元素提取多长时间值。持续时间指定为数字，单位为：


* s - seconds    
* m - minutes    
* h - hours    
* d - days    
* w - weeks    
* y - years


在此示例中，我们选择在过去5分钟内为度量标准名称为`http_requests_total`且`job`标签设置为`prometheus`的所有时间序列记录的所有值：


```sh
http_requests_total{job="prometheus"}[5m]

```


##### 1.4.3偏移修饰符


这个`offset`偏移修饰符允许在查询中改变单个瞬时向量和范围向量中的时间偏移。例如，以下表达式返回过去相对于当前查询评估时间5分钟的`http_requests_total`值：


```sh
http_requests_total offset 5m

```


注意：`offset`偏移修饰符必须直接跟在选择器后面，例如：以下是正确的：


```sh
sum(http_requests_total{method="GET"} offset 5m) // GOOD.

```


然而，下面这种情况是不正确的:


```sh
sum(http_requests_total{method="GET"}) offset 5m // INVALID.

```


同样适用于范围向量。 这将返回`http_requests_total`一周前的5分钟速率：


```sh
rate(http_requests_total[5m] offset 1w)

```


#### 1.5子查询


子查询允许针对给定范围和分辨率运行即时查询。 子查询的结果是范围向量。


语法：`<instant_query>'['<range>'：'[<resolution>]']'[offset <duration>]`


`<resolution>`是可选的。 默认值是全局评估间隔。


#### 1.6操作符


Prometheus支持二元和聚合操作符。详见表达式语言操作符


#### 1.7函数


Prometheus提供了一些函数列表操作时间序列数据。详见[表达式语言函数](https://prometheus.io/docs/prometheus/latest/querying/operators/ "表达式语言函数")


#### 1.8注释


PromQL支持以＃开头的行注释。 如：


```sh
＃这是一条评论

```


#### 1.9陷阱


##### 1.9.1旧数据


运行查询时，独立于当前时间序列的数据选择采样数据的时间戳。这主要是为了支持聚合（总和，平均等）这样的情况，其中多个聚合时间序列在时间上不完全对齐。由于它们的独立性，Prometheus需要在每个相关时间序列的时间戳上分配值。它只需在此时间戳之前采用最新的样本即可。


如果目标抓取或规则评估不再返回先前存在的时间序列的样本，则该时间序列将被标记为旧数据。如果目标被移除，之前很快就会将其先前返回的时间序列标记为旧数据。


如果在时间序列标记为过时后，在采样时间戳处评估查询，则不会为该时间系列返回任何值。如果随后在该时间序列中摄取新样本，它们将照常返回。


如果在采样时间戳前5分钟未找到任何样本（默认情况下），则此时间点不返回该时间序列的值。这实际上意味着时间序列在其最新收集的样本超过5分钟或标记为旧数据之后从图表“消失”。对于在其抓取中包含时间戳的时间序列，不会标记旧数据。在这种情况下，仅应用5分钟的阈值。


##### 1.9.2避免慢查询和过载


如果查询需要对大量数据进行操作，则绘制图表可能会超时或使服务器或浏览器过载。因此，在构建对未知数据的查询时，始终在Prometheus表达式浏览器的表格视图中开始构建查询，直到结果集看起来合理（最多数百个，而不是数千个时间序列）。只有在充分过滤或汇总数据后，才能切换到图表模式。如果表达式仍然需要很长时间来绘制ad-hoc图形，请通过录制规则预先录制它。


这与Prometheus的查询语言尤其相关，其中像`api_http_requests_total`这样的简单度量标准名称选择器可以扩展到具有不同标签的数千个时间序列。还要记住，即使输出只是少量的时间序列，聚合在许多时间序列上的表达式也会在服务器上产生负载。这类似于在关系数据库中对列的所有值求和的速度很慢，即使输出值只是一个数字。

