---
title: Hexo博客使用之旅 
date: 2014-03-21 21:55:16
tags: [hexo,blog]
---

-   Hexo 是什么? 
-   五步拥有 Hexo 
-   Hexo 工作流程 
-   优化配置 

* * * * *

### 篇头语 

1.  Markdown 的语法中注释用 `<!-- -->`.
2.  hexo new "title\_here" 处理 Hexo 这种带 x 的居然有问题?
3.  markdown 简体中文[链接](http://wowubuntu.com/markdown/ "简体中文版"),好好练习吧!!!

<!--more-->

### Hexo 是什么? 

> [tommy351](https://github.com/tommy351/hexo "台湾小伙")发布的基于node的静态博客框架.类似 jekyll, 不过是基于 node/nodejs 的.(我是中英混杂痛恨者,文字前标号需空格执行者, 中英标号统一使用者, 标签化信徒.)

EMACscript 或者俗称 Javascript 越来越强大了. Js 可以写桌面应用, 写 html, 写 css, 写动画, 写动态图表, 写可视化数据, 写服务器, 几乎无所不能了.这个玩具很好玩.

 ### 五步安装 Hexo 到 [Gitcafe](https://gitcafe.com/ "天朝代码托管商")

-   第一步: 安装 Node , 感谢 [Ryan Dahl]("https://www.google.com.hk/search?q=Ryan+Dahl&oq=Ryan+Dahl")(每次都要跟 DHH 搞混)![](http://nodegeek.net/wp-content/uploads/2013/12/ryan_dahl-300x172.jpg)

然后你就有了 npm (Node Package Manager) 这个东西. 打开终端 / CMD, 执行

        npm install -g hexo

-   第二步: 在本地博客文件夹中 (例如 \~\\hexo) 执行

          hexo init

    生成基本文件结构. 你可以随便点开看看. 本地预览你的博客:

          hexo generate
          hexo server

    本地版博客就运行在 [localhost:4000](http://localhost:4000)
    退出还是`Ctrl+c`, 恭喜你拥有了一个完全控制的博客了, 可以写写日志,
    记录感悟......但是不发布怎么行?

-   第三步: 发布. 没有发布怎么体现高度呢? 样式, 模板, 动态, 标签云,
    挂件, Wordpress 有的咱都要有!这里就说说发布到 Gitcafe 吧.
    据说有个专门为极客准备的云主机也不错.自己
    [谷歌](http://www.google.com).

    -   3.1 注册 [gitcafe](http://gitcafe.com), [生成 SSH
        公钥](https://help.github.com/articles/generating-ssh-keys "帮助看这里"),
        在[SSH 公钥管理](https://gitcafe.com/account/public_keys)
        上传公钥 : *\**.pub 文件,
    -   3.2 **坑一** gitcafe 中必须创建一个和用户名相同的公开项目, SSH
        为 git@gitcafe.com:username/username.git 两段必须一致,
    -   3.3 **坑二** 代码必须提交到 gitcafe 的 gitcafe-pages
        分支才能生成页面, github 是 master 分支.
    -   3.4 修改 \~\\hexo 下 \_config.yml, 在结尾 deploy 项添加发布代码:

            deploy:
              type: github
              repository: git@gitcafe.com:\*yourname*\/\*yourname*\.git
              branch: \*gitcafe-pages*\

    -   3.5 下一步:

            hexo generate
            hexo deploy

        提示：每次修改本地文件后，需要 `hexo generate`
        才能保存.每次使用命令时，都要在 \~/hexo 目录下.

    -   3.6 可以通过
        [](http://yourname.gitcafe.com/)[http://yourname.gitcafe.com/](http://yourname.gitcafe.com/)
        访问你的博客了. 搭建好了记得通知我哦, 参考下你们的设计.
        我乃设计麻瓜.

-   第四步: 学习以下技术:

    -   git
    -   js/node
    -   css
    -   ps/gimp
    -   html
    -   markdown

        然后就可以自在编写你的博客咯.

-   第五步: 简化版: Hexo 命令:

    -   hexo g == hexo generate
    -   hexo d == hexo deploy
    -   hexo s == hexo server
    -   hexo n "new post" == hexo new *hexo new page title hexo new post title* 默认是 page
    -   [more](http://hexo.io/docs/commands.html)

当心：

标签里是 tag:_[tag1,tag2] tag 后要有一个空格 否则编译报错？

### TODO: 

1.  hexo 日常工作流程;
2.  多媒体资源管理, 图片, 音频, 视频;
3.  widget 配置;
4.  hexo 内部运行机制.

### 牢骚: 

1.  Markdown 不支持图片设置宽高, 自行使用 标签, 自求多福,
2.  Markdown 莫名其妙的缩进,
    排版啊.(1\>次一级列表只需要比上级多一个空格即可,2\>次级列表中的代码缩进要比上一级多一个单位`四个空格`),
3.  Hexo 版本已经到了 2.4.5 了, 不熟悉代码架构, 添加着附件真困难啊.

感谢以下达人的文章:

-   [zipperary](http://zipperary.com/categories/hexo/)
-   [huaoguo](http://huaoguo.com/work/2013/11/09/%E5%B0%86%E5%8D%9A%E5%AE%A2%E6%89%98%E7%AE%A1%E5%88%B0gitcafe.html)
-   [gitcafe](http://blog.gitcafe.com/116.html)
-   [github](http://github.com/)
-   [google](http://www.google.com/) !== 谷歌你又乱入...

- **感谢 pandoc 帮我把一篇旧文的 HTML 转成 Markdown 

`pandoc -f html -t markdown file.html >> new_markdown.md`

- **感谢 unix 的管道 !==

by **Peter Lau**
