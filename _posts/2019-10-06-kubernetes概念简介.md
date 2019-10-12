# kubernetes概念简介
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
