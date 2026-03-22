# 一、核心高频命令 (Top 5)

| 命令                 | 功能说明                                           | 示例                                                                                                |
| ------------------ | ---------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `kubectl get`      | **列出资源**。查看 Pod、Service、Deployment 等列表。        | `kubectl get pods` (查看所有 Pod)<br>`kubectl get nodes` (查看节点状态)<br>`kubectl get all` (查看当前命名空间所有资源) |
| `kubectl describe` | **查看详情**。当 Pod 启动失败或状态异常时，用来查看具体的报错事件（Events）。 | `kubectl describe pod <pod-name>`                                                                 |
| `kubectl logs`     | **查看日志**。排查应用内部报错（如 Java 堆栈报错）。                | `kubectl logs -f <pod-name>` (实时查看日志)<br>`kubectl logs <pod-name> -c <container-name>` (多容器时指定容器) |
| `kubectl exec`     | **进入容器**。类似 SSH 登录到容器内部进行调试。                   | `kubectl exec -it <pod-name> -- /bin/bash`                                                        |
| `kubectl apply`    | **应用配置**。根据 YAML 文件创建或更新资源（声明式）。               | `kubectl apply -f deployment.yaml`                                                                |
`kubectl attach -it nginx -c shell` 的核心是：**连接到 Kubernetes 集群中名为 `nginx` 的 Pod 里、名为 `shell` 的容器，以交互式终端的方式进入容器内部**。
**和 `kubectl exec -it nginx -c shell /bin/bash` 的区别**：

- `attach`：连接到容器的**主进程**（PID 1），如果主进程退出，你的终端也会断开；
- `exec`：在容器内**新建一个进程**执行命令（比如 bash），更灵活，是日常进入容器的首选。
- 简单说：`exec` 更好用，`attach` 多用于调试主进程。
# 二、故障排查与调试 (Troubleshooting)
当你发现 `get pods` 显示的状态不是 `Running` (比如 `CrashLoopBackOff` 或 `Pending`) 时，使用这些命令：
- **查看最近的启动事件 (非常重要)**：
```
# 查看 Pod 为什么起不来，重点看输出底部的 "Events" 部分
kubectl describe pod <pod-name>
```
- **查看上一次崩溃的日志**：
```
# 如果容器一直在重启，查看它上一次死掉前的日志
kubectl logs <pod-name> --previous
```
- **查看集群事件**：
```
# 按时间顺序列出集群内发生的所有事件（Warning/Normal）
kubectl get events --sort-by='.metadata.creationTimestamp'
```
# 三、创建与删除 (Create & Delete)
- **快速创建一个 Pod (测试用)**：
```
# 启动一个 nginx 实例
kubectl run nginx --image=nginx
```
- **创建 Deployment (生产常用)**：

```
kubectl create deployment my-dep --image=nginx --replicas=3
```
- **删除资源**：
```
kubectl delete pod <pod-name>
kubectl delete -f deployment.yaml  # 根据文件删除
```
# 四、扩缩容与更新 (Scaling & Updating)
- **扩容/缩容 Pod 数量**：
```
# 将副本数调整为 5
kubectl scale deployment/my-dep --replicas=5
```
- **强制重启 Pod (技巧)**： K8s 没有直接的 `restart` 命令，通常通过 rollout 也就是滚动更新来实现：
```
kubectl rollout restart deployment/my-dep
```
- **查看滚动更新状态**：
```
kubectl rollout status deployment/my-dep
```
# 五、实用技巧 (Pro Tips)
**1. 常用缩写 (Aliases)** K8s 资源类型都有简写，可以少打很多字：
- `pods` -> `po`
    
- `services` -> `svc`
    
- `deployments` -> `deploy`
    
- `namespaces` -> `ns`
    
- `nodes` -> `no`
    
**示例：** `kubectl get svc` 等同于 `kubectl get services`。
**2. 试运行 (Dry Run)** 在不实际执行命令的情况下，生成 YAML 文件模板：
```
# 生成一个 pod 的 yaml 结构并打印出来，但不创建 pod
kubectl run nginx --image=nginx --dry-run=client -o yaml > my-pod.yaml
```
对于这条命了的解释：
假如 K8s 是一个装修队，这行命令的对话是这样的：
	1. **`kubectl run nginx --image=nginx`**:
    > 	“老板，我要建一个叫 Nginx 的房子，图纸参考 Nginx 官方标准。”
	2. **`--dry-run=client`**:
    > “但是！**先别动土**，我们在纸上画个草图演练一下就行。”
	3. **`-o yaml`**:
    > “这个草图，请用标准的 YAML 格式画给我看。”
	4. **`> my-pod.yaml`**:
    >“画好之后，把这张纸存到我的文件夹里，名字叫 my-pod.yaml。”
**3. 切换命名空间 (Namespace)** 如果不加 `-n` 参数，默认都是在 `default` 命名空间操作。
```
# 查看特定命名空间的资源
kubectl get pods -n kube-system

# 列出所有命名空间的资源
kubectl get pods --all-namespaces (或 -A)
```
# 本地调试命令
# 一、本地调试神技 (Debugging & Access)
这两个命令是开发人员的最爱，因为它能让你不通过复杂的网络配置（如 Ingress），直接从本地电脑访问集群内部。

| **命令**                 | **功能说明**                                                      | **示例**                                                                                                                 |
| ---------------------- | ------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `kubectl port-forward` | **端口转发**。将集群内的服务端口映射到你本地电脑（localhost）。调试数据库或内部 Admin 面板时极其有用。 | `kubectl port-forward pod/my-db 5432:5432`<br>(现在你可以用本地客户端连接 localhost:5432 来访问集群里的 DB)                                |
| `kubectl cp`           | **文件拷贝**。在你的电脑和容器之间复制文件，用法类似 Linux 的 `scp`。                   | **拷入：** `kubectl cp ./local-file.txt <pod-name>:/tmp/`<br>**拷出：** `kubectl cp <pod-name>:/var/log/app.log ./local.log` |







## 二、在线修改与配置 (Live Editing & Config)
- **直接编辑运行中的配置**： 如果你想快速修改一个 Deployment 的配置（比如改个环境变量），不需要找原始 YAML 文件，可以直接由集群反解出来编辑：
```
# 会自动打开 vim/nano 编辑器，保存退出后 K8s 会自动更新资源
kubectl edit svc <service-name>
```
- **查看文档 (内置说明书)**： 记不住 YAML 里的字段层级？比如 `spec.containers.livenessProbe` 下面有哪些参数？
```
kubectl explain pod.spec.containers.livenessProbe
```
- **切换集群/环境 (Config)**： 如果你同时管理开发环境和生产环境，这个命令必不可少：
```
# 查看当前上下文（连接的是哪个集群）
kubectl config current-context

# 切换上下文（例如从 dev 切换到 prod）
kubectl config use-context prod-cluster
```
## 三、性能与资源监控 (Monitoring)
_前提：集群需要安装 metrics-server 组件。_
- **查看资源占用**： 想知道哪个 Pod 占用了最多的 CPU 或内存？
```
kubectl top pods
kubectl top nodes
```
## 四、 节点维护 (Node Maintenance) - **管理员专用**
如果你是集群管理员，需要对某台物理机/虚拟机进行关机维护，你需要先驱逐上面的 Pod。

1. **禁止调度 (Cordon)**：让新 Pod 别往这台机器上跑。
```
kubectl cordon <node-name>
```
2. **驱逐现有 Pod (Drain)**：把这台机器上的 Pod 安全地迁移到其他机器。
```
kubectl drain <node-name> --ignore-daemonsets
```
3. **恢复调度 (Uncordon)**：维护完后，恢复机器接客。
```
kubectl uncordon <node-name>
```
## 五、格式化输出 (高级技巧)
默认的 `kubectl get` 显示的信息有限。使用 `-o` (output) 参数可以提取任何你想要的信息。
- **显示更多信息 (IP, Node)**：
```
kubectl get pods -o wide
```
- **导出为 YAML** (常用于备份配置)：
```
kubectl get deployment my-dep -o yaml > backup.yaml
```
- **精准提取字段 (JSONPath)**： 比如，只想要获取所有 Pod 的 IP 地址：
```
kubectl get pods -o jsonpath='{.items[*].status.podIP}'
```
