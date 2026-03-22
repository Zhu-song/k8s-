# 一、先搞懂三个角色（一句话）

- **PV（PersistentVolume）**：集群里的**一块存储空间**（硬盘）
- **PVC（PersistentVolumeClaim）**：Pod 对存储的**申请单**
- **StorageClass（SC）**：**自动创建 PV 的模板**
# 二、PV/PVC是什么关系？
- PV 是**资源**
- PVC 是**请求**
- 系统会**自动匹配绑定**（大小、权限、存储类）

> 比喻：
> 
> **PV = 房子
> 
> PVC = 租房需求
> 
> StorageClass = 房产中介（自动盖房子）**
# 三、PV 的回收策略（面试必问）
- **Retain**
    
    保留数据，PVC 删了 PV 还在，数据不删
- **Recycle**
    
    清空数据（旧版本，基本不用）
- **Delete**
    
    删除 PV，数据一起删（云存储常用）
# 四、StorageClass 作用？

- **动态供给 PV**
- PVC 一提交，自动创建 PV
- 不用运维手动建 PV
# 五、PV 访问模式（常考）

- **ReadWriteOnce（RWO）**
    
    只能被**一个节点**挂载读写
- **ReadOnlyMany（ROX）**
    
    可以被**多节点**只读
- **ReadWriteMany（RWX）**
    
    可以被**多节点**读写
# 六、常用命令
```bash
kubectl get pv
kubectl get pvc
kubectl get sc
```
# 七、PV/PVC 高频面试 12 题（精简答案）
## **1.PV 和 PVC 是什么？**
-  PV：集群的存储卷
- PVC：Pod对存储的申请
## **2.为什么要用PV/PVC？**
- 让应用不关心底层存储
- 解偶：应用只管要用多少，运维管怎么存
## **3.PVC 状态一直 Pending 为什么？**
- 没有匹配的 PV
- StorageClass 不对
- 存储插件 / 磁盘有问题
## **4.PV 和 PVC 是 Namespace 资源吗？**
- **PV 是集群级别（不属于任何 ns）**
- **PVC 是命名空间级别**
## **5.PV 三种回收策略？**
Retain（保留）、Recycle（清空）、Delete（删除）
## **6.StorageClass 作用？**
**动态创建 PV**，不用手动管理
## **7.什么是静态供给、动态供给？**
- 静态：运维**提前建好 PV**
- 动态：通过 StorageClass **自动创建 PV**
## **8.PV 访问模式 3 种？**
RWO、ROX、RWX
## **9.PVC 删除后数据会丢吗？**
看 PV 回收策略：

- Retain：**不丢**
- Delete：**丢**
## **10.Pod 如何使用存储？**
Pod 挂载 PVC → PVC 绑定 PV → 真正使用存储
## **11.PV 能被多个 PVC 绑定吗？**
不能，**一对一绑定**
## **12.Deployment 能用 PVC 吗？**
可以，但必须是**RWX 或副本 = 1**

多副本同时读写要用 **RWX 存储**

# 超级一句话总结

**PV 是硬盘，PVC 是申请单，

StorageClass 自动创建硬盘，

Pod 用 PVC 挂载存储。**
