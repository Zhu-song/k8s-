# k8s-
关于k8s的一些基本知识


# K8s有两大组件
Master（大脑）+Node（干活）
## 1）Master组件（控制平面）
### 1.etcd
- 分布式键值存储
- **存集群的所有信息：Pod、service、配置、状态......**
- 相当于K8s的数据库
### 2.Kube-apiserver
- 集群**唯一入口**
- 所有操作都必须走它
- 负责认证、鉴权、校正、操作etcd
### 3.Kube-controller-manager
- 各种控制的大总管
- 保证**期望状态=实际状态**
- 比如：副本少了就拉新、节点挂了就迁移Pod
### 4.Kube-schedluer
- 调度器
- 决定：**这个新Pod应该去哪个Node**
- 看资源、标签、污点、亲和度等
## 2）Node组件（工作节点）
### 1.kubelet
- 节点上的**agent**
- 管本机Pod：创建、启动、监控、销毁
- 跟APIserver汇报
### 2.kube-proxy
- 维护节点上的网络规则
- 实现**Service的负载均衡、访问**
- 模式：iptables/IPVS
### 3.容器运行时（CRI）
- 真正跑容器的：contained、Docker等
- 拉镜像、启停容器
# Pod从敲命令到跑起来，完整流程是什么？
## 1.kubectl提交创建Pod请求→kube-apiserver
## 2.apiserver校验、认证，把Pod信息存入etcd
## 3.kube-scheduler发现有未调度的Pod计算合适节点，把节点名写入etcd
## 4.对应节点的kubelet观察到“有Pod分配给我了”
## 5.kubelet让容器运行时拉镜像、创建容器
## 6.kubelet汇报给apiserver
## 7.Pod运行成功
