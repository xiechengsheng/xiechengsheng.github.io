---
layout:     post
title:      "Shell语法相关知识整理"
subtitle:   "shell"
date:       "2017-07-12"
author:     "xiechengsheng"
header-img: "img/post-bg-13.jpg"
catalog: true
tags:
    - Shell
---

学习部分shell相关语法；虽然记在博客里面还是会忘，但是需要用的时候可以来查一哈，以后不会的、记不住的指令会继续往这里添加；

# if
```sh
# 1.if [ -z STRING ] "STRING" 的长度为零则为真，会进入then程序块
if [-z "${DEBUGGER}"]; then
     echo "the ${DEBUGGER} is null"
fi
```

# shell读取脚本参数
```sh
# 写的shell脚本需要带上参数可以使用${1}、${2}...等获取键入的参数：
# cat test.sh
echo ${1}

# 获取用户键入的第一个参数：
# ./test.sh hello
hello
```

# for循环：
```sh
i=1
for i in $(seq 1 256)
do
  echo "$i times hello world"
  i=$(expr $i + 1)
done
```

# while循环
```sh
# while循环：
i=1
while [ $i -le 256 ]
do
    echo "$i times hello world"
    i=$(expr $i + 1)
done
```

# echo追加写文件
```sh
# 使用echo直接以追加方式将字符串写入文件：
echo "hello" >> test.txt
echo "world" >> test.txt
# 注意是两个>>，如果只有一个>，那么直接就是覆盖写
# cat test.txt
hello
world
```

```sh
# 使用单个'>'会覆盖整个文件：
echo "hello" > test.txt
echo "world" > test.txt
# cat test.txt
world
```

# if...else...
```sh
if [ $j -eq 0 ]
then
    echo "0"
else
    echo "!=0"
fi
```

# 当前路径下面不存在该文件夹的话，一直等待直到存在该文件夹为止：
```sh
while [ ! -d "./folderName/" ]
do
    sleep 1
done
```

# 捕获shell某条指令输出的行数
`ls -lht | wc -l    # 输出当前文件夹下面文件数目`

# 获取Xshell连接到的Linux系统：
```sh
方法1：lsb_release -a
方法2：cat /etc/issue
```