---
title: 开始使用Docker
date: 2014-09-05 20:16:09
tags: [docker, web]
---

# 什么是 Docker?

Go语言编写的应用容器.所谓应用容器,就是可以像虚拟机那样下载下来,已经配置好大部分的开发环境(比如有了一些 Python 包, Gem 装好了一部分).新一代的一次构建到处运行.据说开始是基于 LXC ,这个还没有研究.不过已经宣布将来不再依赖这个, 而是变成可插拔的插件, 插件流行啊.

这货像是个虚拟机,但不带图形界面,当然对我这种命令控没影响.说起来像是通过 SSH 访问Vbox.

# 开始使用

注册 docker.io dockerhub 帐号 不注册也可以用...

1. curl -s https://get.docker.io/ubuntu/ | sudo sh
2. sudo docker run -i -t ubuntu /bin/bash

  下载的是一个ubuntu镜像 或者 ubuntu:14.04 指定是 14.04 版本

  下载后的镜像称为容器, 在容器内可以运行应用如 bash

3. sudo docker run ubuntu:14.04 /bin/echo 'Hello world'

  输出 Hello world 在容器内执行, 返回结果

4. sudo docker run -t -i ubuntu:14.04 /bin/bash

  执行交互式容器, 不过是在14.04版本中运行的

5. 每次 docker run 的时候就是通过一个镜像创建了一个容器, 如后台容器

  守护进程 sudo docker run -d ubuntu:14.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"

  脚本会一直输出"hello word"

  只有通过 sudo docker ps 查看后台进程找到指代名称

  然后用 sudo docker log short-name 查看结果

6. sudo docker stop 结束这个后台

7. 只输入docker没有任何其它选项来查看docker客户端所有的命令

8. 输入Docker [command]，这里会看到docker命令的使用方法：

  $ sudo docker attach

9. sudo docker run -d -P training/webapp python app.py

  -d 后台执行

  -P docker网络端口从主机映射到容器

  training/webapp 指定镜像

  python 容器

  see ubuntu version:
  cat /etc/issue
  see whoami
  whoami

10. 查看web应用容器

  sudo docker ps -l # -l 返回最后的容器状态

  CONTAINER ID        IMAGE                    COMMAND             CREATED             STATUS              PORTS                     NAMES
  9a57847b4be6        training/webapp:latest   "python app.py"     5 minutes ago       Up 5 minutes        0.0.0.0:49153->5000/tcp   trusting_sammet

  容器开启 5000(python) 端口映射到主机 49153 上

  可以访问 localhost:49153 到容器的webapp上

11. 使用docker port可以指定容器ID或者名字映射到主机端的端口号

  sudo docker port nostalgic_morse 5000

  查看到端口5000映射到容器外的主机端口

12. sudo docker logs -f hungry_stallman

  查看容器内应用发生的日志

13. sudo docker top hungry_stallman

  查看应用的过程

14. 使用docker inspect来查看Docker的底层信息。它会返回一个JSON文件记录docker容器的配置和状态信息

15. 
    sudo docker stop hungry_stallman

    停止容器

    sudo docker start hungry_stallman

    重启容器

    sudo docker rm hungry_stallman

    删除容器 == 我们不能删除正在运行的容器

16. 获取一个新的镜像

  sudo docker run -t -i training/sinatra /bin/bash
  
  进入容器的bash中

17. 
  运行一个带标签镜像的容器：

  $ sudo docker run -t -i ubuntu:14.04 /bin/bash

  例如你只使用Ubuntu，Docker将默认使用Ubuntu:latest镜像。

18. sudo docker pull centos 获取新镜像

19. 如何把多个docker容器连接在一起构建一个完整的应用程序

20. *HOWTO* 退出docker 保留docker配置...

  一开始想着试试 perl6 呢,谁知道配置好 vim (对的, vim 都没有, 也没有图形界面 啊) 退出以后就一切重置了, 难道要按照 DOCKERFILE 什么的步骤走吗??? 以后再看吧. Perl 看来得再找时间看看了.
