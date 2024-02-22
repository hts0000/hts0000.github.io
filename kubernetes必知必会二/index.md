# Kubernetes必知必会(二)


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

# Kubernetes应用存储和持久化
## Volumes

## 

# Kubernetes观测性
## Liveness
`Liveness`是存活指针，`Kubernetes`用其来探测Pod是否存活，如果不存活则会杀掉当前`Pod`，由该节点上的`Kubelet`根据Pod的重启策略`(Always/OnFailure/Never)`，决定是否重新拉起该`Pod`。重启策略默认值是`Always`，`Liveness`不配置时，一直认为探活成功，这时`Pod`是否正常就托管给了`Kubernetes`来检测，而`Kubernetes`则是会在`Pod`生命周期状态改变为异常时，才会让`Kubelet`根据重启策略重启`Pod`。也就是说`Liveness`使得用户有了一种手段，可以自定义检测`Pod`是否正常，从而让`Kubernetes`来执行这个自定义的检测。

`Liveness`支持如下检测方式：
- exec：shell命令行检测，返回0检测成功，返回非0检测失败
- httpGet：http接口检测，返回200~399检测成功，其他检测失败
- tcpSocket：tcpsock检测，连通则检测成功，无法连通则失败
- grpc：需要应用支持GRPC健康检测协议

**Liveness模板**  
```yml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-pod
  labels:
    app: liveness-pod
spec:
  containers:
  - image: busybox
    name: liveness-pod
    command: ["sh", "-c", "sleep 1000000"]
    livenessProbe:
      exec:     # 使用shell的方式探测
        command: ["sh", "-c", "exit 1"]     # 让其直接检测失败
    # httpGet:      # 使用httpGet的方式探测
    #   path: /healthz  # 探测路径
    #   port: 8080      # http端口
    #   httpHeaders:    # 设置探测时带上的http header
    #   - name: Custom-Header
    #     value: Awesome
    # tcpSocket:    # 使用tcpSocket的方式探测
    #   port: 8080  # 探测端口
      initialDelaySeconds: 10   # pod启动后延迟10s再开始探测
      periodSeconds: 5     # 探测间隔
```

## Readiness
`Readiness`是就绪探针，`Kubernetes`用其探测`Pod`是否就绪，探测成功后才将其加入`Service`的`Endpoint`中，探测失败了则将其从`Endpoint`中删除。也就是说`Readiness`是和`Service`绑定使用的，探测失败了则认为该`Pod`无法处理请求，相当于做了集群故障节点剔除。

**Readiness模板**  
就绪探针的配置和存活探针的配置相似。 唯一区别就是要使用`readinessProbe`字段，而不是`livenessProbe`字段。
```yml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-pod
  labels:
    app: readiness-pod
spec:
  containers:
  - image: busybox
    name: readiness-pod
    command: ["sh", "-c", "sleep 1000000"]
    readinessProbe:
      exec:     # 使用shell的方式探测
        command: ["sh", "-c", "exit 1"]     # 让其直接检测失败
    # httpGet:      # 使用httpGet的方式探测
    #   path: /healthz  # 探测路径
    #   port: 8080      # http端口
    #   httpHeaders:    # 设置探测时带上的http header
    #   - name: Custom-Header
    #     value: Awesome
    # tcpSocket:    # 使用tcpSocket的方式探测
    #   port: 8080  # 探测端口
      initialDelaySeconds: 10   # pod启动后延迟10s再开始探测
      periodSeconds: 5     # 探测间隔
```

## Startup
`Startup`是启动探针，用于检测`Pod`是否启动成功。在指定时间期间内探测成功，才启动探活指针。

****  
```yml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-pod
  labels:
    app: readiness-pod
spec:
  containers:
  - image: busybox
    name: readiness-pod
    command: ["sh", "-c", "sleep 1000000"]
    startupProbe:
      httpGet:
        path: /healthz
        port: liveness-port
      failureThreshold: 30
      periodSeconds: 10
```
 
# Kubernetes监控和日志

#

