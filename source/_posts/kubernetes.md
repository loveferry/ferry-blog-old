---
title: kubernetes集群部署
date: 2020-11-29 19:55:58
categories: kubernetes
tags:
    - k8s
---

&emsp;&emsp;`Kubernetes`(简称k8s) 协调一个高可用计算机集群，每个计算机作为独立单元互相连接工作。 `Kubernetes` 中的抽象允许您将容器化的应用部署到集群，而无需将它们绑定到某个特定的独立计算机。为了使用这种新的部署模型，应用需要以将应用与单个主机分离的方式打包：它们需要被容器化。与过去的那种应用直接以包的方式深度与主机集成的部署模型相比，容器化应用更灵活、更可用。 `Kubernetes` 以更高效的方式跨集群自动分发和调度应用容器。 `Kubernetes` 是一个开源平台，并且可应用于生产环境。

<!-- more -->

&emsp;&emsp;一个 `Kubernetes` 集群包含两种类型的资源:
            
- Master 控制平面，调度整个集群
- Nodes 工作节点，负责运行应用

&emsp;&emsp;Master 负责管理整个集群。 Master 协调集群中的所有活动，例如调度应用、维护应用的所需状态、应用扩容以及推出新的更新。Node 是一个虚拟机或者物理机，它在 `Kubernetes` 集群中充当工作机器的角色 每个Node都有 `Kubelet` , 它管理 Node 而且是 Node 与 Master 通信的代理。 Node 还应该具有用于​​处理容器操作的工具，例如 `Docker` 或 `rkt` 。处理生产级流量的 `Kubernetes` 集群至少应具有三个 Node 。

&emsp;&emsp;Master 管理集群，Node 用于托管正在运行的应用。在 Kubernetes 上部署应用时，您告诉 Master 启动应用容器。 Master 就编排容器在集群的 Node 上运行。 Node 使用 Master 暴露的 `Kubernetes API` 与 Master 通信。终端用户也可以使用 `Kubernetes API` 与集群交互。

&emsp;&emsp;`Kubernetes` 既可以部署在物理机上也可以部署在虚拟机上。您可以使用 `Minikube` 开始部署 `Kubernetes` 集群。 `Minikube` 是一种轻量级的 `Kubernetes` 实现，可在本地计算机上创建 VM 并部署仅包含一个节点的简单集群。 `Minikube` 可用于 `Linux` ， `macOS` 和 `Windows` 系统。`Minikube CLI` 提供了用于引导集群工作的多种操作，包括启动、停止、查看状态和删除。

&emsp;&emsp;`kubernetes` 集群搭建有两种方式：

- 使用部署工具快速搭建 `kubernetes` 集群，例如 `minikube`（快速搭建一个运行在本地的单节点的 `kubernetes`，一般用于本地学习），`kubeadm` 等
- 从官网下载二进制文件手动部署

&emsp;&emsp;这里记录两种方式安装 `kubernetes` 集群，一种是使用 `kubeadm` 自动部署集群，一种是下载二进制文件手动部署。

### 集群机器检查

&emsp;&emsp;集群内所有节点均需检查是否符合要求。

&emsp;&emsp;安装要求：

> 安装 k8s 的节点必须是大于 1 核心的 CPU
> 建议主机内存都是2G及以上
> 使用的物理机操作系统：Ubuntu 16.04+、Debian 9+、CentOS 7、Red Hat Enterprise Linux (RHEL) 7、Fedora 25+、HypriotOS v1.0.1+、Flatcar Container Linux
> 集群中的机器网络能够互联
> 节点之中不可以有重复的主机名、MAC 地址或 product_uuid
> 禁用交换分区。为了保证 kubelet 正常工作，你 必须 禁用交换分区

&emsp;&emsp;集群安装要求查看并修改：

- 查看逻辑内核数：

```bash
cat /proc/cpuinfo| grep "processor"| wc -l
```

- 查看主机名，mac地址，product_uuid

```bash
hostname
ip addr
cat /sys/class/dmi/id/product_uuid
```

&emsp;&emsp;如果主机名重复，请修改

```bash
hostnamectl set-hostname k8s_master
```

- 永久禁用交换分区(/etc/fstab)

```bash
# /dev/mapper/centos-swap swap                    swap    defaults        0 0
```

- 查看内存大小

```bash
free -m
```

- 查看linux内核版本

```bash
cat /proc/version
```

- 集群机器网络互联

```bash
ping 172.16.98.2
```

- 设置主机映射(/etc/hosts)

```bash
172.16.98.2 k8s-master
```

- 端口设置

| 协议   | 方向  | 端口范围   | 作用                     | 使用者                          |
| ----- | ---- | --------- | ----------------------- | ------------------------------ |
| TCP   | 入站  | 6443      | Kubernetes API 服务器    | 所有组件                        |
| TCP   | 入站  | 2379-2380 | etcd 服务器客户端 API     | kube-apiserver, etcd           |
| TCP   | 入站  | 10250     | Kubelet API             | kubelet 自身、控制平面组件        |
| TCP   | 入站  | 10251     | kube-scheduler          | kube-scheduler 自身             |
| TCP   | 入站  | 10252     | kube-controller-manager | kube-controller-manager 自身   |

```bash
firewall-cmd --zone=public --add-port=6443/tcp --permanent
firewall-cmd --zone=public --add-port=2379/tcp --permanent
firewall-cmd --zone=public --add-port=2380/tcp --permanent
firewall-cmd --zone=public --add-port=10250/tcp --permanent
firewall-cmd --zone=public --add-port=10251/tcp --permanent
firewall-cmd --zone=public --add-port=10252/tcp --permanent
firewall-cmd --reload
```

### 使用kubeadm引导集群

#### 安装容器

&emsp;&emsp;这一步需要在集群所有节点执行。

&emsp;&emsp;在linux上结合 `Kubernetes` 使用的几种通用容器为：`containerd`、`CRI-O`、`Docker`。这里我们选择`Docker`。

&emsp;&emsp;这里提供传送门[docker官网的安装说明](https://docs.docker.com/engine/install/#server),可以查看对应的操作系统安装 `Docker` 的详细步骤。下面的安装是基于centos7安装 `Docker` 的

- docker安装

```bash
yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
yum install -y yum-utils
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install -y docker-ce docker-ce-cli containerd.io
systemctl start docker
systemctl enable docker
```

#### 安装工具

- `kubeadm`：用来初始化集群的指令。
- `kubelet`：在集群中的每个节点上用来启动 Pod 和容器等。
- `kubectl`：用来与集群通信的命令行工具。

&emsp;&emsp;在实际安装之前我们还需要了解一下 `Kubernetes` 版本与版本间的偏差策略

##### 版本差异

&emsp;&emsp;`Kubernetes` 版本号格式为 x.y.z，其中 x 为大版本号，y 为小版本号，z 为补丁版本号。

- `kube-apiserver`

&emsp;&emsp;在 高可用（HA）集群 中， 多个 `kube-apiserver` 实例小版本号最多差1。也就是说，集群的每一个节点版本可以有差异，但是小版本差异最多为1。举例，k8s_master机器上版本号为1.19,那么集群的其他机器上安装的版本只能在1.18,1.19,1.20之间选择

- `kubelet`

&emsp;&emsp;`kubelet` 版本号不能高于 `kube-apiserver`，最多可以比 `kube-apiserver` 低两个小版本。在集群中，则 `kubelet` 版本号不能高于集群中 `kube-apiserver` 最低的版本号，最多可以比集群中 `kube-apiserver` 最高的版本号低两个小版本。举例，存在 `kube-apiserver` 版本1.20，那么 `kubelet` 版本可以选择1.18,1.19,1.20。但如果集群中的 `kube-apiserver` 版本不一致，存在两个版本1.20,1.19，那么可选的 `kubelet` 版本为 1.19,1.18。

- `kube-controller-manager`、 `kube-scheduler` 和 `cloud-controller-manager`

&emsp;&emsp;`kube-controller-manager`、`kube-scheduler` 和 `cloud-controller-manager` 版本不能高于 `kube-apiserver` 版本号。 最好它们的版本号与 `kube-apiserver` 保持一致，但允许比 `kube-apiserver` 低一个小版本（为了支持在线升级）。如果在 HA 集群中，多个 `kube-apiserver` 实例版本号不一致，他们也可以跟任意一个 `kube-apiserver` 实例通信（例如，通过 load balancer）， 但 `kube-controller-manager`、`kube-scheduler` 和 `cloud-controller-manager` 版本可用范围会相应的减小。

- `kubectl`

&emsp;&emsp;`kubectl` 可以比 `kube-apiserver` 高一个小版本，也可以低一个小版本。在集群中，`kubectl` 可以比集群中最高的 `kube-apiserver` 版本号低一个小版本，可以比最低的 `kube-apiserver` 版本号高一个小版本。
            
- `kube-proxy`

&emsp;&emsp;`kube-proxy` 必须与节点上的 `kubelet` 的小版本相同;`kube-proxy` 一定不能比 `kube-apiserver` 小版本高;`kube-proxy` 最多只能比 `kube-apiserver` 低两个小版本


##### 安装

&emsp;&emsp;需要在集群所有机器完成这一步。

- k8s的yum源配置(创建文件`/etc/yum.repos.d/kubernetes.repo`)

```bash
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
```

- 查看可安装的版本

```bash
yum list kubeadm kubectl kubelet
```

- 安装指定版本

```bash
yum install -y kubelet-1.20.2-0 kubeadm-1.20.2-0 kubectl-1.20.2-0
```

- 设置服务开机自启（这里不设置在初始化集群的时候会报错）

```bash
systemctl enable kubelet
```

#### 初始化集群

&emsp;&emsp;这一步只需要在控制平面执行即可。

&emsp;&emsp;在初始化控制平面之前，我们需要验证一下与 gcr.io 容器镜像仓库的连通性。

```bash
kubeadm config images pull
```

&emsp;&emsp;如果这一步报错，说没有响应，那么就是无法连接到该仓库，此时，我们需要从国内的镜像仓库获取对应的镜像了。这一步网上有很多的方法，例如从 `docker hub` 上搜索对应的镜像，下载并重新打标签，这种方式依赖于 `docker hub`上现存的镜像版本，如果你需要的版本没找到，那就有点尴尬了。下面提供的这种方法虽然繁琐了些，但是他能保证你需要获取的版本一定能获取到。

- 查看需要的镜像

```bash
kubeadm config images list
```

![需要的镜像](/kubernetes/k8s_need_image_list.png)

&emsp;&emsp;代码仓操作：

- 登录代码仓创建仓库，将执行 `kubeadm config images list` 查看到的镜像依次创建了仓库。我这里选择的是github，如果无法访问github也可以选择你能访问的任何私仓，只要这个私仓公网可以访问即可。

![创建仓库](/kubernetes/k8s_create_github_repository.png)

- 在仓库中创建 `Dockerfile` 文件,以 `kube-proxy` 为例，仓库名为 `kube-proxy`，`Dockerfile`内容为`FROM k8s.gcr.io/kube-proxy:v1.20.2`。

![创建dockerfile](/kubernetes/k8s_create_github_dockerfile.png)

- 登录[阿里云容器镜像服务](https://cr.console.aliyun.com/cn-hangzhou/instances/repositories),创建镜像仓库。容器镜像服务->默认实例->镜像仓库->创建镜像仓库

![创建dockerfile](/kubernetes/k8s_create_aliyun_docker_repository.png)

![创建dockerfile](/kubernetes/k8s_create_aliyun_docker_repository2.png)

- 创建规则执行构建,若失败则查看日志，大概率是构建规则选的分支，文件，路径与代码仓中的不匹配

![查看镜像仓库](/kubernetes/k8s_click_aliyun_docker_repository.png)

![查看构建](/kubernetes/k8s_click_aliyun_create_rule.png)

![创建构建规则](/kubernetes/k8s_aliyun_create_docker_build_rule.png)

![执行构建](/kubernetes/k8s_aliyun_docker_build.png)

- 设置访问凭证

![执行构建](/kubernetes/k8s_aliyun_set_password.png)

- 在控制平面登录阿里云的镜像仓库，用户名为阿里云的账户名，拉取镜像，这里登录的地址一定要看好，镜像仓库所在的区域不同，登录的仓库地址也是有所区别的。具体的登录地址可以点击仓库进入仓库详情页面，基本信息-操作指南中查看

```bash
docker login --username=ferry_sy registry.cn-shanghai.aliyuncs.com
```

```bash
docker pull registry.cn-shanghai.aliyuncs.com/ferry/kube-apiserver:v1.20.2
docker pull registry.cn-shanghai.aliyuncs.com/ferry/kube-controller-manager:v1.20.2
docker pull registry.cn-shanghai.aliyuncs.com/ferry/kube-scheduler:v1.20.2
docker pull registry.cn-shanghai.aliyuncs.com/ferry/kube-proxy:v1.20.2
docker pull registry.cn-shanghai.aliyuncs.com/ferry/pause:3.2
docker pull registry.cn-shanghai.aliyuncs.com/ferry/etcd:3.4.13-0
docker pull registry.cn-shanghai.aliyuncs.com/ferry/coredns:1.7.0
```

- 镜像重命名

```bash
docker tag registry.cn-shanghai.aliyuncs.com/ferry/kube-apiserver:v1.20.2 k8s.gcr.io/kube-apiserver:v1.20.2
docker tag registry.cn-shanghai.aliyuncs.com/ferry/kube-controller-manager:v1.20.2 k8s.gcr.io/kube-controller-manager:v1.20.2
docker tag registry.cn-shanghai.aliyuncs.com/ferry/kube-scheduler:v1.20.2 k8s.gcr.io/kube-scheduler:v1.20.2
docker tag registry.cn-shanghai.aliyuncs.com/ferry/kube-proxy:v1.20.2 k8s.gcr.io/kube-proxy:v1.20.2
docker tag registry.cn-shanghai.aliyuncs.com/ferry/pause:3.2 k8s.gcr.io/pause:3.2
docker tag registry.cn-shanghai.aliyuncs.com/ferry/etcd:3.4.13-0 k8s.gcr.io/etcd:3.4.13-0
docker tag registry.cn-shanghai.aliyuncs.com/ferry/coredns:1.7.0 k8s.gcr.io/coredns:1.7.0
```

- 删除阿里云镜像

```bash
docker image rm registry.cn-shanghai.aliyuncs.com/ferry/kube-apiserver:v1.20.2
docker image rm registry.cn-shanghai.aliyuncs.com/ferry/kube-controller-manager:v1.20.2
docker image rm registry.cn-shanghai.aliyuncs.com/ferry/kube-scheduler:v1.20.2
docker image rm registry.cn-shanghai.aliyuncs.com/ferry/kube-proxy:v1.20.2
docker image rm registry.cn-shanghai.aliyuncs.com/ferry/pause:3.2
docker image rm registry.cn-shanghai.aliyuncs.com/ferry/etcd:3.4.13-0
docker image rm registry.cn-shanghai.aliyuncs.com/ferry/coredns:1.7.0
```

- 初始化集群，`--pod-network-cidr` 参数用于配置CIDR，参数值可以填写 "内网ip/16"。

```bash
kubeadm init --pod-network-cidr=172.10.0.0/16 --service-cidr=10.96.0.0/12
```

&emsp;&emsp;错误诊断：

- 有警告firewall防火墙是开启状态，并提示需要开放必须的端口，此时我们检查它提示的端口是否已经开放即可。

```bash
firewall-cmd --zone=public --list-ports
```

- 有警告kube和docker使用的 `cgroup` 不一致，此时我们使用 `docker info` 命令，在输出信息中查找 `Cgroup Driver: cgroupfs` 这一行，`kubelet` 的该参数查询我没有找到，所以这里提供修改docker的方法。

```bash
echo '{ "exec-opts": ["native.cgroupdriver=systemd"] }' > /etc/docker/daemon.json
systemctl daemon-reload
systemctl restart docker
```

- 报错 `/proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1`，这是因为未指定kubelet网络插件，则使用 noop 插件，该插件设置 `net/bridge/bridge-nf-call-iptables=1`，以确保简单的配置 （如带网桥的 `Docker` ）与 `iptables` 代理正常工作

- 编辑文件`/etc/sysctl.conf`，添加设置然后重启系统

```bash
net.bridge.bridge-nf-call-iptables = 1
```

&emsp;&emsp;解决了所有错误后我们再次初始化集群，当shell中输出如下日志时说明集群已经初始化成功了。

```bash
kubeadm init --pod-network-cidr=172.10.0.0/16 --service-cidr=10.96.0.0/12
```

```bash
[init] Using Kubernetes version: v1.20.2
[preflight] Running pre-flight checks
	[WARNING Firewalld]: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 172.16.98.2]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-master localhost] and IPs [172.16.98.2 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-master localhost] and IPs [172.16.98.2 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 11.503773 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.20" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node k8s-master as control-plane by adding the labels "node-role.kubernetes.io/master=''" and "node-role.kubernetes.io/control-plane='' (deprecated)"
[mark-control-plane] Marking the node k8s-master as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: ayqhud.254isto3e88zn9ba
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.16.98.2:6443 --token ayqhud.254isto3e88zn9ba \
    --discovery-token-ca-cert-hash sha256:9393b7ecc8c6f81d81eb362e4b74e2635eedda15e749872a74cb6ac77bc8c1e5 
```

&emsp;&emsp;我们可以从中捕获关键信息，在日志末尾它提示我们，如果想要非root用户使用集群，需要做那些操作，如果是root用户需要怎么做，才能够使用集群，将工作节点加入集群执行怎样的命令。

- root用户下添加环境变量(~/.bash_profile) 

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

```bash
source ~/.bash_profile
```

#### CNI(容器网络接口)

```bash
kubectl get pods --all-namespaces
```

&emsp;&emsp;执行上述命令会发现此时集群的 `coredns` 的状态一直是 `Pending`，这是因为系统就是这么设计的。 `kubeadm` 的网络供应商是中立的，因此管理员应该选择 安装 `pod` 的网络解决方案。 你必须完成 `Pod` 的网络配置，然后才能完全部署 `CoreDNS`。 在网络被配置好之前，`DNS` 组件会一直处于 `Pending` 状态。

##### kube-router

&emsp;&emsp;CNI的选择有很多，`Kube-router` 是 `Kubernetes` 的专用网络解决方案， 旨在提供高性能和易操作性。 `Kube-router` 提供了一个基于 `Linux LVS/IPVS` 的服务代理、一个基于 `Linux` 内核转发的无覆盖 `Pod-to-Pod` 网络解决方案和基于 `iptables/ipset` 的网络策略执行器。

&emsp;&emsp;这里选择采用 `Kube-router` 代替 `Kube-proxy` 并且提供网络解决方案。

&emsp;&emsp;首先需要下载 `Kube-router` 的yaml文件，我们需要访问 `raw.githubusercontent.com` 去获取，但是很有可能我们无法访问这个网站，那么我们先[获取该域名对应的ip地址](https://site.ip138.com/raw.Githubusercontent.com)然后在 `/etc/hosts` 文件中配置域名映射,这样我们就可以访问了。

```bash
wget https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml
wget https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter-all-features.yaml
```

&emsp;&emsp;上面两个yaml，第一个是仅提供网络解决方案的，第二个还可以代替 `Kube-proxy`。

- 应用下载的yaml

```bash
kubectl apply -f kubeadm-kuberouter-all-features.yaml
```

&emsp;&emsp;应用之后可以看一下状态，不出意外应该是启动不了。

![错误状态](/kubernetes/k8s_kube-router_apply.png)

&emsp;&emsp;遇到这种情况我们可以和先前下载 `kube-proxy` 等镜像一样，通过阿里云的镜像仓库服务来下载。

&emsp;&emsp;下载好之后查看pod状态如果发现状态是 `CrashLoopBackOff`，此时我们可以查看日志

```bash
kubectl logs kube-router-bxl96 -n kube-system
```

![错误日志](/kubernetes/k8s_kube-router_pod_crashloopbackoff.png)

&emsp;&emsp;这说明我们初始化集群 `kubeadm init` 的时候没有使用参数 `--pod-network-cidr`，这里我们只能添加一下这个参数配置。

- `kubeadm-config` 增加 `podSubnet`

```bash
kubectl edit cm kubeadm-config -n kube-system
```

```bash
data:
  ClusterConfiguration: |
    networking:
      podSubnet: 172.16.0.0/16
```

- `kube-controller-manager` 修改,增加 `allocate-node-cidrs` 和 `cluster-cidr`

```bash
vi /etc/kubernetes/manifests/kube-controller-manager.yaml
```

```bash
spec:
  containers:
  - command:
    - --allocate-node-cidrs=true
    - --cluster-cidr=172.16.0.0/16
```

- 删除 `kube-router` pod

```bash
kubectl delete pod kube-router-bxl96 -n kube-system
```

&emsp;&emsp;删除pod后集群会立刻再创建一个pod，稍等一会可以观察pod的状态，理论上此时就没有什么问题了。`kube-router` 创建好了那么之前一直在 `ContainerCreating` 的 `coredns` 的pod随后状态也会变成 `running`。

&emsp;&emsp;由于这里使用的是 `kube-router` 的全特性，可以代替 `kube-proxy`, 所以我们删除 `kube-proxy`。

```bash
kubectl delete ds kube-proxy -n kube-system
```

&emsp;&emsp;至此，我们工作平面已经初步安装完成，现在需要加入工作节点壮大我们的集群。

&emsp;&emsp;使用weave插件

##### weave

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

##### flannel

```bash
---
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: psp.flannel.unprivileged
  annotations:
    seccomp.security.alpha.kubernetes.io/allowedProfileNames: docker/default
    seccomp.security.alpha.kubernetes.io/defaultProfileName: docker/default
    apparmor.security.beta.kubernetes.io/allowedProfileNames: runtime/default
    apparmor.security.beta.kubernetes.io/defaultProfileName: runtime/default
spec:
  privileged: false
  volumes:
  - configMap
  - secret
  - emptyDir
  - hostPath
  allowedHostPaths:
  - pathPrefix: "/etc/cni/net.d"
  - pathPrefix: "/etc/kube-flannel"
  - pathPrefix: "/run/flannel"
  readOnlyRootFilesystem: false
  # Users and groups
  runAsUser:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  # Privilege Escalation
  allowPrivilegeEscalation: false
  defaultAllowPrivilegeEscalation: false
  # Capabilities
  allowedCapabilities: ['NET_ADMIN', 'NET_RAW']
  defaultAddCapabilities: []
  requiredDropCapabilities: []
  # Host namespaces
  hostPID: false
  hostIPC: false
  hostNetwork: true
  hostPorts:
  - min: 0
    max: 65535
  # SELinux
  seLinux:
    # SELinux is unused in CaaSP
    rule: 'RunAsAny'
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: flannel
rules:
- apiGroups: ['extensions']
  resources: ['podsecuritypolicies']
  verbs: ['use']
  resourceNames: ['psp.flannel.unprivileged']
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - nodes/status
  verbs:
  - patch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: flannel
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: flannel
subjects:
- kind: ServiceAccount
  name: flannel
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: flannel
  namespace: kube-system
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: kube-flannel-cfg
  namespace: kube-system
  labels:
    tier: node
    app: flannel
data:
  cni-conf.json: |
    {
      "name": "cbr0",
      "cniVersion": "0.3.1",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-flannel-ds
  namespace: kube-system
  labels:
    tier: node
    app: flannel
spec:
  selector:
    matchLabels:
      app: flannel
  template:
    metadata:
      labels:
        tier: node
        app: flannel
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/os
                operator: In
                values:
                - linux
      hostNetwork: true
      priorityClassName: system-node-critical
      tolerations:
      - operator: Exists
        effect: NoSchedule
      serviceAccountName: flannel
      initContainers:
      - name: install-cni
        image: quay.io/coreos/flannel:v0.14.0-rc1
        command:
        - cp
        args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conflist
        volumeMounts:
        - name: cni
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      containers:
      - name: kube-flannel
        image: quay.io/coreos/flannel:v0.14.0-rc1
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        resources:
          requests:
            cpu: "100m"
            memory: "50Mi"
          limits:
            cpu: "100m"
            memory: "50Mi"
        securityContext:
          privileged: false
          capabilities:
            add: ["NET_ADMIN", "NET_RAW"]
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        volumeMounts:
        - name: run
          mountPath: /run/flannel
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      volumes:
      - name: run
        hostPath:
          path: /run/flannel
      - name: cni
        hostPath:
          path: /etc/cni/net.d
      - name: flannel-cfg
        configMap:
          name: kube-flannel-cfg
```

```bash
kubectl apply -f flannel.yaml
```

#### 加入工作节点

&emsp;&emsp;这一步需要控制平面和工作节点配合完成。

> 控制平面：用于管理集群的机器
> 工作节点：运行应用服务的机器

&emsp;&emsp;将工作节点加入集群需要令牌认证，令牌默认24小时失效。我们先查询当前未失效的令牌

- 控制平面

```bash
kubeadm token list
```

&emsp;&emsp;如果shell中输出了令牌信息，那么我们就用这个令牌就可以让工作节点加入集群，若没有输出令牌信息，说明已有的令牌都失效了，需要重新生成令牌。

- 控制平面

```bash
kubeadm token create
```

&emsp;&emsp;此外，我们还需要获取ca证书sha256编码hash值

- 控制平面

```bash
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```

&emsp;&emsp;将工作节点加入集群，命令中的ip地址是控制平面的内网ip，端口是 Kubernetes API 服务器的端口

- 工作节点

```bash
kubeadm join 172.16.98.2:6443 --token udl1vd.ln1kzbtertnzbfmi --discovery-token-ca-cert-hash sha256:9393b7ecc8c6f81d81eb362e4b74e2635eedda15e749872a74cb6ac77bc8c1e5
```

![加入节点错误](k8s_join_node_to_master.png)

&emsp;&emsp;出错了，说明我们工作节点的环境准备不是很充分，此时我们按照提示一步步解决。

- 未识别的的主机名,编辑工作节点的 `/etc/hosts` 文件

```bash
172.16.98.6 k8s-node-1
```

- 编辑工作节点文件`/etc/sysctl.conf`，添加设置然后重启系统

```bash
net.bridge.bridge-nf-call-iptables = 1
```

- 工作节点

```bash
kubeadm join 172.16.98.2:6443 --token udl1vd.ln1kzbtertnzbfmi --discovery-token-ca-cert-hash sha256:9393b7ecc8c6f81d81eb362e4b74e2635eedda15e749872a74cb6ac77bc8c1e5
```

&emsp;&emsp;依次将所有工作节点加入集群中，接下来在查看集群中的节点信息。

- 控制平面

```bash
kubectl get node
```

### 集群参数设置

#### kube-proxy模式切换

&emsp;&emsp;`ipvs`相比较`iptables`来说性能更好，我们将`kube-proxy`的模式切换成`ipvs`。

```bash
kubectl edit cm -n kube-system
```

将`data.config.conf.mode`设置成`ipvs`保存退出即可。