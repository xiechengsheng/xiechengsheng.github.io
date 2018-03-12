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

# 配置安装
- 本次的仿真工具打算使用[cluster-scheduler-simulator](https://github.com/google/cluster-scheduler-simulator)，

# 参考
[google/cluster-scheduler-simulator](https://github.com/google/cluster-scheduler-simulator)      