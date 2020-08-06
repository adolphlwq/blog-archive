# Kubernetes是如何调度Pod的？
首先要介绍下Kubernetes(后面简称k8s)的架构。

在k8s集群中，分为Master节点和Node节点，Master节点是整个集群的控制中心，上面运行kube-apiserver、kube-controller-manager和kube-scheduler三个进程。master是整个集群的控制节点，所有的控制命令都发给master，由它负责具体执行。

Node节点是相对于Master的概念，是集群的工作负载节点，上面运行了kubelet、kube-proxy和docker等进程。

k8s集群是一个高度自动化的集群，把资源、pod等结构抽象为`资源`存储在etcd中，每个资源在k8s中都有稳定的状态。当我们操作变更资源的状态是，k8s会变更etcd中对应的状态，Node节点上的kubelet检查到状态变更后，会自动地改变相关资源的状态和etcd一致。

![](res/kube-schedule.jpg)

k8s的调度主要由master上的kube-scheduler负责，调度过程主要分为3步：
1. 产生待调度Pod列表，当用户通过kube-apiserver创建pod或者kube-controller-manager更改pod数量产生新pod的，就会产生带调度的pod列表
2. kube-scheduler根据待调度的pod信息，通过调度算法进行预选，再通过调度策略进行优选，在可选的Node中选择最合适的Node，将状态信息写入etcd集群
3. Node节点上的kubelet更新pod的状态

这样看来调度还是不难的(太天真了...)，还需要通过源代码进一步了解