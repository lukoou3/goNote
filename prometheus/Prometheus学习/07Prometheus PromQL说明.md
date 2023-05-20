## Prometheus学习（七）之Prometheus PromQL说明
https://www.cnblogs.com/even160941/p/15117587.html






### 0.前言


本文来自[Prometheus官网手册](https://prometheus.io/docs/introduction/first_steps/ "Prometheus官网手册")和[Prometheus简介](https://prometheus.fuckcloudnative.io/ "Prometheus简介")


### 1.PromQL操作符


#### 1.1二元操作符


Prometheus的查询语言支持基本的逻辑运算和算术运算。对于两个瞬时向量, 匹配行为可以被改变。


##### 1.1.1算术二元运算符


在Prometheus支持下面的二元算术操作符：


```sh
+ 加法
- 减法
* 乘法
/ 除法
% 模
^ 幂等

```


二元运算操作符定义在`scalar/scalar(标量/标量)`、`vector/scalar(向量/标量)`、和`vector/vector(向量/向量)`之间。


* 在两个标量之间：评估另一个标量，这是运算符应用于两个标量操作数的结果。    
* 在瞬时向量和标量之间：将运算符应用于向量中的每个数据样本的值。 如果时间序列即时向量乘以2，则结果是另一个向量，其中原始向量的每个样本值乘以2。    
* 在两个瞬时向量之间：应用于左侧向量中的每个条目及其右侧向量中的匹配元素。 结果将传播到结果向量中。 右侧向量中（没有匹配条目）不是结果的一部分。


##### 1.1.2比较二元操作符


在Prometheus系统中，比较二元操作符有：


```sh
== 等于
!= 不等于
> 大于
< 小于
>= 大于等于
<= 小于等于

```


比较二元操作符定义在`scalar/scalar（标量/标量）`、`vector/scalar(向量/标量)`，和`vector/vector（向量/向量）`。默认情况下过滤。 可以通过在运算符之后提供bool来修改它们的行为，这将为值返回`0`或`1`而不是过滤。


* 在两个标量之间：必须提供`bool`修饰符，并且这些运算符会产生另一个标量，即`0`（假）或`1`（真），具体取决于比较结果。    
* 在瞬时向量和标量之间：将这些运算符应用于向量中的每个数据样本的值，并且从结果向量中删除比较结果为假的向量元素。 如果提供了`bool`修饰符，则将被删除的向量元素的值为`0`，而将保留的向量元素的值为`1`。    
* 在两个瞬时向量之间：这些运算符默认表现为过滤器，应用于匹配条目。 表达式不正确或在表达式的另一侧找不到匹配项的向量元素将从结果中删除，而其他元素将传播到具有其原始（左侧）度量标准名称的结果向量中 标签值。 如果提供了`bool`修饰符，则已经删除的向量元素的值为`0`，而保留的向量元素的值为`1`，左侧标签值为`1`。


如：


```sh
3 > 2
# 报错 "comparisons between scalars must use BOOL modifier"
 
3 > bool 2 
# 返回 scalar 1
 
1 > bool 2 

```


##### 1.1.3逻辑/集合二元操作符


逻辑/集合二元操作符只能作用在即时向量，包括：


```sh
and 交集
or 并集
unless 补集

```


* vector1 and vector2:得到一个由`vector1`元素组成的向量，其中`vector2`中的元素具有完全匹配的标签集，其他元素被删除。    
* vector1 or vector2:得到包含`vector1`的所有原始元素（标签集+值）的向量以及`vector2`中`vector1`中没有匹配标签集的所有元素。    
* vector1 unless vector2:得到一个由`vector1`元素组成的向量，其中`vector2`中没有元素，具有完全匹配的标签集。 两个向量中的所有匹配元素都被删除。


#### 1.2向量匹配


向量之间的操作尝试在左侧的每个条目的右侧向量中找到匹配元素。 匹配行为有两种基本类型：一对一和多对一/一对多。


##### 1.2.1 一对一向量匹配


一对一从操作的每一侧找到一对唯一条目。 在默认情况下，这是格式为`vector1<operator>vector2`之后的操作。 如果两个条目具有完全相同的标签集和相应的值，则它们匹配。 忽略关键字允许在匹配时忽略某些标签，而`on`关键字允许将所考虑的标签集减少到提供的列表：


```sh
[vector expr] [bin-op] ignoring([label list]) [vector expr]
[vector expr] [bin-op] on([lable list]) [vector expr]

```


例如样本数据：


```sh
 method_code:http_errors:rate5m{method="get", code="500"} 24
 method_code:http_errors:rate5m{method="get", code="404"} 30
 method_code:http_errors:rate5m{method="put", code="501"} 3
 method_code:http_errors:rate5m{method="post", code="404"} 21
 method:http_requests:rate5m{method="get"} 600
 method:http_requests:rate5m{method="delete"} 34
 method:http_requests:rate5m{method="post"} 120

```


查询例子：


```sh
method_code:http_errors:rate5m{code="500"} / ignoring(code) method:http_requests:rate5m

```


这将返回一个结果向量，其中包含每个方法的状态代码为500的HTTP请求部分，在过去的5分钟内进行测量。 没有ignoring(code)就没有匹配，因为度量标准不共享同一组标签。 方法put和del的条目没有匹配，并且不会显示在结果中：


```sh
{method="get"} 0.04 // 24 / 600
{method="post"} 0.05 // 6 / 120

```


##### 1.2.2 多对一和一对多向量匹配


多对一和一对多匹配指的是“一”侧的每个向量元素可以与“多”侧的多个元素匹配的情况。 必须使用group_left或group_right修饰符明确请求，其中left/right确定哪个向量具有更高的基数。


```sh
<vector expr> <bin-op> ignoring(<label list>) group_left(<label list>) <vector expr>
<vector expr> <bin-op> ignoring(<label list>) group_right(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) group_left(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) group_right(<label list>) <vector expr>

```


分组修饰符提供的标签列表包含来自“一”侧的其他标签，以包含在结果度量标准中。 对于标签，只能出现在其中一个列表中。 每次结果向量的序列必须是唯一可识别的。分组修饰符只能用于比较和算术。 默认情况下，操作as和除非和或操作与右向量中的所有可能条目匹配。示例查询：


```sh
method_code:http_errors:rate5m / ignoring(code) group_left method:http_requests:rate5m

```


在这种情况下，左向量每个method标签值包含多个条目。 因此，我们使用group_left表明这一点。 右侧的元素现在与多个元素匹配，左侧具有相同的method标签：


```sh
{method="get", code="500"} 0.04 // 24 /600
{method="get", code="404"} 0.05 // 30 /600
{method="post", code="500"} 0.05 // 6 /600
{method="post", code="404"} 0.175 // 21 /600

```


多对一和一对多匹配是高级用例，应该仔细考虑。 通常正确使用忽略`ignoring(<labels>)`可提供所需的结果。


#### 1.3聚合操作符


Prometheus支持以下内置聚合运算符，这些运算符可用于聚合单个即时向量的元素，从而生成具有聚合值的较少元素的新向量：


```sh
sum (在维度上求和)
max (在维度上求最大值)
min (在维度上求最小值)
avg (在维度上求平均值)
stddev (求标准差)
stdvar (求方差)
count (统计向量元素的个数)
count_values (统计相同数据值的元素数量)
bottomk (样本值第k个最小值)
topk (样本值第k个最大值)
quantile (统计分位数)

```


这些运算符可以用于聚合所有标签维度，也可以通过包含without或by子句来保留不同的维度。


```sh
<aggr-op>([parameter,] <vector expr>) [without | by (<label list>)] [keep_common]

```


`parameter`仅用于`count_values`，`quantile`，`topk`和`bottomk`。不从结果向量中删除列出的标签，而所有其他标签都保留输出。`by`相反并删除未在`by`子句中列出的标签，即使它们的标签值在向量的所有元素之间是相同的。


`count_values`输出每个唯一样本值的一个时间序列。每个序列都有一个额外的标签。该标签的名称由聚合参数给出，标签值是唯一的样本值。每个时间序列的值是样本值存在的次数。


`topk`和`bottomk`与其他聚合器的不同之处在于，输入样本的子集（包括原始标签）在结果向量中返回。`by`和`without`仅用于存储输入向量。


例：如果度量标准`http_requests_total`具有按应用程序，实例和组标签扇出的时间序列，我们可以通过以下方式计算每个应用程序和组在所有实例上看到的HTTP请求总数：


```sh
sum(http_requests_total) without (instance)

```


等价于：


```sh
sum(http_requests_total)

```


要计算运行每个构建版本的二进制文件的数量，我们可以编写：


```sh
count_values("version", build_version)

```


要在所有实例中获取5个最大的HTTP请求计数，我们可以编写：


```sh
topk(5, http_requests_total)

```


#### 1.4二元运算符优先级


以下列表显示了Prometheus中二进制运算符的优先级，从最高到最低。


```sh
^
*, /, %
+, -
==, !=, <=, <, >=, >
and, unless
or

```


具有相同优先级的运算符是左关联的。 例如，`2 * 3％2`相当于`（2 * 3）％2`。但是^是右关联的，因此`2 ^ 3 ^ 2`相当于`2 ^（3 ^ 2）`。


### 2.PromQL函数


一些函数有默认的参数，例如：`year(v=vector(time()) instant-vector)`。意思是有一个参数v是一个瞬时向量，如果没有提供，它将默认为表达式vector(time())的值。可参考：[Prometheus监控学习笔记之PromQL内置函数](https://www.cnblogs.com/JetpropelledSnake/p/10446878.html "Prometheus监控学习笔记之PromQL内置函数")


#### 2.1abs()


```sh
abs(v instant-vector)返回输入向量，所有样本值都转换为其绝对值。

```


#### 2.2absent()


`absent(v instant-vector)`如果传递给它的向量具有任何元素，则返回空向量；如果传递给它的向量没有元素，则返回为1的值。这对于在给定度量标准名称和标签组合不存在时间序列时发出警报非常有用。


```sh
absent(nonexistent{job="myjob"})  
# => {job="myjob"}
absent(nonexistent{job="myjob", instance=~".*"}) 
# => {job="myjob"}
absent(sum(nonexistent{job="myjob"})) 
# => {}

```


在第二个例子中，`absent()`试图从输入向量中导出1元素输出向量的标签。


#### 2.3ceil()


`ceil(v instant-vector)`将v中所有元素的样本值四舍五入到最接近的整数。如：


```sh
node_load5{instance="192.168.1.75:9100"} # 结果为 2.79
ceil(node_load5{instance="192.168.1.75:9100"}) # 结果为 3

```


#### 2.4changes()


输入一个区间向量， 返回这个区间向量内每个样本数据值变化的次数（瞬时向量）。如：


```sh
如果样本数据值没有发生变化，则返回结果为 1
changes(node_load5{instance="192.168.1.75:9100"}[1m]) # 结果为 1 

```


#### 2.5clamp_max()


`clamp_max(v instant-vector, max scalar)`函数，输入一个瞬时向量和最大值，样本数据值若大于 max，则改为 max，否则不变。如：


```sh
node_load5{instance="192.168.1.75:9100"} # 结果为 2.79
clamp_max(node_load5{instance="192.168.1.75:9100"}, 2) # 结果为 2

```


#### 2.6clamp_min()


`clamp_min(v instant-vector, min scalar)`函数，输入一个瞬时向量和最小值，样本数据值若小于 min，则改为 min，否则不变。如：


```sh
node_load5{instance="192.168.1.75:9100"} # 结果为 2.79
clamp_min(node_load5{instance="192.168.1.75:9100"}, 3) # 结果为 3

```


#### 2.7day_of_month()


`day_of_month(v=vector(time()) instant-vector)`返回UTC中每个给定时间的月中的某天。 返回值为1到31。


#### 2.8day_of_week()


`day_of_week(v=vector(time()) instant-vector)`返回UTC中每个给定时间的星期几。 返回值为0到6，其中0表示星期日等。


#### 2.9days_in_month()


`days_in_month(v=vector(time()) instant-vector)`返回UTC中每个给定时间的月中天数。 返回值为28到31。


#### 2.10delta()


`delta(v range-vector)`的参数是一个区间向量，返回一个瞬时向量。它计算一个区间向量 v 的第一个元素和最后一个元素之间的差值。由于这个值被外推到指定的整个时间范围，所以即使样本值都是整数，你仍然可能会得到一个非整数值。 如以下示例表达式返回现在和2小时之前CPU温度的差异：


```sh
delta(cpu_temp_celsius{host="zeus"}[2h])

```


这个函数一般只用在 Gauge 类型的时间序列上。


#### 2.11deriv()


`deriv(v range-vector)`的参数是一个区间向量,返回一个瞬时向量。它使用简单的线性回归计算区间向量 v 中各个时间序列的导数。这个函数一般只用在 Gauge 类型的时间序列上。


#### 2.12exp()


`exp(v instant-vector)`函数，输入一个瞬时向量，返回各个样本值的 e 的指数值，即 e 的 N 次方。当 N 的值足够大时会返回 +Inf。特殊情况为：


```sh
Exp(+inf) = +Inf
Exp(NaN) = NaN

```


#### 2.13floor()


`floor(v instant-vector)`函数与`ceil()`函数相反，将 v 中所有元素的样本值向下四舍五入到最接近的整数。


#### 2.14histogram_quantile()


`histogram_quatile(φ float, b instant-vector)`计算b向量的φ-直方图 (0 ≤ φ ≤ 1) 。（有关φ-分位数的详细解释和直方图度量类型的使用，请参见直方图和摘要。）b中的样本是每个桶中的观察计数。 每个样本必须具有标签le，其中标签值表示桶的包含上限。 （没有这种标签的样本会被忽略。）直方图度量标准类型自动提供带有`_bucket`后缀和相应标签的时间序列。使用`rate()`函数指定分位数计算的时间窗口。


示例：直方图度量标准称为`http_request_duration_seconds`。 要计算过去10m内请求持续时间的第90个百分位数，请使用以下表达式：


```sh
histogram_quantile(0.9, rate(http_request_duration_seconds_bucket[10m]))

```


在`http_request_duration_seconds`中为每个标签组合计算分位数。 要聚合，请在`rate()`函数周围使用`sum()`聚合器。 由于`histogram_quantile()`需要le标签，因此必须将其包含在by子句中。 以下表达式按作业聚合第90个百分点：


```sh
histogram_quantile(0.9, sum(rate(http_request_duration_seconds_bucket[10m])) by (job, le))

```


要聚合所有内容，请仅指定le标签：


```sh
histogram_quantile(0.9, sum(rate(http_request_duration_seconds_bucket[10m])) by (le))

```


`histogram_quantile()`函数通过假设桶内的线性分布来插值分位数值。 最高桶必须具有+Inf的上限。 （否则，返回NaN。）如果分位数位于最高桶中，则返回第二个最高桶的上限。 如果该桶的上限大于0，则假设最低桶的下限为0.在这种情况下，在该桶内应用通常的线性插值。 否则，对于位于最低桶中的分位数，返回最低桶的上限。


如果b包含少于两个桶，则返回NaN。 对于φ<0，返回-Inf。 对于φ> 1，返回+Inf。


#### 2.15holt_winters()


`holt_winters(v range-vector, sf scalar, tf scalar)`函数基于区间向量 v，生成时间序列数据平滑值。平滑因子 sf 越低, 对旧数据的重视程度越高。趋势因子 tf 越高，对数据的趋势的考虑就越多。其中，`0< sf, tf <=1`。仅适用于 Gauge 类型的时间序列。


#### 2.16hour()


`hour(v=vector(time()) instant-vector)`返回UTC中每个给定时间的一天中的小时。 返回值为0到23。


#### 2.17idelta()


`idelta(v range-vector)`的参数是一个区间向量, 返回一个瞬时向量。它计算最新的 2 个样本值之间的差值。这个函数一般只用在 Gauge 类型的时间序列上。


#### 2.18increase()


`increase(v range-vector)`函数获取区间向量中的第一个和最后一个样本并返回其增长量, 它会在单调性发生变化时(如由于采样目标重启引起的计数器复位)自动中断。由于这个值被外推到指定的整个时间范围，所以即使样本值都是整数，你仍然可能会得到一个非整数值。如以下表达式返回区间向量中每个时间序列过去 5 分钟内 HTTP 请求数的增长数：


```sh
increase(http_requests_total{job="api-server"}[5m])

```


increase 的返回值类型只能是计数器类型，主要作用是增加图表和数据的可读性。使用`rate`函数记录规则的使用率，以便持续跟踪数据样本值的变化。


#### 2.19irate


`irate(v range-vector)`函数用于计算区间向量的增长率，但是其反应出的是瞬时增长率。`irate`函数是通过区间向量中最后两个样本数据来计算区间向量的增长速率，它会在单调性发生变化时(如由于采样目标重启引起的计数器复位)自动中断。这种方式可以避免在时间窗口范围内的“长尾问题”，并且体现出更好的灵敏度，通过irate函数绘制的图标能够更好的反应样本数据的瞬时变化状态。如，以下表达式返回区间向量中每个时间序列过去 5 分钟内最后两个样本数据的 HTTP 请求数的增长率：


```sh
irate(http_requests_total{job="api-server"}[5m])

```


`irate`只能用于绘制快速变化的计数器，在长期趋势分析或者告警中更推荐使用`rate`函数。因为使用`irate`函数时，速率的简短变化会重置 FOR 语句，形成的图形有很多波峰，难以阅读。


注意，当将`irate()`与聚合运算符（例如`sum()`）或随时间聚合的函数（任何以`_over_time`结尾的函数）组合时，请始终首先采用`irate()`，然后进行聚合。 否则，当目标重新启动时，`irate()`无法检测计数器重置。


#### 2.20label_join()


函数可以将时间序列 v 中多个标签`src_label`的值，通过 separator 作为连接符写入到一个新的标签`dst_label`中。可以有多个`src_label`标签。如，以下表达式返回的时间序列多了一个 foo 标签，标签值为 etcd,etcd-k8s：


```sh
up{endpoint="api",instance="192.168.123.248:2379",job="etcd",namespace="monitoring",service="etcd-k8s"}
=> up{endpoint="api",instance="192.168.123.248:2379",job="etcd",namespace="monitoring",service="etcd-k8s"} 1
 
label_join(up{endpoint="api",instance="192.168.123.248:2379",job="etcd",namespace="monitoring",service="etcd-k8s"}, "foo", ",", "job", "service")
=> up{endpoint="api",foo="etcd,etcd-k8s",instance="192.168.123.248:2379",job="etcd",namespace="monitoring",service="etcd-k8s"} 1
label_replace()

```


#### 2.21label_replace()


为了能够让客户端的图标更具有可读性，可以通过`label_replace`函数为时间序列添加额外的标签。`label_replace`的具体参数如下：


```sh
label_replace(v instant-vector, dst_label string, replacement string, src_label string, regex string)

```


该函数会依次对 v 中的每一条时间序列进行处理，通过`regex`匹配`src_label`的值，并将匹配部分`relacement`写入到`dst_label`标签中。如下所示：


```sh
label_replace(up, "host", "$1", "instance", "(.*):.*")

```


函数处理后，时间序列将包含一个`host`标签，`host`标签的值为`Exporter`实例的 IP 地址：


```sh
up{host="localhost",instance="localhost:8080",job="cadvisor"} 1
up{host="localhost",instance="localhost:9090",job="prometheus"} 1
up{host="localhost",instance="localhost:9100",job="node"} 1

```


#### 2.22ln()


计算瞬时向量 v 中所有样本数据的自然对数。特殊情况：


```sh
ln(+Inf) = +Inf
ln(0) = -Inf
ln(x<0) = NaN
ln(NaN) = NaN

```


#### 2.23log2()


`log2(v instant-vector)`计算v中所有元素的二进制对数。特殊情况等同于ln中的特殊情况。


#### 2.24log10()


`log10(v instant-vector)`计算v中所有元素的10进制对数。特殊情况等同于ln中的特殊情况。


#### 2.25minute()


`minute(v=vector(time()) instant-vector)`以UTC为单位返回每个给定时间的分钟。 返回值为0到59。


#### 2.26month()


`month(v=vector(time()) instant-vector)`返回UTC中每个给定时间的一年中的月份。 返回值为1到12，其中1表示1月等。


#### 2.27predict_linear()


`predict_linear(v range-vector, t scalar)`函数可以预测时间序列 v 在 t 秒后的值。它基于简单线性回归的方式，对时间窗口内的样本数据进行统计，从而可以对时间序列的变化趋势做出预测。该函数的返回结果不带有度量指标，只有标签列表。如，基于 2 小时的样本数据，来预测主机可用磁盘空间的是否在 4 个小时候被占满，可以使用如下表达式：


```sh
predict_linear(node_filesystem_free{job="node"}[2h], 4 * 3600) < 0

```


通过下面的例子来观察返回值：


```sh
predict_linear(http_requests_total{code="200",instance="120.77.65.193:9090",job="prometheus",method="get"}[5m], 5)

```


结果：


```sh
{code="200",handler="query_range",instance="120.77.65.193:9090",job="prometheus",method="get"} 1
{code="200",handler="prometheus",instance="120.77.65.193:9090",job="prometheus",method="get"} 4283.449995397104
{code="200",handler="static",instance="120.77.65.193:9090",job="prometheus",method="get"} 22.99999999999999

```


#### 2.28rate()


`rate(v range-vector)`函数可以直接计算区间向量 v 在时间窗口内平均增长速率，它会在单调性发生变化时(如由于采样目标重启引起的计数器复位)自动中断。该函数的返回结果不带有度量指标，只有标签列表。


例如，以下表达式返回区间向量中每个时间序列过去 5 分钟内 HTTP 请求数的每秒增长率：


```sh
rate(http_requests_total{job="api-server"}[5m])

```


`rate()`函数返回值类型只能用计数器，在长期趋势分析或者告警中推荐使用这个函数。


注意，当将`rate()`函数与聚合运算符（例如`sum()`）或随时间聚合的函数（任何以`_over_time`结尾的函数）一起使用时，必须先执行`rate`函数，然后再进行聚合操作，否则当采样目标重新启动时`rate()`无法检测到计数器是否被重置。


#### 2.29resets()


`resets(v range-vector)`的参数是一个区间向量。对于每个时间序列，它都返回一个计数器重置的次数。两个连续样本之间的值的减少被认为是一次计数器重置。


这个函数一般只用在计数器类型的时间序列上。


#### 2.30round()


`round(v instant-vector, to_nearest=1 scalar)`函数与`ceil`和`floor`函数类似，返回向量中所有样本值的最接近的整数。`to_nearest`参数是可选的,默认为 1,表示样本返回的是最接近 1 的整数倍的值。你也可以将该参数指定为任意值（也可以是小数），表示样本返回的是最接近它的整数倍的值。


#### 2.31scalar()


`scalar(v instant-vector)`函数的参数是一个单元素的瞬时向量,它返回其唯一的时间序列的值作为一个标量。如果度量指标的样本数量大于 1 或者等于 0, 则返回 NaN。


#### 2.32sort()


`sort(v instant-vector)`函数对向量按元素的值进行升序排序，返回结果：`key: value = 度量指标：样本值[升序排列]`。


#### 2.33sort_desc()


`sort(v instant-vector)`函数对向量按元素的值进行降序排序，返回结果：`key: value = 度量指标：样本值[降序排列]`。


#### 2.34sqrt()


`sqrt(v instant-vector)`函数计算向量 v 中所有元素的平方根。


#### 2.35time()


`time()`函数返回从 1970-01-01 到现在的秒数。注意：它不是直接返回当前时间，而是时间戳。


#### 2.36timestamp()


#### 2.37vector()


`vector(s scalar)`函数将标量 s 作为没有标签的向量返回，即返回结果为：`key: value= {}, s`。


#### 2.38year()


`year(v=vector(time()) instant-vector)`以UTC格式返回每个给定时间的年份。


#### 2.39aggregation>_over_time()


`year(v=vector(time()) instant-vector)`函数返回被给定 UTC 时间的当前年份。


```sh
<aggregation>_over_time()

```


下面的函数列表允许传入一个区间向量，它们会聚合每个时间序列的范围，并返回一个瞬时向量：


```sh
avg_over_time(range-vector) : 区间向量内每个度量指标的平均值。
min_over_time(range-vector) : 区间向量内每个度量指标的最小值。
max_over_time(range-vector) : 区间向量内每个度量指标的最大值。
sum_over_time(range-vector) : 区间向量内每个度量指标的求和。
count_over_time(range-vector) : 区间向量内每个度量指标的样本数据个数。
quantile_over_time(scalar, range-vector) : 区间向量内每个度量指标的样本数据值分位数，φ-quantile (0 ≤ φ ≤ 1)。
stddev_over_time(range-vector) : 区间向量内每个度量指标的总体标准差。
stdvar_over_time(range-vector) : 区间向量内每个度量指标的总体标准方差。

```


请注意，即使值在整个时间间隔内的间隔不均匀，指定时间间隔内的所有值在聚合中都具有相同的权重。注意：即使区间向量内的值分布不均匀，它们在聚合时的权重也是相同的。


### 3.PromQL例子


#### 3.1简单的时间序列选择


使用度量标准`http_requests_total`返回所有时间序列：


```sh
http_requests_total

```


使用度量标准`http_requests_total`以及给定的job和handler标签返回所有时间系列：


```sh
http_requests_total{job="apiserver", handler="/api/comments"}

```


返回相同向量的整个时间范围（在本例中为5分钟），使其成为范围向量：


```sh
http_requests_total{job="apiserver", handler="/api/comments"}[5m]

```


请注意，导致范围向量的表达式不能直接绘制，而是在表达式浏览器的表格（"Console"）视图中查看。


使用正则表达式，您只能为名称与特定模式匹配的作业选择时间序列，在本例中为所有以server结尾的作业。 请注意，这会进行子字符串匹配，而不是完整的字符串匹配：


```sh
http_requests_total{job=~"server$"}

```


要选择除4xx之外的所有HTTP状态代码，您可以运行：


```sh
http_requests_total{status!~"^4..$"}

```


#### 3.2子查询


此查询返回过去30分钟的5分钟`http_requests_total`指标率，分辨率为1分钟：


```sh
rate(http_requests_total[5m])[30m:1m]

```


这是嵌套子查询的示例。 deri函数的子查询使用默认分辨率。 请注意，不必要地使用子查询是不明智的。


```sh
max_over_time(deriv(rate(distance_covered_total[5s])[30s:5s])[10m:])

```


#### 3.3使用函数，操作符等


使用`http_requests_total`指标名称返回所有时间序列的每秒速率，在过去5分钟内的增长率：


```sh
rate(http_requests_total[5m])

```


假设`http_requests_total`时间序列都有标签job（按作业名称扇出）和instance（按作业实例扇出），我们可能想要总结所有实例的速率，因此我们得到的输出时间序列更少，但仍然保留job维度。


```sh
sum(rate(http_requests_total)[5m]) by (job)

```


如果我们有两个具有相同维度标签的不同指标，我们可以对它们应用二元运算符，并且两侧具有相同标签集的元素将匹配并传播到输出。例如，此表达式为每个实例返回MiB中未使用的内存（在虚构的群集调度程序上公开它运行的实例的这些度量标准）：


```sh
(instance_memory_limit_byte - instant_memory_usage_bytes) / 1024 / 1024

```


相同的表达式，但由应用程序总结，可以这样写：


```sh
sum( instance_memory_limit_bytes - instance_memory_usage_bytes) by (app, proc) / 1024 / 1024

```


如果相同的虚构集群调度程序为每个实例公开了如下所示的CPU使用率指标：


```sh
instance_cpu_time_ns{app="lion", pro="web", rev="34d0f99", env="prod", job="cluster-manager"}
instance_cpu_time_ns{app="elephant", proc="worker", rev="34d0f99", env="prod", job="cluster-manager"}
instance_cpu_time_ns{app="turtle", proc="api", rev="4d3a513", env="prod", job="cluster-manager"}

```


我们可以按应用程序（app）和进程类型（proc）分组排名前3位的CPU用户：


```sh
topk(3, sum(rate(instance_cpu_time_ns[5m])) by(app, proc))

```


假设此度量标准包含每个运行实例的一个时间系列，您可以计算每个应用程序运行实例的数量，如下所示：


```sh
count(instance_cpu_time_ns) by (app)

```

