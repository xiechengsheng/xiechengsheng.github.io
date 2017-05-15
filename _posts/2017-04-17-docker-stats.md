---
layout:     post
title:      "使用docker stats收集主机容器运行信息"
subtitle:   "docker stats"
date:       "2017-04-16"
author:     "xiechengsheng"
header-img: "img/post-bg-7.jpg"
catalog: true
tags:
    - docker
---

# 缘起
由于信息采集工具heapster对不同主机的pod资源使用情况采样周期较慢，一般只能为20s，对实时性更新的数据不敏感，于是通过直接访问docker server的RESTFul API的方式来获取服务器端docker消耗资源信息；

# 环境准备
1. 需要修改服务器主机docker的启动项配置，docker在启动的时候可以向外提供 RESTFul API 服务，供外界访问：`-H=unix:///var/run/docker.sock -H=0.0.0.0:1234`
2. [docker stats 指令官方文档](https://docs.docker.com/engine/reference/commandline/stats/#description)：参考这个docker 的官方文档发现可以在本机使用docker stats观测docker 的资源使用率

# 测试
- 提供的RESTFUl API需要隔一秒钟就返回数据，怎么只访问一次：     
在客户端主机通过发送request请求，采用失能 stream 的方式只获取一次stats请求，使用 `/containers/(id)/stats API` 获取：
```python
r = requests.get("http://192.168.3.87:1234/containers/bb20076143e7/stats", {'stream': False})
print r.json()
```

- 返回该容器的资源使用信息数据格式为：
```json
{
u'blkio_stats': {u'io_service_time_recursive': [], u'sectors_recursive': [], u'io_service_bytes_recursive': [{u'major': 252, u'value': 192512, u'minor': 0, u'op': u'Read'}, {u'major': 252, u'value': 0, u'minor': 0, u'op': u'Write'}, {u'major': 252, u'value': 0, u'minor': 0, u'op': u'Sync'}, {u'major': 252, u'value': 192512, u'minor': 0, u'op': u'Async'}, {u'major': 252, u'value': 192512, u'minor': 0, u'op': u'Total'}, {u'major': 253, u'value': 192512, u'minor': 0, u'op': u'Read'}, {u'major': 253, u'value': 9603584, u'minor': 0, u'op': u'Write'}, {u'major': 253, u'value': 9603584, u'minor': 0, u'op': u'Sync'}, {u'major': 253, u'value': 192512, u'minor': 0, u'op': u'Async'}, {u'major': 253, u'value': 9796096, u'minor': 0, u'op': u'Total'}, {u'major': 7, u'value': 4096, u'minor': 0, u'op': u'Read'}, {u'major': 7, u'value': 0, u'minor': 0, u'op': u'Write'}, {u'major': 7, u'value': 0, u'minor': 0, u'op': u'Sync'}, {u'major': 7, u'value': 4096, u'minor': 0, u'op': u'Async'}, {u'major': 7, u'value': 4096, u'minor': 0, u'op': u'Total'}, {u'major': 253, u'value': 4096, u'minor': 2, u'op': u'Read'}, {u'major': 253, u'value': 0, u'minor': 2, u'op': u'Write'}, {u'major': 253, u'value': 0, u'minor': 2, u'op': u'Sync'}, {u'major': 253, u'value': 4096, u'minor': 2, u'op': u'Async'}, {u'major': 253, u'value': 4096, u'minor': 2, u'op': u'Total'}, {u'major': 253, u'value': 3249152, u'minor': 9, u'op': u'Read'}, {u'major': 253, u'value': 65536, u'minor': 9, u'op': u'Write'}, {u'major': 253, u'value': 65536, u'minor': 9, u'op': u'Sync'}, {u'major': 253, u'value': 3249152, u'minor': 9, u'op': u'Async'}, {u'major': 253, u'value': 3314688, u'minor': 9, u'op': u'Total'}], u'io_serviced_recursive': [{u'major': 252, u'value': 28, u'minor': 0, u'op': u'Read'}, {u'major': 252, u'value': 18698, u'minor': 0, u'op': u'Write'}, {u'major': 252, u'value': 18698, u'minor': 0, u'op': u'Sync'}, {u'major': 252, u'value': 28, u'minor': 0, u'op': u'Async'}, {u'major': 252, u'value': 18726, u'minor': 0, u'op': u'Total'}, {u'major': 253, u'value': 28, u'minor': 0, u'op': u'Read'}, {u'major': 253, u'value': 9349, u'minor': 0, u'op': u'Write'}, {u'major': 253, u'value': 9349, u'minor': 0, u'op': u'Sync'}, {u'major': 253, u'value': 28, u'minor': 0, u'op': u'Async'}, {u'major': 253, u'value': 9377, u'minor': 0, u'op': u'Total'}, {u'major': 7, u'value': 1, u'minor': 0, u'op': u'Read'}, {u'major': 7, u'value': 0, u'minor': 0, u'op': u'Write'}, {u'major': 7, u'value': 0, u'minor': 0, u'op': u'Sync'}, {u'major': 7, u'value': 1, u'minor': 0, u'op': u'Async'}, {u'major': 7, u'value': 1, u'minor': 0, u'op': u'Total'}, {u'major': 253, u'value': 1, u'minor': 2, u'op': u'Read'}, {u'major': 253, u'value': 0, u'minor': 2, u'op': u'Write'}, {u'major': 253, u'value': 0, u'minor': 2, u'op': u'Sync'}, {u'major': 253, u'value': 1, u'minor': 2, u'op': u'Async'}, {u'major': 253, u'value': 1, u'minor': 2, u'op': u'Total'}, {u'major': 253, u'value': 80, u'minor': 9, u'op': u'Read'}, {u'major': 253, u'value': 1, u'minor': 9, u'op': u'Write'}, {u'major': 253, u'value': 1, u'minor': 9, u'op': u'Sync'}, {u'major': 253, u'value': 80, u'minor': 9, u'op': u'Async'}, {u'major': 253, u'value': 81, u'minor': 9, u'op': u'Total'}], u'io_time_recursive': [], u'io_queue_recursive': [], u'io_merged_recursive': [], u'io_wait_time_recursive': []}, 
u'precpu_stats': {u'cpu_usage': {u'usage_in_usermode': 2342840000000, u'total_usage': 3253065851002, u'percpu_usage': [817652165400, 820700268313, 797229028908, 817484388381], u'usage_in_kernelmode': 661530000000}, u'system_cpu_usage': 13180102980000000, u'throttling_data': {u'throttled_time': 0, u'periods': 0, u'throttled_periods': 0}}, 
u'read': u'2017-10-11T02:56:25.044461613Z', 
u'memory_stats': {u'usage': 119660544, u'limit': 3975241728, u'failcnt': 0, u'stats': {u'unevictable': 0, u'total_inactive_file': 3723264, u'total_rss_huge': 111149056, u'hierarchical_memsw_limit': 9223372036854775807, u'total_cache': 3731456, u'total_mapped_file': 573440, u'mapped_file': 573440, u'pgfault': 11258904, u'hierarchical_memory_limit': 9223372036854775807, u'total_active_file': 8192, u'rss_huge': 111149056, u'cache': 3731456, u'active_anon': 86568960, u'pgmajfault': 9, u'total_pgpgout': 11256964, u'pgpgout': 11256964, u'swap': 0, u'total_active_anon': 86568960, u'total_unevictable': 0, u'total_pgfault': 11258904, u'total_pgmajfault': 9, u'total_inactive_anon': 0, u'total_swap': 0, u'inactive_file': 3723264, u'pgpgin': 11259095, u'total_pgpgin': 11259095, u'rss': 115929088, u'active_file': 8192, u'inactive_anon': 0, u'total_rss': 115929088}, u'max_usage': 273133568}, 
u'pids_stats': {}, 
u'networks': {u'eth0': {u'tx_dropped': 0, u'rx_packets': 8, u'rx_bytes': 648, u'tx_errors': 0, u'rx_errors': 0, u'tx_bytes': 648, u'rx_dropped': 0, u'tx_packets': 8}}, 
u'cpu_stats': {u'cpu_usage': {u'usage_in_usermode': 2345890000000, u'total_usage': 3256926141491, u'percpu_usage': [818621253198, 821672899753, 798195733077, 818436255463], u'usage_in_kernelmode': 662040000000}, u'system_cpu_usage': 13180106590000000, u'throttling_data': {u'throttled_time': 0, u'periods': 0, u'throttled_periods': 0}}
}
```

- 需要提取数据的部分为：CPU、memory、network、blkIO：
    - 对于CPU，分为`precpu_stats` 与 `cpu_stats`：
    - 对于 `total_usage` 与 `percpu_usage` 的关系，`percpu_usage`是物理机上每个物理核的CPU使用情况构成的数组，因为是4核，所以数组长度为4，因此`total_usage`为`percpu_usage`数组的和：`u'total_usage': 3253065851002, u'percpu_usage': [817652165400, 820700268313, 797229028908, 817484388381]`

- 因此在客户端直接模仿docker stats实现的源代码实现容器状态获取的功能：
```python
#获取磁盘blockIO：
for bioEntry in stats_data["blkio_stats"]["io_service_bytes_recursive"]:
    if bioEntry["op"] == "Read":
        blkRead += float(bioEntry["value"])
    elif bioEntry["op"] == "Write":
        blkWrite += float(bioEntry["value"])

#获取CPU利用率：
cpuPercent = 0.0
cpuDelta = float(stats_data["cpu_stats"]["cpu_usage"]["total_usage"]) - float(stats_data["precpu_stats"]["cpu_usage"]["total_usage"])
systemDelta = float(stats_data["cpu_stats"]["system_cpu_usage"]) - float(stats_data["precpu_stats"]["system_cpu_usage"])
if systemDelta > 0 and cpuDelta > 0:
    cpuPercent = (cpuDelta / systemDelta) * float(len(stats_data["cpu_stats"]["cpu_usage"]["percpu_usage"])) * 100

#网络IO：
for _, value in stats_data["networks"]:
    rx += float(value["rx_bytes"])
    tx += float(value["tx_bytes"])

#内存利用率：
memUsage = float(stats_data["memory_stats"]["usage"])
memLimit = float(stats_data["memory_stats"]["limit"])
if stats_data["memory_stats"]["limit"] != 0:
    memPercent = memUsage / memLimit * 100.0
```


# 实现现象
最后测试发现，如果使用远程访问主机的docker RESTFul API方式，可以在秒级别内返回主机端容器使用的资源信息

# 参考
[docker api 获取stats数据的方式](http://blog.csdn.net/l6807718/article/details/52023577)      
[docker stats命令源码分析结果](http://blog.csdn.net/WaltonWang/article/details/53930070?locationNum=8&fps=1)     
[How to compute CPU usage based on the info of "docker stats"（github issue）](https://github.com/moby/moby/issues/29306)      
[moby/cli/command/container/stats_helpers.go（docker部分源码）](https://github.com/moby/moby/blob/eb131c5383db8cac633919f82abad86c99bffbe5/cli/command/container/stats_helpers.go#L109)      
[moby/api/types/stats.go（docker部分源码）](https://github.com/moby/moby/blob/801230ce315ef51425da53cc5712eb6063deee95/api/types/stats.go#L164)     
