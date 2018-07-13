---
layout:     post
title:      "分布式etcd数据库学习"
subtitle:   "etcd"
date:       "2017-11-10"
author:     "xiechengsheng"
header-img: "img/post-bg-18.jpg"
catalog: true
tags:
    - etcd
    - database
---

- etcd作为Kubernetes后端使用的元数据存储数据库，对于Kubernetes集群的稳定运行起到了重要作用，如果etcd数据库损坏，整个Kubernetes集群对外的服务也会终止；

# 数据库的三大挑战
- 任何分布式系统都无法同时满足的三个性质（三者做一个权衡取其二）：
    1. Consistency（一致性）
    2. Availability（可用性）
    3. **Partition tolerance（分区容错性）**
- 其中，分区容错性是所有分布式系统首要考虑的性质，由于当前的网络硬件肯定会出现延迟丢包等问题，所以分区容忍性是我们必须需要实现的，所以我们只能在一致性和可用性之间进行权衡；
- etcd数据库属于C/S模型，属于典型的KV数据库，采用Raft算法实现分布式etcd集群一致性；

# 分布式etcd
1. 安装部署etcd3.0.6：
```sh
curl -L https://github.com/coreos/etcd/releases/download/v3.0.6/etcd-v3.0.6-linux-amd64.tar.gz -o etcd-v3.0.6-linux-amd64.tar.gz
tar xzvf etcd-v3.0.6-linux-amd64.tar.gz
mv etcd-v3.0.6-linux-amd64 etcd
cd etcd
./etcd --version
#将etcd配置到path中，主要是设置etcdctl客户端的版本，编辑/etc/profile
export PATH=/root/etcd:$PATH
export ETCDCTL_API=3
source /etc/profile
#这里可以避免进程占用终端，可以使得etcd服务器后台运行
./etcd &
#简单测试
etcdctl put aaa 1       # 增
etcdctl get aaa         # 查
```

2. 在单机上面安装etcd集群：
```sh
首先在 $GOPATH 路径下面执行：
# vim Procfile
# Use goreman to run `go get github.com/mattn/goreman`
etcd1: etcd --name infra1 --listen-client-urls http://127.0.0.1:12379 --advertise-client-urls http://127.0.0.1:12379 --listen-peer-urls http://127.0.0.1:12380 --initial-advertise-peer-urls http://127.0.0.1:12380 --initial-cluster-token etcd-cluster-1 --initial-cluster 'infra1=http://127.0.0.1:12380,infra2=http://127.0.0.1:22380,infra3=http://127.0.0.1:32380' --initial-cluster-state new --enable-pprof
etcd2: etcd --name infra2 --listen-client-urls http://127.0.0.1:22379 --advertise-client-urls http://127.0.0.1:22379 --listen-peer-urls http://127.0.0.1:22380 --initial-advertise-peer-urls http://127.0.0.1:22380 --initial-cluster-token etcd-cluster-1 --initial-cluster 'infra1=http://127.0.0.1:12380,infra2=http://127.0.0.1:22380,infra3=http://127.0.0.1:32380' --initial-cluster-state new --enable-pprof
etcd3: etcd --name infra3 --listen-client-urls http://127.0.0.1:32379 --advertise-client-urls http://127.0.0.1:32379 --listen-peer-urls http://127.0.0.1:32380 --initial-advertise-peer-urls http://127.0.0.1:32380 --initial-cluster-token etcd-cluster-1 --initial-cluster 'infra1=http://127.0.0.1:12380,infra2=http://127.0.0.1:22380,infra3=http://127.0.0.1:32380' --initial-cluster-state new --enable-pprof
# in future, use proxy to listen on 2379
#proxy: bin/etcd --name infra-proxy1 --proxy=on --listen-client-urls http://127.0.0.1:2378 --initial-cluster 'infra1=http://127.0.0.1:12380,infra2=http://127.0.0.1:22380,infra3=http://127.0.0.1:32380' --enable-pprof
#下载goreman
go get github.com/mattn/goreman
goreman -f Procfile start &
#简单测试：
#显示etcd集群信息
etcdctl --write-out=table --endpoints=localhost:12379 member list
+------------------+---------+--------+------------------------+------------------------+
|        ID        | STATUS  |  NAME  |       PEER ADDRS       |      CLIENT ADDRS      |
+------------------+---------+--------+------------------------+------------------------+
| 8211f1d0f64f3269 | started | infra1 | http://127.0.0.1:12380 | http://127.0.0.1:12379 |
| 91bc3c398fb3c146 | started | infra2 | http://127.0.0.1:22380 | http://127.0.0.1:22379 |
| fd422379fda50e48 | started | infra3 | http://127.0.0.1:32380 | http://127.0.0.1:32379 |
+------------------+---------+--------+------------------------+------------------------+
# 向其中一个etcd节点写入值
etcdctl --endpoints=localhost:12379 put foo bar     
#可以从不同etcd节点获取键值
etcdctl --endpoints=localhost:12379 get foo
etcdctl --endpoints=localhost:22379 get foo
```

3. 容错性测试
```sh
体验etcd集群的容错性，停掉一个节点：
goreman run stop etcd2
# 向另外的etcd节点写数据
etcdctl --endpoints=localhost:12379 put key hello
etcdctl --endpoints=localhost:12379 get key
#无法从停掉的节点获取数据，但是可以从其他节点获取
etcdctl --endpoints=localhost:22379 get key
#重新启动
goreman run restart etcd2
# 可再次获取数据
etcdctl --endpoints=localhost:22379 get key 
```


# 参考
[老司机带你用 Go 语言实现 Raft 分布式一致性协议](https://github.com/happyer/distributed-computing/tree/master/src/raft)    
[Linux单机部署etcd](https://skyao.gitbooks.io/learning-etcd3/content/installation/linux_single.html)    

