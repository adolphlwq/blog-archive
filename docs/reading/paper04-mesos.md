# 分布式集群调度框架Mesos架构与实现
# 简介
Mesos是一个在多个集群计算框架中**共享集群资源**的管理系统，它提高了集群资源利用率，避免了每个计算框架数据复制。

通过分布式两层调度模型实现了细粒度的资源分配：由Mesos决定为每个框架提供多少资源，框架决定接受哪些资源，以及把计算任务分配到哪里去执行。

# 问题与方案
2010年代计算框架百花齐放，相继出现[MapReduce](https://hadoop.apache.org/docs/r1.2.1/mapred_tutorial.html)、MPI、Dryad、Pregel等。很明显，新的集群计算框架还会不断涌现，`不存在满足所有应用需求的集群计算框架`，因此企业、研究机构会在同一个集群中运行多个计算框架。由于不同的计算框架是相互独立的，**导致不同框架间共享数据和计算资源变得异常困难**。

Mesos实现轻量级的资源共享层，保证不同框架间细粒度的资源共享。它不仅要满足不同计算框架的需求，还要能够满足未来一些新的计算框架的需求。因此对Mesos的扩展性和效率有较高要求。

一个可选方案是中心化调度器，这种方案过于复杂、无法满足所有计算框架的需求，而且会带来大量的重构，并不现实。

Mesos提出一种新的调度抽象`resource offer`，Mesos决定提供多少资源给框架，框架决定接受哪些资源，将任务和资源匹配。这是一种去中心化的调度模型，它简单易于实现，而且给Mesos带来了很高的扩展性和健壮性，并且还有两个额外的优势：
1. 支持运行同一个框架的不同版本
2. 易于新框架的开发

# 架构设计
### 设计哲学
最初的目的是为不同框架提供可扩展、弹性的内核，使得他们高效共享集群。但是考虑到计算框架的多样性和快速迭代，设计哲学演进为实现最小化的接口，保证框架间高效的资源共享，因此将任务调度和执行交给框架去做。

![](http://mesos.apache.org/assets/img/documentation/architecture-example.jpg)

Mesos master通过`resource offer`模型，将集群中的可用资源提供给框架，提供多少资源基于不同的策略，比如公平策略、优先级策略等。同时还提供插件，允许框架实现自己的策略。

每个计算框架包括两部分：scheduler(调度器)和executor(执行器)。调度器注册到Mesos master便可接收资源，执行器运行在salve节点上负责执行具体的任务。框架不会指定自己需要多少资源，而是有Mesos master上报，合适就使用，不合适就拒绝。

上述的机制存在一个问题，如果一个框架需要的资源很多，迟迟得不到满足，就会出现`饥饿`(概率很低)。为此Mesos为框架提供filters机制，框架会告知Mesos自己明确会拒绝某些资源。

### 容错
Mesos master只保留少量必须的状态信息，这样当master崩溃时，新的master可以根据slave和框架上的信息快速恢复。master只包含slave、框架和运行中的任务这三种信息。

运行多个master时，使用zookeeper实现leader选举。

![](http://mesos.apache.org/assets/img/documentation/architecture3.jpg)

# 代码实现
Mesos基于C++实现，借用了很多现有的技术成果，比如C++ actor编程模型库[libprocess](https://github.com/3rdparty/libprocess)、ZooKeeper、[Linux Container](https://linuxcontainers.org/)等。

为了证明Mesos的轻量、易于框架的开发，文章作者基于Mesos开发了面向机器学习中`iterative jobs`的框架Spark，这就是后来名震业界的大数据处理框架。得益于Mesos的优良设计，**Spark原型只用了大约1300行代码**。

# 总结
Mesos的出现有其特定的背景和目的，其解决的是行业内某一领域的痛点问题。Mesos实现了分布式两级调度模型，使得不同的计算框架可以使用同一个计算机集群中的资源，提高了集群资源的利用率。

另外，这一框架也促使另一个著名框架Spark的诞生。

Mesos来自于UC Berkeley AMPLab，从这篇论文中可以看到UC Berkeley计算机研究的显著特点：`理论和实践相结合`，`不纸上谈兵`。这要求研究人员(导师、博士、硕士)不仅要有一流的研究能力，还要有很强的编程实践能力。

# 参考
- [Apache Mesos](http://mesos.apache.org/)