
## Prometheus 快速入门教程（五）：Prometheus 的关键概念
摘抄自：https://www.cnblogs.com/chanshuyi/p/05_key_concept_of_prometheus.html








前面几篇文章，我们单刀直入地讲解了 Prometheus 能做什么。接着用一个例子来让大家知道如何使用 Prometheus，以及如何进行告警配置。最后，还用了一篇文章来讲解如何进行图表配置。但是 Prometheus 里面也有一些关键性的概念，理解这些概念有利于我们后续更深入的学习。


### Metric 指标


在 Prometheus 中，我们所有的信息都以 Metrics（指标） 的形式存在。


Metrics 由 metric name 和 label name 组成。


```go
<metric name>{<label name>=<label value>, ...}

```


例如下面的`api_http_requests_total`就是 metrics name（指标名称），而`method`就是 label name（标签）。而 metric name 加上 label name 就是一个完整的 Metric。


```go
api_http_requests_total{method="POST", handler="/messages"}

```


### Metric Type 指标类型


Prometheus 中主要有四种不同的指标类型，用来适应不同的指标类型。


* counter 计数器    
* gauges 计量器    
* histogram 柱状图    
* summary 汇总


#### counter 计数器


数据从 0 开始累计，理想状态下应该是永远增长或者是不变。


适用于例如机器开机时间、HTTP 访问量等数值。


#### gauges 量器


获取一个返回值，采集回来是多少就是多少。数值可能升高，也可能降低。


适用于例如硬盘容量、CPU 内存使用率等数值。


#### histogram 柱状图


counter 和 gauges 反应的是数值的情况，而 histogram 则是反应数值的分布情况。


histogram 柱状图反映了样本的区间分布梳理，经常用来表示请求持续时间、响应大小等信息。


例如我们 1 分钟内有 1000 个 http 请求，我们想要知道大多数的请求耗时是多少。这时我们使用评价耗时可能不太准，但是我们使用 histogram 柱状图就可以看出这些请求大多数都是分布在哪个耗时区间。


![](assets/061d51fd40e19fcc30642392b603ca55.jpg)


#### Summary 汇总





柱状图同样表示样本的分布情况，与 Histogram 类似，其会有总数、数量表示。**但其多了一个是中位数的表示。**常用来表示类似于：请求持续时间、响应大小的信息。


![](assets/1578cf3a5a42ca3bbd30eee45fcb5784.jpg)


#### Histogram 与 summary


为了区分是平均的慢还是长尾的慢，最简单的方式就是按照请求延迟的范围进行分组。例如，统计延迟在010ms之间的请求数有多少而1020ms之间的请求数又有多少。通过这种方式可以快速分析系统慢的原因。Histogram和Summary都是为了能够解决这样问题的存在，通过Histogram和Summary类型的监控指标，我们可以快速了解监控样本的分布情况。


**Histogram 指标直接反应了在不同区间内样本的个数，区间通过标签len进行定义。而 summary 则是使用中位数反映样本的情况。**


### 任务(Job）和实例（Instance）


在 Prometheus 中抓取数据的应用叫做实例（Instance），而几个为了同个目的的实例组合起来称之为任务（Job）。例如下面是一个有4个实例的服务工作。


```go
job: api-server  // 名为api-server的job
    instance 1: 1.2.3.4:5670  
    instance 2: 1.2.3.4:5671  
    instance 3: 5.6.7.8:5670
    instance 4: 5.6.7.8:5671

```





### 总结


这篇文章我们介绍了 Prometheus 的几个关键概念：


* Metric 指标    
* Metric Type 指标类型    
* 工作(Job）和实例（Instance）


它们之间的关系如下图所示：


![](assets/0640c9971e868de8e760867c926cf336.jpg)


一个任务（Job）可以有多个实例（Instance），一个实例上可以有多个指标（Metric），一个指标只会有一个指标类型（Metric Type）。





