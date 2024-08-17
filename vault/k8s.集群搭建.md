---
id: ev7mst424zjzlmc7z87mgwp
title: 集群搭建
desc: ''
updated: 1723902055655
created: 1723900659033
---

## 实验环境

- 4 台虚拟机
  - 1 台 Master
  - 2 台 Node

网络配置采用 `NAT` + `Host-Only` 模式, 保证集群内部网络互通.

### hostname

- master-node
- node01
- node02
- node03

```
hostnamectl hostname {node1}
```

### 关闭防火墙

```
sudo ufw disable
```

### ip

- master-node: 192.168.56.200
- node01: 192.168.56.201
- node02: 192.168.56.202
- node03: 192.168.56.203

### /etc/hosts

```
# k8s lab
192.168.56.200 master.node
192.168.56.201 n1.node
192.168.56.202 n2.node
192.168.56.203 n3.node
# k8s lab end
```

### ssh 密钥


### 用户名、密码

- username: root
- password: 110119

- username: vboxuser
- password: 110119


## 搭建集群

1. 主机配置, swapoff 和 网络配置
2. 安装容器运行时, `containerd` 或者 `docker`, 需要在所有主机上安装, `containerd` 可以通过从 github 上下载的压缩包安装二进制文件
3. 安装 kubectl, kubelet, kubeadm
4. 集群网络 addon

### 安装容器进行时

文档: 

k8s 通过 CRI 和容器进行时交互, docker 默认不支持 CRI， 需要安装 docker-cri 插件

#### containerd

[Getting started with containerd](https://github.com/containerd/containerd/blob/main/docs/getting-started.md)

scp master.node:/home/vboxuser/install-master.sh .
scp master.node:/home/vboxuser/install-slave.sh .
scp -r master.node:/home/vboxuser/pkgs .


### 安装 kubectl, kubeadm, kubelet

### 部署网络 addon

#### flannel

默认最新镜像 docker.io 无法下载, `rancher/mirrored-flannel` 可以下载

- image: master.node:5000/flannelcni-flannel-cni-plugin:v1.2.0
- image: master.node:5000/flannelcni-flannel:v0.20.2

下载镜像后通过本地镜像 registry 分发, 需要修改 `kube-flannel.yaml` 中的配置:

1. 镜像
2. `net-config.json` 集群网络地址范围需要和 `kubeadm` 启动集群指定的相同
3. `/opt/bin/flanneld` 命令增加一个参数 `--iface=enp0s8`

```
      containers:
      - name: kube-flannel
        image: master.node:5000/flannelcni-flannel:v0.20.2
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        - --iface=enp0s8              # 增加指定网卡参数
```

#### calico


### kubeadm 部署集群

kubeadm 拉取的镜像, docker/nerdctl 都在 `k8s.io` namespace 下

```
# 查看需要的镜像列表
sudo kubeadm config images list

# 拉取 image
sudo kubeadm config images pull --image-repository=registry.aliyuncs.com/google_containers

# master 节点启动
sudo kubeadm init --image-repository registry.aliyuncs.com/google_containers --apiserver-advertise-address=192.168.56.200  --pod-network-cidr=10.244.0.0/16

# 创建 join token, 打印 join 命令
sudo kubeadm token create --print-join-command

# slave 节点加入集群
sudo kubeadm join 192.168.56.200:6443 --token b4jbfn.dkzia9kbua1vxwjj \
        --discovery-token-ca-cert-hash sha256:743089f3672f3773530035114f6cbb3250bdae1de37dae9f7cf5f8d9bc075505

sudo kubeadm join 192.168.56.200:6443 --token zh3iwa.2lpca05blou1e5fc --discovery-token-ca-cert-hash sha256:743089f3672f3773530035114f6cbb3250bdae1de37dae9f7cf5f8d9bc075505

```

```
# 镜像导出
sudo nerdctl --namespace k8s.io images | grep -v none
master.node:5000/flannelcni-flannel-cni-plugin                     v1.2.0      286410072e56    2 days ago    linux/amd64    8.2 MiB      3.7 MiB
master.node:5000/flannelcni-flannel                                v0.20.2     7601046a5aac    2 days ago    linux/amd64    59.8 MiB     19.9 MiB
registry.aliyuncs.com/google_containers/coredns                    v1.11.1     a6b67bdb2a67    4 days ago    linux/amd64    60.9 MiB     17.3 MiB
registry.aliyuncs.com/google_containers/etcd                       3.5.12-0    97a7bb219855    4 days ago    linux/amd64    146.3 MiB    54.6 MiB
registry.aliyuncs.com/google_containers/kube-apiserver             v1.30.3     3c66304a30ef    4 days ago    linux/amd64    114.3 MiB    31.3 MiB
registry.aliyuncs.com/google_containers/kube-controller-manager    v1.30.3     f35091eb7d32    4 days ago    linux/amd64    109.1 MiB    29.7 MiB
registry.aliyuncs.com/google_containers/kube-proxy                 v1.30.3     08e5c999c0a7    4 days ago    linux/amd64    84.4 MiB     27.7 MiB
registry.aliyuncs.com/google_containers/kube-scheduler             v1.30.3     57bb58420452    4 days ago    linux/amd64    62.3 MiB     18.4 MiB
registry.aliyuncs.com/google_containers/pause                      3.9         7031c1b28338    4 days ago    linux/amd64    732.0 KiB    314.0 KiB

sudo nerdctl --namespace k8s.io image save master.node:5000/flannelcni-flannel-cni-plugin:v1.2.0 -o flannelcni-flannel-cni-plugin:v1.2.0.tar
sudo nerdctl --namespace k8s.io image save master.node:5000/flannelcni-flannel:v0.20.2 -o flannelcni-flannel:v0.20.2.tar
sudo nerdctl --namespace k8s.io image save registry.aliyuncs.com/google_containers/coredns:v1.11.1 -o coredns:v1.11.1.tar
sudo nerdctl --namespace k8s.io image save registry.aliyuncs.com/google_containers/etcd:3.5.12-0 -o etcd:3.5.12-0.tar
sudo nerdctl --namespace k8s.io image save registry.aliyuncs.com/google_containers/kube-apiserver:v1.30.3 -o kube-apiserver:v1.30.3.tar
sudo nerdctl --namespace k8s.io image save registry.aliyuncs.com/google_containers/kube-controller-manager:v1.30.3 -o kube-controller-manager:v1.30.3.tar
sudo nerdctl --namespace k8s.io image save registry.aliyuncs.com/google_containers/kube-proxy:v1.30.3 -o kube-proxy:v1.30.3.tar
sudo nerdctl --namespace k8s.io image save registry.aliyuncs.com/google_containers/kube-scheduler:v1.30.3 -o kube-scheduler:v1.30.3.tar
sudo nerdctl --namespace k8s.io image save registry.aliyuncs.com/google_containers/pause:3.9 -o pause:3.9.tar
```