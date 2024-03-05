+++
title = "如何编写 Go - Part2 完结篇"
draft = false
date = "2017-04-23T00:09:39+08:00"

+++

>## 第一个库

我们将会写一个库并在 `hello` 程序中使用它。

第一步是选择一个包路径 (我们会使用 `github.com/user/stringutil` )并创建库文件夹:

    $ mkdir $GOPATH/src/github.com/user/stringutil

下一步，在目录下创建叫做 `reverse.go` 的文件，并输入以下内容

    // Package returns its argument string reversed rune-wise left to right
    package stringutil

    // Reverse 按字符将参数 string 翻转
    func Reverse(s string) string {
        r := []rune(s)
        for i, j := 0, len(r)-1; i < len(r)/2; i,j=i+1,j-1 {
            r[i], r[j] = r[j], r[i]
        }
        return string(r)
    }

现在，用 `go build` 来编译测试这个包:

    $ go build github.com/user/stringutil 

如果你在包文件夹路径，可以只执行:

    $ go build

这不会产生一个输出文件。你必须使用 `go install`，这可以将包对象放在工作空间的 `pkg` 文件夹下。

在确认 `stringutil` 包编译完成后， 修改你原始的 `hello.go` (位于 `$GOPATH/src/github.com/user/hello` 下) 来使用这个包:

    package main

    import (
      "fmt"
      "github.com/user/stringutil"
    )

    func main() {
        fmt.Printf(stringutil.Reverse("!oG ,olleH"))
    }

每当 `go` 工具安装一个包或二进制文件时，它同时安装了这个包包含的依赖。 所以当你安装 hello program 时，

    $ go install github.com/user/hello

stringutil 包会同时被自动安装。

运行新版本的程序，你会看到一个新的翻转的信息:

    $hello
    Hello, Go!

完成以上步骤后，你的工作空间看起来像这样:

    bin/
      hello
    pkg/
      linux_amd64/      # 取决于你使用的操作系统和架构
        github.com/user/
          stringutil.a  # 包项目
    src/
      github.com/user/
        hello/
          hello.go
        stringutil/
          reverse.go

注意 `go install` 将 `stringutil.a` 放在 `pkg/linux_amd64` 下和源码路径一致的子文件夹下。这样方便 `go` 工具找到包文件从而避免重复编译这个包。 `linux_amd64` 这部分是为了辅助交叉编译，并会反映你的操作系统及系统架构。

`Go` 可执行文件是静态链接的；运行 `Go` 程序不需要包文件。

>## 包名

`Go` 源代码第一行必须是 `package name`。 `name` 就是用来导入包的默认名字。(同一个包里的文件需要有相同的包名)。

`Go` 中包名是导入路径最后一个元素: 作为 `"crypto/rot13"` 导入的包，包名是 `"rot13"`。

可执行命令必须使用 `main` 作为 `package name`。

`Go` 不要求链接到一个二进制文件的包名不同，只需要导入路径唯一即可。

查看 `Effective Go` 来学习更多关于 `Go` 命名惯例的内容。

># 测试

`Go` 包含了一个由 `go test` 命令和 `testing` 包组成的轻量级测试框架。
你通过创建一个以 `_test.go` 结尾的文件来写测试，文件中包含命名为 `TestXXX` 参数签名为 ( `t *testing.T` ) 的测试函数。测试框架执行每个这样的函数；如果函数调用了一个类似 `t.Error` 或 `t.Fail` 的失败函数，测试被认为未通过。
创建文件 `$GOPATH/src/github.com/user/stringutil/reverse_test.go` 来添加测试到 `stringutil` 包，包含如下代码:

然后用 `go test` 来运行测试。

    $ go test github.com/user/stringutil
    ok github.com/user/stringutil 0.165s

同样，如果你就在包路径下，可以直接执行:

    $ go test
    ok github.com/user/stringutil 0.165s

执行 `go help test` 可以看到测试包的文档。

>## 远程包

导入路径可以用来描述如何通过 Git 或 Mercurial 这样的版本控制系统来获取包源码。go 工具通过这个特性来从远端自动获取包。例如，本文档中的例子有个托管在 Github 上的 Git 仓库 [github.com/golang/example](https://github.com/golang/example)。如果你在导入时加上这个仓库，go get 会自动获取，构建并安装这个包。

    $ go get github.com/golang/example/hello

    $ $GOPATH/bin/hello
    Hello，Go examples！

如果特定的一个包没有在工作空间中，`go get` 会将它放在 `GOPATH` 指定的第一个工作空间。(如果这个包已经存在，那么 `go get` 会跳过获取远程代码，仅进行 `go install` 步骤)。

执行过上述的 `go get` 命令后，工作目录会变成一下结构:

    bin/
      hello
    pkg/
      linux_amd64/      # 取决于你使用的操作系统和架构
        github.com/golang/example/
          stringutil.a  # 包项目    
        github.com/user/
          stringutil.a  # 包项目
    src/
      github.com/golang/example/
        hello/
          hello.go
        stringutil/
          reverse.go
          reverse_test.go
      github.com/user/
        hello/
          hello.go
        stringutil/
          reverse.go
          reverse_test.go

托管在 `GitHub` 上的 `hello` 命令依赖于同一个仓库里的 `stringutil` 包。`hello.go` 文件里导入包时使用了同样的导入规则，因此 `go get` 命令可以定位并安装这个依赖。

`import "github.com/golang/example/stringutil"` 这种方法是使你的包可以被别人使用的最简单途径。 `Go Wiki` 和 `godoc.org` 提供了其他 `Go` 项目列表。

查看 `go help importpath` 来了解更多通过 `go tool` 使用远程仓库的信息。
