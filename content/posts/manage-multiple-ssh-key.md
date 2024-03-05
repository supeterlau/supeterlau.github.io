---
title: "管理多个 SSH Key"
date: 2019-09-16T13:24:59+08:00
draft: false
---

SSH Key 最大的优势是免输入复杂的密码，一定程度避免中间人攻击。

多站点管理可以通过 SSH Config 解决。

多站点多账户可以用类似方法。 https://gist.github.com/jexchan/2351996

弊端，需要修改 remote url 

更简单方法，使用支持多账户的 Git 客户端管理。

更个人化方法，自己实现 Git 客户端管理。

这里记录一个基于 Docker 的方法。思路很简单，用基于 Alpine 的最小化镜像，安装 Git OpenSSH 后通过 docker run 执行 git 操作。

Generate ssh key

ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

eval "$(ssh-agent -s)"

ssh-add -K ~/.ssh/id_rsa

ssh-keygen -p

docker run -it --rm -v $(pwd):/repo -v $HOME/.ssh/docker_config/ssh:/root/.ssh bebopx/git:alpine cd /repo;ssh -T git@github.com

docker run -it --rm -w /repo -v $(pwd):/repo -v $HOME/.ssh/docker_config/ssh:/root/.ssh bebopx/git:alpine git clone git@github.com:bebopxff/links_share.git

docker run -it --rm -w /repo -v $(pwd):/repo -v $HOME/.ssh/docker_config/ssh:/root/.ssh bebopx/git:alpine git push

docker run -it --rm -w /repo -v $(pwd):/repo -v $HOME/.ssh/docker_config/ssh:/root/.ssh bebopx/git:alpine git clone git@github.com:bebopxff/links_share.git

