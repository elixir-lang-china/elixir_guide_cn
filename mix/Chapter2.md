2 用Mix编写OPT应用

*
  *
  *
*
*
*
*

在Elixir里，我们如何保存状态？

我们的软件需要在运行时系统里保存状态，配置，数据。在之前的[章节](http://elixir-lang.org/getting_started/2.html#receive)里我们已学会了如何用进程/Actor保持状态，在一个循环中如何接受以及回复消息，但这种方式似乎不够可靠。如果我们的进程被一个错误退出了怎么办？难道我们真的需要仅仅为一个配置而去创建一个新的进程？

在这一章，我们将用OTP的方式来回答这些问题。在实践中，我们不必使用Mix来编写这样的应用，然而借此机会正好让我们了解一些Mix提供的一些方便之处。

# 2.1 Stacker服务器

我们的应用将会是一个运行我们推进/推出的简单堆栈。我们管它叫stacker：

```
$ mix new stacker --bare
```

我们的应用将包含一个堆栈，允许同一时间被许多的进程访问。为此，我们将创建一个服务器来负责管理这个堆栈。客户端随时可以向服务器发送消息来从服务器推进或取出某物。

因为在Erlang和Elixir里，创建这样的一个服务器是常见的一个范式，在OTP里有一个被称为**GenServer**的行为封装了这些常见的服务器功能。让我们创建一个文件`lib/stacker/server.ex`， 这就是我们的第一个服务器：

```
defmodule Stacker.Server do
  use GenServer.Behaviour

  def init(stack) do
    { :ok, stack }
  end

  def handle_call(:pop, _from, [h|stack]) do
    { :reply, h, stack }
  end

  def handle_cast({ :push, new }, stack) do
    { :noreply, [new|stack] }
  end
end
```

我们的服务器定义了三个函数： `init/1`，`handle_call/3`和`handle_cast/2`。我们不会直接调用这些函数，它们是当我们请求服务器的时候由OTP来使用的函数。我们很快会了解详情，现在我们无需关心更多。为此，在你的命令行里运行`iex -S mix`来开始用mix来启动iex，输入下面的指令：

```
# Let's start the server using Erlang's :gen_server module.
# It expects 3 arguments: the server module, the initial
# stack and some options (if desired):
iex> { :ok, pid } = :gen_server.start_link(Stacker.Server, [], [])
{:ok,<...>}

# Now let's push something onto the stack
iex> :gen_server.cast(pid, { :push, 13 })
:ok

# Now let's get it out from the stack
# Notice we are using *call* instead of *cast*
iex> :gen_server.call(pid, :pop)
13
```

非常好，我们的服务器工作正常！然而在幕后其实发生了很多的事情，然我们来一一探究。

首先，我们用OTP中的[`:gen_server`](http://www.erlang.org/doc/man/gen_server.html)模块发动服务器。注意我们使用了`start_link`， 它启动了服务器并且把当前的进程与之相连。在这种情况下，如果服务器死了，它将会向我们的当前进程发送一个退出消息，使它也退出。我们将在后面看到这个行为。函数`start_link`返回的是新创建的进程的识别符（`pid`）。

之后，我们向服务器发送了一个**cast**消息。消息的内容是`{ :push, 13 }`，与我们之前定义在`Stacker.Server`中的回调函数`handle_cast/2`里的一致。无论何时我们发送一个`cast`消息，函数`handle_cast/2`会被调用来处理这个消息。

接着，我们最终用发送`call`消息地方时，看到了堆栈里的情况，它触发了回调`handle_call/3`。那么，`cast`和`call`到底有什么不同呢？

`cast`消息是异步的：我们向服务器发送一个消息，然而并不期待回复。这也是为什么我们的`handle_cast/2 `回调返回的是`{ :noreply, [new|stack] }`的缘故。这个元组中的第一个元素表明了无需回复，而第二个元素是包含了新物件的经过升级的堆栈。

相反，`call`消息是同步的。当我们发送一个`call`消息，客户端期待一个回复。在这个例子中，回调`handle_call/3`返回了`{ :reply, h, stack }`，其中第二个元素就是用来返回的内容，而第三个是我们不包含头的堆栈。因为`call`能够向客户端返还消息，所以它的也多了一个有关客户端情况的参数（`_from`）。

# 2.1.1 深入学习回调函数

在GenServer的例子中，类似`handle_call`或`handle_cast`的函数可能返回8种不同的数值：

```
{ :reply, reply, new_state }
{ :reply, reply, new_state, timeout }
{ :reply, reply, new_state, :hibernate }
{ :noreply, new_state }
{ :noreply, new_state, timeout }
{ :noreply, new_state, :hibernate }
{ :stop, reason, new_state }
{ :stop, reason, reply, new_state }
```

一个GenServer的实现必须实现6种不同的回调函数。模块`GenServer.Behaviour`自动定义了这些函数，但又允许我们根据需要来修改。下面是这些函数的列表：

* `init(args) ` - 当服务器启动时调用
* `handle_call(msg, from, state)` - 被调用来处理`call`消息
* `handle_cast(msg, state)` - 被调用来处理`cast`消息
* `handle_info(msg, state)` - 被调用来处理进程所收到的其他消息
* `terminate(reason, state)` - 在服务器当机之前被调用，对清理很有用
* `code_change(old_vsn, state, extra)` - 在应用代码热升级的时候被调用

# 2.1.2 干掉一个服务器

Of what use is a server if we cannot crash it?

实际上要使一个服务器当机并不难。我们的回调`handle_call/3`只有当堆栈不是空的时候才工作正常（想起来了吗，`[h|t]`不能匹配空列表）。让我们在堆栈为空的情况下发送一个消息看看：

```
# Start another server, but with an initial :hello item
iex> { :ok, pid } = :gen_server.start_link(Stacker.Server, [:hello], [])
{:ok,<...>}

# Let's get our initial item:
iex> :gen_server.call(pid, :pop)
:hello

# And now let's call pop again
iex> :gen_server.call(pid, :pop)

=ERROR REPORT==== 6-Dec-2012::19:15:33 ===
...
** (exit)
...
```

你可以看到，这里有两个错误报告。第一个因为当机由服务器产生。因为服务器是同我们的进程相连的，它也发送了一个退出的消息，`IEx as ** (exit) ....`

因为我们的服务器总会崩溃，我们需要监控它们，这也是下面的内容。`GenServer.Behaviour`不仅仅包含我们已经学到的这些。请查看[`GenServer.Behaviour`](http://elixir-lang.org/docs/stable/GenServer.Behaviour.html)的文档来发现更多。

# 2.2 监控服务器

当在Eralng和Elixir里编写应用，一个常用到的哲学是fail first。也许是因为资源不可得，也许是服务之间的超时，或许是其他的什么原因。这也是为什么有能力对这些崩溃作出反应并回复过来，是非常重要的。把这些牢记在心，我们为我们的服务器编写一个supervisor。

用下面的内容创建一个文件，`lib/stacker/supervisor.ex`：

```
defmodule Stacker.Supervisor do
  use Supervisor.Behaviour

  # A convenience to start the supervisor
  def start_link(stack) do
    :supervisor.start_link(__MODULE__, stack)
  end

  # The callback invoked when the supervisor starts
  def init(stack) do
    children = [ worker(Stacker.Server, [stack]) ]
    supervise children, strategy: :one_for_one
  end
end
```

在监控中，唯一需要实现的回调函数是`init(args)`。这个回调必须返回监工的规格，在上面的例子中实际调用了帮助函数`supervise/2`。

我们的监工非常简单：它必须监控我们的工人`Stacker.Server`， 工人的启动需要一个参数，在这是默认的堆栈。完成定义的工人被用`:one_for_one`的策略监控，这意味着每一次工人死亡之后都会被重启。

由于我们的工人由`Stacker.Server `模块指定，并且需要传递`stack`作为参数，在默认下，监工将调用函数`Stacker.Server.start_link(stack)`来启动工人，所以让我们来实现它：

```
defmodule Stacker.Server do
  use GenServer.Behaviour

  def start_link(stack) do
    :gen_server.start_link({ :local, :stacker }, __MODULE__, stack, [])
  end

  def init(stack) do
    { :ok, stack }
  end

  def handle_call(:pop, _from, [h|stack]) do
    { :reply, h, stack }
  end

  def handle_cast({ :push, new }, stack) do
    { :noreply, [new|stack] }
  end
end
```

函数`start_link`同我们之前启动服务器的方式很相似，除了在这里我们需要多传递一个参数`{ :local, :stacker }`。这个参数在本地节点上注册我们的服务器，运行用一个名字（在这里，是`:stacker`）来调用它，而无需直接使用`pid`。

不借助监工，让我们运行`iex -S mix`来再一次打开控制台，这也会再一次编译我们的文件：

```
# Now we will start the supervisor with a
# default stack containing :hello
iex> Stacker.Supervisor.start_link([:hello])
{:ok,<...>}

# And we will access the server by name since
# we registered it
iex> :gen_server.call(:stacker, :pop)
:hello
```

注意监工自动为我们启动了服务器，现在我们可以用名字`:stacker`向服务器发送消息了。如果我们使得服务器当机，会发送甚什么？

```
iex> :gen_server.call(:stacker, :pop)

=ERROR REPORT==== 6-Dec-2012::19:15:33 ===
...
** (exit)
...

iex> :gen_server.call(:stacker, :pop)
:hello
```

它和前面一样当机了，当监工立刻用默认的堆栈重启了它，使得我们可以再一次接受到`:hello`。太棒了！

默认下，监工运行一个工人在5秒内最多当机5次。如果工人当机的频率超过了这个限制，监工就会放弃它，不在重启。让我们尝试接连发送一个未知的消息看看（要快！）：

```
iex> :gen_server.call(:stacker, :unknown)
... 5 times ...

iex> :gen_server.call(:stacker, :unknown)
** (exit) {:noproc,{:gen_server,:call,[:stacker,:unknown]}}
    gen_server.erl:180: :gen_server.call/2
```

第六个消息不在产生错误报告，因为我们的服务器不在自动重启了。Elixir返回`:noproc`（`no process`的简写形式），意味着那里已经没有一个被称为`:stacker`的进程了。重启的次数和时间间隔，可以通过向函数`supervise`传递参数进行修改。除了上面例子中的`:one_for_one`， 监工也能选用不同的重启策略。如果向知道还有那些支持的策略，检查`Supervisor.Behaviour`的[文档](http://elixir-lang.org/docs/stable/Supervisor.Behaviour.html)。

# 2.3 谁来监控监工？

我们已经编写了我们的监工，但有一个问题：谁来监控监工？为了回答这个问题，OTP有一个概念，应用（application）。应用可以被作为一个整体启动或关闭，当这些发生的时候，它们通常和一个监工相连。

在之前的章节中，我们已经看到了Mix如何用文件`mix.exs`中的`application`函数中包含的信息来自动地产生一个`.app`文件，每次编译我们的项目。

这个`.app`文件被称为应用规格，它必须包含我们的应用的依赖，它定义的模块，注册名和其他的许多。其中的一些信息由Mix来自动完成，但另外的一些数据需要手动添加。

在我们的例子里面，我们的应用有一个监工，而且它用名字`:stacker`注册了监工。也就是说，为了避免冲突，我们需要在应用规格里加入所有的注册名。如果两个应用注册了同一个名字，我们能更快地找到冲突。所以，让我们打开文件``，用下面的内容编辑`application`函数：

```
def application do
  [ registered: [:stacker],
    mod: { Stacker, [:hello] } ]
end
```

在`:registered`键中我们指定了我们的应用注册的所有名字。而`:mod`键的用途是，一旦应用启动，它必须去调用应用模块回调函数（appliation module callback）。在我们的例子里面，应用模块回调函数是`Stack`模块，而且它将会接受到默认的参数，堆栈`[:hello]`。这个回调函数必须返回同应用相关的监工的`pid`。

有了这些在心里，让我们打开文件`lib/stacker.ex`，添加以下的内容：

```
defmodule Stacker do
  use Application.Behaviour

  def start(_type, stack) do
    Stacker.Supervisor.start_link(stack)
  end
end
```

`Application.Behaviour`期待两个回调，`start(type, args)`和`stop(state)`。我们需要来实现`start(type, args)`， 而`stop(state)`可以先放在一边不管。

在添加了上面的应用行为之后，你只需要在一次启动`iex -S mix`。我们的文件将被再一次重编译，监工（包括我们的服务器）会自动启动：

```
iex> :gen_server.call(:stacker, :pop)
:hello
```

太棒了，它能性！也许你已经注意到了，应用回调函数`start/2`接受了一个类型参数，虽然我们忽略了它。这个类型控制当我们的监工，自然也包括应用，崩溃的时候，虚拟机应该如何应对。你可以通过阅读`Application.Behaviour`的[文档](http://elixir-lang.org/docs/stable/Application.Behaviour.html)学到更多。


最后，注意`mix new`支持一个`--sup`选项，它告诉Mix产生一个包含应用模块回调的监工，自动地完成了一些上面的工作。你一定要试试！

# 2.3 启动应用

在任何时候，我们都不必自己去启动我们定义的应用。那是因为默认下Mix会启动所有的应用，包括所依赖的应用。我们能通过调用OTP提供的[`:application`模块](http://www.erlang.org/doc/man/application.html)中的函数来手动地启动应用：

```
iex> :application.start(:stacker)
{ :error, { :already_started, :stacker } }
```

在上面的例子中，因为应用已经事先启动了，它返回了一个错误信息。

Mix不仅启动你的应用，而且包括所有的你的应用的依赖。请注意你的项目的依赖（我们在前面几章中讨论过的，定义在键`deps`里的）和应用依赖是不同的。

项目依赖也许会包含你的测试框架或者一个编译时的依赖。应用依赖是运行时你的应用依赖的一切。一个应用依赖需要明确地被加入`appliation`函数里：

```
def application do
  [ registered: [:stacker],
    applications: [:some_dep],
    mod: { Stacker, [:hello] } ]
end
```

当在Mix里运行任务，它将确保应用以及应用的依赖都会被启动。

# 2.5 配置应用

除了我们已经看到的键`:registered`，`:applications`和`:mod`，应用也支持被读取和设置的配置数值。

在命令行里，尝试：

```
iex> :application.get_env(:stacker, :foo)
:undefined
iex> :application.set_env(:stacker, :foo, :bar)
:ok
iex> :application.get_env(:stacker, :foo)
{ :ok, :bar }
```

这个机制非常有用，它无需创建一整个监工链就能为你的应用提供配置数值。应用的默认的配置数值可以通过如下的方式在文件`mix.exs`中定义：

```
def application do
  [ registered: [:stacker],
    mod: { Stacker, [:hello] },
    env: [foo: :bar] ]
end
```

现在，从控制台里退出，然后用`iex -S mix`重启它：

```
iex> :application.get_env(:stacker, :foo)
{ :ok, :bar }
```

例如，IEx和ExUnit是两个Elixir包含的应用，它们的`mix.exs`文件就是使用了这样的配置，文件[IEx](https://github.com/elixir-lang/elixir/blob/master/lib/iex/mix.exs)和[ExUnit](https://github.com/elixir-lang/elixir/blob/master/lib/ex_unit/mix.exs)。这样的应用于是提供了提供了[对读写这些数值的封装](https://github.com/elixir-lang/elixir/blob/d2bfd10299dc0e2df573589f1d33565177d63a7d/lib/ex_unit/lib/ex_unit.ex#L125-L155)。

到这里，我们结束了这一章。我们已经学习了如何创建服务器，监控它们，把它们和我们的应用整合和提供简单的配置选项。在下一章，我们将学习如何创建一个Mix中的定制任务。
