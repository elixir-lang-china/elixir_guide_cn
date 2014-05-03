# 1 Elixir的交互模式，iex

欢迎！

在这个系列的教程中，我们将学习elixr的一些基本知识。在这章之中将会涉及到如何安装和如何使用elixir的交互模式，iex。

[如果你在教程或者网站上发现了错误，请提交报告或者发一个pull request到我们的issue tracker上](https://github.com/kuno/elixir_guide_cn/issues)。如果你怀疑可能遇到了一个语言本身的bug，[请汇报至elixir语言的issue tracker](https://github.com/elixir-lang/elixir/issues)。

让我们开始吧！

## 1.1 安装Erlang

Elixir依赖于17.0版或者更新的Erlang，你可以很容易地通过[预编译的包](https://www.erlang-solutions.com/downloads/download-erlang-otp)来安装。如果你想直接从源代码安装，你可以直接从[Erlang的官方网站](http://www.erlang.org/download.html)上下载源代码，在Riak的官方文档中有一篇非常详尽的[教程](http://docs.basho.com/riak/1.3.0/tutorials/installation/Installing-Erlang/)会指导你如何安装。

如果你打算在Windows上开发，最佳的方式是安装预编译的安装包。如果你是在一个类Unix的系统上，一般来说你系统上的包管理器很可能已经提供了相应的安装包。

> 注意，Elixir需要的是17.0以上的版本的Erlang，由于17.0刚发布不久，你系统上的包管理器很可能还没有升级。

安装完Erlang之后，你应该检查一下安装的结果，在命令行中键入`erl`，如果一切正常你应该能看到这样的一条信息：

```
Erlang/OTP 17 (erts-6) [64-bit] [smp:2:2] [async-threads:0] [hipe] [kernel-poll:false]
```

根据你如何安装，在某些情况下Erlang的二进制命令可能不在你的[`PATH`](http://en.wikipedia.org/wiki/Environment_variable)上。那样的话，你就需要手动地把它们加到`PATH`里面去。

在Erlang一切就绪之后，我们就可以安装Elixir了。你可以通过三种方式安装，发行版，预编译包和从源代码安装。

## 1.2 发行版

到目前为止，Elixir v0.13有希望在某些os的发行版上直接安装了：

  * Mac OSX上的Homebrew
    * 升级Homebrew到最新`brew update`
    * 安装Elixir：`brew install elixir`

  * Fedora 17+和Fedora Rawhide
    * `sudo yum -y install elixir`

  * Arch Linux (Aur)
    * `yaourt -S elixir`

  * OpenSUSE (和SLES 11 SP3+)
    * 添加Erlang开发源 `zypper ar -f obs://devel:languages:erlang/ erlang`
    * 安装Elixir `zypper in elixir`

  * Gentoo
    * `emerge --ask dev-lang/elixir`

  * Chocolatey for Windows
    * `cinst elixir`

如果你用的OS并不是以上的任何之一，别担心，我们也提供了已经编译好的安装包！

## 1.3 预编译安装包

Elixir的每一个release都提供了[预先编译好的安装包](https://github.com/elixir-lang/elixir/releases/)。只要下载之后解压，就可以之间受用`bin`目录下的`elixr`和`iex`命令。同时我们也建议你把`bin`目录加入到系统的`PATH`
里去，这样就能为以后的开发省去不少的麻烦。

##　1.4 从源代码编译（Unix和MinGW）

只需简单的几步就可以下载和编译Elixir。你可以这里下载Elixir的[最新的代码](https://github.com/elixir-lang/elixir/releases/)，在解压缩之后的目录里运行`make`。如果没有出错，你就可以运行`bin`目录下的`elixir`和`iex`命令。同样，我们也建议把`bin`所在的目录加入`PATH`。

```
export PATH="$PATH:/path/to/elixir/bin"
```

万一你想尝鲜的话，你还可以从git masterbianyi：

```
$ git clone https://github.com/elixir-lang/elixir.git
$ cd elixir
$ make clean test
```

如果上面的测试都通过了，就表明一切就绪。否则，请提交一个错误报告在[Github的issue tracker](https://github.com/elixir-lang/elixir)上。

## 1.5 互动模式

Elixir提供了三个新的可执行命令：`iex`， `elixir`和`elixirc`。如果你是从源代码编译或者直接用的打包好的，你会发现这三个命令在`bin`目录下。

现在，让我们开始常识着运行互动模式`iex`， `iex`是“互动Elixir”（Interactive Elixir）的缩写。在互动模式中，我们可以输入任何的Elixir表达式，直接可以看到运行的结果。让我们运行几个例子来热热身：

```
Interactive Elixir - press Ctrl+C to exit (type h() ENTER for help)

iex> 40 + 2
42
iex> "hello" <> " world"
"hello world"
```

看起来我们已经一切就绪了！在下面的章节之中，我们将会频繁地使用到互动模式，它会帮助我们熟悉语言的结构和基本数据类型。就从下一章开始。
