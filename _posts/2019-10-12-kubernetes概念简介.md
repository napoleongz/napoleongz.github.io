## 1、Master

  Kubernetes里的Master指的是集群控制节点， 在每个Kubernetes集群里都需要有一个Master来负责整个集群的管理和控制， 基本上Kubernetes的所有控制命令都发给它， 它负责具体的执行过程， 几乎所有命令基本都是在Master上运行的。 Master通常会占据一个独立的服务器（高可用部署建议用3台服务器），主要原因是它太重要
了， 是整个集群的“首脑”， 如果它宕机或者不可用， 那么对集群内容器应用的管理都将失效。
  在master上运行了kube-apiserver、kube-controller-manager、kube-scheduler关键进程，另外在Master上通常还需要部署etcd服务， 因为Kubernetes里的所
有资源对象的数据都被保存在etcd中。

### 1）、kube-apiserve
  Kubernetes API Server的核心功能是提供Kubernetes各类资源对象（如Pod、 RC、 Service等） 的增、 删、 改、 查及Watch等HTTPRest接口， 成为集群内各功能模块之间数据交互和通信的中心枢纽，是整个系统的数据总线和数据中心。 除此之外， 它还有以下一些功能特性。
（1） 是集群管理的API入口。
（2） 是资源配额控制的入口。
（3） 提供了完备的集群安全机制。
### 2）、kube-controller-manager
  Controller Manager是Kubernetes中各种操作系统的管理者， 是集群内部的管理控制中心， 也是Kubernetes自动化功能的核心。
  Controller Manager内部包含Replication Controller、Node Controller、 ResourceQuota Controller、 Namespace Controller、ServiceAccountController、 Token Controller、 Service Controller及Endpoint Controller这8种Controller， 每种Controller都负责一种特定资源的控制流程， 而Controller Manager正是这些Controller的核心管理者。
### 3）、kube-scheduler
  Kubernetes Scheduler在整个系统中承担了“承上启下”的重要功能， “承上”是指它负责接收Controller Manager创建的新Pod， 为其安排一个落脚的“家”—目Node； “启下”是指安置工作完成后， 目标Node上的kubelet服务进程接管后继工作， 负责Pod生命周期中的“下半生”。
  Kubernetes Scheduler的作用是将待调度的Pod（API新创建的Pod、 Controller Manager为补足副本而创建的Pod等） 按照特定的调度算法和调度策略绑定（Binding） 到集群中某个合适的Node上， 并将绑定信息写入etcd中。 在整个调度过程中涉及三个对象， 分别是待调度Pod列表、 可用Node列表， 以及调度算法和策略。 简单地说， 就是通过调度算法调度为待调度Pod列表中的每个Pod从Node列表中选择一个最适合的Node。
  随后， 目标节点上的kubelet通过API Server监听到KubernetesScheduler产生的Pod绑定事件， 然后获取对应的Pod清单， 下载Image镜像并启动容器。

## 2、Node
  除了Master， Kubernetes集群中的其他机器被称为Node。 与Master一样， Node可以是一台物理主机， 也可以是一台虚拟机Node是Kubernetes集群中的工作负载节点， 每个Node都会被Master分配一些工作负载（Docker容器） ，当某个Node宕机时，其上的工作负载会被Master自动转移到其他节点上。
在Node节点上运行了kubelet、kube-proxy、docker关键进程。Node可以在运行期间动态增加到Kubernetes集群中， 前提是在这个节点上已经正确安装、 配置和启动了上述关键进程， 在默认情况下kubelet会向Master注册自己， 这也是Kubernetes推荐的Node管理方式。一旦Node被纳入集群管理范围， kubelet进程就会定时向Master汇报自身的情报， 例如操作系统、 Docker版本、 机器的CPU和内存情况， 以及当前有哪些Pod在运行等， 这样Master就可以获知每个Node的资源使用情况， 实现高效均衡的资源调度策略。 而某个Node在超过指定时间不上报信息时， 会被Master判定为“失联”， Node的状态被标记为不可用（Not Ready） ， 随后Master会触发“工作负载大转移”的自动流程。
 
### 1）、kubelet
  在Kubernetes集群中， 在每个Node上都会启动一个kubelet服务进程。 该进程用于处理Master下发到本节点的任务， 管理Pod及Pod中的容器。 每个kubelet进程都会在API Server上注册节点自身的信息， 定期向Master汇报节点资源的使用情况， 并通过cAdvisor监控容器和节点资源。
  总结之，kubelet主要有节点管理、Pod管理、容器健康检查、cAdvisor资源监控功能。

### 2）、kube-proxy
  在Kubernetes集群的每个Node上都会运行一个kube-proxy服务进程， 我们可以把这个进程看作Service的透明代理兼负载均衡器， 其核心功能是将到某个Service的访问请求转发到后端的多个Pod实例上。 此外， Service的Cluster IP与NodePort等概念是kube-proxy服务通过iptables的NAT转换实现的， kube-proxy在运行过程中动态创建与Service相关的iptables规则， 这些规则实现了将访问服务（Cluster IP或NodePort） 的请求负载分发到后端Pod的功能。 由于iptables机制针对的是本地的kubeproxy端口， 所以在每个Node上都要运行kube-proxy组件， 这样一来， 在Kubernetes集群内部， 我们可以在任意Node上发起对Service的访问请求。 综上所述， 由于kube-proxy的作用， 在Service的调用过程中客户端无须关心后端有几个Pod， 中间过程的通信、 负载均衡及故障恢复都是透明的。
  
## 3、Pod
  Pod是一组紧密关联的容器集合，是 Kubernetes 项目中的最小编排单位,支持多个容器在一个Pod中共享网络和文件系统，可以通过进程间通信和文件共享这种简单高效的方式完成服务，是Kubernetes调度的基本单位。Pod的设计理念是每个Pod都有一个唯一的IP。
Pod 生命周期的变化，主要体现在 Pod API 对象的Status 部分，这是它除了 Metadata 和 Spec之外的第三个重要字段。其中，pod.status.phase，就是 Pod 的当前状态，它有如下几种可能的情况：
1. Pending。这个状态意味着，Pod 的 YAML 文件已经提交给了 Kubernetes，API 对象已经被创建并保存在 Etcd 当中。但是，这个 Pod 里有些容器因为某种原因而不能被顺利创建。比如，调度不成功。
2. Running。这个状态下，Pod 已经调度成功，跟一个具体的节点绑定。它包含的容器都已经创建成功，并且至少有一个正在运行中。
3. Succeeded。这个状态意味着，Pod 里的所有容器都正常运行完毕，并且已经退出了。这种情况在运行一次性任务时最为常见。
4. Failed。这个状态下，Pod 里至少有一个容器以不正常的状态（非 0 的返回码）退出。这个状态的出现，意味着你得想办法 Debug 这个容器的应用，比如查看 Pod 的 Events 和日志。
5. Unknown。这是一个异常状态，意味着 Pod 的状态不能持续地被 kubelet 汇报给 kubeapiserver，这很有可能是主从节点（Master 和 Kubelet）间的通信出现了问题。

## 4、Label
  Label（标签） 是Kubernetes系统中另外一个核心概念。 一个Label是一个key=value的键值对， 其中key与value由用户自己指定。 Label可以被附加到各种资源对象上， 例如Node、 Pod、 Service、 RC等， 一个资源对象可以定义任意数量的Label， 同一个Label也可以被添加到任意数量的资源对象上。 Label通常在资源对象定义时确定， 也可以在对象创建后动态添加或者删除。  
  我们可以通过给指定的资源对象捆绑一个或多个不同的Label来实现多维度的资源分组管理功能， 以便灵活、 方便地进行资源分配、 调度、 配置、 部署等管理工作。 例如， 部署不同版本的应用到不同的环境中； 监控和分析应用（日志记录、 监控、 告警） 等。
  Label相当于我们熟悉的“标签”。 给某个资源对象定义一个Label，就相当于给它打了一个标签， 随后可以通过Label Selector（标签选择器） 查询和筛选拥有某些Label的资源对象， Kubernetes通过这种方式实现了类似SQL的简单又通用的对象查询机制。
  Label Selector可以被类比为SQL语句中的where查询条件， 例如，name=redis-slave这个Label Selector作用于Pod时， 可以被类比为select *from pod where pod’s name =‘redis-slave’这样的语句。 当前有两种LabelSelector表达式： 基于等式的（Equality-based） 和基于集合的（Setbased） ， 前者采用等式类表达式匹配标签。


## 5、Replication Controller
  Replication Controller（简称RC）是Kubernetes系统中的核心概念之一， 简单来说， 它其实定义了一个期望的场景， 即声明某种Pod的副本数量在任意时刻都符合某个预期值。
  RC主要有Pod期待的副本数量、用于筛选目标Pod的Label Selector、当Pod的副本数量小于预期数量时， 用于创建新Pod的Pod模板（template）三部分组成
  
## 6、Deployment
  Deployment可以看着为RC的一次升级。Deployment相对于RC的一个最大升级是我们可以随时知道当前Pod“部署”的进度。 实际上由于一个Pod的创建、 调度、 绑定节点及在目标Node上启动对应的容器这一完整过程需要一定的时间， 所以我们期待系统启动N个Pod副本的目标状态， 实际上是一个连续变化的“部署过程”导致的最终状态。
  其典型的应用场景包括：
  
    定义Deployment来创建Pod和ReplicaSet
    滚动升级和回滚应用
    扩容和缩容
    暂停和继续Deployment
  
  常用的操作命令如下：

    kubectl run www--image=10.0.0.183:5000/hanker/www:0.0.1--port=8080 生成一个Deployment对象
    kubectlgetdeployment--all-namespaces 查找Deployment
    kubectl describe deployment www 查看某个Deployment
    kubectl edit deployment www 编辑Deployment定义
    kubectldeletedeployment www 删除某Deployment
    kubectl scale deployment/www--replicas=2 扩缩容操作，即修改Deployment下的Pod实例个数
    kubectlsetimage deployment/nginx-deployment nginx=nginx:1.9.1更新镜像
    kubectl rollout undo deployment/nginx-deployment 回滚操作
    kubectl rollout status deployment/nginx-deployment 查看回滚进度
    kubectl autoscale deployment nginx-deployment--min=10--max=15--cpu-percent=80 启用水平伸缩（HPA - horizontal pod autoscaling），设置最小、最大实例数量以及目标cpu使用率
    kubectl rollout pause deployment/nginx-deployment 暂停更新Deployment
    kubectl rollout resume deploy nginx 恢复更新Deployment


  更新策略.spec.strategy 指新的Pod替换旧的Pod的策略，有以下两种类型
  
    RollingUpdate 滚动升级，可以保证应用在升级期间，对外正常提供服务。
    Recreate 重建策略，在创建出新的Pod之前会先杀掉所有已存在的Pod。
 
## 7、DaemonSet 守护进程集


 DaemonSet保证在特定或所有Node节点上都运行一个Pod实例，常用来部署一些集群的日志采集、监控或者其他系统管理应用。典型的应用包括:

    日志收集，比如fluentd，logstash等
    系统监控，比如Prometheus Node Exporter，collectd等
    系统程序，比如kube-proxy, kube-dns, glusterd, ceph，ingress-controller等

 指定Node节点DaemonSet会忽略Node的unschedulable状态，有两种方式来指定Pod只运行在指定的Node节点上:

    nodeSelector:只调度到匹配指定label的Node上
    nodeAffinity:功能更丰富的Node选择器，比如支持集合操作
    podAffinity:调度到满足条件的Pod所在的Node上


目前支持两种策略

    OnDelete: 默认策略，更新模板后，只有手动删除了旧的Pod后才会创建新的Pod
    RollingUpdate: 更新DaemonSet模版后，自动删除旧的Pod并创建新的Pod
 
## 8、StatefulSet
Deployments和ReplicaSets是为无状态服务设计的，那么StatefulSet则是为了有状态服务而设计，其应用场景包括：

    稳定的持久化存储，即Pod重新调度后还是能访问到相同的持久化数据，基于PVC来实现
    稳定的网络标志，即Pod重新调度后其PodName和HostName不变，基于Headless Service(即没有Cluster IP的Service)来实现
    有序部署，有序扩展，即Pod是有顺序的，在部署或者扩展的时候要依据定义的顺序依次进行操作(即从0到N-1，在下一个Pod运行之前所有之前的Pod必须都是Running和Ready状态)，基于init containers来实现
    有序收缩，有序删除(即从N-1到0)


支持两种更新策略：

    OnDelete:当 .spec.template更新时，并不立即删除旧的Pod，而是等待用户手动删除这些旧Pod后自动创建新Pod。这是默认的更新策略，兼容v1.6版本的行为
    RollingUpdate:当 .spec.template 更新时，自动删除旧的Pod并创建新Pod替换。在更新时这些Pod是按逆序的方式进行，依次删除、创建并等待Pod变成Ready状态才进行下一个Pod的更新。
    
        
## 9、Service
  Service是对一组提供相同功能的Pods的抽象，并为他们提供一个统一的入口，借助 Service 应用可以方便的实现服务发现与负载均衡，并实现应用的零宕机升级。Service通过标签(label)来选取后端Pod，一般配合ReplicaSet或者Deployment来保证后端容器的正常运行。
service 有如下四种类型，默认是ClusterIP：

    ClusterIP: 默认类型，自动分配一个仅集群内部可以访问的虚拟IP
    NodePort: 在ClusterIP基础上为Service在每台机器上绑定一个端口，这样就可以通过 NodeIP:NodePort 来访问该服务
    LoadBalancer: 在NodePort的基础上，借助cloud provider创建一个外部的负载均衡器，并将请求转发到 NodeIP:NodePort
    ExternalName: 将服务通过DNS CNAME记录方式转发到指定的域名

  另外，也可以将已有的服务以Service的形式加入到Kubernetes集群中来，只需要在创建 Service 的时候不指定Label selector，而是在Service创建好后手动为其添加endpoint。
  
## 10、job
  Job负责批量处理短暂的一次性任务 (short lived>CronJob即定时任务，就类似于Linux系统的crontab，在指定的时间周期运行指定的任务。它与RC、Deployment、 ReplicaSet、 DaemonSet类似， Job也控制一组Pod容器。 从这个角度来看， Job也是一种特殊的Pod副本自动控制器， 同时Job控制Pod副本与RC等控制器的工作机制有以下重要差别。
    
## 11、Horizontal Pod Autoscaler
  Horizontal Pod Autoscaling可以根据CPU、内存使用率或应用自定义metrics自动扩展Pod数量 (支持replication controller、deployment和replica set)。

    控制管理器默认每隔30s查询metrics的资源使用情况(可以通过 --horizontal-pod-autoscaler-sync-period 修改)
    支持三种metrics类型
        预定义metrics(比如Pod的CPU)以利用率的方式计算
        自定义的Pod metrics，以原始值(raw value)的方式计算
        自定义的object metrics
    支持两种metrics查询方式:Heapster和自定义的REST API
    支持多metrics

可以通过如下命令创建HPA：kubectl autoscale deployment php-apache--cpu-percent=50--min=1--max=10



## 12、Volume
  Volume（存储卷） 是Pod中能够被多个容器访问的共享目录。Kubernetes的Volume概念、 用途和目的与Docker的Volume比较类似， 但两者不能等价。 首先，Kubernetes中的Volume被定义在Pod上， 然后被一个Pod里的多个容器挂载到具体的文件目录下； 其次， Kubernetes中的Volume与Pod的生命周期相同， 但与容器的生命周期不相关， 当容器终止或者重启时， Volume中的数据也不会丢失。 最后， Kubernetes支持多种类型的Volume， 例如GlusterFS、 Ceph等先进的分布式文件系统。
  Volume的使用也比较简单， 在大多数情况下， 我们先在Pod上声明一个Volume， 然后在容器里引用该Volume并挂载（Mount） 到容器里的某个目录上。
  Kubernetes存储卷的生命周期与Pod绑定

    容器挂掉后Kubelet再次重启容器时，Volume的数据依然还在
    Pod删除时，Volume才会清理。数据是否丢失取决于具体的Volume类型，比如emptyDir的数据会丢失，而PV的数据则不会丢

  目前Kubernetes主要支持以下Volume类型：

    emptyDir：Pod存在，emptyDir就会存在，容器挂掉不会引起emptyDir目录下的数据丢失，但是pod被删除或者迁移，emptyDir也会被删除
    hostPath：hostPath允许挂载Node上的文件系统到Pod里面去
    NFS（Network File System）：网络文件系统，Kubernetes中通过简单地配置就可以挂载NFS到Pod中，而NFS中的数据是可以永久保存的，同时NFS支持同时写操作。
    glusterfs：同NFS一样是一种网络文件系统，Kubernetes可以将glusterfs挂载到Pod中，并进行永久保存
    cephfs：一种分布式网络文件系统，可以挂载到Pod中，并进行永久保存
    subpath：Pod的多个容器使用同一个Volume时，会经常用到
    secret：密钥管理，可以将敏感信息进行加密之后保存并挂载到Pod中
    persistentVolumeClaim：用于将持久化存储（PersistentVolume）挂载到Pod中

## 13、Persistent Volume
  PersistentVolume(PV)是集群之中的一块网络存储。跟 Node 一样，也是集群的资源。PersistentVolume (PV)和PersistentVolumeClaim (PVC)提供了方便的持久化卷: PV提供网络存储资源，而PVC请求存储资源并将其挂载到Pod中。
  PV的访问模式(accessModes)有三种:

    ReadWriteOnce(RWO):是最基本的方式，可读可写，但只支持被单个Pod挂载。
    ReadOnlyMany(ROX):可以以只读的方式被多个Pod挂载。
    ReadWriteMany(RWX):这种存储可以以读写的方式被多个Pod共享。

  不是每一种存储都支持这三种方式，像共享方式，目前支持的还比较少，比较常用的是 NFS。在PVC绑定PV时通常根据两个条件来绑定，一个是存储的大小，另一个就是 访问模式。
  PV的回收策略(persistentVolumeReclaimPolicy)也有三种

    Retain，不清理保留Volume(需要手动清理)
    Recycle，删除数据，即 rm -rf /thevolume/* (只有NFS和HostPath支持)
    Delete，删除存储资源
  
## 14、Namespace
  Namespace（命名空间）是对一组资源和对象的抽象集合，比如可以用来将系统内部的对象划分为不同的项目组或者用户组,Kubernetes集群在启动后会创建一个名为default的Namespace。常见的pod、service、replicaSet和deployment等都是属于某一个namespace的(默认是default)，而node, persistentVolumes等则不属于任何namespace。

  常用namespace操作：

    kubectl get namespace, 查询所有namespace
    kubectl create namespace-name，创建namespace
    kubectl delete namespace-name, 删除namespace


  删除命名空间时，需注意以下几点：

    删除一个namespace会自动删除所有属于该namespace的资源。
    default 和 kube-system 命名空间不可删除。
    PersistentVolumes是不属于任何namespace的，但PersistentVolumeClaim是属于某个特定namespace的。
    Events是否属于namespace取决于产生events的对象。
  
## 15、Annotaion
  Annotation（注解） 与Label类似， 也使用key/value键值对的形式进行定义。 不同的是Label具有严格的命名规则， 它定义的是Kubernetes对象的元数据（Metadata） ， 并且用于Label Selector。 Annotation则是用户任意定义的附加信息， 以便于外部工具查找。 在很多时候， Kubernetes的模块自身会通过Annotation标记资源对象的一些特殊信息。
  通常来说， 用Annotation来记录的信息如下。
  
    build信息、 release信息、 Docker镜像信息等， 例如时间戳、
    release id号、 PR号、 镜像Hash值、 Docker Registry地址等。
    日志库、 监控库、 分析库等资源库的地址信息。
    程序调试工具信息， 例如工具名称、 版本号等。
    团队的联系信息， 例如电话号码、 负责人名称、 网址等。
    
## 16、ConfigMap
  ConfigMap用于保存配置数据的键值对，可以用来保存单个属性，也可以用来保存配置文件。ConfigMap跟secret很类似，但它可以更方便地处理不包含敏感信息的字符串。ConfigMap可以通过三种方式在Pod中使用，三种分别方式为:设置环境变量、设置容器命令行参数以及在Volume中直接挂载文件或目录。

 可以使用 kubectl create configmap从文件、目录或者key-value字符串创建等创建 ConfigMap。也可以通过 kubectl create-f value.yaml 创建。
  
  
## 17、Ingress

  Kubernetes中的负载均衡我们主要用到了以下两种机制：

    Service：使用Service提供集群内部的负载均衡，Kube-proxy负责将service请求负载均衡到后端的Pod中
    Ingress Controller：使用Ingress提供集群外部的负载均衡


  Service和Pod的IP仅可在集群内部访问。集群外部的请求需要通过负载均衡转发到service所在节点暴露的端口上，然后再由kube-proxy通过边缘路由器将其转发到相关的Pod，Ingress可以给service提供集群外部访问的URL、负载均衡、HTTP路由等，为了配置这些Ingress规则，集群管理员需要部署一个Ingress Controller，它监听Ingress和service的变化，并根据规则配置负载均衡并提供访问入口。
常用的ingress controller：

    nginx
    traefik
    Kong
    Openresty
    
## 18、Resource Quotas 资源配额

  资源配额(Resource Quotas)是用来限制用户资源用量的一种机制。
资源配额有如下类型：

    计算资源，包括cpu和memory
        cpu, limits.cpu, requests.cpu
        memory, limits.memory, requests.memory
    存储资源，包括存储资源的总量以及指定storage class的总量
        requests.storage:存储资源总量，如500Gi
        persistentvolumeclaims:pvc的个数
        storageclass.storage.k8s.io/requests.storage
        storageclass.storage.k8s.io/persistentvolumeclaims
    对象数，即可创建的对象的个数
        pods, replicationcontrollers, configmaps, secrets
        resourcequotas, persistentvolumeclaims
        services, services.loadbalancers, services.nodeports

  它的工作原理为:

    资源配额应用在Namespace上，并且每个Namespace最多只能有一个 ResourceQuota 对象
    开启计算资源配额后，创建容器时必须配置计算资源请求或限制(也可以 用LimitRange设置默认值)
    用户超额后禁止创建新的资源
