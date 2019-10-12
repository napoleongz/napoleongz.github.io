# kubernetes入门
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
 
 ###  1）、kubelet
  在Kubernetes集群中， 在每个Node上都会启动一个kubelet服务进程。 该进程用于处理Master下发到本节点的任务， 管理Pod及Pod中的容器。 每个kubelet进程都会在API Server上注册节点自身的信息， 定期向Master汇报节点资源的使用情况， 并通过cAdvisor监控容器和节点资源。
  总结之，kubelet主要有节点管理、Pod管理、容器健康检查、cAdvisor资源监控功能。

 ###  2）、kube-proxy
  在Kubernetes集群的每个Node上都会运行一个kube-proxy服务进程， 我们可以把这个进程看作Service的透明代理兼负载均衡器， 其核心功能是将到某个Service的访问请求转发到后端的多个Pod实例上。 此外， Service的Cluster IP与NodePort等概念是kube-proxy服务通过iptables的NAT转换实现的， kube-proxy在运行过程中动态创建与Service相关的iptables规则， 这些规则实现了将访问服务（Cluster IP或NodePort） 的请求负载分发到后端Pod的功能。 由于iptables机制针对的是本地的kubeproxy端口， 所以在每个Node上都要运行kube-proxy组件， 这样一来， 在Kubernetes集群内部， 我们可以在任意Node上发起对Service的访问请求。 综上所述， 由于kube-proxy的作用， 在Service的调用过程中客户端无须关心后端有几个Pod， 中间过程的通信、 负载均衡及故障恢复都是透明的。
  
## 4、Pod
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
  
  
  

