---
title: Hylang 快速指南
date: 2014-10-12 12:30:43
tags: [python, lisp]
---

前几天在 [XahLee][1] 的博客上看到关于 Ruby 和 Python 的讨论, 说 Python 太要求 oop 形式而又不够 oop ,例如, 没有 `3.times` 这种用法, 却有 `len(str)` , 而且不够函数式 \( <<黑客与画家>>提到 "Perl Python Ruby", 越在后越函数式\).

* Markdown 括号加括号 格式就混乱 尽量少用

今天看到了 Hylang, 一个允许用 Lisp 风格写 Python 的解释器.

翻译下5分钟入门:

<img src="http://docs.hylang.org/en/latest/_images/cuddles-transparent-small.png">

\(Thanks to Karen Rustad for Cuddles!\)

快速使用 HY:

1. 创建 Python 虚拟环境 : [见前篇][2]

2. 激活虚拟环境

3. 从 PyPI 安装 hy: `pip install hy`

4. 使用`hy`打开 REPL

    `=> (print "Hy!")`

    `Hy!`

    `=> (defn salutationsnm [name] (print (+ "Hy " name "!")))`

    `=> (salutationsnm "YourName")`

    `Hy YourName!`

    etc

    使用 CTRL + D 退出

5. 脚本运行

    保存`(print "I was going to code in python syntax, but then I got hy.")`为`awesome.hy`
    
    执行 `hy awesome.hy`

Take a deep breath so as to not hyperventilate

Smile villainously and sneak off to your hydeaway and do unspeakable things

既然 Python 中有模块支持函数式编程, 这样一种解释器的意义在哪里?

对于学习两种语言语法树的转换很有意思吧.



[1]: http://xahlee.info/
[2]: http://andyhelix.gitcafe.com/2014/09/10/have-fun-with-django-on-sae/

