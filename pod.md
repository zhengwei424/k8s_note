# Pod

## 创建一个pod

1. 创建前

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']

```

字段解释：

- apiVersion/kind: 这是所有资源共有的基本字段，表示api版本号以及类型

  - apiVersion: 表示资源所属于的group以及version。结构一般为<group/version>，其中v1是特例，其group为“”，省去了中间的/。一般核心资源都是v1，比如Namespace、Pod、ConfigMap等。其他的资源各有各的group以及version
  - kind：资源类型，首字母大写。

- metadata

  - name：pod的名字
  - labels：pod的标签
  - namespace：可选字段。Kubernetes中的资源分为两类，一类属于namespace，一类不属于。Pod属于namespace，如果yaml里没有写namespace，表示属于default namespace

- spec： Pod的主要信息部分

  - containers：一个列表，因为可以有包含多个container

    - name：这个container的名字，==一个pod下面有多个container的名字不能冲突==

    - image：这个container的镜像信息

    - command：启动命令。是可选项，因为一般镜像都有默认值

      

2. pod创建过程大概需要以下步骤：
   1. 调度到某台机器上。kubernetes会根据一定的优先级算法选择一台机器作为pod的运行机器
   2. 拉取镜像
   3. 挂在存储配置
   4. 运行起来。如果有健康检查，会根据检查结果来设置状态。



```yaml
[root@master test]# kubectl get pod myapp-pod  -o yaml 
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/podIP: 10.10.166.180/32
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"app":"myapp"},"name":"myapp-pod","namespace":"default"},"spec":{"containers":[{"command":["sh","-c","echo Hello Kubernetes! \u0026\u0026 sleep 3600"],"image":"busybox","name":"myapp-container"}]}}
  creationTimestamp: "2020-03-23T03:13:18Z"
  labels:
    app: myapp
  name: myapp-pod
  namespace: default
  resourceVersion: "1346785"
  selfLink: /api/v1/namespaces/default/pods/myapp-pod
  uid: f561dd0d-ba5d-464a-9d38-dd12730417e0
spec:
  containers:
  - command:
    - sh
    - -c
    - echo Hello Kubernetes! && sleep 3600
    image: busybox
    imagePullPolicy: Always
    name: myapp-container
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-hljm8
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: node1
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: default-token-hljm8
    secret:
      defaultMode: 420
      secretName: default-token-hljm8
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2020-03-23T03:13:18Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2020-03-23T03:13:52Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2020-03-23T03:13:52Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2020-03-23T03:13:18Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://eb25036fad6901d49ff4b7c45c0fb7ee0b6bf6110bc868744d18f006cad0eb8f
    image: busybox:latest
    imageID: docker-pullable://busybox@sha256:b26cd013274a657b86e706210ddd5cc1f82f50155791199d29b9e86e935ce135
    lastState: {}
    name: myapp-container
    ready: true
    restartCount: 0
    state:
      running:
        startedAt: "2020-03-23T03:13:52Z"
  hostIP: 192.168.80.129
  phase: Running
  podIP: 10.10.166.180
  qosClass: BestEffort
  startTime: "2020-03-23T03:13:18Z"

```



container里主要包含的基本信息有：

- 启动命令：主要是command。
- 镜像信息：
  - image： 镜像地址
  - imagePullPolicy：镜像拉取策略。因为有时候机器上已经有了image，我们就可以不用去远端拉取。这时候可以在pod yaml里设置这个值为IfNotPresent。Always表示不管机器上存不存在都会重新pull镜像。适合用于镜像tag不变但是内容会变化的场景。
- name：容器名称，kubernetes中的所有资源的查找和使用主要都是靠名称。容器也是。比如我们用kubectl去exec到一个容器中时，就会用到这个名称。
- resources：因为我们使用的yaml里面没有这部分信息，所以创建出来的yaml里默认值为{}。它具体描述这个Pod对于计算资源的需求信息。
- terminationMessagePath：用于记录容器退出时的最后信息（成功退出或者异常退出），这些信息就可以用于监控展示或者kubernetes计算container以及pod的状态。==日志文件存放在每个容器（container）内==
- terminationMessagePolicy：从哪些地方取容器最终的状态信息。
  - File：默认值，表示只从上面terminationMessagePath所在位置取状态信息。
  - FallbackTologsOnError：如果上面的文件里没有内容，name就从容器的日志里取一部分数据作为状态信息（一般是stdout的输出）
- restartPolicy：重启策略
  - `Always`：默认策略，当容器失效时，kubelet 会自动重启该容器
  - `OnFailure`：当容器终止运行且退出码不为 0 时，kubelet 会自动重启该容器
  - `Never`：不论容器运行状态如何，kubelet 都不重启该容器



status包含的基本信息有：

- hostIP：pod所在主机的ip地址
- phase：pod的状态，当前为Running，也可能会是其他状态。
  - Pending：一般表示还没有开始调度到某台机器上，如果没有符合条件的主机，就会一直处于Pending状态。或者API Server已经创建了Pod，但在pod内还有一个或多个容器的镜像没有创建，包括正在下载镜像的过程
  - Running：运行中。Pod内所有的容器均已创建，且至少有一个容器处于运行状态，正在启动状态或正在重启状态。
  - Succeeded：有些pod不是长久运行的。 比如：cronjob，一段时间就结束了，需要反馈任务执行的结果。Pod内所有容器均成功执行后退出，且不会再重启。
  - Failed：pod的container异常退出。比如，command写得有问题。POd内所有容器均已退出，但至少有一个容器退出为失败状态。
  - Unknown：未知。比如pod所在的机器无法连接，网络异常。由于某种原因无法获取该Pod的状态，可能由于网络通信不畅导致。
- podIP：pod所分配到的ip，这个ip是全集群唯一的。
- qosClass：资源分配相关。qos（全称quality of service）表示kubernetes对不同的pod因其requests/limits设置而对其运行情况的保障，具体分为以下几类：
  - Guaranteed：requests=limits，二者是一样的值。高优先级，kubernetes保证只要资源使用不超过limits就不会被kill
  - Burstable：requests<limits，pod可以使用requests到limits之间数量的资源，中优先级，但是当机器资源紧张的时候，如果这些pod对资源的使用超过了requests，就何有可能会被kill掉（没有BestEffort的pod的情况下）
  - BestEffort：没有设置具体的requests和limits。低优先级，如果机器资源紧张，这些Pod会优先被kill掉
- startTime：启动时间。

## pod资源控制Requests和Limits

Pod中的资源限制主要是针对cpu和内存。与docker不同的是，它提供了requests和limits两个设置。具体的含义为：

- requests： pod运行所需要的的最少资源。例如kubernetes在调度pod时，就是以这个设置来挑选node
- limits：pod运行的资源上限。也就是说，超过这个limits，pod会被kill掉。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: wp
spec:
  containers:
  - name: wp
    image: wordpress
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"

```

我们可以通过kubectl describe相关的node，查询到node节点的资源使用情况

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200323135031552.png" alt="image-20200323135031552"  />

## pod启动命令command

我们在使用docker run的时候可以指定启动命令，在Dockerfile里也可以设置ENTRYPOINT和RUN命令指令。在pod中，kubernetes使用了更简单的command/args参数来设置，它与docker使用的参数对应如下：

| Image Entrypoint | Image Cmd | Container command | Container args | Command run    |
| ---------------- | --------- | ----------------- | -------------- | -------------- |
| [/ep-1]          | [foo bar] | <not set>         | <not set>      | <ep-1 foo bar> |
| [/ep-1]          | [foo bar] | [/ep-2]           | <not set>      | [ep-2]         |
| [/ep-1]          | [foo bar] | <not set>         | [zoo boo]      | [ep-1 zoo boo] |
| [/ep-1]          | [foo bar] | [/ep-2]           | [zoo boo]      | [ep-2 zoo boo] |

之所以要介绍这部分，是因为本身镜像层面的entrypoint和cmd就容易混淆。再加上pod又抽象出了新的参数，很容易无用出bug。上面的几条规则大概意思如下：

- pod不设置任何参数，直接使用镜像里面自带的参数
- pod的command和args都设置，使用设置的命令和参数
- pod只设置command，完全忽略镜像里的参数
- pod只设置args，镜像中的命令与新参数一起使用

一般来说，我们最好将启动命令在镜像里设置好，这样就不用在pod里设置。如果要在pod里面设置，最好填上command，这样就完全以pod的参数为准，方便理解。如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: command-demo
  labels:
    purpose: demonstrate-command
spec:
  containers:
  - name: command-demo-container
    image: debian
    command: ["printenv"]
    args: ["HOSTNAME", "KUBERNETES_PORT"]
  restartPolicy: OnFailure
```

运行结果如下：

```shell
[root@master test]# kubectl get pod 
NAME           READY   STATUS      RESTARTS   AGE
command-demo   0/1     Completed   0          2m44s
[root@master test]# kubectl logs command-demo 
command-demo
tcp://10.96.0.1:443

```



## 健康检查

docker在较高版本加入了健康检查，pod一开始就支持健康检查。

如果没有健康检查，可能容器在运行，但是服务已经不能正常工作了。

容器并不等于应用/服务。所以我们需要健康检查来同意两者的状态，这样我们就知道当服务异常时，容器也会退出。

kubernetes健康检查分为存活性检查和可用性检查，分别介绍如下：

- 存活性检查（liveness probe）：用于判断容器是否存活（Running状态），如果LivenessProbe探针探测到容器不健康，则kubelet将杀掉该容器，并根据容器的重启策略做相应的处理。如果一个容器没有设置LivenessProbe探针，那么kubelet认为该容器的LivenessProbe探针返回值永远是Success。
- 可用性检查（readiness probe）：用于判断容器服务是否可用（Ready状态），达到Ready状态的Pod才可以接收请求。对于被Service管理的Pod，Service与Pod Endpoint的关联关系也将基于Pod是否Ready进行设置。如果在运行过程中Ready状态变为False，则系统自动将其从Service的后端Endpoint列表中隔离出去，后续再把恢复到Ready状态的Pod加回后端Endpoint列表。这样就能保证客户端在访问Service时不会被转发到服务不可用的Pod实例上。



LivenessProbe和ReadinessProbe均可配置一下三种实现方式：

- exec：在容器内部执行一个命令，如果该命令的返回码为0，则表明容器健康。

- tcpSocket：通过容器的IP地址和端口号执行TCP检查，如果能够建立TCP连接，则表明容器健康。

- httpGet：通过容器的IP地址、端口号及路径调用HTTP Get方法，如果响应的状态码大于等于200且小于400，则认为容器健康。

  - host：连接使用的主机名，默认是pod的ip。也可以在http头中设置“Host”来代替

  - scheme：用于设置连接主机的方式（http或https），默认是http

  - path：访问http服务的路径

  - httpHeaders：请求中自定义的http头。http头字段允许重复。

  - port：访问容器的端口号或者端口名称（在containers.ports.name中定义）如果设置数字，必须在1~65535之间。

  - 对于 HTTP 探测，kubelet 发送一个 HTTP 请求到指定的路径和端口来执行检测。除非 `httpGet` 中的 `host` 字段设置了，否则 kubelet 默认是给 Pod 的 IP 地址发送探测。如果 `scheme`字段设置为了 `HTTPS`，kubelet 会跳过证书验证发送 HTTPS 请求。大多数情况下，不需要设置`host` 字段。这里有个需要设置 `host` 字段的场景，假设容器监听 127.0.0.1，并且 Pod 的 `hostNetwork` 字段设置为了 `true`。那么 `httpGet` 中的 `host` 字段应该设置为 127.0.0.1。可能更常见的情况是如果 Pod 依赖虚拟主机，你不应该设置 `host` 字段，而是应该在 `httpHeaders` 中设置 `Host`。

    对于一次 TCP 探测，kubelet 在节点上（不是在 Pod 里面）建立探测连接，这意味着你不能在 `host` 参数上配置 service name，因为 kubelet 不能解析 service name。

对于每种探针都需要设置initialDelaySeconds和timeoutSeconds两个参数，它们的含义如下：

- initialDelaySeconds：启动容器后进行手册健康检查的等待时间，单位为s。（很多程序启动有初始化时间，比如java程序，初始化比较慢，这时候需要设置initialDelaySeconds来跳过这段时间，等程序完全初始化之后再执行健康检查。）
- periodSeconds：command执行的间隔，健康检查就是一个持续性的过程，需要反复执行。
- timeoutSeconds：健康检查发送请求后等待响应的超时时间，单位为s。当超时发生时，kubelet会认为容器已经无法提供服务，将会重启该容器。
- successThreshold：探测器在失败后，被视为成功的最小连续成功数，默认值是1，LivenessProbe的这个值必须是1，最小值是1。
- failureThreshold：当pod启动了并探测到失败，pod的重启次数。LivenessProbe默认情况下失败就会重新启动。readinessProbe在默认情况下会将失败的pod标记为Unready，默认值为3，最小值是1



1. exec就是通过在容器中执行命令来进行健康检查。

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: liveness-exec
     labels:
       test: liveness
   spec:
     containers:
     - name: liveness
       image: busybox
       args:
       - /bin/sh
       - -c
       - touch /tmp/healthy;sleep 30; rm -rf /tmp/healthy; sleep 600
       livenessProbe:
         exec:
           command:
           - cat
           - /tmp/healthy
         initialDelaySeconds: 5  # 第一次执行livessProbe前要等待5s
         periodSeconds: 5  # 每5s检查一次
   
   ```


注意：

​	例子中为了演示，让command不断的创建和删除文件，健康检查去检查这个文件。



2. http检查就是通过http服务的某个路径，然后根据错误码来判定。http status code的200-400代表成功，其他代表失败。

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     labels:
       test: liveness
     name: liveness-http
   spec:
     containers:
     - name: liveness
       image: kubernetes/liveness
       args:
       - /server
       livenessProbe:
         httpGet:
           path: /healthz
           port: 8080
           httpHeaders:
           - name: X-Custom-Header
             value: Awesome
         initialDelaySeconds: 3
         periodSeconds: 3
   ```



3. 对于监听tcp端口的服务，我们可以蚕食与这个端口建立连接。如果成功，则认为服务正常。

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: goproxy
     labels:
       app: goproxy
   spec:
     containers:
     - name: goproxy
       image: googlecontainer/goproxy:0.1
       ports:
       - containerPort: 8080
       readinessProbe:
         tcpSocket:
           port: 8080
         initialDelaySeconds: 5
         periodSeconds: 10
       livenessProbe:
         tcpSocket:
           port: 8080
         initialDelaySeconds: 15
         periodSeconds: 20
   ```

   

   输出结果：

   ![image-20200323185641582](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200323185641582.png)

   

   timeout：超时时间，如果超时了就认为失败了

   success/failure：图中的配置是——>如果成功了一次就认为正常，如果失败了三次才认为失败。这样就可以有效地避免因为偶然的偏差导致容器被标记为异常。

## 多容器pod

这样的模式通常是一个容器主要用来提供服务，另一个做一些其他零碎的工作。比如：

- 收集日志。这样不用修改原来的服务，可以用另外的容器来适配各种日志收集系统。
- 做Proxy。比如我们的程序需要访问外部服务（db等），可以固定配置为localhost，由其他的容器来决定如何转发请求，相当于将动态配置的需求交由其他容器来做。

综合来讲，这样做的好处就是： 让主要的服务容器不做修改，就能更好地适配各种系统。另外，也能较好地坐到职责分离，不需要由一个容器来处理过多的任务。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: two-containers
spec:
  restartPolicy: Never
  volumes:
  - name: shared-data
    emptyDir: {}
  containers:
  - name: nginx-container
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html

  - name: debian-container
    image: debian
    volumeMounts:
    - name: shared-data
      mountPath: /pod-data
    command: ["/bin/sh"]
    args: ["-c", "echo Hello from the debian container > /pod-data/index.html"]

```



运行结果：

```shell
[root@master test]# kubectl get pods 
NAME             READY   STATUS    RESTARTS   AGE
two-containers   1/2     Running   0          16m
[root@master test]# vi test.yaml 
[root@master test]# kubectl exec -it two-containers -c nginx-container bash 
root@two-containers:/# cd /usr/share/nginx/
root@two-containers:/usr/share/nginx# ls
html
root@two-containers:/usr/share/nginx# cd html/
root@two-containers:/usr/share/nginx/html# ls
index.html
root@two-containers:/usr/share/nginx/html# cat index.html 
Hello from the debian container

```

因为debian-container运行完command就结束了，所以pod最终只有nginx-container在运行，进入到nginx-container中，发现已经写入成功。



## initContainer

initContainer是pod提供的另外一个非常有用的功能。它的结构与普通的container类似。但是在作用上有很大的区别：

- 它们是短期运行的程序，不是持久运行的进程。
- 顺序执行，并且每一个initContainer必须等待前一个执行成功才能继续执行。正常的container必须等待前面的initContainer都执行完成之后才能开始执行。

业务场景：

- 程序在运行前需要等待某个条件才能执行。比如服务A需要等待服务B可用时才能开始运行。
- 程序需要某些动态的配置信息生成之后才能正常运行
- 程序需要同步好某些数据之后才能正常运行，比如mysql-slave

综合来讲，就是某个服务的运行，需要一些前置条件（服务依赖、文件等等）才能正常运行。没有initContainer的情况下，我们需要在正常的container里做一些hack才能做到，这样不好维护并且比较复杂。有了initContainer，不管是结构上还是可维护性上都会好很多。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: inin-ctr-demo
spec:
  volumes:
  - name: data
    emptyDir: {}
  initContainers:
  - name: init-1
    image: busybox
    command: ["sh", "-c", "echo start 1 >> /data/file"]
    volumeMounts:
    - name: data
      mountPath: /data
  - name: init-2
    image: busybox
    command: ["sh", "-c", "echo start 2 >> /data/file"]
    volumeMounts:
    - name: data
      mountPath: /data
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "sleep 1000"]
    volumeMounts:
    - name: data
      mountPath: /data

```



运行结果：

```shell
[root@master test]# kubectl get pods -w
NAME            READY   STATUS     RESTARTS   AGE
inin-ctr-demo   0/1     Init:0/2   0          6s   # 有两个initContainer，正在执行第一个
inin-ctr-demo   0/1     Init:1/2   0          23s  #正在执行第二个
inin-ctr-demo   0/1     PodInitializing   0          51s#initContainer执行完成，执行正常的容器
inin-ctr-demo   1/1     Running           0          79s
[root@master test]# kubectl exec -it inin-ctr-demo -c busybox sh 
/ # 
/ # cd /data/
/data # ls
file
/data # cat file 
start 1
start 2
```



## pod日志查询

```shell
kubectl logs pod名  [-c]  容器名  -n  名字空间
```

==注意：==

- 在linux中，一般docker的日志文件存储在/var/lib/docker/containers/container_id/ 目录下的 各个容器ID对应的目录下的*-json.log 文件中

- 每天或是每次日志文件达到 10M 大小之后，容器日志会自动轮替，kubectl logs 命令只显示最后一次轮替后的日志（通过docker配置文件可以设置日志文件大小）

## 端口转发port-forward

创建一个pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.9
    ports:
    - containerPort: 80

```

创建pod之后，将本地的8080端口映射到pod容器内的80端口即可从本地访问到容器

```shell
[root@master test]# kubectl port-forward nginx 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
# 终端会一直停留在此，监听8080端口，使用ctrl + c结束
```

访问nginx

```shell
[root@master ~]# curl 127.0.0.1:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```

## pod标签

### I. 标签的基本操作（增删改查）

当集群种pod较多的时候，可以通过labels来有效地区分和管理pod。通过标签将pod分组，这样可以对同一个分组的pod进行操作，不需要逐一操作每个pod

例：

手动创建两个nginx，分别设置不同的标签

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    creation_method: manual
    rel: beta
spec:
  containers:
  - name: nginx
    image: nginx:1.9
    ports:
    - containerPort: 80
      protocol: TCP
```

结果：

查看pod的标签

```shell
[root@master test]# kubectl get pods --show-labels
NAME      READY   STATUS    RESTARTS   AGE    LABELS
nginx     1/1     Running   0          6m2s   creation_method=manual,rel=beta
nginx-2   1/1     Running   0          21s    creation_method=manual-2,rel=beta-2  
```

查看pod某个具体的标签（即根据标签的key查询value）

```shell
[root@master test]# kubectl get pods -L creation_method,rel
NAME      READY   STATUS    RESTARTS   AGE     CREATION_METHOD   REL
nginx     1/1     Running   0          6m23s   manual            beta
nginx-2   1/1     Running   0          42s     manual-2          beta-2
```

添加或修改标签

```shell
[root@master test]# kubectl label pod nginx creation_method=manual-1 --overwrite
pod/nginx labeled
[root@master test]# kubectl get pods -L creation_method
'NAME      READY   STATUS    RESTARTS   AGE     CREATION_METHOD
nginx     1/1     Running   0          12m     manual-1
nginx-2   1/1     Running   0          6m21s   manual-2

```

为整个namespace的pod设置标签

```shell
# Update all pods in the namespace
kubectl label pods --all key=value
```

删除一个标签

```shell
[root@master test]# kubectl get pods --show-labels 
NAME      READY   STATUS    RESTARTS   AGE   LABELS
nginx     1/1     Running   0          22m   creation_method=manual-1,rel=beta
nginx-2   1/1     Running   0          16m   creation_method=manual-2,rel=beta-2
[root@master test]# kubectl label pod nginx rel-    # 删除rel标签，在其后加减号
pod/nginx labeled
[root@master test]# kubectl get pods --show-labels 
NAME      READY   STATUS    RESTARTS   AGE   LABELS
nginx     1/1     Running   0          23m   creation_method=manual-1
nginx-2   1/1     Running   0          17m   creation_method=manual-2,rel=beta-2
```

### II. 标签选择器(使用小写L)

标签选择器使用的3种方式：

- 包含或不包含使用特定键的标签
- 包含具有特定键和值的标签
- 包含有特定键、值可以为任意值的标签

筛选出所有手动创建的pod：

```shell
[root@master test]# kubectl get pods --show-labels 
NAME      READY   STATUS    RESTARTS   AGE   LABELS
nginx     1/1     Running   0          47m   creation_method=auto,rel=beta-1
nginx-2   1/1     Running   0          42m   creation_method=manual,rel=beta-2
[root@master test]# 
[root@master test]# 
[root@master test]# kubectl get pods -l creation_method=manual
NAME      READY   STATUS    RESTARTS   AGE
nginx-2   1/1     Running   0          42m

```

列出含有rel标签的所有pod：

```shell
[root@master test]# kubectl get pods --show-labels 
NAME      READY   STATUS    RESTARTS   AGE   LABELS
nginx     1/1     Running   0          50m   creation_method=auto
nginx-2   1/1     Running   0          45m   creation_method=manual,rel=beta-2
[root@master test]# kubectl get pod -l rel
NAME      READY   STATUS    RESTARTS   AGE
nginx-2   1/1     Running   0          45m


```

列出不含有rel标签的所有pod：

```shell
[root@master test]# kubectl get pods --show-labels 
NAME      READY   STATUS    RESTARTS   AGE   LABELS
nginx     1/1     Running   0          50m   creation_method=auto
nginx-2   1/1     Running   0          45m   creation_method=manual,rel=beta-2

[root@master test]# kubectl get pod -l '!rel'  # 必须使用单引号
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          51m

```

其他方式：

```shell
# 列出含有 creation_method 标签，但是其值不能为 manual 的 Pod
kubectl get pod -l 'creation_method!=manual'
# 列出含有 rel 标签，且其值为 beta 或是 stable 的 Pod
kubectl get pod -l 'rel in (beta,stable)'
# 列出含有 rel 标签，且其值不为 beta 和 stable 的 Pod
kubectl get pod -l 'rel notin (beta,stable)'
# 列出含有标签 creation_method=manual 和 rel=stable 的 Pod
kubectl get pod -l creation_method=manual,rel=stable
```

### III. 约束pod调度（nodeSelector）

例：

将pod调度到node标签为gpu=true的node上

1. 设置node标签

```shell
[root@master test]# kubectl label nodes node1 gpu=true
node/node1 labeled
[root@master test]# kubectl get nodes -l gpu=true
NAME    STATUS   ROLES    AGE    VERSION
node1   Ready    <none>   120d   v1.15.3

```

2. 创建pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  nodeSelector:
    gpu: "true"
  containers:
  - name: busybox
    image: busybox:1.28
    command:
    - /bin/sh
    args:
    - -c
    - echo Hello world;sleep 3600

```

结果busybox被调度到node1上：

```shell
[root@master test]# kubectl get pods -o wide 
NAME      READY   STATUS    RESTARTS   AGE   IP              NODE    NOMINATED NODE   READINESS GATES
busybox   1/1     Running   0          68s   10.10.166.128   node1   <none>           <none>
nginx     1/1     Running   0          14h   10.10.166.186   node1   <none>           <none>
nginx-2   1/1     Running   0          14h   10.10.166.187   node1   <none>           <none>

```

## 练习

创建一个完整的 Pod, 要求包含如下信息：

- requests 和 limits （给一个容器配置即可）
- 自定义的 command 和 args （给一个容器配置即可）
- Liveness 的健康检查 (EXEC/HTTP/TCP 均可) （给一个容器配置即可）
- 两个容器 （一个要求持续运行，能够查看 initContainers 里写入的数据，一个正常退出，往 volume 里写入数据，并能在另一个容器里看到）
- 一个 initContainers （往 volume 里写入数据)

例：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  volumes:
  - name: share-data
    emptyDir: {}
  initContainers:
  - name: busybox-1
    image: busybox:1.28
    command:
    - /bin/sh
    args:
    - -c
    - echo "I'm initcontainer named busybox" > /tmp/test-init
    volumeMounts:
    - name: share-data
      mountPath: /tmp/
  containers:
  - name: busybox-2
    image: busybox
    command:
    - /bin/sh
    args:
    - -c
    - echo "I'm busybox" > /haha/test;ping 127.0.0.1
    volumeMounts:
    - name: share-data
      mountPath: /haha/
    resources:
      limits:
        cpu: 200m
        memory: 500Mi
      requests:
        cpu: 200m
        memory: 500Mi
    livenessProbe:
      exec:
        command:
        - cat
        - /haha/test
      initialDelaySeconds: 5
      periodSeconds: 5
```



# ReplicationController

仅支持==基于相等的标签选择器==

## 1. 工作原理

当 pod 数量过多时，ReplicationController 会终止多余的 pod。当 pod 数量太少时，ReplicationController 将会启动新的 pod。 与手动创建的 pod 不同，由 ReplicationController 创建的 pod 在失败、被删除或被终止时会被自动替换。 例如，在中断性维护（如内核升级）之后，您的 pod 会在节点上重新创建。 因此，即使您的应用程序只需要一个 pod，您也应该使用 ReplicationController 创建 Pod。 ReplicationController 类似于进程管理器，但是 ReplicationController 不是监控单个节点上的单个进程，而是监控跨多个节点的多个 pod

## 2. 示例

例：

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80

```

完整的yaml

```yaml
apiVersion: v1    # 与pod相同，而不是deployment的apps/v1
kind: ReplicationController
metadata:
  annotations: {省略}
  creationTimestamp: "2020-04-07T03:47:03Z"
  generation: 1
  labels:   # 如果没有设置标签，默认使用template.metadata.labels
    app: nginx
  name: nginx
  namespace: default
  resourceVersion: "3307039"
  selfLink: /api/v1/namespaces/default/replicationcontrollers/nginx
  uid: d73c17de-e33e-4052-b66a-e7c236512d31
spec:
  replicas: 3    # 不设置，默认值是1
  selector:     # 标签选择器与service的设置相同，不是matchLabels
    app: nginx
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
      name: nginx
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always 
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always    # 只能是Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30

```



## 3. 使用方法

### 1. 删除一个rc以pod

```shell
# kubectl delete  rc   rc名称  -n  名字空间
```

### 2. 只删除rc不影响pod

```shell
# kubectl delete  rc   rc名称  -n  名字空间  --cascade=false
```

### 3. 从rc中隔离pod

在rc生成的pod实例中，修改pod的labels，就可以将pod从rc的管理中移除（此时rc会新创建一个pod），主要用于删除pod进行调试、数据恢复等

![image-20200407125503557](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200407125503557.png)

### 4. 裸pod运行

当只运行一个pod时，我们也建议使用rc部署，因为它可以自动创建因某些原因被删除或被终止的pod。











# ReplicaSet（不建议单独使用）

是ReplicationController的升级版，支持新的==基于集合的标签选择器==，主要被Deployment用来作为一种编排pod创建、删除及更新的机制。

官方建议使用Deployment而不是直接使用ReplicaSet，除非你需要自定义更新编排或根本不需要更新。



# Deloyments

管理ReplicaSet和Pod，功能更加强大，推荐使用。

## 1. deployment创建

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.4
        ports:
        - containerPort: 80

```

重点关注spec部分：

- replicas：pod的数量，也就是创建这个deployment之后他会创建几个pod
- selector：deployment管理pod，可以理解为pod的一个父资源。一个namespace下有很多个pod，deployment通过selector字段，使用常见的标签关联方式，管理自己的pod。在本例中表示：任何有app:nginx标签的pod，都归该deployment管理。
- template：deployment的定义其实就是pod的定义加上其他的属性。这个template就是deployment启动的pod应该长什么样子的定义，它跟单独的一个pod的定义非常像
  - labels：定义了每个pod应该有什么样的标签，这个是和selector一起使用的。
  - spec：里面的内容和pod的spec部分是完全相同的，主要内容也是container列表

运行结果：

```shell
[root@master test]# kubectl get deploy
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploymennt   3/3     3            3           15m
[root@master test]# kubectl get pods 
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deploymennt-9b644dcd5-6nmlq   1/1     Running   0          16m
nginx-deploymennt-9b644dcd5-g97sf   1/1     Running   0          16m
nginx-deploymennt-9b644dcd5-zgvrg   1/1     Running   0          16m
[root@master test]# kubectl get pods --show-labels
NAME                               READY   STATUS    RESTARTS   AGE    LABELS
nginx-deployment-9b644dcd5-766qk   1/1     Running   0          108m   app=nginx,pod-template-hash=9b644dcd5
nginx-deployment-9b644dcd5-g645j   1/1     Running   0          108m   app=nginx,pod-template-hash=9b644dcd5
nginx-deployment-9b644dcd5-t7tbs   1/1     Running   0          108m   app=nginx,pod-template-hash=9b644dcd5

```

各个字段的涵义如下：

- NAME: deployment 的名字
- CURRENT: 当前有多少个 Pod 处于运行中
- UP-TO-DATE: 达到最新状态的 Pod 的数量。当 Deployment 在进行更新时，会有新老版本的 Pod 同时存在，这时候这个字段会比较有用
- AVAILABLE: 可用的 Pod。上个实验我们讲了健康检查的相关配置，Pod 运行中和可以提供服务是不同的概念
- AGE: deployment 运行的时间

--show-labels：

- NAME: 格式为: ` <deployment name> - <id> - <random string> ` 的结构。每次更新了 deployment 里面的 pod template 部分， `<id>` 值也会对应更新。
- labels: 我们在 deployment 的 yaml 里指定的标签，也有一个自动添加的用于标记其版本的 pod-template-hash, 同一个 pod template 生成的 pod 的 hash 都是一样的值。

完整的deployment：

```yaml
[root@master test]# kubectl get deploy nginx-deployment -o yaml 
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app":"nginx"},"name":"nginx-deployment","namespace":"default"},"spec":{"replicas":3,"selector":{"matchLabels":{"app":"nginx"}},"template":{"metadata":{"labels":{"app":"nginx"}},"spec":{"containers":[{"image":"nginx:1.15.4","name":"nginx","ports":[{"containerPort":80}]}]}}}}
  creationTimestamp: "2020-03-24T04:14:13Z"
  generation: 1
  labels:
    app: nginx
  name: nginx-deployment
  namespace: default
  resourceVersion: "1485012"
  selfLink: /apis/extensions/v1beta1/namespaces/default/deployments/nginx-deployment
  uid: 08e33a26-9a5e-462e-95c9-914ec2b54364
spec:
  progressDeadlineSeconds: 600
  replicas: 3
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: nginx
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.15.4
        imagePullPolicy: IfNotPresent
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 3
  conditions:
  - lastTransitionTime: "2020-03-24T04:14:17Z"
    lastUpdateTime: "2020-03-24T04:14:17Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2020-03-24T04:14:13Z"
    lastUpdateTime: "2020-03-24T04:14:17Z"
    message: ReplicaSet "nginx-deployment-9b644dcd5" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 3
  replicas: 3
  updatedReplicas: 3

```

- apiVersion： 注意这里的APIVersion和我们填写的完全不一样。这是因为除了属于core group的resource（pod，configmap）其他很多resource的定义都是不断变化的。大致都有一个alpha-beta-stable的阶段。这个过程中，group和version都有可能发生变化。在每个kubernetes版本中，每个resource都会有一个推荐的默认版本，所有的资源不管以什么版本创建，都会被修改成这个默认版本。目前deployment/stateful/daemonset等最新的apiVersion是apps/v1，但kubernetes 1.10版本时的默认版本是extensions/v1beta1。deployment等workload的版本变化比较多，好在具体的yaml结构变化并不大。

- progressDeadlineSeconds：过了多久时间Deployment会被认为失败。这时候controller-manager会停止继续部署deployment

- revisionHistoryLimit：历史版本，用于更新及回滚

- selector：基于集合的标签选择器

  - matchLabels

  - matchExpressions：包含3个部分：key、operator、values列表

    ```yaml
    selector:
      matchExpressions:
        - key: app
          operator: In
          values:
            - nginx
        ......
        - key:
    ```

    operator运算符一般有4种：

    - In：标签的值必须与其中一个指定的values匹配
    - NotIn：标签的值必须与指定的values不匹配
    - Exists：Pod必须包含一个指定名称的标签（值不重要），这时候就不需要指定values
    - DoesNotExist：Pod不能够包含有指定名称的标签，这时候也不需要指定values

  - ==如果指定了多个表达式或者同时混合使用了matchLabels和matchExpressions属性，这表达式必须都为true才能和pod进行匹配。==

- strategy（策略）：更新策略。默认为RollingUpdate，滚动更新

  - maxSurge：就是最多会多部署多少个pod。假设一个deployment有10个pod，当更新时，可以最多先启动4个新版本的pod，然后等他们成功后，再去kill旧版本的pod。

  - maxUnavailable：最多有几个pod不可用。假设一个deployment有10个pod。当更新时，最多可以先kill掉4个旧的pod，然后启动新的pod

  - type：RollingUpdate。更新策略，RollingUpdate是默认选项且是最优选项，其他的选项都是一些老版本策略，并不适用了。

    RollingUpdate的设置能够适用于多种集群资源的状态，富余时可以先启动新的pod，资源紧张时可以先kill老的pod。这些配置能够最大限度的保证deployment更新成功。



deployment的status部分与pod类似——>基本状态 + condition的结构。



## 2. deployment更新

deployment的yaml包含两部分：

- 属于deployment自己的，比如它的基本信息： metadata、spec、标签等
- 属于pod的信息——>template里面的内容

==更新deployment自身的信息，template的内容没有发生变化，正在运行的pod不需要更新。==

==如果更新了template的内容，所有的pod都需要删掉并创建新的pod，才会记录revision版本信息==

使用命令更新pod：

```yaml
kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.9.1 --record
```

--record表示：会将上面这条kubectl更新命令写入到deployment的yaml文件中的annotations里：

![image-20200324170356577](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200324170356577.png)

更新结果：

```shell
[root@master test]# kubectl get pods --show-labels
NAME                               READY   STATUS              RESTARTS   AGE     LABELS
nginx-deployment-9b644dcd5-766qk   1/1     Running             0          5h1m    app=nginx,pod-template-hash=9b644dcd5
nginx-deployment-9b644dcd5-t7tbs   1/1     Running             0          5h1m    app=nginx,pod-template-hash=9b644dcd5
nginx-deployment-dcff897bd-qk62f   0/1     ContainerCreating   0          9m52s   app=nginx,pod-template-hash=dcff897bd
nginx-deployment-dcff897bd-rxft6   1/1     Running             0          15m     app=nginx,pod-template-hash=dcff897bd


```

更新之后，template  hash随之更新了

## 3. deployment回滚



deployment本身保存和很多历史版本信息（具体多少条可以配置，参考revisionHistoryLimit），我们可以查看下：

```shell
[root@master test]# kubectl rollout history deployment.v1.apps/nginx-deployment
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.9.1 --record=true

```

history里面CHANGE-CAUSE字段就保存了我们用--record记录的更新原因。



如果使用kubectl  edit直接修改deployment，也会有history记录——>只是没有更新的内容记录，显示none：

```shell
[root@master test]# kubectl rollout history deployment nginx-deployment   # 也可以“--reversion=版本号”查看指定版本的历史信息
deployment.extensions/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

```



回滚到上一个版本：

```shell
kubectl rollout undo deployment nginx-deployment
```

回滚到特定版本：

```shell
kubectl rollout undo deployment nginx-deployment --to-revision=2
```



## 4. deployment水平扩容缩容

### 手动扩容

deployment的replicas指定了目标实例数。但是在实际运行过程中，我们经常会有调整是实例数的需求。比如业务的高峰期和低峰期，临时性的debug等。

- 可以使用kubectl  edit来编辑spec.replicas字段

- 也可以使用kubectl  scale命令操作

  ```shell
  [root@master test]# kubectl scale deployment  nginx-deployment  --replicas=5
  deployment.extensions/nginx-deployment scaled
  
  ```



### 自动扩容

Pod 水平自动伸缩（Horizontal Pod Autoscaler）特性， 可以基于CPU利用率自动伸缩 replication controller、deployment和 replica set 中的 pod 数量，（除了 CPU 利用率）也可以 基于其他应程序提供的度量指标[custom metrics](https://git.k8s.io/community/contributors/design-proposals/instrumentation/custom-metrics-api.md)（自定义指标）。 pod 自动缩放不适用于无法缩放的对象，比如 DaemonSets

自动伸缩原理：

![水平自动伸缩示意图](https://d33wubrfki0l68.cloudfront.net/4fe1ef7265a93f5f564bd3fbb0269ebd10b73b4e/1775d/images/docs/horizontal-pod-autoscaler.svg)

pod的水平自动伸缩式一个控制循环，由controller manager的`--horizontal-pod-autoscaler-sync-period` 参数指定周期（默认值为15秒）。每个周期内，controller manager 根据每个 HorizontalPodAutoscaler 定义中指定的指标查询资源利用率。 controller manager 可以从 以下接口获取资源指标

resource metrics API（每个pod 资源指标）：通过metric-server实现（heapster监控已经废弃），负责采集Node、Pod的核心资源数据

custom metrics API（其他指标）：通常使用prometheus实现，负责自定义的指标数据采集，比如：网卡流量、磁盘IOPS、HTTP请求数、数据库连接数等。

#### 扩容算法

autoscaler控制器从聚合API获取到Pod性能指标数据之后，通过下面的算法计算目标pod副本数量：

```shell
desiredReplicas = ceil[currentReplicas * ( currentMetricValue / desiredMetricValue )]
目标副本数量 = 当前副本数 * (当前指标值/期望指标值)
```



注意上面公式的计算结果需要向上取整，然后将目标副本数量与当前副本数量进行对比，就可以判断是否需要进行扩容或是缩容。

我们以 CPU 请求数量为例，假设当前系统运行了一个 Pod 副本，用户设置的期望值为 100m：

- 如果当前实际使用的指标值为 200m，则：目标副本数量 = 1 * (200/100) = 2，也就是期望 Pod 副本数量为 2，这个时候就会进行 Pod 扩容
- 如果当前实际使用的指标值为 50m，则：目标副本数量 = 1 * (50/100) = 0.5，将计算结果向上取整为 1，也就是期望 Pod 副本数量为 1，这个时候就不会改变 Pod 的数量

当计算结果与 1 非常接近时，可以设置一个容忍度使系统不做扩缩容处理，通过 kube-controller-manager 服务的启动参数 `--horizontal-pod-autoscaler-tolerance` 进行设置，该参数的默认值为 0.1(10%)，所以上述算法计算结果在正负 10% 的区间中都不会执行扩缩容操作。

期望指标值（desiredMetricValue）也可以是指标的平均值，比如：targetAverageValue，这时当前指标值（currentMetricValue）的算法为：

```shell
当前指标值 = 所有 Pod 副本当前指标值总和 / Pod 副本数量
```




## 5. 暂停和恢复deployment

避免频繁的deployment更新操作（适用于kubectl命令行更新），可以先暂停deployment，待所有更新的命令（如更新镜像信息，resources资源信息等）都执行完之后，再恢复deployment

如：

```shell
# 更新镜像
# kubectl set image deployment/nginx-deployment nginx=nginx：1.9.1
# 更新resources
# kubectl set resources deployment nginx-deployment -c=nginx --limits=cpu=200m,memory=512Mi
```



暂停命令：

```shell
# kubectl rollout pause deploy  deploy名称 -n 名称空间
```

恢复命令：

```shell
# kubectl rollout resume deploy  deploy名称 -n 名称空间
```

==注意：==

在恢复暂停的deployment之前，无法回滚deployment

# StatefulSets













# DaemonSet

==ds资源文件与deployment的唯一区别就是不用设置replicas==

提供特定node节点监控或日志收集等功能，这些pod的生命周期与服务器的生命周期绑定，它们需要在启动之前在node上运行，在节点准备重启或者关闭时安全地终止。

与deployment一样，daemonset也是一种workload，用来管理多个pod，但是使用场景不同。

主要特点：

- 每个node上都有一个实例，且只有一个。如果有新的node节点加入，daemonset也会在新的节点上起一个新的pod
- 不能扩缩容，因为它的实例个数是随node数量的



应用场景：

- 各类需要在每个机器上部署agent的组件，比如监控、日志、存储。这些组件一般都是每个机器都需要有，本身自己也不需要保存持久状态，一般都是向外部汇报数据。

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: logging
spec:
  selector:
    matchLabels:
      app: logging-app
  template:
    metadata:
      labels:
        app: logging-app
    spec:
      nodeSelector:
        app: logging-node
      containers:
      - name: webserver
        image: nginx
        ports:
        - containerPort: 80

```

注意：

- 没有replicas字段
- spec.nodeSelector：pod与主机标签的匹配。在kubernetes中，标签广泛应用于各种resource中，所有的resource都可以有标签，某些resource在binding的过程中，可以通过标签来进行匹配。匹配的概念就是说目标resource的标签必须包含selector中的全部标签。



由于设置了nodeSelector，所以在创建daemonset之前，需要将待创建daemonset的node节点打上nodeSelector设置的标签，否则会因为找不到对应标签的node而创建失败。

```shell
# --overwrite 表示如果标签已经存在，强制覆盖
[root@master test]# kubectl label nodes node2 app=logging-node --overwrite  
node/node2 labeled

```

创建daemonset：

```shell
[root@master test]# kubectl apply -f test.yaml 
daemonset.apps/logging created
[root@master test]# 
[root@master test]# kubectl get pods -o wide 
NAME            READY   STATUS              RESTARTS   AGE     IP              NODE    NOMINATED NODE   READINESS GATES
logging-7knbb   1/1     Running             0          5m11s   10.10.166.131   node1   <none>           <none>
logging-7svqq   0/1     ContainerCreating   0          5m11s   <none>          node2   <none>           <none>


```



# jobs

执行一个可完成的任务，当进程终止后pod不再重新启动。

kubernetes支持以下几种job：

- 非并行job：通常创建一个pod直至其成功结束
- 固定结束次数的job：设置spec.completions，创建多个pod，知道spec.completions个pod成功结束
- 带有工作队列的并行job：设置spec.parallelism但不设置spec.completions，当所有pod结束并且至少一个成功时，job就认为是成功。

==注：==

- restartPolicy必须设置为OnFailure或者Never
- bare pods也可以使用job，这样能保证节点重启之后，pod能继续运行。

一个简单的job：

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    metadata:
      name: pi
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```

固定结束次数的job：

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: busybox
spec:
  completions: 3
  template:
    metadata:
      name: busybox
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ["echo", "hello"]
      restartPolicy: Never
```

带有工作队列的job：

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: busybox
spec:
  parallelism: 3    # 将completions替换为parallelism即可
  template:
    metadata:
      name: busybox
    spec:
      restartPolicy: Never
      containers:
      - name: busybox
        image: busybox:1.28
        command: ["echo", "hello"]
```

工作队列&&固定次数：

```yaml
spec:
  completions: 10
  parallelism: 3
  template:
    ...
```



# CronJob







# 垃圾收集

