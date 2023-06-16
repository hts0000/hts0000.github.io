# Kubernetes必知必会(一)


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

# Kubernetes架构
`K8S`使用CS架构，也就是`Server-Client`架构。客户端通过`WebUI`、`CLI`等工具与`Kubernetes Master`连接下达命令，`Master`再将这些命令下发给对应`Node`进行执行。  
![](https://raw.githubusercontent.com/hts0000/images/main/202306121400078.png)

`K8S Master`包含四个核心组件：
- `API Server`：处理`API`请求，`Kubernetes`中所有组件都会与`API Server`进行连接
- `Controller`：进行集群状态管理，比如容器故障恢复、自动水平扩容等
- `Scheduler`：进行调度管理，比如观察和计算节点与容器负载，决定将容器调度到那个节点
- `ETCD`：分布式存储系统，存储`Kubernetes`集群所需的元信息

![](https://raw.githubusercontent.com/hts0000/images/main/202306121416910.png)

`K8S Node`是真正运行业务负载的地方，需要运行的容器会经过`Scheduler`计算决定运行在哪个`Node`上，计算完之后通知`API Server`，`API Server`通知对应`Node`上的`Kubelet`组件运行对应的容器。  
![](https://raw.githubusercontent.com/hts0000/images/main/202306121546038.png)

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

# Kubernetes元信息
## Labels
标签是`Kubernetes`中最重要的元数据，它使用`Key:Value Pair`的方式来标识资源对象，以便后续`Selector`能够筛选和组合资源。`Selector`支持与或非、集合的的逻辑组合来筛选资源。

```bash
# 给资源打上标签，<resource>表示资源的类型，<resource-name>表示具体资源名称
kubectl label <resource> <resource-name> key=value

# 重写已有标签
kubectl label <resource> <resource-name> key=value --overwrite

# 查看资源标签
kubectl get <resource> --show-labels

# 给资源删除标签，key后面加 '-' 表示删除改标签
kubectl label <resource> <resource-name> key-

# 筛选资源时指定标签
kubectl get <resource> --show-labels -l 'key=value'
kubectl get <resource> --show-labels -l 'key in (value1, value2)'
```

## Annotation
注解用于记录资源的非标示性信息，扩展资源的描述。

```bash
# 给资源加上注解，<resource>表示资源的类型，<resource-name>表示具体资源名称
kubectl annotate <resource> <resource-name> describe='some describe for resource.'

# 查看资源注解信息，在annotation中可以看到加上的注解
kubectl get <resource> <resource-name> -o yaml

# kubectl工具apply文件创建资源时，会创建一个独特的annotation叫：kubectl.kubernetes.io/last-configuration，里面记录了apply是文件的内容，是一个json串
kubectl get deployments nginx -o yaml | grep -A 5 annotations
```

## OwnerReference
所有者用于标识资源的所有者/创建者。比如有些资源是一个集合，比如`Pod`集合会由`replicaset`或`statefulset`创建，而想要删除整个集合资源时，就可以用到`OwnerReference`来找到属于同一集合的资源进行统一删除。

```bash
# Deployment类型的资源是Replicaset更高级的封装，因此查看Replicaset类型的资源也可以看到Deployment的资源，因此也可以看到Deployment资源的OwnerReference
kubectl get replicasets nginx -o yaml
```

# 控制器模式
控制器模式指的是在一个控制循环中，通过控制器、被控系统及观测传感器这三个逻辑组件，驱使被控系统达到期望状态。

`Kubernetes`中的`声明式API`就是声明资源期望达到的状态(spec)，通过控制模式不断地将当前状态(status)向期望状态逼近。这在资源扩容或故障恢复时，有明显的逻辑优势。在扩容时，`声明式API`理解的是当前状态与期望状态不一致，为了达到状态一致而触发扩容，而`命令式API`理解的是收到一条命令，该命令要求扩容机器。

这两种逻辑在出错后重试时，会产生巨大的差异。试想一下扩容成功了，但是扩容程序认为自己失败了，会怎么样？`声明式API`重试时，会观测到当前状态与期望状态其实已经一致了，从而停止扩容，就算这个时间错开，扩容了两次，在下个观测周期到来，传感器依然会发现状态不一致而触发缩容，这种设计下当前状态总是同期望状态一起变化的。

`命令式API`如果发生两次扩容，系统很难理解和避免这种错误，特别是期望状态不断变化时，程序必须非常小心的处理每一个可能发生错误的地方，加上大量的判断和回滚，这严重影响性能和故障恢复时间，而且也无法完全保证多次故障发生后，状态仍然与期望的一致。究其根源在于`命令式API`只有一次任务的过程，如果该过程发生错误，必须通过强一致的事务来保证回滚，然后再继续尝试。  
![](https://raw.githubusercontent.com/hts0000/images/main/202306131157408.png)

# Kubernetes中的资源
## Deployment
`Deployment`是对`Replicaset`的高级封装，`Replicaset`控制`Pod`的副本数量，`Deployment`负责控制使用哪个`Replicaset`。创建或修改`Deployment`就会新创建一个`Replicaset`，每个`Replicaset`都记录了管理的`Pod`的副本、容器等等信息。`Deployment`回滚就是回滚使用历史版本的`Replicaset`信息来创建`Pod`。

**Deployment资源文件模板**  
```yml
apiVersion: app/v1
kind: Deployment
# 声明资源元信息：名称、标签、注解、命名空间等
metadata:
  name: nginx-deployment
  labels:
    app: nginx
# 声明资源期望的状态
spec:
  # 具体配置需要根据资源类型进行配置
  # 期望Pod数量
  replicas: 3
  # Pod的选择器
  selector:
    # 配置标签匹配
    matchLabels:
      app: nginx
  # 每个Pod的模板
  template:
    # Pod元信息
    metadata:
      labels:
        app: nginx
    # Pod期望状态
    spec:
      # Pod中的containers
      containers:
      - name: nginx
        image: nginx:1.18
        ports:
        - containerPort: 80
```

**查看Deployment资源的字段解释**  
```bash
# 查看Deployment资源
# READY：就绪Pod个数
# UP-TO-DATE：达到最新版本Pod个数
# AVAILABLE：可以Pod个数
kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
nginx-depoloyment     4/4     4            4           23h
nginx-depoloyment-2   2/2     2            2           22h
web                   3/3     3            3           12d
```

**更新Deployment资源Pod的镜像**  
```bash
# 使用kubectl工具更新镜像
# deployment.v1.apps：指定资源类型和资源组，可以忽略
# nginx-deployment：资源名称
# nginx=nginx:1.19.1：nginx指的是Pod中具体哪一个container，以及修改成什么镜像
kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.19.1

# 也可以修改资源的yaml文件，重新apply来修改
# 具体操作为打印资源的yaml格式配置到文件中，修改指定容器的镜像版本，重新apply
kubectl get deployments nginx-deployment -o yaml > deployment-update.yaml
kubectl apply -f deployment-update.yaml

# 其实每一次变更，Kubernetes都会保留历史的信息，方便回滚，默认会保留10次的变更历史
kubectl get deployments nginx-deployment -o yaml | grep -A revisionHistoryLimit

# 因为Deployment是对Replicaset的高级封装，多次修改Deployment会保留多份Replicaset
# 因为保存了历史信息，所以可以很快回退
# 回退时会设置当前版本的Replicaset副本数为0，增加回退版本Replicaset副本数，当然会控制两边达到一个平滑过渡。
```

**回滚操作**  
```bash
# 查看历史版本
kubectl rollout history deployments.apps/nginx-deployment

# 回退上一版本
kubectl rollout undo deployments.apps/nginx-deployment

# 回退指定版本
kubectl rollout undo deployments.apps/nginx-deployment --to-revision=2
```

**Deployment状态流转图**  
每一个资源都有其对应的`当前状态(status)`。  
![](https://raw.githubusercontent.com/hts0000/images/main/202306131544588.png)

**spec字段解释**  
- MinReadySeconds：Deployment 会根据 Pod ready 来看 Pod 是否可用，但是如果我们设置了 MinReadySeconds 之后，比如设置为 30 秒，那 Deployment 就一定会等到 Pod ready 超过 30 秒之后才认为 Pod 是 available 的。Pod available 的前提条件是 Pod ready，但是 ready 的 Pod 不一定是 available 的，它一定要超过 MinReadySeconds 之后，才会判断为 available
- revisionHistoryLimit：保留历史 revision，即保留历史 ReplicaSet 的数量，默认值为 10 个。这里可以设置为一个或两个，如果回滚可能性比较大的话，可以设置数量超过 10
- paused：paused 是标识，Deployment 只做数量维持，不做新的发布，这里在 Debug 场景可能会用到
- progressDeadlineSeconds：前面提到当 Deployment 处于扩容或者发布状态时，它的 condition 会处于一个 processing 的状态，processing 可以设置一个超时时间。如果超过超时时间还处于 processing，那么 controller 将认为这个 Pod 会进入 failed 的状态

**升级策略字段解析**
- MaxUnavailable：滚动过程中最多百分之几 Pod 不可用
- MaxSurge：滚动过程中最多存在百分之几 Pod 超过预期 replicas 数量

## Job
`Job`就是一个任务，对于资源文件指定的任务启动一个对应的Pod执行，如果该任务要求执行10次，而且并发执行，那么将会创建10个Pod并发来执行同一个任务。

**配置文件模板**  
```yaml
apiVersion: batch/v1
kind: Jon
metadata:
  name: hello-world
  labels:
    type: bash
# 描述Job
spec:
  # 该Job需要执行多少次
  completions: 10
  # 并发执行的Pod数
  parallelism: 2
  # 重试次数
  backoffLimit: 4
  # 描述执行Job的Pod
  template:
    spec:
      # 有三种策略
      # Never：
      # OnFailure：
      # Always：
      restartPolicy: Never
      conatiners:
      - name: hello-world
        image: busybox
        command: ["sh", "-c", "echo hello world && sleep 60"]
```

**查看Job资源字段解释**  
```bash
# COMPLETIONS：一共几次任务，完成了几次
# DURATION：Pod中业务运行具体时长
# AGE：
kubectl get jobs
NAME                      COMPLETIONS   DURATION   AGE
hello-world               1/1           80s        33m
hello-world-parallelism   8/8           5m20s      6m44s
```

**查看Job执行日志**  
```bash
kubectl logs <pod-id>
```

## CronJob
`CronJob`与`Job`类似，增加了定时执行功能，使用习惯和语法与`Linux`的`crontab`类似。

**配置文件模板**  
```yml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-world-cron
  labels:
    type: bash
spec:
  schedule: "*/1 * * * *"
  # 超过指定时间Job未启动，就停止这个Job
  startingDeadlineSecond: 10
  # 是否允许并行运行
  concurrencyPolicy: Allow
  # 允许留存历史Job个数
  successfulJobsHistoryLimit: 10
  jobTemplate:
    spec:
      template:
        spec:
          # 有三种策略
          # Never：
          # OnFailure：
          # Always：
          restartPolicy: OnFailure
          containers:
          - name: hello-world-cron
            image: busybox
            command:
            - /bin/sh
            - -c
            - echo hello world from CronJob; sleep 60
```

## DaemonSet
`DaemonSet`叫做守护进程控制器，主要作用如下：
- 保证每一个`Node`上面运行同一个`Pod`
- 新增或移除`Node`时能动态感知到并自动添加或删除
- 跟踪每一个`Pod`的状态，在`Pod`异常时尝试恢复

**配置文件模板**  
```yml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # 这些容忍度设置是为了让该守护进程集在控制平面节点上运行
      # 如果你不希望自己的控制平面节点运行 Pod，可以删除它们
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

# Kubernetes应用配置管理
首先看一下需求来源。大家应该都有过这样的经验，就是用一个容器镜像来启动一个container。要启动这个容器，其实有很多需要配套的问题待解决：
- 第一，比如说一些可变的配置。因为我们不可能把一些可变的配置写到镜像里面，当这个配置需要变化的时候，可能需要我们重新编译一次镜像，这个肯定是不能接受的；
- 第二就是一些敏感信息的存储和使用。比如说应用需要使用一些密码，或者用一些 token；
- 第三就是我们容器要访问集群自身。比如我要访问 kube-apiserver，那么本身就有一个身份认证的问题；
- 第四就是容器在节点上运行之后，它的资源需求；
- 第五个就是容器在节点上，它们是共享内核的，那么它的一个安全管控怎么办？
- 最后一点我们说一下容器启动之前的一个前置条件检验。比如说，一个容器启动之前，我可能要确认一下 DNS 服务是不是好用？又或者确认一下网络是不是联通的？那么这些其实就是一些前置的校验。

`Kubernetes`中对这些配置进行管理，有配套的工具和资源：
- 可变配置就用 ConfigMap；
- 敏感信息是用 Secret；
- 身份认证是用 ServiceAccount 这几个独立的资源来实现的；
- 资源配置是用 Resources；
- 安全管控是用 SecurityContext；
- 前置校验是用 InitContainers 这几个在 spec 里面加的字段，来实现的这些配置管理。

![](https://raw.githubusercontent.com/hts0000/images/main/202306141154350.png)

## ConfigMap
存储`Pod`的配置，用于实现`Pod`与配置的解耦。

**配置文件模板**  
```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: hello-world-config
  labels:
    hello: world
# 具体的配置以key:value的方式存储
data:
  # 简单的标量类型key:value形式
  MAX_CONNECTION: "10"
  FILE_NAME: hello_world
  player: hts0000

  # 文件名和内容形式的key:value
  hello-world-config.json: |
    {
        "name": "hts0000",
        "age": 10,
        "sex": boy
    }
  # 类文件键
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
```

**通过命令行创建**  
```bash
kubectl create configmap hello-config --from-literal=hello=world --from-literal=MaxConnection=10
```

**使用ConfigMap**  
可以在`Deployment`等资源中使用`ConfigMap`，下面以`Deployment`为例，打印`ConfigMap`中定义的各项配置。  
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-deployment
  labels:
    hello: world
spec:
  replicas: 3
  selector:
    matchLabels:
      hello: world
  template:
    metadata:
      labels:
        hello: world
    spec:
      volumes:  # 将ConfigMap作为卷，Pod内就可以将这个卷挂载到容器内部
      - name: hello-world-config-volumes
        configMap:
          name: hello-world-config  # ConfigMap的名字
          items:    # 来自ConfigMap的一组键，将被创建为文件
          - key: hello-world-config.json
            path: hello-world-config.json
          - key: game.properties
            path: game.properties
          - key: user-interface.properties
            path: user-interface.properties
      containers:
      - name: say-hello
        image: busybox
        env:    # 从hello-world-config中获取配置，写入容器的环境变量
        - name: MAX_CONNECTION  # 这个名字可以和ConfigMap中不同
          valueFrom:
            configMapKeyRef:    # value的来源为ConfigMap
              name: hello-world-config
              key: MAX_CONNECTION
        - name: FILE_NAME
          valueFrom:
            configMapKeyRef:
              name: hello-world-config
              key: FILE_NAME
        - name: MY_NAME
          valueFrom:
            configMapKeyRef:
              name: hello-world-config
              key: player
        volumeMounts:   # 将hello-world-config-volumes挂载到容器内部
        - name: hello-world-config-volumes
          mountPath: "/config"
          readOnly: true
        command:    # 打印上面加载进来的环境变量和文件
        - /bin/sh
        - -c
        - echo hello world; echo MAX_CONNECTION $(MAX_CONNECTION); \
          echo FILE_NAME $(FILE_NAME); echo player $(MY_NAME); \
          echo '########################################'; \
          cat /config/hello-world-config.json; \
          echo '########################################'; \
          cat /config/game.properties; \
          echo '########################################'; \
          cat /config/user-interface.properties; \
          echo '########################################'; \
          sleep 10000000000
```

## Secret
`Secret`是一个主要用来存储密码`token`等一些敏感信息的资源对象，里面的敏感信息是采用 `base-64`编码保存起来的。

**配置文件模板**  
```yml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
  namespace: default
# 指定Secret的一个类型
# Opaque：用户定义的任意数据
# kubernetes.io/service-account-token：服务账号令牌
# kubernetes.io/dockercfg：~/.dockercfg文件的序列化形式
# kubernetes.io/dockerconfigjson：~/.docker/config.json文件的序列化形式
# kubernetes.io/basic-auth：用于基本身份认证的凭据
# kubernetes.io/ssh-auth：用于 SSH 身份认证的凭据
# kubernetes.io/tls：用于 TLS 客户端或者服务器端的数据
# bootstrap.kubernetes.io/token：启动引导令牌数据
type: Opaque
data:
  aHRzMDAwMAo=: MTIzNDU2Cg==    # 敏感信息经过base64加密后存储在Secret中
```

## ServiceAccount
`ServiceAccount`用于解决`Pod`在集群里面的身份认证问题，`ServiceAccount`会去关联一个底层的`Secret`，也就是说身份认证信息实际存储于`Secret`里面。

## Resources
`Resources`用于配置容器使用的资源，比如配置使用的CPU时间、内存大小、硬盘大小或GPU算力等，也支持自定义实现资源限制。

```yml
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-ctr
    image: polinux/stress
    resources:
      requests:
        cpu: "250m"
        memory: "100Mi"
      limits:
        cpu: "300m"
        memory: "200Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]

```

## SecurityContext
`SecurityContext`主要是用于限制容器的一个行为，它能保证系统和其他容器的安全。

## InitContainers
`InitContainer`会在普通的`Container`之前启动，如果定义了多个`InitContainer`，那么他们会按定义顺序依次启动。`InitContainer`常用与为普通的`Container`进行初始化。

# 实践
本实践通过`helm`在`Kubernetes`上部署`Prometheus`，采集自定义的`Demo`应用的指标数据，并通过`Grafana`展示出来。

实践使用到了`Kubernetes`的`Deployment`、`ConfigMap`、`服务发现`、`Service`等一系列能力，结合`Prometheus`和`Grafana`实现监控系统的搭建。

## 自定义Exporter
根据`Prometheus`提供的`SDK`，实现一个可以向外暴露`Metrics`指标的应用。

官方教程：https://prometheus.io/docs/guides/go-application/  
实现demo：https://github.com/hts0000/exporter-demo

## 打包成Docker镜像
使用以下`Dockerfile`文件将`Demo`打包成镜像。打包是使用多阶段编译的方式减小镜像的体积。
```Dockerfile
FROM golang:1.20.3-alpine AS builder

RUN go env -w GO111MODULE=on
RUN go env -w GOPROXY=https://goproxy.cn,direct

COPY . /go/src/exporter-demo

WORKDIR /go/src/exporter-demo

RUN go build -o exporter-demo .

FROM alpine

COPY --from=builder /go/src/exporter-demo/exporter-demo /bin/exporter-demo

EXPOSE 18080

ENTRYPOINT [ "/bin/exporter-demo" ]
```
执行`docker build -t demo:v1 .`命令进行打包。

## 将Demo在K8S上运行起来
以`Deployment`的方式，运行三个`Demo`副本，同时使用`ConfigMap`定义副本将会使用到的配置。

`demo-deployment.yaml`文件配置
```yml
# 副本使用到的配置存储在demo-configmap这个ConfigMap里
apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-configmap
  namespace: demo
data:
  ADDR: ':18080'

---

# demo副本信息
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-deployment
  labels:
    app: demo
  namespace: demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: demo
  template:
    metadata:
      labels:
        app: demo
      annotation:
        desc: 'simple app, expose metric from :18080/metrics url'
    spec:
      containers:
      - name: demo
        image: demo:v1
        ports:
        - containerPort: 18080  # 定义程序向外暴露的端口，需要跟ConfigMap中的配置对应
        env:
        - name: ADDR  # 程序会读取ADDR环境变量获取程序启动监听地址
          valueFrom:
            configMapKeyRef:
              name: demo-configmap   # 此处为ConfigMap的名称
              key: ADDR
```

执行`kubectl apply -f demo-deployment.yaml`命令，创建`Deployment`。

执行`kubectl get deployments -n demo`查看`Deployments`信息。`-n`选项指定命名空间，因为我们将`demo-deployment`创建在`demo`这个命名空间里的。  
![](https://raw.githubusercontent.com/hts0000/images/main/202306161442413.png)

执行`kubectl get pods -n demo`查看`Pods`信息。  
![](https://raw.githubusercontent.com/hts0000/images/main/202306161453080.png)

确保`Pods`已经正常运行。

## 使用Helm安装Prometheus、Altermanager和Grafana
`Helm`是`Kubernetes`的包管理工具，可以方便的在`Kubernetes`上部署软件。

如果我们要手动部署`Prometheus`到`Kubernetes`上可能需要如下步骤：
1. 使用`Prometheus`官方镜像包甚至自行给`Prometheus`打包
2. 根据产品特性，选择部署资源。比如单实例的部署为`Pod`，集群的部署为`Deployment`或`DaemonSet`等等
3. 抽离其中可定制化的配置，比如监听地址、采集间隔、监控服务器信息等等，根据`Kubernetes`的最佳实践，这些配置最好写到`ConfigMap`中去
4. 还需要考虑访问权限和安全性的问题，如果不希望其他用户访问`Prometheus`这个资源，还要单独配置`ServiceAccount`等
5. ...

需要做的事情很多，而且要按照最佳实践来部署，有很高的学习成本。因此`Helm`就是来解放这一过程的，用户只需要一行命令，就可以安装官方提供的`Chart(Helm里一个包称之为Chart)`，官方提供的无疑是最贴合最佳实践的，用户只需要简单的配置，即可在`Kubernetes`上启动一个实例，甚至一个集群。

`Helm`官方文档：https://helm.sh/zh/docs/

需要注意的是，`Helm`与`Kubernetes`有版本支持的要求，不同版本的`Helm`支持的`Kubernetes`版本参考文档：https://helm.sh/zh/docs/topics/version_skew/。  
![](https://raw.githubusercontent.com/hts0000/images/main/202306161513189.png)

首先安装`Helm`很简单，下载下来就是个二进制包，放到`/usr/bin`目录即可。下载地址：https://github.com/helm/helm/releases


前往`Helm Charts Hub`查找`Prometheus Chart`。  
`Charts Hub`地址：https://artifacthub.io/。  
![](https://raw.githubusercontent.com/hts0000/images/main/202306161532346.png)

一般来说里面的教程会提示增加一个仓库，但是这些仓库下载拉取镜像时，经常会失败，所以我们需要配置`Helm`的下载仓库，`Helm`支持配置多个仓库，下载的时候需要加上指定仓库的前缀。  
```bash
# 添加仓库地址
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add stable http://mirror.azure.cn/kubernetes/charts
helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
helm repo add incubator https://charts.helm.sh/incubator
```

执行`helm repo list`查看所有仓库。  
![](https://raw.githubusercontent.com/hts0000/images/main/202306161534906.png)

我们在`bitnami`仓库和官方提供的`prometheus-community`仓库分别查找`promethues`，看看他们的版本差异。  
![](https://raw.githubusercontent.com/hts0000/images/main/202306161537507.png)

可以看到两个仓库的版本是同步的。为了下载安装顺利，我们选择使用`bitnami`仓库。

`Helm`在下载安装`Chart`时，需要指定仓库作为前缀。
```bash
# 在bitnami/prometheus仓库中下载prometheus
helm install prometheus bitnami/prometheus

# 如法炮制下载grafana
helm install grafana bitnami/grafana
```
下载完之后查看，altermanager也安装上了。  
![](https://raw.githubusercontent.com/hts0000/images/main/202306161544667.png)

## 暴露Grafana和Prometheus访问地址
默认情况下`Grafana`和`Prometheus`的`Service Type`为`ClusterIP`，只能通过`Service`网段访问，也就是只有集群内可访问。我们希望集群外也能访问，就需要将`Service Type`修改为`NodePort`，并给资源分配一个`nodePort`，即`Node`上的`Port`。

首先查看所有的`Service`。  
![](https://raw.githubusercontent.com/hts0000/images/main/202306161553981.png)

执行`kubectl edit services prometheus-server`实时修改配置，将`type`修改为`NodePort`，增加`nodePort`配置。`grafana`也是一样的操作。  
![](https://raw.githubusercontent.com/hts0000/images/main/202306161600238.png)

![](https://raw.githubusercontent.com/hts0000/images/main/202306161601718.png)

然后就可以通过`Node IP`来访问`Grafana`和`Prometheus`了。  
![](https://raw.githubusercontent.com/hts0000/images/main/202306161601956.png)

![](https://raw.githubusercontent.com/hts0000/images/main/202306161603709.png)

## Prometheus访问K8S动态发现Pod
现在`Prometheus`默认只有`Prometheus`自己和`Altermanager`这两个采集源。我们增加上`demo-deployment`中的所有`pod`。但是`pod ip`是不固定的，如果`pod`重启或滚动升级，ip地址会变化。这时就需要将`Prometheus`配置为主动与`K8S`读取`Pod`配置。

`Prometheus`的配置存储在`ConfigMap`中，我们先查看所有`ConfigMap`。  
![](https://raw.githubusercontent.com/hts0000/images/main/202306161608184.png)

执行`kubectl edit configmap prometheus-server`命令进行编辑。增加针对`demo`这个`namespace`的指标采集。  
![](https://raw.githubusercontent.com/hts0000/images/main/202306161609919.png)

`Prometheus`启动后配置就固定了，需要重启，我们可以把`prometheus`这个`pod`删掉，让`Kubernetes Deployment`自动帮我们重启。重启之后就能看到三个`demo`副本的监控信息了。  
![](https://raw.githubusercontent.com/hts0000/images/main/202306161612121.png)

## 配置Grafana
在`Grafana`中配置`Prometheus`数据源，并展示指标的数据。

配置数据源时，地址就是`Prometheus`所在`Node`的地址，以及`nodePort`配置的端口。  
![](https://raw.githubusercontent.com/hts0000/images/main/202306161614866.png)

添加一个`Panel`。  
![](https://raw.githubusercontent.com/hts0000/images/main/202306161621557.png)

在`Options`和侧边栏里还可以配置指标和`Panel`的名称。  
![](https://raw.githubusercontent.com/hts0000/images/main/202306161623406.png)

## 访问测试
使用`ab`工具访问这些`Pod`，就可以在`Grafana`中看到指标的变化了。需要注意执行`ab`命令的机器，必须是`Kubernetes`集群内的机器，`ab`访问的地址为`Pod`在集群内的地址，如果希望集群外访问，可以创建一个`Services`，通过向外暴露`demo-deployment`的统一入口。  
![](https://raw.githubusercontent.com/hts0000/images/main/202306161625914.png)
