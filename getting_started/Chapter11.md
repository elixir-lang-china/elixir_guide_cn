# 11 进程

Elixir的所有代码都运行在进程。每个进程之间互相隔离，同时运行，通过传递消息进行通讯。进程不仅是Elixir并发的基础，而且它们也是编写分布式和高可靠性程序的基础。

和其他语言中的线程不同，就内存和CPU消耗来说，进程是极为轻量级的。同时运行上千个进程是家常便饭。

在这一章，我们将学习如何创建一个新的进程，包括在进程之间首发信息。

## 11.1 spawn

创建一个进程最基本的方式是使用内置的`spawn/1`函数：

```
iex> spawn fn -> 1 + 2 end
#PID<0.43.0>
```

`spawn/1`接受一个函数，然后在另一个进程中执行。

注意`spawn/1`返回一个PID（进程识别符）。在这个时候看起来你的进程很可能已经死掉了。被创建的进程在执行完传进来的函数之后会自动退出：

```
iex> pid = spawn fn -> 1 + 2 end
#PID<0.44.0>
iex> Process.alive?(pid)
false
```

> 注意：自己动手的话，你应该会得到一个和我们上面例子不同的PID

我们能通过调用`self/0`得到当前进程的PID：

```
iex> self()
#PID<0.41.0>
iex> Process.alive?(self())
true
```

只有当我们能够在进程之间发送信息时，它才会变得更加有趣。

## 11.2 发和送

我们能用`send/2`向一个进程发送信息，能用`receive/1`接受信息。

```
iex> send self(), {:hello, "world"}
{:hello, "world"}
iex> receive do
...>   {:hello, msg}  -> msg
...>   {:world, msg} -> "won't match"
...> end
"world"
```

当一个消息被送到一个进程的时候，它首先被储存在进程的邮箱里。`receive/2`会在邮箱中寻找第一个匹配模式的消息，`receive/1`支持许多子句，比如`case/2`，包括子句中的守护。

如果邮箱中没有消息匹配指定的模式，当前的进程会一直等待直到来了一个匹配的消息。当然你可以指定等待的最长时间：

```
iex> receive do
...>   {:hello, msg}  -> msg
...> after
...>   1_000 -> "nothing after 1s"
...> end
"nothing after 1s"
```

如果早就确信消息就在邮箱里，那可以把timeout设为0。

让我们汇总一下看看如何在进程间发送消息：

```
iex> parent = self()
#PID<0.41.0>
iex> spawn fn -> send(parent, {:hello, self()}) end
#PID<0.48.0>
iex> receive do
...>   {:hello, pid} -> "Got hello from #{inspect pid}"
...> end
"Got hello from #PID<0.48.0>"
```

如果你在控制台里，那函数`flush/0`可能就对你比较有用。它能打印并清空整个邮箱。

```
iex> send self(), :hello
:hello
iex> flush()
:hello
:ok
```

在我们完成这章之前，让我们谈一谈进程之间的链接。

# 11.3 链接

实际上，在Elixir中最长用到的创建进程的方式是调用函数`spawn_link/1`。在展示`spawn_link/1`的例子之前，让我们试着看看当进程出错的时候会发生什么：

```
iex> spawn fn -> raise "oops" end
#PID<0.58.0>
```

正如你所见。。。什么也没有发生。那是因为进程都是独立的。如果我们希望一个进程的问题能够传导至另一个进程，我们就应该把它们链接起来。这个时候就该`spawn_link/1`出场了：

```
iex> spawn_link fn -> raise "oops" end
#PID<0.60.0>
** (EXIT from #PID<0.60.0>) {RuntimeError[message: "oops"], [{:erlang, :apply, 2, []}]}
```

当控制台中除了错，控制台自动处理它们，并且打印出整齐的错误信息。为了能真正理解刚才在我们的代码中发生了什么，让我们使用在一个脚本中使用`spawn_link/1`，并运行看看：

```
# spawn.exs
spawn_link fn -> raise "oops" end

receive do
  :hello -> "let's wait until the process fails"
end
```

这一次，当子进程退出，也使得母进程退出了，因为它们是链接在一起的。链接可以通过调用`Process.link/2`手动完成。我们见你你看看[Process模块](http://elixir-lang.org/docs/stable/Process.html)中还有那些其他的函数。

在编写高可靠性的系统时， 进程和链接扮演了重要的角色。在Elixir的应用中，我们常常把我们的进程同监工链接在一起，它的任务是当发现有进程退出的时候，在原地重启一个新的进程。之所以能这么干，就是得益于进程之间互相独立，某一个进程的问题不会导致其他进程出问题。

在其他的一些语言中，它也许要求你捕捉和处理意外，在Elixir的中我们可以随它去，因为我们知道监工会去重启进程。“早出问题”是编写Elixir软件时的常用哲学思想。

下面我们将探索IO的世界。
