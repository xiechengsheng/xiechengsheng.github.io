---
layout:     post
title:      "在centos上部署安装mysql"
subtitle:   "安装mysql"
date:       "2017-04-10"
author:     "xiechengsheng"
header-img: "img/post-bg-6.jpg"
catalog: true
tags:
    - tools
---

CentOS上面由于没有现成的mysql包，因此在系统上面安装mysql数据库比Ubuntu复杂一点；

# CentOS安装Mysql
1. 首先查看本机是否已经安装mysql：`yum list installed | grep mysql`
2. 查看yum源中提供的mysql包：`yum list | grep mysql`
3. 发现centos7.2下面，没有可以直接用于安装的mysql包，手动下载安装包：`wget http://dev.mysql.com/get/mysql57-community-release-el7-8.noarch.rpm`
4. 安装mysql源：`yum localinstall mysql57-community-release-el7-8.noarch.rpm -y`
5. 安装mysql服务器和客户端：`yum install mysql-community-server -y`
6. 启动mysql服务：`systemctl start mysqld`
7. 查看mysql服务启动状态，并配置成开机自启动：
    ```sh
    systemctl status mysqld
    systemctl enable mysqld
    systemctl daemon-reload
    ```
8. 安装成功之后，在 `/var/log/mysqld.log` 文件中会生成root的默认密码，查询到root默认密码并登陆：
    ```sh
    # grep 'temporary password' /var/log/mysqld.log
    2017-09-08T02:14:29.220897Z 1 [Note] A temporary password is generated for root@localhost: vzSi.Xz>;0Uy
    ```
    - 登录输入密码的时候需要输入默认设置好的密码：
        ```sh
        mysql -u root -p
        Enter password:
        ```

9. 登陆后将密码设置成root，首先需要修改有效密码的规则，其次再修改密码：    
    ```sh
    #设置密码的规则，相当于正则表达式
    set global validate_password_policy=0;
    set global validate_password_mixed_case_count=0;
    set global validate_password_special_char_count=0;
    set global validate_password_length=4;
    set global validate_password_number_count=0;

    mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('root');
    Query OK, 0 rows affected, 1 warning (0.00 sec)
    ```

10. 要想从远地机器远程访问本机mysql服务，使用：
    ```sh
    # mysql -u root -h 192.168.3.89 -P 3306 -p
    Enter password:
    ERROR 1130 (HY000): Host 'k8s-mst' is not allowed to connect to this MySQL server
    ```

- 发现报错，原因是因为系统数据库mysql中的user表的host是localhost，因此只能使用localhost方式登陆服务器，**将host修改成为服务器的ip**，可解决问题：

    ```sh
    mysql -u root -p
    Enter password:

    mysql> use mysql;
    mysql> update user set host='192.168.3.89';
    ```

- 客户端退出mysql，之后重新启动mysql服务：`systemctl restart mysqld`
- 做出这样的修改之后，以后想登陆mysql，必须使用IP的方式登陆，在本地不能使用localhost的方式了：
```sh
# mysql -p
Enter password:
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
```

11. 但是，客户端（192.168.3.87）在远程登陆mysql服务器还是报错：
```sh
# mysql -h 192.168.3.89 -P 3306 -p
Enter password:
ERROR 1130 (HY000): Host 'k8s-nod1' is not allowed to connect to this MySQL server
```

- 解决方案是在MySQL本地服务器中添加授权信息，允许root用户在任何机器上以root密码登陆mysql服务器：
```sh
set global validate_password_policy=0;
set global validate_password_mixed_case_count=0;
set global validate_password_special_char_count=0;
set global validate_password_length=4;
set global validate_password_number_count=0;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

# 参考
[CentOS 7.2 安装MySQL 5.7](http://jingyan.baidu.com/article/455a9950448580a166277887.html)    
[mysql5.7设置简单密码报错ERROR 1819 (HY000): Your password does not satisfy the current policy requirements](http://blog.csdn.net/kuluzs/article/details/51924374)    
[mysql远程连接：ERROR 1130 (HY000): Host ‘*.*.*.*’ is not allowed to connect to this MySQL](http://www.th7.cn/db/mysql/201606/191057.shtml)
[报错:1130-host ... is not allowed to connect to this MySql server 开放mysql远程连接 不使用localhost](http://www.cnblogs.com/xyzdw/archive/2011/08/11/2135227.html)
