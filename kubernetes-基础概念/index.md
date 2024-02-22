# Kubernetes 基础概念


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

## Pod
### 自主式Pod
### 控制器管理的Pod
下面是所有控制器
#### ReplicationController(RC)
用来确保容器副本数始终保持在用户定义的副本数

#### ReplicaSet(RS)
RS在RC的基础上增加了集合式选择器，根据不同RS的标签，来进行集合操作（交集、并集等）

#### Deployment
Deployment运行在RS之上，支持滚动更新和回滚

#### HPA(HorizontalPodAutoScale)
处于RC、RS、Deployment之上，根据CPU或其他用户自定义的指标来自动扩缩容

#### StatefullSet
用于解决有状态服务。用过稳定的持久化存储、稳定的网络标志、有序部署、有序扩展、有序收缩和有序删除来保证有状态服务在Pod中的运行

#### DaemonSet
用于保证集群的NODE节点中都运行这至少一个Pod副本，用于保证集群中的Pod高可用

经常用于ceph等存储系统中

#### Job
负责批处理任务，仅执行一次的任务，保证批处理任务的一次或多次成功结束

#### Cronjob
定时周期的执行Job

#### 自定义控制器
kubernetes二次开发的重要内容，自己开发适合自己的控制器

## 网络
主要是解决集群中不同物理机之间pod通信的问题
### 网络通讯方式

同pod间通信
- 通过回环地址

不同pod但同物理机通信
- 利用docker网桥

不同物理机通信
- flannel网络插件

#### flannel
基本原理是利用etcd集群，存储一个网段，每个pod生成时flannel都通过请求这个网段生成全集群唯一的虚拟IP

flannel对udp数据包做了二次封装，数据包的内容是flannel实现的协议数据包，etcd中存储了虚拟IP到实际IP的映射，可以直接找到虚拟IP对应的实际IP是谁

![](https://cdn.jsdelivr.net/gh/hts0000/images/202202262318287.png)

## kubernetes组件
![](https://cdn.jsdelivr.net/gh/hts0000/images/202202271556249.png)
kubectl

schedule

etcd

kubecontroller

kubelet

kubeadm

apiserver

## kubernetes部署
master要求：
- cpu >= 2核
- 内存 >= 4G
- 操作系统内核版本要求4.x(node机器也是)
### 二进制部署

### kubeadm容器化部署
先下载kubeadm

所有机器都需要安装docker-ce

需要拉取的镜像在国外，下载很慢，可以先拉取对应的镜像，导入到本地再执行kubeadm

容器化方式安装的kubernetes有证书时限限制，一年后失效，集群无法工作

解决方法：
- 升级集群可以重置证书时间
- 修改kubeadm源码中颁发证书时的时间限制

容器化部署的kubernetes有自愈性，如果kubernetes的组件异常了，可以再启动一个新的组件

## kubernetes概念
k8s中所有内容都抽象为资源，资源实例化之后叫做对象

所有组件都通过yaml格式的文件描述，称为一个资源，k8s可以通过资源文件的描述，实例化一个对应的pod

### 资源分类
#### 名称空间级别
工作负载型资源：
- Pod
- RS
- Deployment

服务发现和负载均衡型资源：
- Service
- Ingress

配置与存储型资源：
- Volume
- CSI

特殊类型存储卷：
- ConfigMap
- Secre

#### 集群级资源
- NameSpace
- Node
- ClusterRole
- ClusterRoleBinding

#### 元数据型资源
- HPA
- PodTemplate
- LimitRange

## 资源清单
通常使用yaml格式的文件来编写

kubectl create -f 指定资源文件生成实例，生成时kubectl会把yaml格式转化成json发送给apiserver

kubectl可以装在任何地方，只要能通过config配置文件找到k8s集群即可

资源清单文件基本格式：
```yaml
apiVersion: group/apiversion  # 如果没有给定 group 名称，那么默认为 core，可以使用 kubectl api-versions # 获取当前 k8s 版本上所有的 apiVersion 版本信息( 每个版本可能不同 )
kind:       #资源类别
metadata：  #资源元数据
   name
   namespace
   lables
   annotations   # 主要目的是方便用户阅读查找
spec: # 期望的状态（disired state）
status：# 当前状态，本字段有 Kubernetes 自身维护，用户不能去定义
```

kubectl实时获取资源信息：
kubectl get pods -o yaml

所有资源信息都存储在etcd中，kubectl是通过请求apiserver，apiserver再请求etcd来获取的资源信息
