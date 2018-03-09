---
layout:     post
title:      "CentOS下面安装Kubernetes"
subtitle:   "安装Kubernetes"
date:       "2017-08-29"
author:     "xiechengsheng"
header-img: "img/post-bg-15.jpg"
catalog: true
tags:
    - tools
    - K8S
    - docker
---

Kubernetes作为优秀的容器编排工具，在业界已经得到广泛的认可与应用，与docker swarm、mesos竞争激烈；下面在CentOS系统上面部署安装Kubernetes；

# 准备
1. 至少两台已安装好CentOS7.2操作系统的物理机或者虚拟机；
2. 设置master主机和node从机的hostname命令：
```sh
master端： hostnamectl set-hostname k8s-mst
node 端： hostnamectl set-hostname k8s-nodx
```
3. 在集群内所有机器的host文件中添加各主机名与其对应的IP地址；
4. 关闭Node节点上的防火墙：
```sh
systemctl stop firewalld
systemctl disable firewalld
```
5. 为了让各个节点的时间保持一致，需要为所有节点安装NTP模块：
```sh
yum -y install ntp
systemctl start ntpd
systemctl enable ntpd
```

# 编译Kubernetes源码
- 由于本次使用的是二进制方式部署安装Kubernetes，因此使用直接编译源码的方式生成Kubernetes的相关可执行命令；
1. 安装docker：由于CentOS7.2自带Docker源，直接安装docker1.12版本：
```sh
sudo yum install -y docker
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker    # 查看docker 状态
```
2. Kubernetes采用go编写，安装go：`sudo yum install -y golang`
3. 下载Kubernetes：
```sh
git clone https://github.com/kubernetes/kubernetes.git
git checkout v1.x.x     # 选择自己想要的版本
```
4. 编译Kubernetes：
```sh
cd /path/to/kubernetes/
make    # 静静地等待15-20min...
ls _output/     # Kubernetes的所有可执行文件都在该文件夹下面
```

# 配置master
1. 安装并配置etcd：
```sh
yum -y install etcd
# 修改etcd配置文件：
vim /etc/etcd/etcd.conf：
ETCD_NAME=default
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://localhost:2379"    # 注意这里的etcd的ETCD_ADVERTISE_CLIENT_URLS地址最好使用http://0.0.0.0:2379
# 启动etcd数据库：
sudo systemctl enable etcd
sudo systemctl start etcd
```
2. 配置etcd的网络：`etcdctl mk /atomic.io/network/config '{"Network":"172.17.0.0/16"}'`
3. 将可执行文件夹`_output/`下面的可执行文件拷贝kube-apiserver、kube-controller-manager、kube-scheduler、kubectl到master端点的/usr/bin/目录下面，使用shell脚本执行该操作并配置好服务的相关信息：
- 根据自己的的集群配置修改MASTER_ADDRESS以及ETCD_SERVERS
```sh
# cat service.sh:
#!/bin/bash
# Copyright 2016 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
MASTER_ADDRESS=${1:-$master_ip}
ETCD_SERVERS=${2:-"http://$etcd_host_ip:2379"}
SERVICE_CLUSTER_IP_RANGE=${3:-"10.254.0.0/16"}
ADMISSION_CONTROL=${4:-"NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ResourceQuota"}
cat <<EOF >/etc/kubernetes/config
# --logtostderr=true: log to standard error instead of files
KUBE_LOGTOSTDERR="--logtostderr=true"
# --v=0: log level for V logs
KUBE_LOG_LEVEL="--v=0"
# --allow-privileged=false: If true, allow privileged containers.
KUBE_ALLOW_PRIV="--allow-privileged=false"
# How the controller-manager, scheduler, and proxy find the apiserver
KUBE_MASTER="--master=${MASTER_ADDRESS}:8080"
EOF
mkdir /etc/kubernetes
cat <<EOF >/etc/kubernetes/apiserver
# --insecure-bind-address=127.0.0.1: The IP address on which to serve the --insecure-port.
KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0"
# --insecure-port=8080: The port on which to serve unsecured, unauthenticated access.
KUBE_API_PORT="--insecure-port=8080"
# --kubelet-port=10250: Kubelet port
NODE_PORT="--kubelet-port=10250"
# --etcd-servers=[]: List of etcd servers to watch (http://ip:port), # comma separated. Mutually exclusive with -etcd-config
KUBE_ETCD_SERVERS="--etcd-servers=${ETCD_SERVERS}"
# --advertise-address=<nil>: The IP address on which to advertise # the apiserver to members of the cluster.
KUBE_ADVERTISE_ADDR="--advertise-address=${MASTER_ADDRESS}"
# --service-cluster-ip-range=<nil>: A CIDR notation IP range from which to assign service cluster IPs. # This must not overlap with any IP ranges assigned to nodes for pods.
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=${SERVICE_CLUSTER_IP_RANGE}"
# --admission-control="AlwaysAdmit": Ordered list of plug-ins # to do admission control of resources into cluster. # Comma-delimited list of: # LimitRanger, AlwaysDeny, SecurityContextDeny, NamespaceExists, # NamespaceLifecycle, NamespaceAutoProvision,# AlwaysAdmit, ServiceAccount, ResourceQuota, DefaultStorageClass
KUBE_ADMISSION_CONTROL="--admission-control=${ADMISSION_CONTROL}"
# Add your own!
KUBE_API_ARGS=""
EOF
KUBE_APISERVER_OPTS=" \${KUBE_LOGTOSTDERR} \\
\${KUBE_LOG_LEVEL} \\
\${KUBE_ETCD_SERVERS} \\
\${KUBE_API_ADDRESS} \\
\${KUBE_API_PORT} \\
\${NODE_PORT} \\
\${KUBE_ADVERTISE_ADDR} \\
\${KUBE_ALLOW_PRIV} \\
\${KUBE_SERVICE_ADDRESSES} \\
\${KUBE_ADMISSION_CONTROL} \\
\${KUBE_API_ARGS}"
cat <<EOF >/usr/lib/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
After=network.target
After=etcd.service
[Service]
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/apiserver
ExecStart=/usr/bin/kube-apiserver ${KUBE_APISERVER_OPTS}
Restart=on-failure
Type=notify
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF
cat <<EOF >/etc/kubernetes/controller-manager
#### The following values are used to configure the kubernetes controller-manager
# defaults from config and apiserver should be adequate
# Add your own!
KUBE_CONTROLLER_MANAGER_ARGS=""
EOF
KUBE_CONTROLLER_MANAGER_OPTS=" \${KUBE_LOGTOSTDERR} \\
\${KUBE_LOG_LEVEL} \\
\${KUBE_MASTER} \\
\${KUBE_CONTROLLER_MANAGER_ARGS}"
cat <<EOF >/usr/lib/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes
[Service]
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/controller-manager
ExecStart=/usr/bin/kube-controller-manager ${KUBE_CONTROLLER_MANAGER_OPTS}
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF
cat <<EOF >/etc/kubernetes/scheduler
#### kubernetes scheduler config
# Add your own!
KUBE_SCHEDULER_ARGS=""
EOF
KUBE_SCHEDULER_OPTS=" \${KUBE_LOGTOSTDERR} \\
\${KUBE_LOG_LEVEL} \\
\${KUBE_MASTER} \\
\${KUBE_SCHEDULER_ARGS}"
cat <<EOF >/usr/lib/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes
[Service]
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/scheduler
ExecStart=/usr/bin/kube-scheduler ${KUBE_SCHEDULER_OPTS}
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
```

4. master端运行相应的Kubernetes命令，启动Kubernetes服务：
```sh
for svc in kube-apiserver kube-controller-manager kube-scheduler; do
    systemctl restart $svc
    systemctl enable $svc
    systemctl status $svc
done
```

# Node配置工作
1. 安装并配置flannel：
```sh
# 安装flannel：
yum -y install flannel
# 修改flannel的配置文件：
vim /etc/sysconfig/flanneld:
FLANNEL_ETCD="http://master-ip:2379"
FLANNEL_ETCD_KEY="/atomic.io/network"
# 启动flannel服务：
systemctl restart flanneld
systemctl enable flanneld
systemctl status flanneld
```

2. 向etcd服务器上传本地的网络配置：
    - 创建json文件：
    ```sh
    # cat config.json:
    {
    "Network": "172.17.0.0/16",
    "SubnetLen": 24,
    "Backend": {
        "Type": "vxlan",
        "VNI": 7890
        }
    }    
    ```
    - 将配置上传到etcd服务器上：`curl -L http://etcd-host-ip:2379/v2/keys/atomic.io/network/config -XPUT --data-urlencode value@config.json`

3. Node节点端的Kubernetes配置：
    - 将`_output/`下面的`kube-proxy`、`kubelet` 复制到Node节点的/usr/bin/可执行文件目录下
    - 创建配置node上面Kubernetes服务脚本：
```sh
# 根据自己的的集群信息修改MASTER_ADDRESS和NODE_HOSTNAME
# cat service.sh
#!/bin/bash
# Copyright 2016 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
MASTER_ADDRESS=${1:-"master-ip"}
NODE_HOSTNAME=${2:-"k8s-nodx"}
cat <<EOF >/etc/kubernetes/config
# --logtostderr=true: log to standard error instead of files
KUBE_LOGTOSTDERR="--logtostderr=true"
# --v=0: log level for V logs
KUBE_LOG_LEVEL="--v=0"
# --allow-privileged=false: If true, allow privileged containers.
KUBE_ALLOW_PRIV="--allow-privileged=false"
# How the controller-manager, scheduler, and proxy find the apiserver
KUBE_MASTER="--master=${MASTER_ADDRESS}:8080"
EOF
cat <<EOF >/etc/kubernetes/proxy
#### kubernetes proxy config
# default config should be adequate
# Add your own!
KUBE_PROXY_ARGS=""
EOF
KUBE_PROXY_OPTS=" \${KUBE_LOGTOSTDERR} \\
\${KUBE_LOG_LEVEL} \\
\${KUBE_MASTER} \\
\${KUBE_PROXY_ARGS}"
cat <<EOF >/usr/lib/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Proxy
After=network.target
[Service]
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/kube-proxy
ExecStart=/usr/bin/kube-proxy ${KUBE_PROXY_OPTS}
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF
mkdir /etc/kubernetes
cat <<EOF >/etc/kubernetes/kubelet
# --address=0.0.0.0: The IP address for the Kubelet to serve on (set to 0.0.0.0 for all interfaces)
KUBELET__ADDRESS="--address=0.0.0.0"
# --port=10250: The port for the Kubelet to serve on. Note that "kubectl logs" will not work if you set this flag.
KUBELET_PORT="--port=10250"
# --hostname-override="": If non-empty, will use this string as identification instead of the actual hostname.
KUBELET_HOSTNAME="--hostname-override=${NODE_HOSTNAME}"
# --api-servers=[]: List of Kubernetes API servers for publishing events, # and reading pods and services. (ip:port), comma separated.
KUBELET_API_SERVER="--api-servers=http://${MASTER_ADDRESS}:8080"
# pod infrastructure container
KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=registry.access.redhat.com/rhel7/pod-infrastructure:latest"
# Add your own!
KUBELET_ARGS=""
EOF
KUBE_PROXY_OPTS=" \${KUBE_LOGTOSTDERR} \\
\${KUBE_LOG_LEVEL} \\
\${KUBELET__ADDRESS} \\
\${KUBELET_PORT} \\
\${KUBELET_HOSTNAME} \\
\${KUBELET_API_SERVER} \\
\${KUBE_ALLOW_PRIV} \\
\${KUBELET_POD_INFRA_CONTAINER}\\
\${KUBELET_ARGS}"
cat <<EOF >/usr/lib/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
After=docker.service
Requires=docker.service
[Service]
WorkingDirectory=/var/lib/kubelet
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/kubelet
ExecStart=/usr/bin/kubelet ${KUBE_PROXY_OPTS}
Restart=on-failure
[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
```
4. 启动Node节点上面的Kubernetes服务：
```sh
for svc in docker kubelet kube-proxy; do 
    systemctl restart $svc
    systemctl enable $svc
    systemctl status $svc
done
```

# 验证集群安装
- master端获取集群信息：
```sh 
# kubectl get nodes 
 NAME      STATUS    AGE
 nod1       Ready     3h 
 nod2       Ready     3h
```
- Done.


# 参考
[CentOS7.2中使用Kubernetes（k8s)1.4.6源码搭建k8s容器集群环境](http://www.cnblogs.com/yujinyu/p/6092572.html)   
[CentOS部署Kubernetes集群](https://www.kubernetes.org.cn/doc-16)      