---
layout:     post
title:      "服务器压测与监控工具"
subtitle:   "压测与监控工具"
date:       "2017-06-16"
author:     "xiechengsheng"
header-img: "img/post-bg-11.jpg"
catalog: true
tags:
    - tools
---

工欲善其事，必先利其器；总是要在服务器上面运行各种各样的应用，可以使用服务器监控工具查看自己的服务器状态如何；如果需要了解下服务器的性能如何，需要压测工具；

# 监控工具
- 查看服务器CPU核的使用率：`top`
- 查看服务器内存使用情况： `free -m`
- 查看服务器磁盘IO使用情况：`iotop`
- 查看服务器磁盘使用情况：`df -h`
- 查看服务器网络IO使用情况：`iftop`
- 升级版top指令： `htop`

# 压测
- 对服务器的不同种类的资源进行压测的工具较多；只写自己会使用的；
- CPU压力测试：
```sh
# 使用lookbusy
./lookbusy -h   # 打印帮助信息
./lookbusy --n=4 --cpu-util=80 --mem-util=4096MB    # 将4个逻辑核的使用率变为80%，内存使用4096MB
```

- 内存压力测试：
```sh
# 使用stress-ng
# 下载与安装stress-ng
wget http://kernel.ubuntu.com/~cking/tarballs/stress-ng/stress-ng-0.08.01.tar.gz
tar -xvf stress-ng-0.08.01.tar.gz
cd stress-ng-0.08.01
make && make install
# help
stress-ng --help
#在4个CPU逻辑核上面产生80%的负载（不耗用内存利用率）：
stress-ng --cpu 4 --cpu-load 80
#在4个CPU逻辑核上面产生20%的负载，并使用两个进程总共占用1024MB内存：
stress-ng --cpu 4 --cpu-load 20 --vm 2 --vm-bytes 1024M #此时可以查看系统的CPU使用率，发现约为70%左右，因为在分配并耗费内存资源的时候，两个进程对两个CPU逻辑核的利用率是100%，因此：综合利用率是100%*0.5+20%=70%
```

- 磁盘IO性能测试工具：fio、dd、iozone
- 网络带宽测试工具：使用netperf对两台机器之间的通信网络进行性能测试
    - netperf包含两个组件：
        - 客户端netperf
        - netserver
```sh
#物理机上测试，在服务器端和客户端安装程序
wget https://github.com/HewlettPackard/netperf/archive/netperf-2.7.0.tar.gz
tar -xvf netperf-2.7.0.tar.gz
cd netperf-netperf-2.7.0/
./configure && make && make install
# 安装完成
# 在服务器端运行：
netserver
# 在客户端运行
netperf -H $ip -l 60 -t TCP_STREAM  # ip是服务器端的ip地址
```


# 参考
[stress-ng](http://kernel.ubuntu.com/~cking/stress-ng/)    
[掌握 Linux PC 性能之基准测试](http://blog.csdn.net/u014743697/article/details/52774140)    
[使用NETPERF测试网络性能](http://www.cnblogs.com/wensiyang0916/p/6558699.html)     
[网络测试工具Netperf安装使用](http://blog.csdn.net/miaohongyu1/article/details/12751295)    
