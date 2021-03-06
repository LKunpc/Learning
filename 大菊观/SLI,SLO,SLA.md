SLO和SLA是大家常见的两个名词：服务等级目标和服务等级协议。

云计算时代，各大云服务提供商都发布有自己服务的SLA条款，比如Amazon的EC2和S3服务都有相应的SLA条款。这些大公司的SLA看上去如此的高达上，一般是怎么定义出来的呢？本文就尝试从技术角度解剖一下SLA的制定过程。

说SLA不能不提SLO，这个是众所周知的，但是还有一个概念知道的人就不多了，那就是SLI（Service Level Indicator），定义一个可执行的SLA，好的SLO和SLI是必不可少的。

再有就是SLI/SLO/SLA都是和服务联系在一起的，脱离了服务这三个概念就没有什么意义了。

## Service
什么是服务？

简单说就是一切提供给客户的有用功能都可以称为服务。

服务一般会由服务提供者提供，提供这个有用功能的组织被称为服务提供者，通常是人加上软件，软件的运行需要计算资源，为了能对外提供有用的功能软件可能会有对其他软件系统的依赖。

客户是使用服务提供者提供的服务的人或公司。

## SLI
SLI是经过仔细定义的测量指标，它根据不同系统特点确定要测量什么，SLI的确定是一个非常复杂的过程。

SLI的确定需要回答以下几个问题：

要测量的指标是什么？

测量时的系统状态？

如何汇总处理测量的指标？

测量指标能否准确描述服务质量？

测量指标的可靠度(trustworthy)？

### 1. 常见的测量指标有以下几个方面：
性能

响应时间(latency)

吞吐量(throughput)

请求量(qps)

实效性(freshness)

可用性

运行时间(uptime)

故障时间/频率

可靠性

质量

准确性(accuracy)

正确性(correctness)

完整性(completeness)

覆盖率(coverage)

相关性(relevance)

内部指标

队列长度(queue length)

内存占用(RAM usage)

因素人

响应时间(time to response)

修复时间(time to fix)

修复率(fraction fixed)

下面通过一个例子来说明一下：hotmail的downtime SLI

错误率(error rate)计算的是服务返回给用户的error总数

如果错误率大于X%，就算是服务down了，开始计算downtime

如果错误率持续超过Y分钟，这个downtime就会被计算在内

间断性的小于Y分钟的downtime是不被计算在内的。

### 2. 测量时的系统状态，在什么情况下测量会严重影响测量的结果
测量异常(badly-formed)请求，还是失败(fail)请求还是超时请求(timeout)

测量时的系统负载（是否最大负载）

测量的发起位置，服务器端还是客户端

测量的时间窗口（仅工作日、还是一周7天、是否包括计划内的维护时间段）

### 3. 如何汇总处理测量的指标？
计算的时间区间是什么：是一个滚动时间窗口，还是简单的按照月份计算

使用平均值还是百分位值，比如：某服务X的ticket处理响应时间SLI的

测量指标：统计所有成功解决请求，从用户创建ticket到问题被解决的时间

怎么测量：用ticket自带的时间戳，统计所有用户创建的ticket

什么情况下的测量：只包括工作时间，不包含法定假日

用于SLI的数据指标：以一周为滑动窗口，95%分位的解决时间

### 4. 测量指标能否准确描述服务质量？
性能：时效性、是否有偏差

准确性：精度、覆盖率、数据稳定性

完整性：数据丢失、无效数据、异常(outlier)数据

### 5. 测量指标的可靠度
是否服务提供者和客户都认可

是否可被独立验证，比如三方机构

客户端还是服务器端测量，取样间隔

错误请求是如何计算的

## SLO
SLO(服务等级目标)指定了服务所提供功能的一种期望状态。SLO里面应该包含什么呢？所有能够描述服务应该提供什么样功能的信息。

服务提供者用它来指定系统的预期状态；开发人员编写代码来实现；客户依赖于SLO进行商业判断。SLO里没有提到，如果目标达不到会怎么样。

SLO是用SLI来描述的，一般描述为：
比如以下SLO：

每分钟平均qps > 100k/s

99% 访问延迟 < 500ms

99% 每分钟带宽 > 200MB/s

设置SLO时的几个最佳实践：

指定计算的时间窗口

使用一致的时间窗口(XX小时滚动窗口、季度滚动窗口)

要有一个免责条款，比如：95%的时间要能够达到SLO

如果Service是第一次设置SLO，可以遵循以下原则

测量系统当前状态

设置预期(expectations)，而不是保证(guarantees)

初期的SLO不适合作为服务质量的强化工具

改进SLO

设置更低的响应时间、更改的吞吐量等

保持一定的安全缓冲

内部用的SLO要高于对外宣称的SLO

不要超额完成

定期的downtime来使SLO不超额完成

设置SLO时的目标依赖于系统的不同状态(conditions)，根据不同状态设置不同的SLO：总SLO = service1.SLO1 weight1 service2.SLO2 weight2 …

为什么要有SLO，设置SLO的好处是什么呢？

对于客户而言，是可预期的服务质量，可以简化客户端的系统设计

对于服务提供者而言

可预期的服务质量

更好的取舍成本/收益

更好的风险控制(当资源受限的时候)

故障时更快的反应，采取正确措施

SLO设好了，怎么保证能够达到目标呢？
需要一个控制系统来：

监控/测量SLIs

对比检测到的SLIs值是否达到目标

如果需要，修证目标或者修正系统以满足目标需要

实施目标的修改或者系统的修改

该控制系统需要重复的执行以上动作，以形成一个标准的反馈环路，不断的衡量和改进SLO/服务本身。

我们讨论了目标以及目标是怎么测量的，还讨论了控制机制来达到设置的目标，但是如果因为某些原因，设置的目标达不到该怎么办呢？

也许是因为大量的新增负载；也许是因为底层依赖不能达到标称的SLO而影响上次服务的SLO。这就需要SLA出场了。

SLA
SLA是一个涉及2方的合约，双方必须都要同意并遵守这个合约。当需要对外提供服务时，SLA是非常重要的一个服务质量信号，需要产品和法务部门的同时介入。

SLA用一个简单的公式来描述就是： SLA = SLO 后果

SLO不能满足的一系列动作，可以是部分不能达到

比如：达到响应时间SLO 未达到可用性SLO

对动作的具体实施

需要一个通用的货币来奖励/惩罚，比如：钱

SLA是一个很好的工具，可以用来帮助合理配置资源。一个有明确SLA的服务最理想的运行状态是：增加额外资源来改进系统所带来的收益小于把该资源投给其他服务所带来的收益。

一个简单的例子就是某服务可用性从99.9%提高到99.99%所需要的资源和带来的收益之比，是决定该服务是否应该提供4个9的重要依据。
