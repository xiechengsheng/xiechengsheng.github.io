---
layout:     post
title:      "vim指令及其相关插件"
subtitle:   "vim"
date:       "2017-07-15"
author:     "xiechengsheng"
header-img: "img/post-bg-14.jpg"
catalog: true
tags:
    - tools
---

- 虽然使用vim写代码效率会降低，但有些时候只能使用vim进行操作；
- 记录下普通vim下面一些快捷键；以及vim安装tmux插件与oh-my-zsh插件；

# vim指令基本知识
- 基本快捷键：
```sh
:w     ：将文件存盘但是不退出当前编辑界面
h       ：光标左移一位
j          ：光标下移一位
k          ：光标上移一位
l          ：光标右移一位
sp + 文件名：可以水平分割窗口
vs + 文件名：可以垂直分割窗口
Ctrl + w：可以快速在窗口间切换
/word    ：向下查找 word 的字符串；例如  /ABCD   向下查找字符ABCD，绿色光标处即为查找结果，使用n查找下一个字符串，使用N查找上一个字符串
?word    ：向上查找word 的字符串
dd ：可以删除一行
```

- 关于复制粘贴删除（使用块选择实现）：
```sh
v：选中字符
V：选中一行
Ctrl+v：长方形的方式选择块数据
######################################
选中之后就可以进行复制或者删除操作：
y：将选中的部分复制
p：将选中的部分粘贴
d：将选中的部分删除
全部删除：dG
全部复制：ggyG
######################################
0或者HOME键：移动到该行的最前面的字符处
$或者END键：移动到该行的最后面的字符处
######################################
Ctrl + f：屏幕向下移动一页 ，相当于page down
Ctrl + b：屏幕向上移动一页 ，相当于page up
```

- 一键下载多种vim插件并安装：
`curl https://raw.githubusercontent.com/gavin-hu/oh-my-vim/master/bootstrap.sh -L -o - | sh`

- 一些花哨的vim插件，时间过得久了自己好像也没有怎么用，自己感觉最有用的还是tmux和oh-my-zsh；

# tmux
- 使用Tmux之后，我可以开出很多窗口，将其拆分成很多面板，接管和分离会话等等；也就是说，这里看到的所谓终端控制台应该称作tmux的一个面板；
- tmux使用C/S模型构建，主要包括以下单元模块：
    - 一个`tmux`命令执行后启动一个tmux服务
    - 一个tmux服务可以拥有多个`session`，一个session可以看作是tmux管理下的伪终端的一个集合
    - 一个session可能会有多个`window`与之关联，每个window都是一个伪终端，会占据整个屏幕
    - 一个window可以被分割成多个`pane`

- 相关操作指令：
```
tmux ls：列出所有会话
ctrl+b是基础，先同时按下ctrl+b之后，再按下：
    数字键：切换到相应数字的窗口
    c：创建新窗口（window）
    &：关闭当前窗口
    %：将当前面板分成左右两块
    "：将当前面板分成上下两块
    x：关闭当前面板
    上下左右方向键：在不同面板之间跳转
    ctrl+上下左右方向键：调整当前面板大小
```
- 发现tmux不能查看终端显示的之前的内容，之前的内容被当前内容刷新后，再次显示的时候是乱码；解决方案：
```
Ctrl+b，之后按下[，进入翻页模式，按下q退出翻页模式
或者直接Ctrl+b，之后按下Pgup翻页，按q退出
```

# oh-my-zsh
- oh-my-zsh最大的方便之处就在于：可以对添加在配置文件`~/.zshrc`中**plugins里面的指令，按下TAB之后可以智能提示**，其实就是将这些指令添加到zsh的环境变量中；
```sh
首先安装zsh：
yum install -y zsh
其次安装zsh配置文件：
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
配置zsh主题文件：
vim ~/.zshrc
主要配置，将自己想要有智能提示的指令添加到plugins里面：
plugins=(git docker vim go python gcc bundler osx rake ruby)
ZSH_THEME="robbyrussell"
重启生效：
source /root/.zshrc
最后修改默认的shell：
chsh -s /bin/zsh
出现：chsh: Shell not changed，其实已经被修改成zsh了
若不想使用zsh，可以切换回bash模式：
chsh -s /bin/bash
```
- 注意：因为和bash是两种不同的模式，因此**原来在/root/.bashrc下面配置的环境变量的信息在zsh模式下面会完全失效**


# 参考
[How to improve your productivity in terminal environment with Tmux](http://xmodulo.com/improve-productivity-terminal-environment-tmux.html)    
[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)    