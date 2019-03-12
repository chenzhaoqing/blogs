title: 'YARN（《Hadoop: The Definitive Guide》第4版 读书笔记）'
tags:
  - yarn
  - hadoop
categories: reading notes
date: 2018-09-13 15:43:14
---
# YARN
Apache **YARN** (Yet Another Resource Negotiator) 是Hadoop集群的资源管理系统

**Resource Manager** (每个集群只运行一个)：管理整个集群资源的使用

**Node Manager** (运行在集群中的所有节点)：启动并监控容器
<!--more-->
### YARN 执行应用的过程

{% asset_img how-yarn-runs-an-application.png YARN执行应用的过程（图片引自原书）  %}
（上图引自原书）
1. 客户端联系resource manager，请求执行一个应用管理进程（Application Master）

2. RM寻找一个node manager，该node manager在一个容器中启动 Application Master进程

3. Application master的执行过程，由应用自己决定。

   Application master可能只在自己所在的容器中进行计算，并返回计算结果。或者它会向resource manager请求更多的容器（containers），以便执行一个分布式计算。


### 资源请求

1. container: CPU, memory
2. 本地性：分布式数据处理算法高效使用**带宽**资源的重要保证



### 应用生存期

YARN 应用的生存期可以短至数秒，长至数天甚至数月

执行模式：

1. 每个用户作业，1个应用。如MapReduce任务
2. 一个工作流或用户会话期的作业，1个应用。该方式更有效率，作业之间的资源可以重复利用，中间数据可缓存。如Spark应用，使用该模式
3. 多用户共享的长期运行的应用。该模式的应用常常扮演某种协调的角色。如Apache Slider, Impala



### 对比YARN(MapReduce 2)和MapReduce 1

- MapReduce 1

  jobtracker：通过调度作业在tasktracker上执行，来协调所有作业执行；跟踪每个作业的整体进度；匹配作业和tasktracker；追踪任务，重启失败或慢的任务；对任务进行记录，如维护counter总量；存储作业历史（可以启动单独的服务来完成）。

  tasktracker：执行任务，发送进度报告给jobtracker。

- YARN

  resource manager： 作业调度

  application master：作业进度监控

  timeline：存储应用历史。 

  node manager：执行任务，发送进度报告给jobtracker。



| MapReduce 1 | YARN                                                  |
| ----------- | ----------------------------------------------------- |
| Jobtracker  | Resource manager, application master, timeline server |
| TaskTracker | Node manager                                          |
| Slot        | Container                                             |

- 可扩展性 (scalability)

|       | MapReduce 1 | YARN    |
| ----- | ----------- | ------- |
| nodes | 4,000       | 10,000  |
| tasks | 40,000      | 100,000 |

- 可用性 (availability)

  YARN中的resouce manager 和 application master分别处理原jobtracker负责的工作，这样更容易分别对二者实现HA (high availability)。事实上，Hadoop 2已经支持对二者的HA。

- 资源利用率 (utilization)

| MapReduce 1: slot                                            | YARN: container                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 每个tasktracker配置固定大小的slots  <br/>slots分为map slots和reduce slots，二者不可互用 | node manager管理一个资源池，而不是固定数量的指定的slots<br />不会像MapReduce1中那样遇到只有map slots而导致reduce作业无法执行的情形 <br />资源细粒度的划分，使应用可以申请合适的资源。 |

- 多租户性 (multitenancy)

  从某种角度说，YARN所带来的最大的益处是它将Hadoop开放给其他类型分布式应用，而不仅限于MapReduce.



### YARN的调度

YARN提供三种调度器： FIFO Scheduler, Capacity Schedulers, Fair Scheduler

 FIFO Scheduler: 将应用置于队列中，依据提交时间的顺序来执行。
**TO BE CONTINUED**