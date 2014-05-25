# 简介

*
  *
  *
  *
  *
*
*
*
  *
  *
  *
  *
  *
*
*

Elixir自带了几个应用使得编写和部署Elixir项目更加容易，其中Mix是关键。

Mix是一个提供了用愈创建，编译，测试（很快就会有发布功能）的编译工具。Mix的灵感来自于Clojure的编译工具Leiningen，并且作者本人就是它的其中一个开发者。

在这一章，我们将学习如何用mix来创建项目，安装依赖。在之后的部分，我们将雪鞋如何创建OTP应用，和定制mix的任务。

# 1.1 始动

要开始你的第一个项目，你只需要输入`mix new` 并把项目的路径作为参数。现在，我们将在当前目录创建一个被称为``的项目：

```
$ mix new my_project --bare
```

Mix将常见一个名为``的目录，包含一下这些内容：

```
.gitignore
README.md
mix.exs
lib/my_project.ex
test/test_helper.exs
test/my_project_test.exs
```

让我们来简单地看一下其中的一些。

> 注意：Mix是一个Elixir的可执行文件。这意味着为了能够运行`mix`，elixir的可执行文件许需要在你的`PATH`里面。如果不是，你可以直接把脚本作为参数传递给elixir来执行：

>```
>$ bin/elixir bin/mix new ./my_project
>```
>注意你也能通过``选项来让Elixir运行任何PATH里的脚本：
>```
>$ bin/elixir -S mix new ./my_project
>```
>当用了``， elixir会遍历PATH，寻找并运行那个脚本

# 1.1.1 mix.exs

这是包含了你项目配置的文件。它看起来是这样的：

```
defmodule MyProject.Mixfile do
  use Mix.Project

  def project do
    [app: :my_project,
     version: "0.0.1",
     deps: deps]
  end

  # Configuration for the OTP application
  def application do
    []
  end

  # Returns the list of dependencies in the format:
  # {:foobar, git: "https://github.com/elixir-lang/foobar.git", tag: "0.1"}
  #
  # To specify particular versions, regardless of the tag, do:
  # {:barbat, "~> 0.1", github: "elixir-lang/barbat"}
  defp deps do
    []
  end
end
```

我们的``定义了两个函数：``， 用来返回项目的配置比如项目名称和版本，和``，用来产生一个被Erlang运行时管理的Erlang应用。在这一章，我们将谈一谈函数``。我们将在下一章详谈``函数。

# 1.1.2

这个文件包含了一个简单的模块，它定义了我们代码的基本机构：

```
defmodule MyProject do
end
```

# 1.1.3

这个文件包含了项目的一个测例：

```
defmodule MyProjectTest do
  use ExUnit.Case

  test "the truth" do
    assert true
  end
end
```

有几点请注意：

1） 注意这个文件是一个Elixir的脚本文件（``）。作为一个约定，我们不需要在运行之前编译测试。

2） 我们定义了一个测试模块``，用``来注入默认的行为，并定义了一个简单测试。你可以在ExUnit那一章学习到有关测试框架的更多内容。

# 1.1.4

我们将查看的最后一个文件是``，它的任务是启动测试框架：

```
ExUnit.start
```
# 1.2 探索

现在我们已经创建了新项目，接下去做什么？要了解还有其他什么命令可以使用的话，运行`help`任务：

```
$ mix help
```

它将会打印出所有可得任务，你能得到用`mix help TASK`得到更多的信息。

运行其中一些命令试试，比如`mix compile`和`mix test`，在你的项目里运行看看会发生什么。

# 1.3 编译

Mix可以我们编译项目。默认的设置是用``放源代码，``放编译后的beam文件。你无需提供任何的编译相关的设置，但如果你决定这么做，有一些选项可以用。例如，如果你打算把你的编译后的beam文件放在`ebin`之外的文件夹里，只需要在``里设置``：

```
def project do
  [compile_path: "ebin"]
end
```

总的来说，Mix会尽力表现的聪明一些，只在必须的时候编译。

注意在你第一次编译之后，Mix会在你的`ebin`文件夹里产生了一个``文件。这个文件里定义的Erlang应用是用到了你的项目中的``函数里的内容。

这个``文件存储在和应用有关的信息，它的依赖，它所依赖的模块㩐等。每次你用mix运行命令的时候，这个应用会自动被启动，我们将在下一章学习如何配置它。

1.4 依赖

Mix也能用来管理依赖。依赖应被列在项目配置中，例如：

```
def project do
  [app: :my_project,
   version: "0.0.1",
   deps: deps]
end

defp deps do
  [{:some_project, ">= 0.3.0"},
   {:another_project, git: "https://example.com/another/repo.git", tag: "v1.0.2"}]
end
```

**注意：** 虽然并非必须，常见的做法是把依赖分散到它们自己的函数里。

某个依赖有一个原子来表示，跟着是一个需求和一些选项。在默认情况下，Mix使用``来获取依赖，但它也能从git库或直接从文件系统来获取。

当我们使用Hex， 你必须在需求里指定所接受的依赖的版本。它支持一些基本的操作符，例如``，``，``，``：

```
# Only version 2.0.0
"== 2.0.0"

# Anything later than 2.0.0
"> 2.0.0"
```

需求也支持用`and`和`or`表达复杂的情况：

```
# 2.0.0 and later until 2.1.0
">= 2.0.0 and < 2.1.0"
```

类似上面的例子非常地常见，所以它也能用简单的方式表达：

```
"~> 2.0.0"
```

注意为git库设置版本需求不会影响到取出的分支和标签，所以类似下面这样的定义是合法的：

```
{ :some_project, "~> 0.5.0", github: "some_project/other", tag: "0.3.0" }
```

但它会导致一个依赖永远无法满足，因为被取出的标签总不能和需求的版本匹配。

# 1.4.1 源代码管理（scm）

Mix的设计就考虑到了支持多种的SCM工具，Hex包是默认，但``和``是可选项。常见的一些选项是：

* `` - 依赖是一个git版本库，Mix可以来获取和升级。
* `` - 依赖是文件系统中的一个路径
* `` - 如何编译依赖
* `` - 依赖所定义的应用的路径
* `` - 依赖所用的环境（详情在后），默认是``；

每个SCM也许支持特有的选项，yi``为例，支持下面这些：

* `` - 一个可选的索引（提交）用来取出版本库；
* `` - 一个可选的标签用来取出版本库；
* `` - 一个可选的分支用来取出版本库；
* `` - 当为真，在依赖中设置submodule；

# 1.4.2 编译依赖

为了编译依赖，Mix会选择最适合的方式。依赖所包含的文件不同，编译的方式也不一样：

1， `` - 直接用Mix的`compile`任务编译依赖；
2， ``或`` - 用``编译，`DEPS`是Mix会默认安装依赖的文件夹；
3， `` - 直接`make`；

如果编译的代码里没有包含以上的任何，你可以在``选项里直接指定一个命令：

```
{:some_dep, git: "...", compile: "./configure && make"}
```

如果``被设为`false`， 这一步会被忽略。

# 1.4.3 可重复性

任何一个依赖管理工具的重要特性是可重复性。因此当你初次获取依赖，Mix将创建一个文件``，用来包含每个依赖所取出的索引。

当另一个开发者得到这个项目的拷贝，Mix将取出相同的那个索引，保证其他的开发者能“重复”同样的设置。

运行``能自动升级锁，用``任务来移除锁。

# 1.4.4 依赖任务

Elixir自带了许多用来管理项目依赖的任务：

* `mix deps` - 列出所有的依赖和它的情况；
* `` - 获取所有可得的依赖
* `` - 编译依赖
* `` - 升级依赖；
* `` - 移除依赖文件；
* `` - 解锁依赖
用`mix help`来获取更多信息。

# 1.4.5 依赖的依赖

如果你的依赖是一个Mix或rebar的项目，Mix知道如何应付：它将自动获取和处理你的依赖的所有的依赖。然而，如果你的项目中有两个依赖共享了同一个依赖，但它们的SCM信息又无法互相匹配的话，Mix将标明这个依赖是分裂的，并发出警告。要解决而这个问题，你可以在你项目中声明选项``， Mix将根据这个信息来获取依赖。

# 1.5 伞形项目

你是否想过，如果能将几个Mix项目打包在一起，只需一个命令就可以运行各自的Mix任务， 该有多方便？。这种将项目打包在一起使用的情况被称为伞形项目。一个伞形项目可以用下面的命令来创建：

```
$ mix new project --umbrella
```

这将会创建一个包含以下内容的``文件：

```
defmodule Project.Mixfile do
  use Mix.Project

  def project do
    [apps_path: "apps"]
  end
end
```

这个``选项指定了子项目将会落户的文件夹。在伞形项目中运行的Mix任务，会对``文件夹中的每一个子项目起作用。例如`mix compile`和`mix test`将编译或测试文件夹下的每一个项目。值得注意的是，伞形项目既不是一个普通的Mix项目，也不是一个OTP应用，也不能修改其中的源码。

如果在子项目之间有互相依赖的存在，需要指定它们的顺序，这样Mix才能正确编译。如果项目A依赖于项目B，这个依赖关系必须在项目A的``文件里指定。修改文件``来指定这个依赖：

```
defmodule A.Mixfile do
  use Mix.Project

  def project do
    [app: :a,
     deps_path: "../../deps",
     lockfile: "../../mix.lock",
     deps: deps]
  end

  defp deps do
    [{ :b, in_umbrella: true }]
  end
end
```

注意上面的子项目中``和``选项。如果在你的伞形项目的所有子项中都有它们，它们将共享依赖。apps文件夹中的`mix new`将自动用这些预设的选型创建一个项目。

# 1.6 环境

Mix有环境的概念，能让一个开发者去基于一个外部的设定来定制编译和其他的选项。默认下，Mix能理解项目三类环境：

* `dev` - mix任务的默认环境；
* `test` - 用于`mix test`；
* `prod` - 在这个环境下，依赖将被载入和编译；

在默认情况下，这些环境的行为没有不同，我们之前看到的所有配置都将会影响到这三个环境。针对某个环境的定制，可以通过访问``来实现：

```
def project do
  [deps_path: deps_path(Mix.env)]
end

defp deps_path(:prod), do: "prod_deps"
defp deps_path(_), do: "deps"
```

Mix默认为`dev`环境（除了测试）。可以通过修改环境变量``来改变环境。

```
$ MIX_ENV=prod mix compile
```

在下一章，我们将学习如何用Mix编写OTP应用和如何创建你自己的任务。
