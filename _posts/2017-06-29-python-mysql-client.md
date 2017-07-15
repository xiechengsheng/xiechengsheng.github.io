---
layout:     post
title:      "SQL相关知识整理"
subtitle:   "SQL"
date:       "2017-06-29"
author:     "xiechengsheng"
header-img: "img/post-bg-12.jpg"
catalog: true
tags:
    - MySQL
    - Python
---

由于在平时在服务器端需要对采集到的各类信息进行保存到数据库，因此记录下和MySQL的相关操作；

# BS与CS模型
- bs模型：客户端通过浏览器，浏览web服务器上的网页，这样的模型叫bs模型，b指客户端browser，s指服务端server。在客户端和浏览器端之间走的报文是http协议（即超文本传输协议）
- cs模型：客户端（client）发报文，服务器（server）收报文，服务器收到报文之后处理。这与bs模式没有很大区别，只不过是c与s间可以自定义数据传送报文。cs模式一般走的协议是tcp协议；数据库服务器和客户端之间是CS关系；

# SQL语句
- 基础sql语句
```sql
创建数据库 create database dbname
删除数据库 drop database dbname
创建新表 create table tablename(col1 type1[primary key], col2 type2)
根据已有的表创建新表 create table tablenew like table old /create table tablenew as select col1,col2 from tableold definition only
增加列 alter table tablename add column coltype
增加主键 alter table tablename add primary key(col)
删除主键 alter table tablename drop primary key(col)
创建视图 create view viewname as select statement
删除视图 drop view viewname
选择 select *from table1 where...
范围插入 insert into table1(field1, field2)
删除 delete from table1 where ...
范围更新 update table1 set field1 = 1 where...
范围查找 select * from table1 where field1 like....
排序 select * from table1 order by field1,field2[desc]
总数 select count * as totalcount from table1
求和 select sum(field1) as sum from table1
平均 select avg(field1) as avg from table1
最大 select max(field1) as max from table1
最小 select min(field1) as min from table1
```

- msql服务器操作：服务器的一些安装、配置、客户端登陆等等
    - 对整个数据库与表的操作（不涉及数据）：
        ```sql
        列举MySQL中所有的数据库：
        show databases;
        创建数据库名为test的数据库：
        create database test;
        使用test数据库：
        use test;
        删除test数据库：
        drop database test;
        ```

    - 对表的操作：
        ```sql
        首先选择数据库：
        use dutylist;
        在数据库中创建一个表：
        create table if not exists table_test(id int, name varchar(64));
        打印所有的表：
        show tables;
        描述表的结构：
        mysql> describe du_category;
        +-------------+-------------+------+-----+---------+-------+
        | Field       | Type        | Null | Key | Default | Extra |
        +-------------+-------------+------+-----+---------+-------+
        | category_id | int(11)     | YES  |     | NULL    |       |
        | name        | varchar(64) | YES  |     | NULL    |       |
        +-------------+-------------+------+-----+---------+-------+
        也可以这样来描述表的结构，功能是一样的：
        mysql> show columns from du_category;
        向表中插入三条数据数据：
        mysql> insert into du_category(category_id, name) values(123, "hello"), (456, "world"), (789, "python");
        可以只向表的部分列中插入数据；
        查看表中的数据：
        > select * from du_category;
        +-------------+--------+
        | category_id | name   |
        +-------------+--------+
        |         123 | hello  |
        |         456 | world  |
        |         789 | python |
        +-------------+--------+
        清空一个表的所有数据（表还在）：
        delete from du_category;
        删除数据库中的一个表（表的结构都被删除）：
        drop table du_category;
        ```

# python操作MySQL
```python
# -*- coding: utf-8 -*-
# encoding = utf-8
# 导入数据库模块
import MySQLdb
# 连接数据库
conn = MySQLdb.connect(host='localhost', user='user', passwd='passwd', port='port')
# 获取操作游标
cursor = conn.cursor()
# 创建数据库
cursor.execute('create database if not exists dutylist')
# 选择数据库
conn.select_db('dutylist')
# 创建表
cursor.execute('create table if not exists du_category(category_id int, name varchar(64))')
cursor.execute('create table if not exists du_duty(duty_id int, category_id int, user_id int, title varchar(64), '
               'status int, is_show int, create_time int)')
cursor.execute('create table if not exists du_user(user_id int, username varchar(64), phone varchar(64), '
               'password varchar(64), create_time int)')
conn.commit()
# 写信息到表中方式1
cursor.execute("insert into du_category(category_id, name) values(1234567, 'hello world')")
conn.commit()
# 写信息到表中方式2，需要注意的是不论是些什么数据类型都是需要用%s来表示
cursor.execute('insert into du_category(category_id, name) values(%s, %s)', (7654321, "world hello"))
conn.commit()
# 使用方式2同时写入多条信息
cursor.executemany('insert into du_category(category_id, name) values(%s, %s)', ((123, "hello"), (456, "world"),
                   (789, "python")))
conn.commit()
# 刪除表中指定列的信息
cursor.execute('delete from du_category where name = "hello world"')
conn.commit()
# 获取表的信息，获取的信息列表中没有字段信息，需要手动输入
cursor.execute('select * from du_category')
information = cursor.fetchall()
for info in information:
    print 'category_id: %d; name: %s' % (info[0], info[1])
cursor.execute('delete from du_category where category_id = 123')
conn.commit()
# 更新信息
cursor.execute('update du_category set name = "magic" where category_id = 456')
conn.commit()
cursor.execute('delete from du_category')
conn.commit()
# 获取表的信息，列表中包含字段信息
cursor = conn.cursor(cursorclass=MySQLdb.cursors.DictCursor)
cursor.execute('select * from du_category')
information = cursor.fetchall()
for info in information:
    print info
cursor.close()
conn.close()    # 断开连接
```

# 参考
[bs模型与cs模型](http://www.cnblogs.com/baoxiaofei/p/4101241.html)    