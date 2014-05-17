# 3 定制Mix任务

* [3.1 通用API](#toc_1)
* [3.2 命名空间任务](#toc_2)
* [OptionParser](#toc_3)
* [3.4 共享任务](#toc_4)
  * [3.4.1 作为一个依赖](#toc_5)
  * [3.4.2 作为一个存档](#toc_6)
  * [3.4.3 MIX_PATH](#toc_7)

在Mix里，一个任务只不过是一个包含`run/1`函数，`Mix.Tasks`命名空间下的Elixir模块而已。例如，任务`compile`其实是一个名为`Mix.Tasks.Compile`的模块。

让我们创建一个简单的模块：

```
defmodule Mix.Tasks.Hello do
  use Mix.Task

  @shortdoc "This is short documentation, see"

  @moduledoc """
  A test task.
  """
  def run(_) do
    IO.puts "Hello, World!"
  end
end
```

把这段代码保存到文件`hello.ex`里，并编译和运行：

```
$ elixirc hello.ex
$ mix hello
Hello, World!
```

上面的模块定义了一个名为`hello`的任何。函数`run/1`接受一个类型是二进制字符串列表的参数，它就是通过命令行传递过来的参数。

当你调用`mix hello`， 这个任务被运行并打印`Hello, World!`。Mix使用它的第一个参数（在这里是`hello`），来查找任务模块，一旦找到就执行其中的`run`函数。

你也许会好奇为何我们有一个`@moduledoc`和`@shortdoc`。这两者都被`help`任务用来表列任务和提供文档。前者被用于当`mix help TASK`的时候，而后者在总体表列`mix help`的时候使用。

除了这两，还有一个``属性，当被设成`true`的时候，表明这个任务是隐藏的，不会被显示在`mix help`里。任何没有`@shortdoc`的任务也不会被显示。

# 3.1 通用API

当编写任务的时候，有一些常用的Mix函数可以使用。它们是：

* `Mix.Project.config` - 返回函数`project`中定义的项目的配置；注意，如果项目中没有文件`mix.exs`，这个函数返回一个空配置。这运行众多的Mix函数即时没有`mix.exs`项目也能工作。
* `Mix.Project.get!` - 访问当前项目的模块，这可以让你访问项目中的特殊函数。如果项目不存在，将导致一个错误。
* `Mix.shell` - 这个壳在Mix做IO的简单抽象。这个抽象使得比较容易地测试现存的Mix任务。在未来，这个壳还将轻松地提供彩色输出和得到用户输入。
* `Mix.Task.run(task, args) ` - 这是你如何从一个任务中调用另一个Mix任务。注意如果任务已经被调用，it works as no-op;

更多的有关MixApi的文档，请访问[这里](http://elixir-lang.org/docs/stable/Mix.html)，特别注意其中的[`Mix.Task`](http://elixir-lang.org/docs/stable/Mix.Task.html)和[`Mix.Project`](http://elixir-lang.org/docs/stable/Mix.Project.html)。

# 3.2 命名空间任务

虽然任务是非常简单的，但它们能被用于完成复杂的事情。因为它们仅仅是Elixir代码，任何你能用普通Elixir代码做的事情，也能用于Mix任务。你能和普通代码库一样发布任务，也就是说它们能在别的项目中被复用。

所以，当你有一堆的相关的任务时该怎么办？如果你按照`foo`，`bar，`baz`的方式命名它们，很快你就会很其他人的任务冲突了。为了避免这样的情况，Mix运行你对任务使用命名空间。

让我们假设你有一堆用于Riak的任务：

```
defmodule Mix.Tasks.Riak do
  defmodule Dostuff do
    ...
  end

  defmodule Dootherstuff do
    ...
  end
end
```

现在你在模块`Mix.Tasks.Riak.Dostuff`和`Mix.Tasks.Riak.Dootherstuff`下有了两个不同的任务。你可以想遮阳调用这些函数：`mix riak.dostuff`和`riak.dootherstuff`。很酷，不是吗？

当你有一堆相互关联的任务，而对它们单独命名有会非常不方便时使用这个特性。如果你只有不多的几个任务，那就随你怎么处理了。

# 3.3 OptionParser

虽然并非是一个Mix的特性，Elixir随带了一个`OptionParser`，当用于创建Mix任务和处理接受到选项是比较有用。`OptionParser`接受一个列表作为参数，返回一个包含经过处理的选项和未能被处理的参数：

```
OptionParser.parse(["--debug"])
#=> { [debug: true], [] }

OptionParser.parse(["--source", "lib"])
#=> { [source: "lib"], [] }

OptionParser.parse(["--source", "lib", "test/enum_test.exs", "--verbose"])
#=> { [source: "lib", verbose: true], ["test/enum_test.exs"] }
```

访问``文档来获得跟多信息。

# 3.4 共享任务

当你创建了自己的任务，你也许会希望能同别的开发者分享或在现有的项目就中复用。在这一部分，我们将看到在Mix中不同的分享的办法。

# 3.4.1 作为一个依赖

想象一已经创建了一个Mix项目，`my_tasks`， 提供了一些任务。把`my_tasks`作为一个依赖加入到其他项目中，`my_tasks`中的所有任务都将可以父项目中使用。就这么简单！

# 3.4.2 作为一个存档

Mix任务不仅在项目内有用，而且能被用于创建新项目，自动化复杂任务和避免重复劳动。在这些场景中，你会希望一个任务始终能在工作流中被使用，无论是否在项目中。

针对这些需求，Mix允许开发者在本地安装和卸载存档。要从当前项目产生一个存档并在本地安装，你可以运行：

```
$ mix do archive, local.install
```

存档能从一个路径或者其他任何URL安装：

```
$ mix local.install http://example.org/path/to/sample/archive.ez
```

存档安装完之后，你就可以运行其中的所有任务了，用`mix local`来表列它们，或用`mix local.uninstall archive.ez`来卸载包裹。

# 3.4.3 MIX_PATH

最后一个用于分享的机制是`MIX_PATH`。通过设置你的`MIX_PATH`， 任何在`MIX_PATH`路径内的任务都能自动被Mix可见。这里是一个例子：

```
$ export MIX_PATH="/full/path/to/my/project/ebin"
```

这对那些必须安装在`/usr`和`/opt`目录下的，但有希望仍然能和Mix绑定的复杂项目比较有用。

学习了这些选项，你就可以出师， 创建并安装你自己的任务了！加油！
