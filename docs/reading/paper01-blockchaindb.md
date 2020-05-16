# BlockchainDB - 构建于区块链之上的分片数据库
>2019.12.1

![](https://raw.githubusercontent.com/adolphlwq/osshub/master/oss/blog/2019/12/bcdb_arg.png)

这篇论文主要分为9部分，其中1 2节介绍BlockchainDB要解决的问题，3 4 5 6四节介绍了BlockchainDB的架构和技术细节，7节对其性能进行评估，最后总结展望。

## 问题及挑战
区块链技术的兴起催生了很多新的应用场景，一个非常重要的场景是：**互不信任的多方实现`数据共享读写`**，比如供应链上的货物追踪。但是，原生区块链存在如下缺陷：
1. 区块链的性能和扩展性存差：区块链的事务处理能力在10-100 tx/s
2. 缺少易于使用的抽象层：区块链没有提供像数据库那样简单方便的查询接口、一致性等功能

上面两个主要缺陷极大阻碍了区块链在`数据共享读写`场景的使用和推广。为了解决上面的问题，这篇论文基于区块链开发了BlockchainDB，它主要有以下优势：
1. 把区块链作为存储层(Storage Layer)，不需要修改底层区块链代码，同时充分利用其去中心化、tamper-proof log等原生技术优势
2. 提供易用的查询接口和一致性保证
3. 提高区块链的性能、降低使用复杂度

## 具体实现
### 架构
![](https://raw.githubusercontent.com/adolphlwq/osshub/master/oss/blog/2019/12/bcdb_network.png)

上图是BlockchainDB的架构图，自下而上分为存储层和数据库层，数据存储在底层区块链上，通过Client提供简单易用的put/get/verify接口。

peer指区块链里面的多个节点，常用于区块链场景，类比分布式数据库中的node。分为fully peer和thin peer，前者存储数据、资源要求高，后者不存储数据，资源要求低。

需要注意的是，每个party在加入BlockchainDB网络时需要进行认证并且做到彼此知晓，认证通过后才能加入。

### 约定
为了方便后续讨论，这里做如下约定：
1. 加入BlockchainDB网络的peer需要认证，认证通过后才能加入
2. peer可以代表本地所有的client进行验证操作
3. peer信任本地读写的数据
4. 通过**大多数**进行认证，如果大多数peer上的数据是可信的，那么数据是可信的

基于上述约定，在进行验证时，主要验证下面两个方面:
1. put操作是否被恶意丢弃
2. get得到的数据真假

### Database Layer
![](https://raw.githubusercontent.com/adolphlwq/osshub/master/oss/blog/2019/12/bcdb_layer_arc.png)

数据库层为用户提供client，封装了put/get/verify接口：
- get(t, k) --> v，返回表t对应主键k的值
- put(t, k, v) --> void 所有数据被编码到v中，作为文档插入表t中，主键为k
- verify() --> bool 该方法用于在线验证，put/get后调用verify()
    - put：插入操作(事务)是否commit
	- get：返回的结果是否正确、真实

如图3所示：client读写请求先到off-chain verifer，然后放到请求队列中。由事务管理(Tx Mgr)和分片管理(Shard Mgr)处理后转发到对应的底层区块链上，最后返回结果。

需要注意，Backend Connector用于连接底层不同的区块链。

### 一致性
针对目前提供的功能，论文介绍了`get`和`put`的一致性保证，put操作的事务处理要复杂些，get相对简单。

##### Processing Model in Blockchain
区块链是如何处理写数据的，它采用“a global order of write”比如blockchain transaction，即所有的写会根据replica附加到底层的log中。因此区块链自身的机制就支持sequential consistency，但是区块链的事务处理速度很慢，一个事务往往花费数秒到数分钟，这严重影响写密集型负载的吞吐量。

另外，有些区块链存在特殊的机制，比如Ethereum中的fork机制，它会重新执行事务，需要额外的时间。

#### 顺序一致性(Sequential Consistency)
区块链能够保证全局写的顺序，但是处理速度慢，这里没有利用区块链的机制。顾名思义，顺序一致性(Sequential)保证时间上的先后关系，每个写操作按顺序进行事务提交。这是一个同步执行过程。

#### 最终一致性(Eventual Consistency)
最终一致性(Eventual Consistency)是异步执行过程，put操作后不等待事务结果。因此put执行后BlockchainDB不会立刻进入一致性状态，但等一会后会达到一致性。

为了协调一致性和执行速度（吞吐量），BlockchainDB提供了参数用于调优，下文会详细介绍。

### Storage Layer

#### shared table
“数据共享读写”场景首先从数据库建表开始，这里将介绍表的定义和创建过程。

shared table数据结构是用来存储数据的，它包括表模式(schema)和分片配置(sharding configuration)：
1. 表的schema通过key/value数据模型来定义，key表示主键(primary key)，value表示数据(payload of a tuple)
2. 分片配置包括shard的数量、replica数量等。这些参数不仅影响整体性能，还会影响table提供的一些保证(gurantee)

#### storage interface
存储层提供与put/get/verify相关的低水平接口：
1. read(s, k) --> v 读取分片s上k的值
2. write-async(s, k, v) --> tx-id 把k和v写到分片s上，异步操作，返回事务id
3. check-tx-status(s, tx-id) --> TX-STATUS, 检查事务的状态，有：COMMITED 写成功、ABORTED 事务失败、PENDDING 事务还没有被添加到区块链的区块中
4. get-writeset(s, e) --> ws 返回分片s上epoch e上所有被执行的写，用于offline verification

前三个函数由数据库层调用，用于提供一致性保证。

#### backend connector
![](https://raw.githubusercontent.com/adolphlwq/osshub/master/oss/blog/2019/12/bcdb_func_call.png)

如上图，存储层提供的函数都由backend connector实现，backend connector向下屏蔽所有区块链的技术细节，向上提供统一的函数接口：
1. write-async(s, k, v) 连接器根据s查找分片连接信息，然后把k v转换成字节表示，这样能节约存储消耗、增加区块链处理事务的速度。
2. read(s, k) 该方法不依赖区块链事务，速度很快
3. check-tx-status(s, tx-id)

### Off-Chain Verification
off-chain分为online和offline，这里的**off-chain**强调是BlockchainDB提供的验证机制。

#### Online
![](https://raw.githubusercontent.com/adolphlwq/osshub/master/oss/blog/2019/12/bcdb_online.png)

**在线验证中，每个client发起的操作都是顺序执行**。

get验证过程：
1. 存储层提供获取shard number的接口
2. 根据number获取大多数节点上的数据，对比验证即可

put验证过程：
1. put是有事务的，通过查询大多数区块链put事务的最新状态来确认
2. 如果事务没有在大多数peer上被记录为committed，则认为该put事务被丢弃
3. 不需要验证block的内容，因为内容是被client签名的，client只有被认证了才能加入到BlockchainDB中

#### Offline
![](https://raw.githubusercontent.com/adolphlwq/osshub/master/oss/blog/2019/12/bcdb_offline.png)

在线验证可以立刻知道结果，但是存在缺陷。首先，online是同步过程，结果没返回前会阻塞client其它操作；其次，吞吐量低，事务以分组的形式存储在区块中，online没有利用这个特点，如果分批执行的话可以提高吞吐量。

**离线验证的核心思想是不立刻执行验证，而是分批执行验证**。

#### 参数
![](bcdb_epoch_offline.png)
结合图6，介绍|e|和delta-e的概念：
1. |e|：表示每个epoch队列的大小
2. delta-e：epoch满以后，每个shard未验证的epoch数量

|e|和delta-e能够影响验证的速度和一致性，可以用来调优BlockchainDB的性能：
1. |e|>=一个区块(block)能够处理的事务数，这是理想值。如果小于，执行一次验证的事务数达不到区块处理的事务能力，吞吐量小，没有充分利用计算能力
2. delta-e=0，队列中的epoch处于“关闭”状态后必须立刻进行验证，验证结束才能接受新的写。delta-e可以用来调节不同peer间的进度“倾斜”，比如运行快的peer达到delta-e时会阻塞，这时慢的peer可以赶上来
3. 只有验证过程是分批进行的，其它put/get操作不是分批执行的。离线验证不会增加单个put/get的延迟，但是会影响吞吐量：
    - |e| delta-e太小，吞吐量有负面影响
    - |e| delta-e太大，shard较长时间处于unverified状态，检查出问题的时间增加

## 实验评估
测试集和方法基于[YCSB标准](https://dl.acm.org/citation.cfm?id=1807152)，目的是评估不同技术对BlockchainDB性能的影响，并展示BlockchainDB允许应用改变参数调优。

### 环境和参数
| Parameter   |  Description |
| :--------: | :--------:|
| shardCount  | 分片数量 |
| repFactor | 副本数量 |
| consistency  | 一致性策略 |
| numPeers  | peer数量 |
| numClients  | client数量 |
| workload  | 负载 |
| opsCount  | 操作数量 |

1. Java8实现BlockchainDB原型，底层使用Ethereum(geth v1.8.23-stable)
2. Azure 16vcpu 32g mem Ubuntu16.04 LTS
3. 测试共分两部分，第一部分测试性能，关闭验证功能。第二部分测试验证功能的影响、不同验证策略的影响
4. 详细参数见table 1，采用控制变量法。repFactor=numPeer/shardCount，opsCount=numClient*ops。

### 实验一
实验过程：  
1. 采用顺序一致性
2. 操作中100%write  0%read
6. repFactor=numPeer/numShard
2. 每个peer配一个client
1. 先建表，然后插入4000条数据
4. 每个client执行1000次操作
7. 对于相同peer，调整shard和replica

![](https://raw.githubusercontent.com/adolphlwq/osshub/master/oss/blog/2019/12/bcdb_exp1.png)

结果分析：
1. 分片(shard)和副本(replica)会影响吞吐量（性能）和信任，绿线和蓝线代表的是两个极端情况
    - 绿线：只有1个分片，每增加一个peer，会在上面增加一份分片的副本，所以性能不高，但是信任有保证，数据很难丢失
    - 蓝线：只有一份副本，每增加一个peer，会在上面增加一个分片，所以吞吐量高，但是信任没有保证，peer挂了数据就没了
2. 可以根据测试结果在吞吐量和信任之前寻找折中/平衡，比如4分片 4副本

### 实验二
实验过程：
1. peer数量为16不变、副本为4不变，分片数量从1-16，每个分片插入4000条数据
2. numClient=16 each client send 2000 ops
3. opsCount=16*2000 = 32000
4. 100%write 0% read
5. 顺序一致性

![](https://raw.githubusercontent.com/adolphlwq/osshub/master/oss/blog/2019/12/bcdb_exp2.png)

结果分析：吞吐量线性增加，因为shard增加，增加了并发能力。


### 实验三
![](https://raw.githubusercontent.com/adolphlwq/osshub/master/oss/blog/2019/12/bcdb_exp3a.png)

实验3a：测试不同一致性策略下read延迟，结果分析：
1. 最终一致性不受影响
2. 顺序一致性中，读比重越大，延迟越低，因为阻塞的写少

![](https://raw.githubusercontent.com/adolphlwq/osshub/master/oss/blog/2019/12/bcdb_exp3b.png)

实验3b：值新旧程度的上下界限。结果分析：
1. queue越小（数据越新鲜），读延迟越大
2. queue 900时，读延迟依然在20s，这也侧面验证区块链处理事务速度确实低

### 实验四
离线验证中不同参数的影响

![](https://raw.githubusercontent.com/adolphlwq/osshub/master/oss/blog/2019/12/bcdb_exp4b1.png)

实验4b1：
1. delta-e固定，分片中每次只能有1个未验证的epoch
2. 每张表只有一个分片

结果分析：
1. 绿线表示只有一个peer，replica factor=1，epoch zise小于70时，吞吐量不断增加，到达70后增速变慢甚至下降。可以得出70是区块链底层block的事务处理能力
2. 蓝线有两个peer时，请求被分发到两个节点，所以吞吐量比一个peer高

![](https://raw.githubusercontent.com/adolphlwq/osshub/master/oss/blog/2019/12/bcdb_exp4b2.png)

实验4b2：
1. 2个peer，一个速度快、一个速度慢
2. skew=x，表示速度慢的peer验证事务数平均落后速度快的peer x个区块
3. |e|=70，之前测试的底层区块链处理事务的上限，delta-e取值从1到20
4. 运行10次取平均值，折线图里还有标准差

结果分析：
1. delta-e和skew越大，整体吞吐量越高
2. delta-e需要根据skew的值来设置，当skew=14、delta-e=14效果最好，skew=4、delta-e=8效果最好
3. delta-e小于skew，fast peer会被slow peer拉慢，整体吞吐量变小

## 总结与趋势分析

### 相关工作
![](https://raw.githubusercontent.com/adolphlwq/osshub/master/oss/blog/2019/12/bcdb_related_work.png)

业界关于区块链和数据库的工作主要集中在三个方面：可验证数据库(Verifiable Database)、可扩展区块链(Scalable Blockchain)和分布式数据库(Distributed Database)，最后一个论文不讨论。

可验证数据库，主要是让数据库和表可以验证和共享，有些论文提出把数据存储在传统数据库中，把数据的摘要(digest)存储在底层区块链中。

可扩展区块链，该领域主要讨论如何提高区块链的扩展性和性能。有些论文提出在区块链中实现分片功能一部分，或者向传统数据库中添加区块链的功能（比如BigChainDB）。

### 总结
BlockchainDB是在区块链之上实现的数据库，通过底层区块链原生机制保证数据可共享、可验证，它为用户提供了简单易用的抽象层，降低了使用复杂度，同时提高了区块链的性能，解决了**数据共享读写**场景下的存在的问题。

## Reference
- [BlockchainDB - A Shared Database on Blockchains](https://dl.acm.org/citation.cfm?id=3360366)