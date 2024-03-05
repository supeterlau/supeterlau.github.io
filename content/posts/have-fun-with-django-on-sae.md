---
title: 新浪云平台使用Django填坑指南
date: 2014-09-10 15:09:30
tags: [django, sae] 
---

## 新浪云平台使用手册

[for python][1]

注意 django 的开发框架默认是 1.4 的, 如果你已经安装了 1.6 甚至 1.7 怎么办呢?

当然用 virtualenv + virtualenvwrapper 咯

使用 `pip install virtualenvwrapper` 在环境配置文件 `$HOME/.bashrc` 中添加 `source /usr/local/bin/virtualenvwrapper.sh` 然后 `mkvirtualenv` 命令创建新环境 比如 **django14** 这样将在 $HOME/.virtualenvis/ 中建立隔离的开发环境

>    切换环境

>    - workon [env]

>    随时使用“workon 环境名”可以进行环境切换，如果不带环境名参数，则显示当前使用的环境

>    - deactivate

>    在某个环境中使用，切换到系统的python环境

>      其他命令

>      - showvirtualenv [env] 显示指定环境的详情。

>      - rmvirtualenv [env] 移除指定的虚拟环境，移除的前提是当前没有在该环境中工作。如在该环境工作，先使用deactivate退出。

>      - cpvirtualenv [source] [dest] 复制一份虚拟环境。

>      - cdvirtualenv [subdir] 把当前工作目录设置为所在的环境目录。

>      - cdsitepackages [subdir] 把当前工作目录设置为所在环境的sitepackages路径。

>      - add2virtualenv [dir] [dir] 把指定的目录加入当前使用的环境的path中，这常使用于在多个project里面同时使用一个较大的库的情况。

>      - toggleglobalsitepackages -q 控制当前的环境是否使用全局的sitepackages目录。

接下来使用 django14 环境中的 django-admin.py 创建项目

` ~/.virtualenvs/django14/bin/django-admin.py startproject djangosite`



## sae._restful_mysql 问题

本身用于在本地直接生成远端数据库表的库, 需要安装 sae-python-dev 直接 pip install sae-python-dev 即可

在本地使用 python manager.py syncdb 就可以创建远端数据库表了

本地创建完了, 是不是挺高兴, 但是上传到 SAE 就会提示 找不到 _restful_mysql 为何?

可见 sae-python-dev 是作为第三方库提供的. 因此或者用[安装第三方库的方法][2]或者上传前注释掉这两句

PS: 使用 django1.6 实际是可以用的, 甚至不需要这个 monkey 就可以在远端创建表, 看来 SAE 已经更新了版本了?

PSS: 总之, 已经可以在 SAE 上开发带数据库支持的 Django 应用了. 让我们开始海盗之旅咯

[1]: http://sae.sina.com.cn/doc/python/index.html
[2]: http://sae.sina.com.cn/doc/python/faq.html
