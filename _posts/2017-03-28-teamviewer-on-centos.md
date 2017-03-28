---
layout:     post
title:      "在centos上部署安装teamviewer"
subtitle:   "安装teamviewer"
date:       "2017-03-28"
author:     "xiechengsheng"
header-img: "img/post-bg-3.jpg"
catalog: true
tags:
    - tools
---

有时候如果自己的工作站和服务器不在同一局域网下面，使用Xshell无法连接到服务器，teamviewer是一个穿透内网的远程控制服务器的很好的解决方案；

## CentOS下面使用指令安装teamviewer
1. 下载安装包，选择11版本安装包：
`teamviewer_11.0.67687.i686.rpm`
可以在官网上面查看各种系统各种版本安装包：[官网下载链接](https://www.teamviewer.com/zhcn/download/linux/)

2. 安装一系列依赖包：
`yum install libpng12.so.0 libjpeg.so.62 libXdamage.so.1 libXfixes.so.3 libXinerama.so.1 libXrandr.so.2  libgcc_s.so.1 libfreetype.so.6 libfontconfig.so.1 libdbus-1.so.3 libasound.so.2 libXtst.so.6 libXrender.so.1 libSM.so.6 libdamage.so.1 libz.so.1 -y`
* 如果报错，提示缺少依赖包：
` Error: Package: teamviewer-13.0.6634-0.x86_64 (/teamviewer.x86_64)
           Requires: libQt5WebKitWidgets.so.5()(64bit) >= 5.5 `
* 去官网上面下载对应Qt包安装：
```sh
wget http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/q/qt5-qtwebkit-5.6.2-1.el7.x86_64.rpm
rpm -Uvh qt5-qtwebkit-5.6.2-1.el7.x86_64.rpm --force --nodeps
```

3. 此时teamviewer已经在系统上面正常安装；首先停止teamviewer进程：
`teamviewer --daemon stop`

4. 修改teamviewer配置文件：
`vim /opt/teamviewer/config/global.conf`，在配置文件中添加：
```
[int32] EulaAccepted = 1
[int32] EulaAcceptedRevision = 6
```

5. 启动teamviewer服务：`teamviewer --daemon start
`
6. 设置新密码：`teamviewer --passwd [NEWPASSWORD]`
**注意：必须先设置teamviewer新密码，不然查看不到teamviewer id**

7. 获取teamviewer id与密码： `teamviewer --info print id`

8. 设置为开机自启动：
```sh
systemctl enable teamviewerd.service
systemctl start teamviewerd.service
systemctl status teamviewerd.service
```

其他比较优秀的远程桌面控制工具，比如向日葵、QQ等，有机会可以尝试下......

## 参考
[CentOS 7下安装TeamViewer](http://blog.csdn.net/RBPicsdn/article/details/78972100)