- ## kubernates

  ### kubernetes是什么

  ##### kubernetes是一个可移植的，可扩展的开源平台，用于管理容器化的工作负载和服务，方便了声明式配置和自动化。

  ##### 名字起源于希腊语，意为舵手或飞行员，k8s缩写是因为k和s之间有8个字符

  #### 为什么要用kubernetes

  - 资源分配问题
    - 传统时代：物理机上如果部署多个应用，容易出现一个应用占用资源很多，导致其他应用性能下降的情况。
    - 虚拟化部署时代：虚拟化技术允许在单个物理机的CPU上运行多个虚拟机，允许应用程序在VM之间隔离，提供一定程度的安全。实现更好的伸缩性，降低硬件成本。每个VM就是一个独立的计算机，独立的操作系统
    - 容器化部署时代：容器类似于VM，但是容器之间是可以共享操作系统的，容器有自己的文件系统、CPU、内存、进程空间等，但是由于和基础架构分离，所以可以跨云和OS发行版本进行移植。
    - 为什么有虚拟化技术还需要容器化技术？容器的优势？
      - 敏捷应用的创建和部署：与VM镜像相比，提高的容器创建的简洁性和效率
      - 持续开发、集成部署：通过快速的回滚（镜像不可变性），支持可靠频繁的容器构建和部署
      - 开发和运维分离：在构建/发布时创建应用程序的镜像而不是部署时，从而将应用程序和基础架构分离
      - 可观察性：不仅显示操作系统级别的信息和指标，还可以显示应用程序的运行状况和其他指标信息
      - 0开发、测试、生产环境的一致性：镜像是一样的，不同环境执行相同的指令都可以运行
      - 跨云和操作系统的可移植性：可以在任何地方运行
      - 以应用程序为中心的管理：提高抽象级别
      - 松散耦合、分布式、弹性、解放的微服务：单独的微服务可以单独的运行了
      - 资源隔离：可以做到应用程序间的隔离
      - 资源利用：高效率和高密度
  - 服务发现和负载均衡：kubernetes可以使用DNS名称或者自己的IP地址公开容器，如果进入容器的流量很大，k8s可以负载均衡分配网络流量，从而使部署稳定
  - 存储编排：k8s允许你自动挂载到你选择的存储系统，例如本地存储、公共云提供商（COS、NFS等）
  - 自动部署和回滚：可以使用k8s描述已部署容器的所需状态，可以以受控的速率将实际状态更改为期望状态
  - 自动完成装箱计算：k8s允许指定每个容器所需要的CPU和内存（RAM），当指定资源的时候，k8s可以做出更好的决策管理容器的资源
  - 自我修复：k8s会重新启动失败的容器、替换容器、杀死不响应用户定义的运行状态检查的容器，并且在容器准备好之前不将其返回给客户端
  - 密钥与配置管理：k8s允许存储敏感信息，可以在不重建容器的情况下更新秘钥和应用程序配置，可以不在堆栈中暴露密钥

  #### kubernetes特点

  - 不限制支持的应用程序类型
  - 不部署源代码，也不构建应用程序，集成、部署、交付工作流取决于组织偏好
  - 不提供应用程序级别的服务作为内置服务，服务之间可以通过可移植机制（例如开放服务代理）来访问
  - 不要求日志记录、监视、或警报解决方案
  - 不提供或者不要求配置语言、系统
  - 不提供也不采用任何全面的机器配置、维护、管理和自我修复系统
  - 不仅仅是一个编排系统，实际上它消除了编排的需要

  ### kubernetes组件

  ![Kubernetes 组件](https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg)

  #### 控制平面组件（Control Plane Components)

  #####  控制平面组件对集群做出全局决策（比如调度），以及检测和响应集群事件。控制平面组件可以在集群的任何节点上运行，但是一般会把所有的控制平面组件运行在一个机器上，并且这个机器不会运行其他容器，一般就是master节点上

  - kube-apiserver

    ##### API服务时K8s的请求入口服务

    ##### APi服务时kubernetes控制面的前端，公开了kubernetes API。apiserver是可伸缩的，可以部署apiserver的多个实例，在这些实例之间负载、平衡流量。所有组件都通过该前端进行交互

  - etcd

    ##### K8s的存储服务

    ##### etcd是一致性、高可用的键值数据库，可以用来保存kubernetes所有集群数据的后台数据库

  - kube-scheduler

  ##### k8s所有worker Node的调度器

  ##### 运行控制器进程的控制平面组件，负责监视新创建的、未指定运行节点的pods，选择节点让pod在上面运行。

  ##### 调度决定因素包括单个Pod和Pod集合的资源请求、硬件/软件/策略约束、亲和性和反亲和性规范、数据位置、工作负荷间的干扰和最后时限。

  - kube-controller-manager

    ##### K8s所有worker Node的监视器

    ##### 运行控制器进程的控制平面组件。控制器包括下面这些：

    - 节点控制器（Node Controller）：负责在节点出现故障时进行通知和响应
    - 任务控制器（job controller）：监测代表一次性任务的job对象，然后创建pods来运行这些任务直至完成
    - 端点控制器（Endpoints Controller)：填充端点（Endpoints)对象（即加入Service与Pod)
    - 服务账户和令牌控制器（Service Account & Token Controller)：为新的命名空间创建默认账户和API访问令牌

  - cloud-controller-manager

    ##### 云控制管理器是指嵌入特定云的控制逻辑的控制平面组件。云控制管理器使得你可以将你的集群连接到云提供商的API上，并将于该平台交互的组件于你集群交互的组件分离开。

    ##### 于kube-controller-manager类似，cloud-controller-manager将若干逻辑独立的控制回路组合放到同一个可执行文件中，下面的控制器都包含对云平台驱动的依赖：

    - 节点控制器（Node Controller)：用于再节点终止响应后检查云提供商以确定节点是否已被删除
    - 路由控制器（Route Controller)：用于在底层云基础架构中设置路由
    - 服务控制器（Service Controller)：用于创建、更新和删除云提供商负载均衡器

  ### Node 组件

  ##### 节点组件在每个节点上运行，维护运行的pod并提供kubernetes运行环境

  - kubelet

    ##### Woker Node的监视器，以及与Master Node的通讯器，定期向Master节点汇报自己Node上运行服务状态，并接受指示调整

    ##### 一个在集群中每个节点上运行的代理，它保证容器都运行在pod中

    ##### kubelet接收一组通过各类机制提供给它的PodSpecs，保证这些PodsSpecs中描述的容器处于运行状态且健康

  - kube-proxy

    ##### k8s的网络代理

    ##### kube-proxy中集群中每个节点上运行的网络代理，维护节点上的网络规则。这些网络规则允许从集群内部或外部的网络会话与Pod进行网络通讯。

    ##### proxy 不仅知道自己节点的网络，他知道整个集群的网络。

    ##### 如果操作系统提供了数据包过滤层并可用的话，kube-proxy会通过它来实现网络规则，否则，kube-proxy仅转发流量本身。

  - 容器运行时（Container Runtime)

    ##### Worker Node的运行环境

    ##### 容器运行环境是负责运行容器的软件

    ##### kubernetes支持多个容器运行环境：Docker、containerd、CRI-O以及任何实现Kubernetes CRI（容器运行环境接口）。

    

  ### 插件

    - DNS
    - Web界面（仪表盘）
    - 容器资源监控
    - 集群层面日志

  

  ### 部署一个应用各个组件的调用情况

  - 首先命令发送到master节点的网关apiserver
  - apiserver将命令请求交给controller mannager进行应用部署解析
  - controller mannager会生成一次部署信息，并通过apiserver将信息存储到etcd中
  - scheduler调度器通过apiserver从etcd中，拿到要部署的应用，开始调度那个节点有资源去部署
  - scheduler把计算出来的调度信息通过apiserver再放回到etcd中
  - 每一个node节点的监控组件kubelet，随时和master保存联系（给apiserver发送请求不断的拿数据），拿到master节点存储在etcd中的部署信息
  - 当某个节点的kubelet拿到部署信息，发现是自己要部署应用，kubelet会自己启动一个应用在当前机器上，并随时给master节点汇报当前应用的状态信息

    

### Kubernetes API

##### Kubernetes 控制面的核心是API server。API server负责提供HTTP API，以供用户、集群中的不同部分和集群外部组件相互通信。

##### Kubernetes API使你可以查询和操控kubernetes API中对象（例如Pod、Namespace、ConfigMap、Event）的状态

##### 大部分操作都可以通过kubectl命令行接口或者类似命令行工具来执行，或者用REST请求调用、客户端库调用，其实所有的方式最后都是调用API server

#### OpenAPI规范

- 完整的API细节是用OpenAPI来表述的。API服务器通过/openapi/v2末端提供OpenAPI规范
- Kubernetes为API实现了一种基于Protobuf的序列化格式，主要用于集群内部通信。



#### API约定

- 详细说明：https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md
- 各个对象操作文档：https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.22/#-strong-write-operations-pod-v1-core-strong-





#### API变更

- kubernetes保证在项目不断的更新中，不会引发现有客户端的兼容性，并在一定时间内维护这种兼容性，以便其他项目有机会做出适应性变更。

#### API组和版本

- 为了简化操作，kubernetes支持多个API版本，每个版本在不同API路径下，例如/api/v1。
- 版本化是在API级别而不是资源级别，这样的目的是保证清晰、一致的视图，所以多个版本访问的资源是一致的



#### API扩展

- 可以使用自定义资源来以声明式方式定义API服务器如何提供你选择的资源API
- 也可以选择实现自己的聚合层来扩展API



##### 官方支持的客户端库包括以下这些：

- dotnet：[ github.com/kubernetes-client/csharp](https://github.com/kubernetes-client/csharp)
- Go：[ github.com/kubernetes/client-go/](https://github.com/kubernetes/client-go/)
- Haskell：[github.com/kubernetes-client/haskell](https://github.com/kubernetes-client/haskell)
- Java：[github.com/kubernetes-client/java](https://github.com/kubernetes-client/java/)
- JavaScript：[github.com/kubernetes-client/javascript](https://github.com/kubernetes-client/javascript)
- Python：[github.com/kubernetes-client/python/](https://github.com/kubernetes-client/python/)



##### 社区维护的客户端库包括以下这些：

- Clojure	github.com/yanatan16/clj-kubernetes-api
- DotNet	github.com/tonnyeremin/kubernetes_gen
- DotNet (RestSharp)	github.com/masroorhasan/Kubernetes.DotNet
- Elixir	github.com/obmarg/kazan
- Elixir	github.com/coryodaniel/k8s
- Go	github.com/ericchiang/k8s
- Java (OSGi)	bitbucket.org/amdatulabs/amdatu-kubernetes
- Java (Fabric8, OSGi)	github.com/fabric8io/kubernetes-client
- Java	github.com/manusa/yakc
- Lisp	github.com/brendandburns/cl-k8s
- Lisp	github.com/xh4/cube
- Node.js (TypeScript)	github.com/Goyoo/node-k8s-client
- Node.js	github.com/ajpauwels/easy-k8s
- Node.js	github.com/godaddy/kubernetes-client
- Node.js	github.com/tenxcloud/node-kubernetes-client
- Perl	metacpan.org/pod/Net::Kubernetes
- PHP	github.com/allansun/kubernetes-php-client
- PHP	github.com/maclof/kubernetes-client
- PHP	github.com/travisghansen/kubernetes-client-php
- PHP	github.com/renoki-co/php-k8s
- Python	github.com/fiaas/k8s
- Python	github.com/mnubo/kubernetes-py
- Python	github.com/tomplus/kubernetes_asyncio
- Python	github.com/Frankkkkk/pykorm
- Ruby	github.com/abonas/kubeclient
- Ruby	github.com/Ch00k/kuber
- Ruby	github.com/k8s-ruby/k8s-ruby
- Ruby	github.com/kontena/k8s-client
- Rust	github.com/clux/kube-rs
- Rust	github.com/ynqa/kubernetes-rust
- Scala	github.com/hagay3/skuber
- Scala	github.com/joan38/kubernetes-client
- Swift	github.com/swiftkube/client



#### kubernetes client 和 fabric8io/kubernetes-client 区别

- kubernetes client是官方的，更新频率比较高，社区不活跃
- fabric8io是社区版的，比较活跃，但是更新慢



### fabric8io/kubernetes-client

#### 用法

- 创建客户端

  - KubernetesClient client = new DefaultKubernetesClient();

- 配置客户端

  ##### 将按以下优先级顺序使用来自不同源的设置：

  - 系统属性

  - 环境变量

  - kube配置文件

  - 服务账户令牌和挂载的CA证书

    ##### 或者可以使用ConfigBuilder为Kubernetes客户端创建配置对象：config = new ConfigBuilder().withMasterUrl()；

- 通过客户端可以获取资源列表、资源、删除、编辑等操作

- 监听事件

<details>
<summary>监听代码</summary>

  ```Java
        client.events().inAnyNamespace().watch(new Watcher<Event>() {
  @Override
  public void eventReceived(Action action, Event resource) {
    System.out.println("event " + action.name() + " " + resource.toString());
  }
  @Override
  public void onClose(KubernetesClientException cause) {
    System.out.println("Watcher close due to " + cause);
  }
});
  ```
 </details>

- 使用扩展

  - 通过kubernetes API定义的jobs、ingresses这些都是可以获取的，例如：jobs = client.batch().jobs().list()；

- 从外部资源加载资源

  - 客户端允许从下列位置加载资源：

    - 一个文件
    - 一个网址
    - 一个输入流

  - 读取之后可以直接使用，比如：

    ```java
    Pod refreshed = client.load('/path/to/a/pod.yml').fromServer().get();
    Boolean deleted = client.load('/workspace/pod.yml').delete();
    LogWatch handle = client.load('/workspace/pod.yml').watchLog(System.out);
    ```

- 将资源的引用传递给客户端

  - 可以使用外部创建的对象，例如：

  ```java
  Pod pod = someThirdPartyCodeThatCreatesAPod();
  Boolean deleted = client.resource(pod).delete();
  ```

- 客户端支持可插入适配器
- 模拟Kubernetes，除了客户端，该项目还提供了了kubernetes模拟服务器，可以用于测试。模拟服务器基于mockwebserver，mockwebserver支持两种操作模式：
  - 期望模式
  - CRUD模式
- 通过扩展支持Junit5



#### 谁在使用这个客户端



#### Kubectl Java 等价物

| kubectl                                                      | Fabric8 Kubernetes Client                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `kubectl config view`                                        | [ConfigViewEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/ConfigViewEquivalent.java) |
| `kubectl config get-contexts`                                | [ConfigGetContextsEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/ConfigGetCurrentContextEquivalent.java) |
| `kubectl config current-context`                             | [ConfigGetCurrentContextEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/ConfigGetCurrentContextEquivalent.java) |
| `kubectl config use-context minikube`                        | [ConfigUseContext.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/ConfigUseContext.java) |
| `kubectl config view -o jsonpath='{.users[*].name}'`         | [ConfigGetCurrentContextEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/ConfigGetCurrentContextEquivalent.java) |
| `kubectl get pods --all-namespaces`                          | [PodListGlobalEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/PodListGlobalEquivalent.java) |
| `kubectl get pods`                                           | [PodListEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/PodListEquivalent.java) |
| `kubectl get pods -w`                                        | [PodWatchEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/PodWatchEquivalent.java) |
| `kubectl get pods --sort-by='.metadata.creationTimestamp'`   | [PodListGlobalEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/PodListGlobalEquivalent.java) |
| `kubectl run`                                                | [PodRunEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/PodRunEquivalent.java) |
| `kubectl create -f test-pod.yaml`                            | [PodCreateYamlEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/PodCreateYamlEquivalent.java) |
| `kubectl exec my-pod -- ls /`                                | [PodExecEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/PodExecEquivalent.java) |
| `kubectl delete pod my-pod`                                  | [PodDelete.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/PodDelete.java) |
| `kubectl delete -f test-pod.yaml`                            | [PodDeleteViaYaml.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/PodDeleteViaYaml.java) |
| `kubectl cp /foo_dir my-pod:/bar_dir`                        | [UploadDirectoryToPod.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/UploadDirectoryToPod.java) |
| `kubectl cp my-pod:/tmp/foo /tmp/bar`                        | [DownloadFileFromPod.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/DownloadFileFromPod.java) |
| `kubectl cp my-pod:/tmp/foo -c c1 /tmp/bar`                  | [DownloadFileFromMultiContainerPod.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/DownloadFileFromMultiContainerPod.java) |
| `kubectl cp /foo_dir my-pod:/tmp/bar_dir`                    | [UploadFileToPod.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/UploadFileToPod.java) |
| `kubectl logs pod/my-pod`                                    | [PodLogsEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/PodLogsEquivalent.java) |
| `kubectl logs pod/my-pod -f`                                 | [PodLogsFollowEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/PodLogsFollowEquivalent.java) |
| `kubectl logs pod/my-pod -c c1`                              | [PodLogsMultiContainerEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/PodLogsMultiContainerEquivalent.java) |
| `kubectl port-forward my-pod 8080:80`                        | [PortForwardEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/PortForwardEquivalent.java) |
| `kubectl get pods --selector=version=v1 -o jsonpath='{.items[*].metadata.name}'` | [PodListFilterByLabel.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/PodListFilterByLabel.java) |
| `kubectl get pods --field-selector=status.phase=Running`     | [PodListFilterFieldSelector.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/PodListFilterFieldSelector.java) |
| `kubectl get pods --show-labels`                             | [PodShowLabels.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/PodShowLabels.java) |
| `kubectl label pods my-pod new-label=awesome`                | [PodAddLabel.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/PodAddLabel.java) |
| `kubectl annotate pods my-pod icon-url=http://goo.gl/XXBTWq` | [PodAddAnnotation.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/PodAddAnnotation.java) |
| `kubectl get configmap cm1 -o jsonpath='{.data.database}'`   | [ConfigMapJsonPathEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/ConfigMapJsonPathEquivalent.java) |
| `kubectl create -f test-svc.yaml`                            | [LoadAndCreateService.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/LoadAndCreateService.java) |
| `kubectl create -f test-deploy.yaml`                         | [LoadAndCreateDeployment.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/LoadAndCreateDeployment.java) |
| `kubectl set image deploy/d1 nginx=nginx:v2`                 | [RolloutSetImageEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/RolloutSetImageEquivalent.java) |
| `kubectl scale --replicas=4 deploy/nginx-deployment`         | [ScaleEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/ScaleEquivalent.java) |
| `kubectl rollout restart deploy/d1`                          | [RolloutRestartEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/RolloutRestartEquivalent.java) |
| `kubectl rollout pause deploy/d1`                            | [RolloutPauseEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/RolloutPauseEquivalent.java) |
| `kubectl rollout resume deploy/d1`                           | [RolloutResumeEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/RolloutResumeEquivalent.java) |
| `kubectl rollout undo deploy/d1`                             | [RolloutUndoEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/RolloutUndoEquivalent.java) |
| `kubectl create -f test-crd.yaml`                            | [LoadAndCreateCustomResourceDefinition.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/LoadAndCreateCustomResourceDefinition.java) |
| `kubectl create -f customresource.yaml`                      | [CustomResourceCreateDemo.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/CustomResourceCreateDemo.java) |
| `kubectl create -f customresource.yaml`                      | [CustomResourceCreateDemoTypeless.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/CustomResourceCreateDemoTypeless.java) |
| `kubectl get ns`                                             | [NamespaceListEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/NamespaceListEquivalent.java) |
| `kubectl apply -f test-resource-list.yml`                    | [CreateOrReplaceResourceList.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/CreateOrReplaceResourceList.java) |
| `kubectl get events`                                         | [EventsGetEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/EventsGetEquivalent.java) |
| `kubectl top nodes`                                          | [TopEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/TopEquivalent.java) |
| `kubectl auth can-i create deployment.apps`                  | [CanIEquivalent.java](https://github.com/fabric8io/kubernetes-client/blob/master/kubernetes-examples/src/main/java/io/fabric8/kubernetes/examples/kubectl/equivalents/CanIEquivalent.java) |





#### kubernetes 对象

- kubernetes对象是持久化的实体，kubernetes使用这些实体去表示整个集群的状态，如果：容器运行在哪个节点、哪些资源可以被使用、应用运行时的策略

- 通过创建对象可以告诉kubernetes系统自己需要的状态是什么样的，也就是kubernetes集群的期望状态

- 操作对象是通过API实现的

##### 对象字段

- Spec 对象约束：创建对象的时候必须设置Spec内容，用来描述自己的期望状态
- status 状态：status字段描述了当前状态，kubernetes控制平面管理对象的实际状态，通过更新状态使之与期望状态匹配。
- apiVersion：创建该对象所使用的的kubernetes API版本
- kind：想要创建的对象类型
- metedata： 帮助唯一性标识对象的一些数据，包括name、UID、namespace

##### 对象管理

- 指令式命令，命令的方式直接对对象进行操作，例如：kubectl create deployment nginx --image nginx
- 指令式对象配置，通过yaml文件或者JSON格式配置对象的属性，通过命令指定文件，例如：kubectl create -f nginx.yaml、kubectl replace -f nginx.yaml
- 声明式对象配置，可以在目录下工作，根据目录中配置文件对不同的对象执行不同的操作。在目录下，可以有多个对象配置文件，kubectl会自动检测每个文件的创建、更新、删除操作。例如：kubectl diff -f configs/、kubectl apply -f configs/

##### 三种对象管理方式的优缺点

- 指令式优点：上手简单、操作简单
- 指令式缺点：命令不与流程集成、不提供更改记录、不提供记录源、不提供创建新对象的模板
- 指令式配置优点：配置与流程集成、配置可以存储在git上、提供创建新对象的模板
- 指令式配置缺点：需要单独配置yaml文件、只能操作单个对象、对对象的跟新必须反映到配置文件中，否则更新时候会丢失
- 声明式配置优点：对对象做的更新即使没有合并到配置文件中，也会被保留、支持目录操作
- 声明式配置缺点：调试困难，出现异常不易解决、使用diff产生的更新会创建复杂的合并和补丁操作

##### 对象名称和IDS

- 名称，客户端提供的字符串，同一个类型的对象名称不能相同

  ##### 资源命名约束

  - DNS子域名
  - DNS标签名
  - 路径分段名称

- UIDS，kubernetes系统生成的字符串，唯一标识对象

##### 命名空间

namespace其实就是将同一个物理集群拆分为多个虚拟集群，这些虚拟集群就叫做命名空间。

只有在用户量很大的情况下才需要考虑使用命名空间，否则没必要

命名空间之间是隔离的，命名空间内的资源名称是唯一的

##### kubernetes会创建四个初始命名空间

- default：默认命名空间，没有指定其他命名空间时候默认用的
- kube-system：kubernetes创建对象使用的命名空间
- kube-public：公用命名空间，集群内的所有用户都可以读取他，主要用于集群之间，以防某些资源在整个集群中应该是可见的
- kube-node-lease：这个命名空间用于与各个节点相关的租期对象，在集群规模大的时候可以提升心跳检测性能

##### 命名空间相关命令

- 查看命名空间：kubectl get namespace
- 为当前请求设置命名空间：kubectl run nginx --image=nginx --namespace=<实际命名空间>
- 设置当前命名空间，后面的命令自动在这个命名空间执行：kubectl config set-context --current --namespace=<实际命名空间>，验证：kubectl config view | grep namespace;
- 查看哪些对象在命名空间中，哪些不在：kubectl api-resources --namespaced=true



#### 标签和选择算符 labels

标签（labels）是附加到对象上的键值对，用于指定对用户有意义的对象的标识属性。对象中键唯一，不同对象可以重复。

##### 动机

标签是为了使用户能够以松耦合的方式将自己的组织结构映射到系统对象中去

##### 示例标签

“environment”：“dev”

“tire":"cache"

##### 语法和字符集

标签是键值对，有效的标签有两段，可选的前缀和名称，用斜杠/分隔，名称是必需的，前缀是可选的，如果指定，前缀必须是DNS子域，由点.分隔。

如果省略前缀，则默认标签键值对是私有的。向系统组件（kube-scheduler、kube-controller-manager）加标签必须加前缀

##### 标签选择算符

通过标签选择算符，可以识别一组对象。主要有两种类型选择算符：

- 等值，支持 = 、==、!= ，例如：tier != cache,tier=b
- 集合，支持in、notin、exists，例如：tier in (a,b)



##### 标签在API中的使用

- 命令行

  - kubectl get pods --show-labels
  - kubectl get pods -l tier=a,public=b
  - kubectl get pods -l 'tier in (a,b)'

- YAML

  - selector: tier:a

  - ```yaml
    selector:
      matchLabels:
        component: redis
      matchExpressions:
        - {key: tier, operator: In, values: [cache]}
        - {key: environment, operator: NotIn, values: [dev]}
    ```



#### 注解 annotations

可以使用注解为对象附加非标识的元数据，客户端可以获取到这些元数据信息

##### 语法和字符集

与标签一致

##### 注解和标签的区别

- 标签是用来标识对象的，需要用标签对对象做过滤筛选，注解只是存放一些元数据信息，比如构建信息、git地址、联系电话等，不做筛选用



#### finalizers

finalizers用来标记删除资源前要执行的任务，在删除目标资源时候会回调finalizers，只有finalizers为空才会真正的删除资源。

##### finalizers工作原理

- 创建资源时候，可以指定metadata.finalizers字段
- 删除资源时候，控制器会先修改metadata.deletionTimestamp，然后判断finalizers是否存在
- 如果不为空，控制器会满足finalizers指定的要求，满足一个之后会删除对应的键
- 当finalizers为空的时候可以删除资源

##### finalizers的作用

- 当需要回收的资源失败的时候，可以阻止资源删除以查看问题



#### owner and Dependents

- kubernetes中，某些对象是其他对象的所有者，例如：一个副本集是多个pod的所有者，这些pod对象是所有者的依赖对象

- 对象中通过metadata.ownerReferences引用其所有者对象的字段，有效的所有者引用由对象名称+UID组成。
- kubernetes会自动为依赖于其他对象的对象设置ownerReferences字段，也可以手动设置，一般kubernetes自动管理
- ownerReferences.blockOwnerDeletion字段可以控制特定的从属对象是否可以阻止垃圾收集删除其所有者对象。
- owner的作用是为了控制访问权限、删除权限，防止未经授权的用户删除所有者对象。

#### 使用通用标签描述对象

| 键                             | 描述                                               | 示例                 | 类型   |
| ------------------------------ | -------------------------------------------------- | -------------------- | ------ |
| `app.kubernetes.io/name`       | 应用程序的名称                                     | `mysql`              | 字符串 |
| `app.kubernetes.io/instance`   | 用于唯一确定应用实例的名称                         | `mysql-abcxzy`       | 字符串 |
| `app.kubernetes.io/version`    | 应用程序的当前版本（例如，语义版本，修订版哈希等） | `5.7.21`             | 字符串 |
| `app.kubernetes.io/component`  | 架构中的组件                                       | `database`           | 字符串 |
| `app.kubernetes.io/part-of`    | 此级别的更高级别应用程序的名称                     | `wordpress`          | 字符串 |
| `app.kubernetes.io/managed-by` | 用于管理应用程序的工具                             | `helm`               | 字符串 |
| `app.kubernetes.io/created-by` | 创建该资源的控制器或者用户                         | `controller-manager` | 字符串 |



例如：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: wordpress
    app.kubernetes.io/instance: wordpress-abcxzy
    app.kubernetes.io/version: "4.9.4"
    app.kubernetes.io/managed-by: helm
    app.kubernetes.io/component: server
    app.kubernetes.io/part-of: wordpress
```

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: wordpress
    app.kubernetes.io/instance: wordpress-abcxzy
    app.kubernetes.io/version: "4.9.4"
    app.kubernetes.io/managed-by: helm
    app.kubernetes.io/component: server
    app.kubernetes.io/part-of: wordpress
```

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  labels:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/instance: mysql-abcxzy
    app.kubernetes.io/version: "5.7.21"
    app.kubernetes.io/managed-by: helm
    app.kubernetes.io/component: database
    app.kubernetes.io/part-of: wordpress
```

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/name: mysql
    app.kubernetes.io/instance: mysql-abcxzy
    app.kubernetes.io/version: "5.7.21"
    app.kubernetes.io/managed-by: helm
    app.kubernetes.io/component: database
    app.kubernetes.io/part-of: wordpress
```





### kubernetes架构



































