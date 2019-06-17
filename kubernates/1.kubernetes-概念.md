## 概念

概念部分可以帮助你了解 Kubernetes 的各个组成部分以及 Kubernetes 用来表示集群的一些抽象概念，并帮助你更加深入的理解 Kubernetes 是如何工作的。

* [**概述**](#概述)
* [ **Kubernetes 对象** ](# Kubernetes 对象)
* [**Kubernetes 控制面**](#Kubernetes 控制面)
* [**接下来**](#接下来)

## 概述

要使用 Kubernetes，你需要使用 Kubernates API 对象来描述集群的预期状态 (desired state) ：包括你需要运行的应用或者负载，它们使用的镜像、副本数，以及所需网络和磁盘资源等等。你可以使用命令行工具 `kubectl` 来调用 Kubernetes API 创建对象，通过所创建的这些对象来配置预期状态。你也可以直接调用 Kubernetes API 和集群进行交互，设置或者修改预期状态。

一旦你设置了所需的目标状态， Kubernetes 控制面 (control plane) 会促成集群的当前状态符合其预期状态。为此， Kubernetes 会自动执行各类任务，比如运行或者重启容器、调整给定应用的副本数等等。

Kubernetes 控制面由一组运行在集群上的进程组成：

- **Kubernetes 主控组件（Master）** 包含三个进程，都运行在集群中的某个节上，通常这个节点被称为 master 节点。这些进程包括：[kube-apiserver](https://kubernetes.io/docs/admin/kube-apiserver/)、[kube-controller-manager](https://kubernetes.io/docs/admin/kube-controller-manager/) 和 [kube-scheduler](https://kubernetes.io/docs/admin/kube-scheduler/)。
- 集群中的每个非 master 节点都运行两个进程：
  - **[kubelet](https://kubernetes.io/docs/admin/kubelet/),**  和 master 节点进行通信。
  - **[kube-proxy](https://kubernetes.io/docs/admin/kube-proxy/),** 一种网络代理，将 Kubernetes 的网络服务代理到每个节点上。

## Kubernetes 对象

Kubernetes 包含若干抽象用来表示系统状态，包括：已部署的容器化应用和负载、与它们相关的网络和磁盘资源以及有关集群运行的其他操作信息。这些抽象使用 Kubernetes API 对象来表示。参阅 **[Kubernetes 对象概述](https://kubernetes.io/docs/concepts/abstractions/overview/)** 来了解详细信息。

基本的 Kubernetes 对象包括：

	* [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)
	* [Service](https://kubernetes.io/docs/concepts/services-networking/service/)
	* [Volume](https://kubernetes.io/docs/concepts/storage/volumes/)
	* [Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

另外， Kubernetes 包含大量的被称作 *控制器 (controllers)*  高级抽象。控制器基于基本对象构建并提供额外的功能和方便使用的特性。具体包括：

* [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)
* [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
* [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
* [Job](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/)

## Kubernetes 控制面

关于 Kubernetes 控制平面的各个部分， 如 Kubernetes 主控组件和 kubelet 进程，管理着 Kubernetes 如何与你的集群进行通信。控制平面维护着系统中所有的 Kubernetes 对象的状态记录，并且通过连续的控制循环来管理这些对象的状态。在任一的给定时间点，控制面的控制环都能相应集群中的变化，并且让系统中所有对象的实际状态与你提供的预期状态项匹配。

比如，当你通过 Kubernetes API 创建一个 Deployment 对象，你就为系统增加了一个新的目标状态。

Kubernetes 控制平面记录着对象的创建，并启动必要的应用然后将他们调度至集群的某个节点上来执行你的指令，以此来保持集群的实际状态和目标状态的匹配。

**Kubernetes Master 节点**

Kubernetes master 节点负责维护集群的目标状态。当你与 Kubernetes 通信时，使用如 `kubectl` 的命令行工具，就可以直接与 Kubernetes master 节点进行通信。

"master" 是指管理集群状态的一组进程的集合。通常这些进程都跑在集群中的一个单独的节点上，并且这个几点被称为 master 几点。 master 节点也可以扩展副本数，来获取更好的性能及冗余。

**Kubernates Node 节点**

集群中的 node 节点 (虚拟机、物理机等等) 都是用来运行你的应用和云工作流的机器。 Kubernetes master 节点控制所有 node 节点；你很少需要和 node 节点进行直接通信。

**对象元数据**

* [注释](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/)

## 接下来



