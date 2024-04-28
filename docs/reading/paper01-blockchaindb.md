# BlockchainDB - 构建于区块链之上的分片数据库

> 2019.12.1

![](https://raw.githubusercontent.com/alwqx/picx-images-hosting/master/blog/2019/12/bcdb_arg.png)

这篇论文主要分为 9 部分，其中 1 2 节介绍 BlockchainDB 要解决的问题，3 4 5 6 四节介绍了 BlockchainDB 的架构和技术细节，7 节对其性能进行评估，最后总结展望。

## 问题及挑战

区块链技术的兴起催生了很多新的应用场景，一个非常重要的场景是：**互不信任的多方实现`数据共享读写`**，比如供应链上的货物追踪。但是，原生区块链存在如下缺陷：

1. 区块链的性能和扩展性存差：区块链的事务处理能力在 10-100 tx/s
2. 缺少易于使用的抽象层：区块链没有提供像数据库那样简单方便的查询接口、一致性等功能

上面两个主要缺陷极大阻碍了区块链在`数据共享读写`场景的使用和推广。为了解决上面的问题，这篇论文基于区块链开发了 BlockchainDB，它主要有以下优势：

1. 把区块链作为存储层(Storage Layer)，不需要修改底层区块链代码，同时充分利用其去中心化、tamper-proof log 等原生技术优势
2. 提供易用的查询接口和一致性保证
3. 提高区块链的性能、降低使用复杂度

## 具体实现

### 架构

![](https://raw.githubusercontent.com/alwqx/picx-images-hosting/master/blog/2019/12/bcdb_network.png)

上图是 BlockchainDB 的架构图，自下而上分为存储层和数据库层，数据存储在底层区块链上，通过 Client 提供简单易用的 put/get/verify 接口。

peer 指区块链里面的多个节点，常用于区块链场景，类比分布式数据库中的 node。分为 fully peer 和 thin peer，前者存储数据、资源要求高，后者不存储数据，资源要求低。

需要注意的是，每个 party 在加入 BlockchainDB 网络时需要进行认证并且做到彼此知晓，认证通过后才能加入。

### 约定

为了方便后续讨论，这里做如下约定：

1. 加入 BlockchainDB 网络的 peer 需要认证，认证通过后才能加入
2. peer 可以代表本地所有的 client 进行验证操作
3. peer 信任本地读写的数据
4. 通过**大多数**进行认证，如果大多数 peer 上的数据是可信的，那么数据是可信的

基于上述约定，在进行验证时，主要验证下面两个方面:

1. put 操作是否被恶意丢弃
2. get 得到的数据真假

### Database Layer

![](https://raw.githubusercontent.com/alwqx/picx-images-hosting/master/blog/2019/12/bcdb_layer_arc.png)

数据库层为用户提供 client，封装了 put/get/verify 接口：

- get(t, k) --> v，返回表 t 对应主键 k 的值
- put(t, k, v) --> void 所有数据被编码到 v 中，作为文档插入表 t 中，主键为 k
- verify() --> bool 该方法用于在线验证，put/get 后调用 verify()
  - put：插入操作(事务)是否 commit
  - get：返回的结果是否正确、真实

如图 3 所示：client 读写请求先到 off-chain verifer，然后放到请求队列中。由事务管理(Tx Mgr)和分片管理(Shard Mgr)处理后转发到对应的底层区块链上，最后返回结果。

需要注意，Backend Connector 用于连接底层不同的区块链。

### 一致性

针对目前提供的功能，论文介绍了`get`和`put`的一致性保证，put 操作的事务处理要复杂些，get 相对简单。

##### Processing Model in Blockchain

区块链是如何处理写数据的，它采用“a global order of write”比如 blockchain transaction，即所有的写会根据 replica 附加到底层的 log 中。因此区块链自身的机制就支持 sequential consistency，但是区块链的事务处理速度很慢，一个事务往往花费数秒到数分钟，这严重影响写密集型负载的吞吐量。

另外，有些区块链存在特殊的机制，比如 Ethereum 中的 fork 机制，它会重新执行事务，需要额外的时间。

#### 顺序一致性(Sequential Consistency)

区块链能够保证全局写的顺序，但是处理速度慢，这里没有利用区块链的机制。顾名思义，顺序一致性(Sequential)保证时间上的先后关系，每个写操作按顺序进行事务提交。这是一个同步执行过程。

#### 最终一致性(Eventual Consistency)

最终一致性(Eventual Consistency)是异步执行过程，put 操作后不等待事务结果。因此 put 执行后 BlockchainDB 不会立刻进入一致性状态，但等一会后会达到一致性。

为了协调一致性和执行速度（吞吐量），BlockchainDB 提供了参数用于调优，下文会详细介绍。

### Storage Layer

#### shared table

“数据共享读写”场景首先从数据库建表开始，这里将介绍表的定义和创建过程。

shared table 数据结构是用来存储数据的，它包括表模式(schema)和分片配置(sharding configuration)：

1. 表的 schema 通过 key/value 数据模型来定义，key 表示主键(primary key)，value 表示数据(payload of a tuple)
2. 分片配置包括 shard 的数量、replica 数量等。这些参数不仅影响整体性能，还会影响 table 提供的一些保证(gurantee)

#### storage interface

存储层提供与 put/get/verify 相关的低水平接口：

1. read(s, k) --> v 读取分片 s 上 k 的值
2. write-async(s, k, v) --> tx-id 把 k 和 v 写到分片 s 上，异步操作，返回事务 id
3. check-tx-status(s, tx-id) --> TX-STATUS, 检查事务的状态，有：COMMITED 写成功、ABORTED 事务失败、PENDDING 事务还没有被添加到区块链的区块中
4. get-writeset(s, e) --> ws 返回分片 s 上 epoch e 上所有被执行的写，用于 offline verification

前三个函数由数据库层调用，用于提供一致性保证。

#### backend connector

![](https://raw.githubusercontent.com/alwqx/picx-images-hosting/master/blog/2019/12/bcdb_func_call.png)

如上图，存储层提供的函数都由 backend connector 实现，backend connector 向下屏蔽所有区块链的技术细节，向上提供统一的函数接口：

1. write-async(s, k, v) 连接器根据 s 查找分片连接信息，然后把 k v 转换成字节表示，这样能节约存储消耗、增加区块链处理事务的速度。
2. read(s, k) 该方法不依赖区块链事务，速度很快
3. check-tx-status(s, tx-id)

### Off-Chain Verification

off-chain 分为 online 和 offline，这里的**off-chain**强调是 BlockchainDB 提供的验证机制。

#### Online

![](https://raw.githubusercontent.com/alwqx/picx-images-hosting/master/blog/2019/12/bcdb_online.png)

**在线验证中，每个 client 发起的操作都是顺序执行**。

get 验证过程：

1. 存储层提供获取 shard number 的接口
2. 根据 number 获取大多数节点上的数据，对比验证即可

put 验证过程：

1. put 是有事务的，通过查询大多数区块链 put 事务的最新状态来确认
2. 如果事务没有在大多数 peer 上被记录为 committed，则认为该 put 事务被丢弃
3. 不需要验证 block 的内容，因为内容是被 client 签名的，client 只有被认证了才能加入到 BlockchainDB 中

#### Offline

![](https://raw.githubusercontent.com/alwqx/picx-images-hosting/master/blog/2019/12/bcdb_offline.png)

在线验证可以立刻知道结果，但是存在缺陷。首先，online 是同步过程，结果没返回前会阻塞 client 其它操作；其次，吞吐量低，事务以分组的形式存储在区块中，online 没有利用这个特点，如果分批执行的话可以提高吞吐量。

**离线验证的核心思想是不立刻执行验证，而是分批执行验证**。

#### 参数

![](bcdb_epoch_offline.png)
结合图 6，介绍|e|和 delta-e 的概念：

1. |e|：表示每个 epoch 队列的大小
2. delta-e：epoch 满以后，每个 shard 未验证的 epoch 数量

|e|和 delta-e 能够影响验证的速度和一致性，可以用来调优 BlockchainDB 的性能：

1. |e|>=一个区块(block)能够处理的事务数，这是理想值。如果小于，执行一次验证的事务数达不到区块处理的事务能力，吞吐量小，没有充分利用计算能力
2. delta-e=0，队列中的 epoch 处于“关闭”状态后必须立刻进行验证，验证结束才能接受新的写。delta-e 可以用来调节不同 peer 间的进度“倾斜”，比如运行快的 peer 达到 delta-e 时会阻塞，这时慢的 peer 可以赶上来
3. 只有验证过程是分批进行的，其它 put/get 操作不是分批执行的。离线验证不会增加单个 put/get 的延迟，但是会影响吞吐量：
   - |e| delta-e 太小，吞吐量有负面影响
   - |e| delta-e 太大，shard 较长时间处于 unverified 状态，检查出问题的时间增加

## 实验评估

测试集和方法基于[YCSB 标准](https://dl.acm.org/citation.cfm?id=1807152)，目的是评估不同技术对 BlockchainDB 性能的影响，并展示 BlockchainDB 允许应用改变参数调优。

### 环境和参数

|  Parameter  | Description |
| :---------: | :---------: |
| shardCount  |  分片数量   |
|  repFactor  |  副本数量   |
| consistency | 一致性策略  |
|  numPeers   |  peer 数量  |
| numClients  | client 数量 |
|  workload   |    负载     |
|  opsCount   |  操作数量   |

1. Java8 实现 BlockchainDB 原型，底层使用 Ethereum(geth v1.8.23-stable)
2. Azure 16vcpu 32g mem Ubuntu16.04 LTS
3. 测试共分两部分，第一部分测试性能，关闭验证功能。第二部分测试验证功能的影响、不同验证策略的影响
4. 详细参数见 table 1，采用控制变量法。repFactor=numPeer/shardCount，opsCount=numClient\*ops。

### 实验一

实验过程：

1. 采用顺序一致性
2. 操作中 100%write 0%read
3. repFactor=numPeer/numShard
4. 每个 peer 配一个 client
5. 先建表，然后插入 4000 条数据
6. 每个 client 执行 1000 次操作
7. 对于相同 peer，调整 shard 和 replica

![](https://raw.githubusercontent.com/alwqx/picx-images-hosting/master/blog/2019/12/bcdb_exp1.png)

结果分析：

1. 分片(shard)和副本(replica)会影响吞吐量（性能）和信任，绿线和蓝线代表的是两个极端情况
   - 绿线：只有 1 个分片，每增加一个 peer，会在上面增加一份分片的副本，所以性能不高，但是信任有保证，数据很难丢失
   - 蓝线：只有一份副本，每增加一个 peer，会在上面增加一个分片，所以吞吐量高，但是信任没有保证，peer 挂了数据就没了
2. 可以根据测试结果在吞吐量和信任之前寻找折中/平衡，比如 4 分片 4 副本

### 实验二

实验过程：

1. peer 数量为 16 不变、副本为 4 不变，分片数量从 1-16，每个分片插入 4000 条数据
2. numClient=16 each client send 2000 ops
3. opsCount=16\*2000 = 32000
4. 100%write 0% read
5. 顺序一致性

![](https://raw.githubusercontent.com/alwqx/picx-images-hosting/master/blog/2019/12/bcdb_exp2.png)

结果分析：吞吐量线性增加，因为 shard 增加，增加了并发能力。

### 实验三

![](https://raw.githubusercontent.com/alwqx/picx-images-hosting/master/blog/2019/12/bcdb_exp3a.png)

实验 3a：测试不同一致性策略下 read 延迟，结果分析：

1. 最终一致性不受影响
2. 顺序一致性中，读比重越大，延迟越低，因为阻塞的写少

![](https://raw.githubusercontent.com/alwqx/picx-images-hosting/master/blog/2019/12/bcdb_exp3b.png)

实验 3b：值新旧程度的上下界限。结果分析：

1. queue 越小（数据越新鲜），读延迟越大
2. queue 900 时，读延迟依然在 20s，这也侧面验证区块链处理事务速度确实低

### 实验四

离线验证中不同参数的影响

![](https://raw.githubusercontent.com/alwqx/picx-images-hosting/master/blog/2019/12/bcdb_exp4b1.png)

实验 4b1：

1. delta-e 固定，分片中每次只能有 1 个未验证的 epoch
2. 每张表只有一个分片

结果分析：

1. 绿线表示只有一个 peer，replica factor=1，epoch zise 小于 70 时，吞吐量不断增加，到达 70 后增速变慢甚至下降。可以得出 70 是区块链底层 block 的事务处理能力
2. 蓝线有两个 peer 时，请求被分发到两个节点，所以吞吐量比一个 peer 高

![](https://raw.githubusercontent.com/alwqx/picx-images-hosting/master/blog/2019/12/bcdb_exp4b2.png)

实验 4b2：

1. 2 个 peer，一个速度快、一个速度慢
2. skew=x，表示速度慢的 peer 验证事务数平均落后速度快的 peer x 个区块
3. |e|=70，之前测试的底层区块链处理事务的上限，delta-e 取值从 1 到 20
4. 运行 10 次取平均值，折线图里还有标准差

结果分析：

1. delta-e 和 skew 越大，整体吞吐量越高
2. delta-e 需要根据 skew 的值来设置，当 skew=14、delta-e=14 效果最好，skew=4、delta-e=8 效果最好
3. delta-e 小于 skew，fast peer 会被 slow peer 拉慢，整体吞吐量变小

## 总结与趋势分析

### 相关工作

![](https://raw.githubusercontent.com/alwqx/picx-images-hosting/master/blog/2019/12/bcdb_related_work.png)

业界关于区块链和数据库的工作主要集中在三个方面：可验证数据库(Verifiable Database)、可扩展区块链(Scalable Blockchain)和分布式数据库(Distributed Database)，最后一个论文不讨论。

可验证数据库，主要是让数据库和表可以验证和共享，有些论文提出把数据存储在传统数据库中，把数据的摘要(digest)存储在底层区块链中。

可扩展区块链，该领域主要讨论如何提高区块链的扩展性和性能。有些论文提出在区块链中实现分片功能一部分，或者向传统数据库中添加区块链的功能（比如 BigChainDB）。

### 总结

BlockchainDB 是在区块链之上实现的数据库，通过底层区块链原生机制保证数据可共享、可验证，它为用户提供了简单易用的抽象层，降低了使用复杂度，同时提高了区块链的性能，解决了**数据共享读写**场景下的存在的问题。

## Reference

- [BlockchainDB - A Shared Database on Blockchains](https://dl.acm.org/citation.cfm?id=3360366)
