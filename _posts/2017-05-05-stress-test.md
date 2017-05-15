---
layout:     post
title:      "网站并发压力测试"
subtitle:   "stress test"
date:       "2017-05-05"
author:     "xiechengsheng"
header-img: "img/post-bg-9.jpg"
catalog: true
tags:
    - tools
    - docker
---

寻找在docker上面部署较为广泛的体现服务器性能的测试工具；

# nginx in docker
Nginx (engine x) 是一个高性能的HTTP和反向代理服务器，其特点是占有内存少，并发能力强，事实上nginx的并发能力确实在同类型的网页服务器中表现较好；
```sh
下载容器：docker pull nginx
以交互方式启动nginx（不能对外提供转发服务）：docker run -it -p 8080:80 docker.io/library/nginx /bin/bash
以后台方式启动nginx：docker run -d -p 8080:80 docker.io/library/nginx
```

- 查看本机8080端口号监听情况：`netstat -ntlp | grep 8080`
- 打开本地浏览器，输入 `http://host_ip:8080` 可查看到nginx的hello欢迎界面
- 安装ab测试工具：`yum install httpd-tools -y`

并发性测试：
```sh
# ab -kc 1000 -n 1001 http://192.168.3.87:8080/index.html
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 192.168.3.87 (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Completed 600 requests
Completed 700 requests
Completed 800 requests
Completed 900 requests
Completed 1000 requests
Finished 1001 requests

Server Software:        nginx/1.13.5
Server Hostname:        192.168.3.87
Server Port:            8080

Document Path:          /index.html
Document Length:        612 bytes

Concurrency Level:      1000
Time taken for tests:   0.273 seconds
Complete requests:      1001
Failed requests:        0
Write errors:           0
Keep-Alive requests:    1001
Total transferred:      850850 bytes
HTML transferred:       612612 bytes
Requests per second:    3669.93 [#/sec] (mean)
Time per request:       272.485 [ms] (mean)
Time per request:       0.272 [ms] (mean, across all concurrent requests)
Transfer rate:          3046.33 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0   69  24.6     69     111
Processing:    30   81  28.0     81     151
Waiting:        2   81  28.1     80     151
Total:         30  150  15.6    148     248

Percentage of the requests served within a certain time (ms)
  50%    148
  66%    150
  75%    151
  80%    151
  90%    153
  95%    154
  98%    240
  99%    246
 100%    248 (longest request)
```
- 测试参数说明：测试指令中 -n表示请求数，-c表示并发数, -k表示使用http的keepalive特性
- 测试结果说明：
```sh
Requests per second: 3669.93 [#/sec] (mean)     #表示当前测试的服务器每秒可以处理3669.93个静态html的请求事务，后面的mean表示平均。这个数值表示当前机器的整体性能，值越大越好。
Time per request: 272.485 [ms] (mean)     #衡量单个请求的延迟，cpu是分时间片轮流执行请求的，多并发的情况下，一个并发上的请求时需要等待这么长时间才能得到下一个时间片。计算方法Time per request: 0.272 [ms] (mean, across all concurrent requests)*并发数，通俗点说就是当以-c 10的并发下完成-n 1000个请求的同时，额外加入一个请求，完成这个请求平均需要的时间。
Time per request: 0.272 [ms] (mean, across all concurrent requests)     #衡量性能的标准，它反映了完成一个请求需要的平均时间,在当前的并发情况下，增加一个请求需要的时间。计算方法Time taken for tests: 0.273 seconds/Complete requests: 1000，通俗点说就是当以-c 10的并发下完成-n 1001个请求时，比完成-n1000个请求多花的时间。
```

- 测试效果：在 响应 客户端 ab 进程 的并发请求时， 耗费整个CPU资源，和部分网络IO资源（带宽大约为1MB/s-3MB/s），不耗费内存和Blk IO 资源；并且使用ab 进程对其进行测试，响应10000个并发请求的时间为 **20秒** 左右；

# Tomcat in docker
- Tomcat 服务器是一个免费的开放源代码的Web 应用服务器，同样也适用于高并发服务器中； 
- 启动Tomcat in docker：
```
# 限制资源为 128MB内存，0.5 核CPU
docker run --cpu-quota 50000 --cpu-period 100000 -m 512MB -it -p 8080:8080 --rm tomcat:8.0
```
- 使用ab测试：`ab -kc 10 -n 100000 http://192.168.3.87:8080/`，测试输出和上面nginx的输出一致；
- 测试效果：网络IO大概10MB/s-20MB/s，不占用Blk IO， 占用全部的CPU和部分内存；


# 参考
[NGINX压力测试](https://github.com/bingbo/blog/wiki/NGINX%E5%8E%8B%E5%8A%9B%E6%B5%8B%E8%AF%95)      
[使用ab对nginx进行压力测试](http://www.nginx.cn/110.html)
