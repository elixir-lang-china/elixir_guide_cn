# 17 `try`, `catch`和`rescue`

Elixir有三个错误处理机制：错误，抛出和退出。在这一章我们将一一探索它们，包括应该在什么时候使用它们。

## 17.1 错误

第一个典型的错误是试图把一个数字和原子相加：

```
iex> :foo + 1
** (ArithmeticError) bad argument in arithmetic expression
     :erlang.+(:foo, 1)
```

在运行时调用宏`raise/1 `导致一个错误：

```
iex> raise "oops"
** (RuntimeError) oops
```

另一些错误能通过给`raise/2`传递一个错误名和一个关键字列表来形成：

```
iex> raise ArgumentError, message: "invalid argument foo"
** (ArgumentError) invalid argument foo
```

你也可以用宏`defexception/2`定义你自己的错误。最常见的例子是定义一个带有message域的异常：

```
iex> defexception MyError, message: "default message"
iex> raise MyError
** (MyError) default message
iex> raise MyError, message: "custom message"
** (MyError) custom message
```

异常可以被通过`try/rescue`挽救：

```
iex> try do
...>   raise "oops"
...> rescue
...>   e in RuntimeError -> e
...> end
RuntimeError[message: "oops"]
```

上面的例子挽救了一个运行时错误并返回这个错误，并在`iex`的session中打印出来。在实践中Elixir开发者很少使用`try/rescue`结构。例如，许多语言中但一个文件无法被打开时，会强制你去挽救一个错误。相反Elixir提供了一个函数`File.read/1`，它返回一个包含文件是否被成功打开的相关信息的元组。

```
iex> File.read "hello"
{:error, :enoent}
iex. File.write "hello", "world"
:ok
iex> File.read "hello"
{:ok, "world"}
```

这里没有`try/rescue`。如果你想要处理开打文件的不同后果，你可以非常方便地使用`case`做模式识别：

```
iex> case File.read "hello" do
...>   {:ok, body} -> IO.puts "got ok"
...>   {:error, body} -> IO.puts "got error"
...> end
```

当然，最终还是取决你自己的应用来决定打开一个文件是不是一个错误。这也是为什么Elixir没有在`File.read/1`和其他函数中只用异常。它把选择最佳的处理方式的决定留给了开发者。

在某些情况下，当你期待一个文件的确存在（并如果没有这个文件，这的确是一个错误），有可以轻松地嗲用`File.read!/1`：

```
iex> File.read! "unknown"
** (File.Error) could not read file unknown: no such file or directory
    (elixir) lib/file.ex:305: File.read!/1
```

用另一种话来说，我们避免使用`try/rescue`因为我们不用错误来做控制流程。在Eliixr中，我们对错误的理解是字面意义上的：它们就是没有预期的或留给意外的或情况的。如果你的确需要流程控制结构，就需要用到throw。这也是下一章的内容。

## 17.2 抛出

在Elixir中，一个抛出的值能之后被捕获。`throw`和`catch`是用于除了是用`throw`和`catch`之外没法获取一个值的情况。

这些情况在时间中是不常见的，除非当你要和一些API定义地不好的库打交道的时候。例如，让我们想象`v`模块没有提供提供寻找值的API，所以我们必须去找到第一个是13的倍数的数字：

```
iex> try do
...>   Enum.each -50..50, fn(x) ->
...>     if rem(x, 13) == 0, do: throw(x)
...>   end
...>   "Got nothing"
...> catch
...>   x -> "Got #{x}"
...> end
"Got -39"
```

然而，在实际上`Enum.find/2`就可以轻松做到：

```
iex> Enum.find -50..50, &(rem(&1, 13) == 0)
-39
```

## 17.3 退出

所有的Eliixr代码都运行在进程中，进程之间互相通信。当一个进程死亡，它会发送一个`exit`信号。也可以手动发送一个退出信号来杀死一个进程：

```
iex> spawn_link fn -> exit(1) end
#PID<0.56.0>
** (EXIT from #PID<0.56.0>) 1
```

在上面的例子中，我们链接了死亡的进程之后发送了退出信号，它的值是1.Elixir控制台自动处理这些消息，并把它们打印出来：

`exit`也能被``捕获：

```
iex> try do
...>   exit "I am exiting"
...> catch
...>   :exit, _ -> "not really"
...> end
"not really"
```

用`try/catch`已经是非常罕见的，用它们来捕获退出更是少见。

`exit`信号是Eralng虚拟机提供的tolerant机制的重要组成部分。进程通常在监控树之下运行，它们其实是一个等待被监控的进程的退出信号的进程。当一个收到一个退出信号，它们的监控策略被触发并将死亡的被监控进程重启。

正式这个监控系统使得`try/catch`和`try/rescue`在Elixir中用的这么少。与其挽救一个错误，我们更愿意让他`先失败`，因为监控树会保证我们的应用在错误之后，会重回到一个已知的初始状态。

## 17.4 之后

有时的确有必要时候`try/after`来保证一个资源在某些特定的动作之后被清理。例如，我们打开了一个文件并用` try/after`来保证它的关闭。

```
iex> {:ok, file} = File.open "sample", [:utf8, :write]
iex> try do
...>   IO.write file, "josé"
...>   raise "oops, something went wrong"
...> after
...>   File.close(file)
...> end
** (RuntimeError) oops, something went wrong
```

## 17.5 变量作用域

牢记在`try/catch/rescue/after `内部定义的变量并不会泄漏到外部环境中。这是因为`try`块也会会失败，所以变量也许从一开始就是找不到的。换一句话来说，这些代码是非法的：

```
iex> try do
...>   from_try = true
...> after
...>   from_after = true
...> end
iex> from_try
** (RuntimeError) undefined function: from_try/0
iex> from_after
** (RuntimeError) undefined function: from_after/0
```

到这里，我们结束了我们对`try`，`catch`和`rescue`的介绍。你会发现和在其他语言中相比，它们在Elixir中用的不多。当然在某些特定的情况下当一个库或一些特定的代码“不按规矩出牌”的时候，也许它们会有用。

是时候让我们谈谈一些Elixir的构建比如comprehension和sigil了。
