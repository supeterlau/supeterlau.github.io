+++
date = "2017-04-19T11:49:05+08:00"
title = "如何编写 Go - Part1"
draft = false

+++

### 简介

本文档描述了如何开发一个简单的 `Go` 包，并介绍了获取，构建和安装 `Go` 包和命令的标准方法 `go tool`。

`go tool` 要求你以一种特殊的方式组织代码。请详细阅读本文档。它介绍了设置和运行 `Go` 的最简单方法。

### 代码组织

#### 概览

- `Go` 程序员通常将全部的 `Go` 代码放在单独的工作空间。
- 一个工作空间包含了很多版本控制仓库 (例如由 `Git` 管理)。
- 每个仓库包含一个或多个包。
- 每个包由单独目录下一个或多个 `Go` 源代码组成。
- 包文件夹路径决定了包的 `import path` 。

注意，`Go` 的代码组织与其他编程语言不同在每个项目由自己独立的工作空间， 而且工作空间与版本控制仓库紧密联系。

#### 工作空间

工作空间是一个由三个子目录组成的文件夹

- `src` 包含 `Go` 源代码文件
- `pkg` 包含 `Go` 包
- `bin` 包含可执行命令

`go tool` 编译源码包并将生成的二进制文件安装到 `pkg` 和 `bin` 文件夹。

`src` 子目录通常包含多个版本控制仓库(如 `Git` 或 `Mercurial` )，跟踪一个或多个包的开发。

下边的例子让你看下实际的工作空间样子：

    bin/
        hello                          # command executable
        outyet                         # command executable
    pkg/
        linux_amd64/
            github.com/golang/example/
                stringutil.a           # package object
    src/
        github.com/golang/example/
            .git/                      # Git repository metadata
        hello/
            hello.go                   # command source
        outyet/
            main.go                    # command source
            main_test.go               # test source
        stringutil/
            reverse.go                 # package source
            reverse_test.go            # test source
        golang.org/x/image/
            .git/                      # Git repository metadata
        bmp/
            reader.go                  # package source
            writer.go                  # package source
        ... (many more repositories and packages omitted) ...
上边的文件树展示了一个包含两个仓库(`example` 和` image`)的工作空间。`example` 仓库包含两个命令(`hello` 和 `outyet`)以及一个包(`stringutil`)。`image` 仓库包含 `bmp` 包和其他包。

一个典型的工作空间包含多个仓库和命令。 大部分 `Go` 程序员将全部 `Go` 源代码和依赖放在一个单独的工作空间中。

命令和库是由多种源码包构建而来。 我们稍后会讨论这个区别。

#### `GOPATH` 环境变量

`GOPATH` 环境变量指定了工作空间的位置。默认是你的家目录下叫做 `go` 的文件夹( `UNIX` 下是 `$HOME/go`，`Paln9` 下是 `$home/go`， `Windows` 下 `%USERPROFILE%\go` ( 通常是 `C:\Users\YourName\go` ) )。如果你想在另外一个位置下工作， 你需要设置 `GOPATH` 为另外一个位置。(另一个常用设置方法是设置 `GOPATH` 为 `$HOME`。) 注意 `GOPATH` 不能是 `Go` 安装路径。

`go env GOPATH` 命令打印出当前有效 `GOPATH` ；如果没设置这个值，就打印出默认位置。

简便起见，将工作空间的 `bin` 子目录添加到 `PATH` 变量里:

`$ export PATH=$PATH:$(go env GOPATH)/bin` 

文档剩余的代码片段使用 `$GOPATH` 替代 `$(go env GOPATH)`。 为了让脚本可以在你没设置 `GOPATH` 的情况下运行，你可以在这些命令里替换 `$HOME/go` 或者运行

`$ export GOPATH=$(go env GOPATH)`

想了解更多关于 `GOPATH` 环境变量的信息，可以查看 `go help gopath`

关于使用自定义工作空间的知识，可查阅 [set the GOPATH environment variable](https://golang.org/wiki/SettingGOPATH).

#### Import paths

导入路径 (`import paths`) 是唯一地标识一个包的字符串。 一个包的导入路径对应工作空间的位置或者一个远端的代码仓库。

标准库的包被赋予了类似` "fmt"` 和` "net/http"` 的缩写形式。对你自己的包来说，你必须选择一个不大可能和标准库或其他外部库冲突的基路径。

如果你将代码保存在某个源代码库中， 你应该使用源码地址的根地址作为你的基路径。例如， 如果你有个位于 `github.com/user` 的帐号， 这应该作为包的基路径。

注意你构建包并不需要将它推送到远端。 通过假定你未来会推送到远端，而学习着这样组织代码，仅仅是为了帮助你养成一个好习惯。实际上你可以选取任何路径名，只需要保证它足够长从而与 `Go` 标准库生态群中的其他包不发生命名冲突。

我们会使用 `github.com/user` 作为基路径。 在你的工作空间创建一个目录保存代码:

`$ mkdir -p $GOPATH/src/github.com/user`

#### 你的第一个程序

为了编译并运行一个简单程序， 首先选择一个包路径 (我们会使用 `github.com/user/hello` ) 并创建一个对应的文件夹:

`$ mkdir -p $GOPATH/src/github.com/user/hello` 

下一步，创建一个叫做 `hello.go` 的文件， 包含以下 `Go` 代码。

    package main
    import "fmt"
    func main() {
        fmt.Printf("Hello, world.\n")
    }
现在你可以通过 `go tool` 编译并安装这个程序:

`go install github.com/user/hello`

你可以在系统的任何地方执行这个命令(当然需要在当前的 `shell session` 下)。 `go tool` 通过 `GOPATH` 工作空间下 `github.com/user/hello` 找到源代码。

如果从包目录安装也可以忽略包路径

`$ cd $GOPATH/src/github.com/user/hello`

`go install`

这一命令编译 `hello` 命令， 产生一个可执行文件。随后将其安装到工作空间的 bin 文件夹下。 在我们的例子中， 这个路径是 `$GOPATH/bin/hello`。

`go tool` 只会在出现错误时打印输出， 如果上述命令没有任何输出就说明执行成功了。

我们现在可以通过全路径调用生成的可执行文件

    $ $GOPATH/bin/hello
    Hello, world.

另外，如果你已经将 $GOPATH/bin 添加到 PATH，只需要直接调用二进制文件名字：

    $hello 
    Hello, world. 

如果你使用版本控制系统， 现在你就可以初始化一个仓库了， 添加文件，提交你的第一个修改。 再说一下， 这一步是可选的：不用源码管理系统，你也可以写 `Go` 代码。

`$ cd $GOPATH/src/github.com/user/hello`

    $ git init
    Initialized empty Git repository in /home/user/work/src/github.com/user/hello/.git/
    $ git add hello.go
    $ git commit -m 'initial commit'
    
将代码推送到远端仓库就留给读者作为练习了。