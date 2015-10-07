# 12 IO

这一章将简单介绍一下Elixir的输入输出机制，和相关的模块，比如 [IO](http://elixir-lang.org/docs/stable/IO.html)，[File](http://elixir-lang.org/docs/stable/File.html) 和 [Path](http://elixir-lang.org/docs/stable/Path.html)。

在一开始的时候，我们在写这个系列的教程的时候，有一个很早的比较粗躁的版本。但我们发现IO系统其实是提供了一个很好的机会来管窥Elixir和Erlang虚拟机中的一些哲学和有意思的地方。

## 12.1 IO模块

Elixir中的IO模块是它的主要机制用来读写标准输入（`:stdio`），标准错误（`:stderr`），文件和其他IO设备。它的用法也是非常直观的：

```
iex> IO.puts "hello world"
"hello world"
:ok
iex> IO.gets "yes or no? "
yes or no? yes
"yes\n"
```

在默认的情况下，IO模块中的函数使用标准输入输出。我们可以通过传递参数`:stderr`来让它写进标准错误设备：

```
iex> IO.puts :stderr, "hello world"
"hello world"
:ok
```
# 12.2 文件模块

[File](http://elixir-lang.org/docs/stable/File.html)模块包含了一些可以把文件打开当成IO设备的函数。默认情况下，文件都是用二进制模式打开，这需要开发者指明使用`IO`模块中的函数`IO.binread/2`和`IO.binwrite/2`：

```
iex> {:ok, file} = File.open "hello", [:write]
{:ok, #PID<0.47.0>}
iex> IO.binwrite file, "world"
:ok
iex> File.close file
:ok
iex> File.read "hello"
{:ok, "world"}
```

文件也能用`:utf8`编码模式打开，这样就可以使用`IO`模块中的其他函数了。

除了那些能打开，读写文件的函数之外，`File`模块还有很多在文件系统领域工作的函数。这些函数的名字对应于UNIX命令。例如，`File.rm/1`用于删除文件，`File.mkdir/1`用来创建文件夹， `File.mkdir/1`用来创建包含父目录的文件夹，甚至有`File.cp_r/2`和`File.rm_rf/2`用来复制和移除文件和文件夹。

你也会注意到，`File`模块中的函数可以分为两种模式，一种带有名字里带有`!`（bang），另一些没有。例如，在上面读取文件“hello”的时候，我们已经用到了一些，现在让我们在试试其他的例子：

```
iex> File.read "hello"
{:ok, "world"}
iex> File.read! "hello"
"world"
iex> File.read "unknown"
{:error, :enoent}
iex> File.read! "unknown"
** (File.Error) could not read file unknown: no such file or directory
```

注意：如果文件不存在，那个带`！`的版本会抛出一个错误。也就是说，当你想自己通过模式匹配来处理不同的情况是，就选择不带`！`
的，如果你确信文件就在那儿，这个时候用带`！`的版本才有意义。简单来说，不要写成这样：

```
{:ok, body} = File.read(file)
```

要么这样：

```
case File.read(file) do
  {:ok, body} -> # handle ok
  {:error, r} -> # handle error
end
```
或

```
File.read!(file)
```

## 12.3 Path模块

文件模块中的大部分函数，需要的参数都是路径。最常见的情况下，这些路径都是二进制的，它们可以被通过[`Path`模块](http://elixir-lang.org/docs/stable/Path.html)来操作：

```
iex> Path.join("foo", "bar")
"foo/bar"
iex> Path.expand("~/hello")
"/Users/jose/hello"
```

到这里我们已经完成了IO相关和同文件系统互动的主要模块。下面我们将讨论和IO相关的趣闻和高级主题。这部分并不是编写Elixir程序必须的知识，要跳过请随意，但它们揭示了在Erlang虚拟机中IO系统是如何实现的一些趣闻。

## 12.4 进程和组领头人

你也许注意到了`File.open/2 `返回了一个包含PID的元组：

```
iex> {:ok, file} = File.open "hello", [:write]
{:ok, #PID<0.47.0>}
```

那是因为IO模块实际上是通过进程工作的。当你说`IO.write(pid, binary)`，IO模块会向一个进程发送消息，告诉它该干什么。让我们看一看如果我们使用自己的模块会发生什么：

```
iex> pid = spawn fn ->
...>  receive do: (msg -> IO.inspect msg)
...> end
#PID<0.57.0>
iex> IO.write(pid, "hello")
{:io_request, #PID<0.41.0>, #PID<0.57.0>, {:put_chars, :unicode, "hello"}}
** (ErlangError) erlang error: :terminated
```

在`IO.write/2`之后，我们能看到打印出来IO模块发送的请求，它失败了是因为，IO模块期待得到一些回复，而我们并没有这么做。

[StringIO](http://elixir-lang.org/docs/stable/elixir/StringIO.html)模块提供了在字符串之上的IO设备消息实现：

```
iex> {:ok, pid} = StringIO.start("hello")
{:ok, #PID<0.43.0>}
iex> IO.read(pid, 2)
"he"
```

通过用进程将IO设备模型化，Erlang虚拟机允许同一网络中的不同的节点之间交互文件进程，读写文件。在所有的IO设备中，有一个对所有每个进程都特殊的，称为组领头人：

当你写入`:stdio`，你实际上向组领头人发送了一个消息，通过它来写入`:stdio`：


```
iex> IO.puts :stdio, "hello"
hello
:ok
iex> IO.puts Process.group_leader, "hello"
hello
:ok
```

每个进程的组领头都是可以配置的，能在不同的场景中使用。例如，当在远程节点执行代码时，它保证在远程节点打印的消息会返还到“执行者”并打印。

## 12.5 `iodata` 和`char_data`

在上面的所有例子中，当我们写文件的时候，我们用到了二进制/字符串。在“二进制，字符串和字符列表”那一章，我们提到过字符串实际上就是字节，而字符列表就是codepoint的列表。

`IO`和`File`模块中的函数也能接受列表作为参数。不仅如此，它们还允许接受列表中混合了列表，整数和二进制：


```
iex> IO.puts 'hello world'
hello world
:ok
iex> IO.puts ['hello', ?\s, "world"]
hello world
:ok
```

然而，这里需要注意一些事情。一个字符列表当它被写入磁盘的时候需要被编码成字节，而这依赖于IO设备的编码。如果文件打开时没有编码，文件就处在原始模式，只有`IO`模式中一`bin`开头的模式才能对付。这些函数需要一个`iodata`作为第一个参数。例如，它期待给予的参数是一个字节和二进制的列表。如果你提供了一个codepoint的列表，而且这些codepoint都大于255，这个操作将会失败，因为我们不知道如何编码。

在另一方面，`:stdio`和用`:utf8`编码打开的文件可以用IO模块中的其他函数处理，它们需要一个`char_data`的参数，例如，它们期待的参数是一个字符列表或者字符串。

虽然有这样的不同，只有当你有意传递一个列表给这些函数的时候，才需要担心。二进制的底层早就是字节了，所以它们一直就是原始模式。

当处理io数据和chat数据的时候，Elixir提供了`iodata_to_binary/1`能把任何的`iodata`转换成二进制。函数`String.from_char_data!/1`和`List.from_char_data!/1 `能被用于把chat data转成字符串或字符列表。除非你需要和一些设备打交道，不然你不会经常用到这些函数。

到这里我们就结束了IO设备和相关的函数的旅程。我们已经学到了四个有关的Elixir模块，[`IO`](http://elixir-lang.org/docs/stable/IO.html)，[`File`](http://elixir-lang.org/docs/stable/File.html)，[`Path`](http://elixir-lang.org/docs/stable/Path.html)和[`StringIO`](http://elixir-lang.org/docs/stable/StringIO.html)，包括在底层虚拟机是如何用进程来实现IO机制，和如何用（字符和io）列表来做IO操作。
