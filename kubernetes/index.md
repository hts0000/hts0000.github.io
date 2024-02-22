# Kubernetes


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

### Kubernetes是操作系统

### 编排对象

K8S中，编排对象分为Pod（需要持续提供服务的应用）、Job（只运行一次的任务）、CronJob（定时运行的任务），这些对象用于描述你试图管理的应用

### 服务对象

K8S中，服务对象用于描述平台级的服务，比如Service（Pod的代理）、Secret（信息加密）、Horizontal Pod Autoscaler（自动水平扩展器）等。

由下图可以看到，K8S首先抽象出了需要编排的对象—Pod，再根据Pod运行时需要的服务，声明了一系列的API对象（服务对象），这就是所谓的声明式API。

![image-20201202104037532](https://gitee.com/hts0000/my-images/raw/master/img/20201202111315.png)



### Pod

#### 一般来说，Pod启动过程如下：

1. 调度到某台机器。kubernetes根据一定的优先级算法选择一台机器将其作为pod运行的机器
2. 拉取pod资源文件中指定的镜像
3. 挂载存储配置等
4. 运行pod，进行健康检查



#### pod 的状态有：

- `Pending`: 一般表示还没有开始调度到某台机器上。如果没有符合条件的主机，就会一直处于 Pending 状态
- `Running`: 运行中
- `Succeeded`：有些 pod 不是长久运行的，比如 cronjob,一段时间就结束了。需要反馈任务执行的结果
- `Failed`：pod 的 container 异常退出，比如 command 写的有问题
- `Unknown`：未知。比如 pod 所在的机器无法连接
- `podIP`: pod 所分配到的 ip, 这个 ip 是全集群唯一的
- `qosClass`: 资源分配相关，在后面小节介绍
- `startTime`: 启动时间



### kubeadm单机部署

1. 安装kubeadm、kubectl和kubelet
2. 初始化当前节点为master节点（初始化完成后会生成token，用于后续node节点接入集群）
3. 配置$HOME/.kube/config文件
4. 将当前节点标记为可用节点（master默认Taints为不可调度，需要修改或者删除Taints）
5. 安装网络插件flannel（需要flannel.yml文件）
6. 查看各个pod运行状态kubectl get pods -n kube-system
7. 查看master节点是否运行正常kubectl get node

[kubeadm单机部署K8S参考](https://www.freeaihub.com/kubernetes/setup.html)



### Resource

Resource 是 Kubernetes 的一个基础概念，kubernets 用 Resource 来表示集群中的各个资源。比如`Pod`节点（主机）、`Container`容器、`Proxy`路由、`Config`配置文件等等。这些概念听起来差别很大，但是却有很多共同的基本属性。比如名称、创建时间、标签、uuid 等。kubernetes 对这些信息进行抽象，提供了一个通用的 metadata 结构，再加上各个 resource 自己的其它信息，构成了一个形式类似的数据结构。

#### ConfigMap

很像环境变量或者很多软件的配置文件，存储了一些 key-value 键值对。



#### Pod

Pod是一组container的集合，集合内的container默认共享Network NameSpace和Mount NameSpace，也支持配置其他资源进行共享。



### kubectl基本使用

#### 查看

kubectl配置文件存放路径：~/.kube/config

kubectl cluster-info：查看K8S集群的版本和基本信息

kubectl get ns：查看所有namespace

kubectl get namespace kube-system -o yaml：查看某一个namespace，以yaml方式输出

kubectl get configmap cluster-info -n kube-system -o yaml：查看namespace为kube-system的cluster-info的配置文件，以yaml方式输出

kubectl get node master -o yaml：查看节点的信息

kubectl get pod pod名称 -o yaml：查看pod的信息

kubectl get pod pod名称 -w：持续监控pod的信息

kubectl logs --tail 10 pod-sample：查看pod名为pod-sample的日志信息

kubectl logs --tail 10 pods/etcd-node1 --namespace kube-system：查看namespace为kube-system中pod名为pods/etcd-node1的日志，只看后10行





#### 增加

kubectl create namespace 资源名称：创建一个namespace

kubectl create ns 资源名称-v=9：创建一个namespace，并显示创建过程日志，9表示日志级别，数字越大日志越详细

```shell
root@freeaihub:~# kubectl create ns test1 -v=9
# kubectl载入本地配置文件
I1203 02:57:29.070303   32654 loader.go:375] Config loaded from file:  /root/.kube/config
# 根据命令行参数生成合适的body发送给apiserver
I1203 02:57:29.078742   32654 request.go:1068] Request Body: {"apiVersion":"v1","kind":"Namespace","metadata":{"creationTimestamp":null,"name":"test1"},"spec":{},"status":{}}
# https请求调用api
I1203 02:57:29.078950   32654 round_trippers.go:423] curl -k -v -XPOST  -H "Accept: application/json" -H "User-Agent: kubectl/v1.18.2 (linux/amd64) kubernetes/52c56ce" 'https://172.87.64.11:6443/api/v1/namespaces'
I1203 02:57:29.099372   32654 round_trippers.go:443] POST https://172.87.64.11:6443/api/v1/namespaces 201 Created in 20 milliseconds
I1203 02:57:29.099422   32654 round_trippers.go:449] Response Headers:
I1203 02:57:29.099548   32654 round_trippers.go:452]     Content-Type: application/json
I1203 02:57:29.099650   32654 round_trippers.go:452]     Content-Length: 455
I1203 02:57:29.099671   32654 round_trippers.go:452]     Date: Thu, 03 Dec 2020 02:57:29 GMTI1203 02:57:29.099794   32654 request.go:1068] Response Body: {"kind":"Namespace","apiVersion":"v1","metadata":{"name":"test1","selfLink":"/api/v1/namespaces/test1","uid":"51c4e17b-a5fc-4dfb-93f1-07a701997a6a","resourceVersion":"6720","creationTimestamp":"2020-12-03T02:57:29Z","managedFields":[{"manager":"kubectl","operation":"Update","apiVersion":"v1","time":"2020-12-03T02:57:29Z","fieldsType":"FieldsV1","fieldsV1":{"f:status":{"f:phase":{}}}}]},"spec":{"finalizers":["kubernetes"]},"status":{"phase":"Active"}}
# 创建成功
namespace/test1 created
```

kubectl create -f 资源文件路径：更常用的创建资源的方式，在资源文件中编辑好需要创建的资源类型（kind）、元数据（metadata）、资源名称（name）。

kubectl create configmap 资源名称 --from-literal=a=a --from-literal=b=b -n default -v=9：创建一个configmap资源，创建步骤与namespace相似。



#### 删除

kubectl delete cm 资源名称 -n default：删除default namespace中指定的configmap资源

kubectl delete ns 资源名称：删除default namespace中指定的configmap资源



#### 操作pod和容器

kubectl exec -it pod名称 执行的命令

kubectl exec -it pod-sample sh

kubectl exec -it pod-sample -c pod-sample-container -- sh：进入pod-sample pod中的pod-sample-container容器，执行命令为sh

### K8S的调度算法


### 应用部署在K8S上的流程

1. 开发应用
2. 打包需要上线的应用，制作成镜像，提交到Docker Registry
3. 选择应用合适的K8S API对象（Pod、Deployment等），编写相应的YAML文件
4. 使用kubectl创建该对象
5. 后续滚动升级等操作都使用kubectl和更新YAML文件的方式来维护
