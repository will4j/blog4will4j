---
slug: "build-k8s-cluster-with-kubekey"
title: "使用 Kubekey 构建 k8s 集群"
date: 2024-02-07T19:42:35+08:00
author:
  name: 水王
tags:
  - k8s
  - kubekey
categories:
  - cloud-native
draft: false
comment: true
description: 使用 Kubekey 工具构建生产环境可用的 k8s 集群。
keywords: ["k8s", "kubernetes", "kubekey", "helm"]
license:
hiddenFromHomePage: false
hiddenFromSearch: false
summary:
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: false
---

自 2014 年 k8s 开源至今，其作为容器编排事实标准的地位已无人能够撼动，围绕着 k8s 发展出来的云原生生态圈枝繁叶茂，云原生全景图[[1]]已经无法在一个屏幕中完整显示，一眼望去尽显开源开放的活力。

要安装体验 k8s 有多种方式[[2]]，开发环境单机版可以选择 Minikube，集群版则有 Kind，生产环境一般使用 Kubeadm 工具安装，鉴于 Kubeadm 需要逐个节点手动配置，可以使用集群自动化部署工具，比如接下来要介绍的使用 Go 语言编写的 Kubekey [[3]]。

## 为什么选择 Kubekey
Kubekey 声称自己要优于其他基于 ansible 的自动化部署工具[[4]]，一方面因为 kubekey 本身基于 Go 构建，云原生生态下的诸多组件（如 helm、docker）都可以通过库依赖直接在编译时打包到执行程序，另一方面，Kubekey 采用将二进制安装包 scp 到远程节点安装的暴力美学方式，优点是简单粗暴又实用，缺点也很明显，没有预置安装包的依赖版本和系统版本无法直接支持。

除此之外，Kubekey 可以说面面俱到：

+ 支持 Ubuntu、CentOS 等多种操作系统；
+ 支持大多数 k8s 版本，从 1.19.x 到最新的 1.29.x，支持单节点、集群、在线、离线部署；
+ 支持 Docker、Containerd、CRI-O 等容器运行环境；
+ 支持 Calico、Flannel、Cilium 等网络插件；
+ 支持 k3s、k8e 等 k8s 扩展版本；
+ 支持节点增删、集群升级、集群卸载、自动集群证书刷新；

Kubekey 可以做到使用裸机一键部署 k8s 集群，并且通过并行化加快部署速度，通过扩展插件丰富集群基础环境。

## 集群配置

可参考示例仓库[[5]]，集群构建脚本的目录结构如下：

```shell
.
├── README.md
├── Taskfile.yaml
├── addons # 插件目录
│   ├── charts # helm charts 目录
│   │   ├── argo-cd-5.53.14.tgz
│   │   ├── cert-manager-v1.14.1.tgz
│   │   ├── csi-driver-nfs-v4.6.0.tgz
│   │   ├── ingress-nginx-4.9.1.tgz
│   │   └── kubernetes-dashboard-7.0.0-alpha1.tgz
│   ├── values # helm values 文件目录
│   │   ├── argo-cd.yaml
│   │   ├── cert-manager.yaml
│   │   ├── csi-driver-nfs.yaml
│   │   ├── ingress-nginx.yaml
│   │   ├── kubernetes-dashboard.yaml
│   │   └── metrics-server.yaml
│   └── yaml # yaml 资源目录
│       ├── cluster-issuer.yaml
│       └── nfs-storage-class.yaml
├── bin # Kubekey 命令路径，通过 Kubekey 源码编译获得
│   ├── kk # Kubekey 命令行程序，编译自：https://github.com/will4j/kubekey
│   └── kubekey # kk 执行本地缓存目录
│       └── ...
├── kubekey-config.yaml # kubekey 集群配置文件
└── scripts # 自定义 shell 脚本
    ├── control_plane_taint.sh
    └── install_nfs_server.sh
```

核心是 `kubekey-config.yaml` 配置文件，具体配置可参考[[6]]，主要配置块包含：

### 节点信息

Kubekey 管理的节点必须能够使用 SSH 登录，登录方式支持密码和 SSH 密钥，登录账号需要是 root 或者具备 sudo 权限的普通用户，其他信息还包括节点名称、ip 地址等。

```yaml
hosts:
  - { name: example-master-01, address: 192.168.10.1, internalAddress: 192.168.10.1, user: example, password: "P@ssw0rd" }
  - { name: example-master-02, address: 192.168.10.2, internalAddress: 192.168.10.2, user: example, password: "P@ssw0rd" }
```

### 节点角色

定义节点角色分组，一个节点可以同时属于多个分组。除了预置的 `control-plane`、`master` 和 `worker` 角色外，也支持自定义角色，如下面例子中的 `nfs-server`，也可以按照系统类型定义 `ubuntu` 、`gpu` 等分组，方便后续对不同分组执行特殊操作。

```yaml
roleGroups:
  controller:
  - example-master-01
  etcd:
  - example-master-01
  - example-master-02
  - example-master-03
  control-plane:
  - example-master-01
  - example-master-02
  - example-master-03
  nfs-server:
    - example-master-03
```

### k8s 配置

目标 k8s 集群配置信息，包括集群版本、Api Server 配置、集群根证书自动刷新配置等。Kubekey 支持的 k8s 版本可通过 `kk version --show-supported-k8s` 命令查看。

```yaml
kubernetes:
  version: v1.29.1 # k8s 版本
  clusterName: cluster.local
  autoRenewCerts: true # 自动刷新集群根证书
  containerManager: containerd # 容器环境类型，支持 docker、containerd、cri-o、isula
  apiserverArgs: # api server 特殊配置
    - service-node-port-range=80-32767
```

### 网络插件配置

### 操作系统配置

### DNS 配置

### 扩展插件配置

## 集群构建

### kk 命令编译

kk 是 Kubekey 编译生成的命令行程序，下载 Kubekey 源代码[[7]]（笔者的个人仓库，Fork 自 Kubekey 开源仓库[[3]]，新增部分功能增强），执行编译命令：

```shell
# 编译构建当前平台可执行程序
$ make kk
# 编译构建 Linux amd64 可执行程序
$ make kk-linux-amd64
```

将编译生成的 `bin/kk` 可执行程序拷贝到集群构建脚本的 `bin` 目录下，将用于后续集群构建。

### 操作系统初始化

```shell
$ task os:init
task: [os:init] ./bin/kk init os -f kubekey-config.yaml
...
```

初始化主要安装前置必备软件 socat 及 conntrack，以及配置文件中指定的软件依赖，这里以 nfs 客户端为例，配置文件中分别设置了 rpm 系统和 debian 系统对应的安装包：

```yaml
...
spec:
  system:
  rpms:
    - nfs-utils # 对应 yum 安装
  debs:
    - nfs-common # 对应 apt 安装
...
```

### 集群创建命令执行

```shell

```

## 插件安装

## 参考资料

\[1\]. [Cloud Native Landscape.][1]  
\[2\]. [Kubernetes installation methods the complete guide.][2]  
\[3\]. [Kubesphere Kubekey, GitHub.][3]  
\[4\]. [Why Kubekey, Kubesphere Documents.][4]  
\[5\]. [Kubekey Cluster Example, GitHub.][5]
\[6\]. [Kubekey config example, GitHub.][6]  
\[7\]. [Will4j Kubekey, GitHub.][7]  

[1]: https://landscape.cncf.io/
[2]: https://kubedemy.io/kubernetes-installation-methods-the-complete-guide-update-2022
[3]: https://github.com/kubesphere/kubekey
[4]: https://kubesphere.io/docs/v3.4/installing-on-linux/introduction/kubekey/#why-kubekey
[5]: https://github.com/will4j/kubekey-cluster-example
[6]: https://github.com/kubesphere/kubekey/blob/master/docs/config-example.md
[7]: https://github.com/will4j/kubekey
