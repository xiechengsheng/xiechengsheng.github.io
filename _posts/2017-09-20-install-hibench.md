---
layout:     post
title:      "安装测试集hibench"
subtitle:   "学习hibench"
date:       "2017-09-20"
author:     "xiechengsheng"
header-img: "img/post-bg-16.jpg"
catalog: true
tags:
    - tools
    - scheduler
    - hadoop
---

# 缘起
- 发现好多优化hadoop yarn论文都会使用hibench作为其基准测试工具集，想着同样是一个调度系统，看看人家是调度的什么官方负载，能不能用到容器调度测试性能中来呢？（当前来看应该是不能的......）
- Hibench是intel为评估各大数据框架（例如hadoop、spark）而设计的测试集，从普通的排序，字符串统计到机器学习，数据库操作，图像处理和搜索引擎，都能够涵盖；

# 开始安装
1. 首先安装需要的软件：Java：
    - Java环境需要jdk 与 jre 安装，跳过配置
    - 测试java环境：`java -version`

2. Maven：
```
下载maven包：wget http://apache.fayea.com/maven/maven-3/3.5.0/binaries/apache-maven-3.5.0-bin.zip
解压：unzip apache-maven-3.5.0-bin.zip -d /usr/local/
设置环境变量：
vim .bashrc， 在.bashrc中添加Maven的PATH：
# set maven environment
export M3_HOME=/usr/local/apache-maven-3.5.0
export PATH=$M3_HOME/bin:$PATH
更新环境变量：
source .bashrc
测试Maven环境：
mvn -v
```

3. 下载Hibench测试工具集（从网络上下载一系列包）：
```
git clone https://github.com/intel-hadoop/HiBench
cd HiBench
# 安装测试hadoop框架下面的sql模块：
mvn -Phadoopbench -Dmodules -Psql -Dscala=2.11 clean package
```

- 可能是由于国内网络原因，遇到`wget apache-hive-0.14.0-bin.tar.gz` 包无法下载的的情况，需要手动在本地下载，然后手动指定maven下载apache-hive-0.14.0-bin.tar.gz 包的地址的配置文件；

- 经过一段时间的等待之后，可以看到成功日志：
```sh
...
[INFO] hibench ............................................ SUCCESS [  0.328 s]
[INFO] hibench-common ..................................... SUCCESS [ 22.319 s]
[INFO] HiBench data generation tools ...................... SUCCESS [ 20.008 s]
[INFO] hadoopbench ........................................ SUCCESS [  0.005 s]
[INFO] hadoopbench-sql .................................... SUCCESS [08:14 min]
...
```

4. 配置HiBench（主要需要配置的是 `conf/hadoop.conf` 及 `conf/hibench.conf` 两个文件）：
首先查询master端的hadoop服务端口号：netstat -ntlp，发现端口号为9000，并且hadoop是安装在/usr/local/hadoop文件夹下面：
```sh
# cp conf/hadoop.conf.template conf/hadoop.conf
# nano conf/hadoop.conf     （编辑/hadoop.conf文件成如下形式）
'''
# Hadoop home
hibench.hadoop.home     /usr/local/hadoop
# The path of hadoop executable
hibench.hadoop.executable     /usr/local/hadoop/bin/hadoop
# Hadoop configraution directory
hibench.hadoop.configure.dir  /usr/local/hadoop/etc/hadoop
# The root HDFS path to store HiBench data
hibench.hdfs.master       hdfs://localhost:9000/user/hadoop/HiBench
# Hadoop release provider. Supported value: apache, cdh5, hdp
hibench.hadoop.release    apache
'''
```

- `cat conf/hibench.conf`：（主要设置测试集运行时的数据量和并发度）
```
# The definition of these profiles can be found in the workload's conf file i.e. con/workloads/micro/wordcount.conf
hibench.scale.profile                  tiny
# Mapper number in hadoop, partition number in Spark
hibench.default.map.parallelism         8
# Reducer nubmer in hadoop, shuffle partition number in Spark
hibench.default.shuffle.parallelism     8
```

# 测试运行HiBench
- 安装完成后，可以运行其中的测试集。首先要启动hadoop：`./start-hadoop.sh`
- 以运行Hadoop框架下micro集的sort为例：
```sh
bin/workloads/micro/sort/prepare/prepare.sh
bin/workloads/micro/sort/hadoop/run.sh
```
- 等待读条MapReduce完毕，可以在`report/sort/hadoop/bench.log`处查看具体的运行结果和日志。
- 一些额外的用于验证hadoop集群是否正常工作的方法：
    1. 在master端和slave端可以使用 `jps` 指令检测是否存在hadoop相关进程
    2. master端可以使用 `start-dfs.sh`、`start-yarn.sh` 开启hadoop服务，使用`stop-yarn.sh`、`stop-dfs.sh`关闭hadoop服务

# 参考
[Hadoop常用测试集HiBench配置指南](https://kimihe.github.io/2017/05/11/Hadoop%E5%B8%B8%E7%94%A8%E6%B5%8B%E8%AF%95%E9%9B%86HiBench%E9%85%8D%E7%BD%AE%E6%8C%87%E5%8D%97/)        
[intel-hadoop/HiBench](https://github.com/intel-hadoop/HiBench/wiki)        