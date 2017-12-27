---
layout:     post
title:      "安装集群调度仿真器"
subtitle:   "simulator"
date:       "2017-10-15"
author:     "xiechengsheng"
header-img: "img/post-bg-17.jpg"
catalog: true
tags:
    - tools
    - scheduler
    - simulator
---

# 缘起
- 由于学生党能够拥有的机器集群规模非常非常有限，因此在超小规模的集群进行任务调度的实验测试结果不容易让人信服，此时可以使用仿真器模拟大规模集群对任务进行调度；
- 如果自己有多余时间的话，也可以自己写一个针对google cluster trace 的仿真调度器，顺便也可以熟悉google trace的数据，毕竟是比较公认的测试数据集；
- 本次的仿真工具打算使用[cluster-scheduler-simulator](https://github.com/google/cluster-scheduler-simulator)，其作者[@ms705](https://github.com/google/cluster-scheduler-simulator)是大名鼎鼎的[Omega](https://dl.acm.org/citation.cfm?id=2465386)共享状态式调度器论文的一作，这篇论文是他在google实习后发表的，并且文章的测试工具集也是使用到google cluster teace，值得一读；

# 配置安装
1. java
- java1.8安装
```sh
wget http://download.oracle.com/otn-pub/java/jdk/8u121-b13/e9e7ea248e2c4826b92b3f075a80e441/jdk-8u121-linux-x64.tar.gz
tar xvf jdk-8u121-linux-x64.tar.gz -C /usr/lib/jvm/
vim /etc/profile
#在末尾添加
export JAVA_HOME=/usr/lib/jvm/java1.8.0_121
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH={JAVA_HOME}/bin:$PATH
source /etc/profile
```

- 更新配置java
```sh
update-alternatives --install /usr/bin/java java /usr/lib/jvm/jdk1.8.0_121/bin/java 300
update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/jdk1.8.0_121/bin/javac 300
update-alternatives --config java
update-alternatives --config javac
```

- 验证
```sh
安装完java环境之后，可以在终端运行java或者javac验证；
在Linux终端下面编译运行java程序：
javac program.java     //编译java程序，生成class文件，不会生成program可执行文件，使用ls无法查看
java program               //执行program程序
```

2. scala
- 安装
```sh
wget https://downloads.lightbend.com/scala/2.12.1/scala-2.12.1.tgz
mkdir /opt/scala
tar xvf scala-2.12.1.tgz -C /opt/scala/
vim ~/.bashrc
source ~/.bashrc
在结尾添加：
export PATH=/opt/scala/scala-2.12.1/bin:$PATH
export SCALA_HOME=/opt/scala/scala-2.12.1
```

- 验证：
`scala -version`

3. 运行
- 获取源码
```sh
git clone https://github.com/google/cluster-scheduler-simulator
```

- 在cluster-scheduler-simulator文件夹下面运行bin/sbt run，出现下面错误：
![error](/img/in-post/cluster-scheduler-simulator/error.png)

- 原因是由于java1.8与scala2.12.1版本不兼容导致，切换java和javac版本到1.7，scala版本到2.9.3解决问题。原来java和scala的版本还需要对应的；

# 感想
- scala代码难看懂，该工程运行的原理还没弄清楚
- 有机会用脚本语言造仿真器

# 参考
[google/cluster-scheduler-simulator](https://github.com/google/cluster-scheduler-simulator)      