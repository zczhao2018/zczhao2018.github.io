---
layout:     post                    # 使用的布局（不需要改）
title:      My First Post               # 标题 
subtitle:   Hello World, Hello Blog #副标题
date:       2017-02-06              # 时间
author:     BY                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 生活
---

Linux命令:alias

功能说明:设置指令的别名。
语 法:alias[别名]=[指令名称]
补充说明:用户可利用alias,自定指令的别名。若仅输入alias,则可列出目前所有的别名设置。
alias的效力仅及于该次登入的操作。若要每次登入是即自动设好别名,可在~/.bashrc中设定指令的别名。
**1.设置别名,修改**
```
vim ~/.bashrc
```
在vim编译器下,输入i进行编写,找到下面代码,在下面添加即可
```
# some more ls aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
```

**2.示例一:冗长路径简约化**
```
# define alias for zczhao
alias documents='cd /home/zczhao/Documents'
alias downloads='cd /home/zczhao/Downloads'
```
**3.示例二:ssh**
远程登陆服务器,每次都要输入密码是一件很烦的事情,这里有一个命令就是sshpass,首先需要安装sshpass
安装sshpass
```
sudo apt-get install sshpass
```
之后的登陆命令就变成了
```
sshpass -p password ssh usernname@xxx.xxx.xxx.xxx
```
从此不用再输入密码,然后修改alias即可
```
alias xxx='sshpass -p password ssh usernname@xxx.xxx.xxx.xxx'
```
保存:wq
**4.重新载入**
记得修改完之后一定要重新载入,不然不会生效,博主就是修改了很久,忘了载入,还以为是命令出现了问题呢,其实不是,只是没有激活

```
source ~/.bashrc
```
**5.查看alias设置情况**

```
alias
```
然后愉快的使用骚气的命令吧!
