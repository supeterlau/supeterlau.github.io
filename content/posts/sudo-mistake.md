---
title: 一次由 Linux 下无声音引发的权限错误惊魂记
date: 2014-10-04 13:57:00
tags: [ubuntu, sudo, permission, alasmixer]
---

今天遇到了一个惊险的事情, 笔记本的 Ubuntu 系统没有声音.

开始以为是模块没有加载, 检查后发现自己认不出来哪个是的. 用 alasmixer 结果显示 ` alsamixer cannot open mixer: No such file or directory` 在网上搜了一通以后发现, 有个命令需要 sudo 权限.

于是就 sudo 执行, 出来问题了, 居然说我不在 sudoer 用户组里.

使用 groups 查看用户组, 发现只有 username 和 www-data

结果很明显:

昨天在给 yii 框架用户数据赋予服务器写权限时, 我用的是该死的 `usermod -G groupname username` 这就使得我的用户名离开了其他所有的组.

实际应该使用 `usermod -a -G groupname username`

或者 `gpasswd -a username groupname`

好吧, 我现在的问题就是要在一个只有 root 和一个 user 的机器上, 给这个 user sudo 权限...

首先我没有一个能有 sudo 权限的帐号啊, root 不知道有没有设密码...

恩, 破解 root 密码.

使用 grub 进入 `init 1` 的单用户界面

对了, 网上一堆 grub 的命令, 我的是 grub2 ... 囧

其实还算简单, 在启动页对应启动项按 e , 在 `linux /boot ...` 一堆里的 root 参数后加上 `init 1` 再后边的参数删去, 然后就 F10 启动. OK

**反思**

    1. 命令行连个恐怖提示都没有 下次可能就 `rm -rf /` 了吧...
    2. 一定添加一个备用帐号
    3. 我一定记得给 root 加上密码, 这次好好设置这里了. 诶要, 我的 root 密码多少来着???
