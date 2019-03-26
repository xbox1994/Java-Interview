## 第一步	：弄清用例与约束

> 收集需求和问题的范围
> 通过问问题来弄清用例与约束
> 讨论假设

我们假定以下用例

### 用例
解决这个问题需要采用迭代的方法：

1. 基准/负载测试
2. 瓶颈检测
3. 评估替代方案来解决瓶颈
4. 重复以上

这是将基本设计升级为可扩展设计的良好模式

除非你有AWS的背景或者正在申请AWS的相关职位，否则在AWS上的实现细节不需要了解。然而**大部分在这里讨论的原理可以应用到除了AWS以外更通用的地方**

#### 我们将问题约束到如下范围
* **用户**发送读或写请求
    * **服务**处理，存储用户数据然后返回结果
* **服务**需要从少量用户发展到数百万用户
    * 在我们升级架构来处理大量用户请求时，讨论通用的扩展模式
* **服务**需要高可用

### 约束和假设

#### 状态假设

* 流量分布不均
* 需要关系型数据
* 从单个用户扩展到千万级用户
    * 用户增加的标识：
        * 用户数+
        * 用户数++
        * 用户数+++
        * ...
    * 一千万用户
    * 每月10亿次写入
    * 每月1000亿次读取
    * 100:1读写比
    * 每次写入1KB内容

#### 计算方式

**如果你想做一个大致估算，请向你的面试官表明以下数据：**

* 每月1TB数据写入
    * 每次写入1KB数据 * 每月10亿次写入
    * 3年有3TB数据写入
    * 假设大多数写入是新的内容而不是已有内容的更新
* 平均每秒400次写入
* 平均每秒40000次读取

方便的转换公式:

* 每月有250万秒
* 每秒一个请求 = 每月250万个请求
* 每秒40个请求 = 每月1亿个请求
* 每秒400个请求 = 每月10亿个请求

## 第二步：创建高层设计

> 大致写出包含所有重要组件的高层设计

![](https://github.com/xbox1994/2018-Java-Interview/raw/master/images/aws1.png)

## 第三步：设计核心组件

> 深入每个核心组件的细节

### 用例：用户发送读或写的请求

#### 目标

* 对于仅仅的1-2个用户，你只需要一个基本的配置
    * 简单的单体应用
    * 当需要的时候垂直缩放
    * 监控来确定瓶颈

#### 从单体应用开始

* EC2上的**服务器**
    * 存储用户数据
    * [**MySQL数据库**](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#%E5%85%B3%E7%B3%BB%E5%9E%8B%E6%95%B0%E6%8D%AE%E5%BA%93%E7%AE%A1%E7%90%86%E7%B3%BB%E7%BB%9Frdbms)

使用**垂直扩展**:

* 选择更好性能的机器
* 密切关注监控指标以确定如何扩大规模
    * 使用基本监控来确定瓶颈：CPU，内存，IO，网络等
    * CloudWatch, top, nagios, statsd, graphite等
* 垂直缩放可能会很昂贵
* 没有故障转移措施

*替代方案和其他细节：*

* **垂直扩展**的替代是[**水平扩展**](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#%E6%B0%B4%E5%B9%B3%E6%89%A9%E5%B1%95)

#### 从SQL开始，考虑NoSQL

约束里我们需要关系型数据。我们在开始的时候可以在单机上用**MySQL数据库**.

*替代方案和其他细节：*

* [关系型数据库](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#%E5%85%B3%E7%B3%BB%E5%9E%8B%E6%95%B0%E6%8D%AE%E5%BA%93%E7%AE%A1%E7%90%86%E7%B3%BB%E7%BB%9Frdbms)
* 使用[SQL还是NoSQL](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#sql-%E8%BF%98%E6%98%AF-nosql)的原因

#### 分配公网静态IP

* 弹性IP提供一个重启之后不会更改的公网端口
* 有效的帮助故障转移，只需要将域名指向新IP

#### 使用DNS

使用Route 53添加**DNS**将域名映射到实例的公共IP

*替代方案和其他细节：*

* [DNS](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#%E5%9F%9F%E5%90%8D%E7%B3%BB%E7%BB%9F)

#### 保护web服务器

* 开启必要的端口
    * 允许web服务器对于以下端口回复:
        * 80 - HTTP
        * 443 - HTTPS
        * 22 - SSH（白名单）
    * 阻止web服务器进行出站连接

*替代方案和其他细节：*

* [安全](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#%E5%AE%89%E5%85%A8)

## 第四步：扩展设计

> 鉴于约束条件，确定并解决瓶颈

### 用户数+

![](https://github.com/xbox1994/2018-Java-Interview/raw/master/images/aws2.png)

#### 假设

我们的用户数正在增加并且在我们单体应用上的负载也在增加。我们的**基准/负载测试**和**瓶颈**指向了**MySQL数据库**占用更多内存和CPU资源，同时用户内容正在填满磁盘空间

到目前为止我们可以通过**水平扩展**解决问题。但不幸的是已经变得非常昂贵并且**MySQL数据库**和**web服务器**无法独立扩展

#### 目标

* 减轻单体应用的负载并且允许独立扩展
    * 将静态内容分开存储到**AWS对象存储**
    * 移动**MySQL数据库**到独立的服务上
* 缺点
    * 这些改变将增加复杂度并且需要**Web服务器**指向**对象存储**和**MySQL数据库**
    * 新组件额外的安全措施
    * AWS的费用将会增加但应该与自己管理类似系统成本进行权衡

#### 分离存储静态内容

* 考虑使用S3作为**对象存储**
    * 高扩展和可靠性
    * 服务端加密
* 移动静态内容到S3
    * 用户文件
    * JS
    * CSS
    * 图片
    * 视频

#### 移动MySQL数据库到独立的服务

* 考虑使用RDS服务管理**MySQL数据库**
    * 扩展和管理简单
    * 多个可用区
    * 静态加密

#### 保护系统

* 在传输和静止时加密数据
* 使用虚拟私有网络
    * 为单个**Web服务器**创建公共子网以便可以发送和接收网上的流量
    * 为其他组件创建私有网络，组织外部访问
    * 每个组件仅仅对白名单IP开放端口

### 用户数++

![](https://github.com/xbox1994/2018-Java-Interview/raw/master/images/aws3.png)

#### 假设

我们的**基准/负载测试**和**瓶颈检测**表明我们的单体**Web服务器**在高峰期出现瓶颈，导致回应慢，在某些情况下宕机。随着服务的成熟，我们希望提高可用性和冗余度

#### 目的

* 以下目标尝试解决**Web服务器**的扩展问题
    * 基于**基准/负载测试**和**瓶颈检测**，你可能只需要实现这些技术中的一个或者两个
* 使用[**水平扩展**](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#%E6%B0%B4%E5%B9%B3%E6%89%A9%E5%B1%95)处理不断增加的负载并解决单体故障
    * 添加[**负载均衡器**](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E5%99%A8)
        * ELB是高可用的
        * 如果你想配置自己的**负载均衡器**, 在多个可用区配置[主-主](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#%E5%8F%8C%E5%B7%A5%E4%BD%9C%E5%88%87%E6%8D%A2active-active)或[主-备](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#%E5%B7%A5%E4%BD%9C%E5%88%B0%E5%A4%87%E7%94%A8%E5%88%87%E6%8D%A2active-passive)可以提高可用性
        * 在**负载均衡器**上关闭SSL去减少在后端服务器上的计算负载并简化证书管理
    * 使用多个**Web服务器**分布到多个区域
    * 使用多个[**主从故障切换**](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6)模式的**MySQL**实例来增进冗余度
* 将**Web服务器**和[**应用服务器**](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#%E5%BA%94%E7%94%A8%E5%B1%82)分开
    * 独立扩展和配置这两层
    * **Web服务器**可以作为[**反向代理服务器**](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86web-%E6%9C%8D%E5%8A%A1%E5%99%A8)
    * 比如你可以添加**应用服务器**处理**读API**而其他**应用服务器**处理**写API**
* 移动静态（和一些动态)内容到[**CDN**](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#%E5%86%85%E5%AE%B9%E5%88%86%E5%8F%91%E7%BD%91%E7%BB%9Ccdn)比如CloudFount去减少负载和延迟

### 用户数+++

![](https://github.com/xbox1994/2018-Java-Interview/raw/master/images/aws4.png)

**注意:** 为了避免过于混乱，没有显示**内部负载均衡器**

#### 假设

我们的**基准/负载测试**和**瓶颈检测**表明我们的读请求很多(100:1读写比)，我们的数据库因为大量读取请求导致性能不佳

#### 目标

* 以下目标尝试去解决在**MySQL数据库**上的问题
    * 基于**基准/负载测试**和**瓶颈检测**，你可能只需要实现这些技术中的一个或者两个
* 移动以下数据到[**内存缓存**](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#%E7%BC%93%E5%AD%98)，比如Elasticache去减少负载和延迟：
    * 在**MySQL**中经常读取的内容
        * 首先，在实现**内存缓存**之前试图配置**MySQL数据库**的缓存看是否足以解决瓶颈
    * 来自**Web服务器**的session数据
        * **Web服务器**变成无状态服务，允许**自动缩放**
    * 从内存读取1MB需要250微秒，而SSD需要4倍的时间，从硬盘读取需要80倍时间
* 添加[**MySQL只读副本**](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6)来减少主服务器的负载
* 添加更多**Web服务器**和**应用服务器**来提升响应

#### 添加MySQL只读副本

* 除了增加和扩展**内存缓存**外, **MySQL只读副本**也能帮助减轻**MySQL主节点**的负载
* 添加**Web服务器**的逻辑来分开读写数据
* 在**MySQL只读副本**前添加**负载均衡器**(图里没画)

### 用户数++++

![](https://github.com/xbox1994/2018-Java-Interview/raw/master/images/aws5.png)

#### 假设

我们的**基准/负载测试**和**瓶颈检测**表明在正常工作时间内流量激增，在用户离开办公室时显著下降。我们认为我们可以根据实际负载自动调整服务器来降低成本。我们是个小公司，因此我们希望尽可能多地**自动缩放**

#### 目标

* 添加**自动缩放**来根据需求提供实例数量
    * 跟上流量的高峰
    * 通过关闭未使用的实例来减少费用
* DevOps自动化
    * Chef, Puppet, Ansible等
* 继续监控指标以解决瓶颈
    * **主机级别** - 查看单个EC2实例
    * **汇总级别** - 查看负载均衡器统计信息
    * **日志分析** - CloudWatch, CloudTrail, Loggly, Splunk, Sumo
    * **外部网站性能** - Pingdom或New Relic
    * **处理通知和时间** - PagerDuty
    * **错误报告** - Sentry

#### 添加自动缩放

* 考虑AWS的托管服务**自动缩放**
    * 为每个**Web服务器**和**应用服务器**创建一个组, 每个组放到多个可用区中
    * 设置最小和最大实例数
    * 通过CloudWatch触发向上和向下扩展
        * 一段时间内的指标：
            * CPU负载
            * 延迟
            * 网络流量
            * 自定义指标
    * 缺点
        * 自动缩放可能会带来复杂性
        * 系统可能需要一段时间才能适当扩展以满足不断增长的需求，或者在需求下降时缩小规模

### 用户数+++++

![](https://github.com/xbox1994/2018-Java-Interview/raw/master/images/aws6.png)

**注意:** **自动缩放**组未在图中显示

#### 假设

随着服务继续朝着约束中的数字增长, **基准/负载测试**和**瓶颈检测**继续迭代来发现和解决新的瓶颈

#### 目标

由于问题的限制，我们将继续解决扩展问题：

* 如果我们的**MySQL数据库**开始变得非常大，我们可能会考虑只将有限时间段的数据存储在数据库中，同时将其余数据存储在Redshift等数据仓库中
    * 像Redshift这样的数据仓库可以轻松处理每月1TB的新内容
* 每秒平均读取请求4万次，读取常用数据的流量可以通过扩展**内存缓存**来解决，这对于处理不均匀分布的流量和流量峰值也很有用
    * **SQL只读副本**可能在处理缓存未命中时遇到问题，我们可能需要采用其他SQL扩展模式
* 对于单个**SQL写服务**来说，每秒400次平均写入次数（可能更高的峰值）可能很难，同时也表明需要额外的缩放技术

SQL扩展模式包括:

* [联合](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#%E8%81%94%E5%90%88)
* [分片](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#%E5%88%86%E7%89%87)
* [非规范化](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#%E9%9D%9E%E8%A7%84%E8%8C%83%E5%8C%96)
* [SQL调优](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#sql-%E8%B0%83%E4%BC%98)

为了进一步解决高读取和写入请求，我们还应考虑将适当的数据移动到[**NoSQL数据库**](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#nosql)，例如DynamoDB

我们可以进一步分离[**应用服务器**](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#%E5%BA%94%E7%94%A8%E5%B1%82)来允许独立的缩放。不需要实时完成的批处理和计算可以使用**队列**和**工作程序**[**异步**](https://github.com/donnemartin/system-design-primer/blob/master/README-zh-Hans.md#%E5%BC%82%E6%AD%A5)完成:

* 例如，在照片服务中，照片上传和缩略图创建可以分开：
    * **客户端**上传图片
    * **应用程序服务器**放一个任务到**队列**
    * **工作服务**从**队列**中拉取到任务：
        * 创建缩略图
        * 上传到**数据库**
        * 存储缩略图到**对象存储**

欢迎光临[我的博客](http://www.wangtianyi.top/?utm_source=github&utm_medium=github)，发现更多技术资源~
