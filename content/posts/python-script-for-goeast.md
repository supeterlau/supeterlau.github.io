---
title: 针对 GOEAST 的 Python 脚本
date: 2014-11-16 19:49:49
tags: [go, goeast, python]
---

真没打算有人能搜到这个文章, 过去两周时间, 为了能批量从 GOEAST (中国科学院基因所的 GO 富集聚类分析服务) 下载分析结果, 我写了这个脚本. 等于是复习了 Requests 这个http网络库.

整个的风格是过程式的. 多文件任务没有进行多线程或多进程优化, 导致效率极低下. 更多代码见[Github][1]

[1]: https://github.com/Peter Lau/Userscript/blob/master/python/spiders/dataup.py
