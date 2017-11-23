# 14 模块属性

Elixir中的模块属性有三个作用：

    1. 它们提供对模块的注释， 很多信息对用户和Erlang虚拟机都有用。

    2. 它们也是恒量。

    3. 在编译时，它们也是模块的零时存储。

让我们一一来看看

## 14.1 注释

Elixir从Erlang中借用了模块属性的概念。例如

```
defmodule MyServer do
  @vsn 2
end
```
在上面的例子中，我们显明地设置了那个模块的版本。Erlang的代码重载机制使用`@vsn`来检查一个模块是否已经是最新的。如果版本没有指定，版本就是模块函数的md5数值。

Elixir有一些保留的属性。下面这些只是其中最常用的部分：

* `@moduledoc` 提供当前模块的文档
* `@doc` 提供紧跟这个属性的函数或宏的文档
* `@behaviour` （注意英式英语拼法）用来指定一个OTP或用户指定的行为
* `@before_compile` 提供了一个在模块被编译之前的调用的hook。我们可以利用它在模块编译之前注入新的函数。

`@moduledoc`和`@doc`是最常用的两个属性，我们会经常用到。Elixir把文档当成第一等公民，提供了很多用于访问文档的函数：

```
iex> defmodule MyModule do
...>  @moduledoc "It does **x**"
...>
...>  @doc """
...>  Returns the version
...>  """
...>  def version, do: 1
...> end
{:module, MyModule, <<70, 79, 82, ...>>, {:version, 0}}
iex> h MyModule
* MyModule

It does **x**

iex> h MyModule.version
* def version()

Returns the version
```

Elixir的惯例是用markdown和heredocs来编写注重可读性的文档。Heredocs是多行的字符串，它们以三联的引号结束，这样就不用破坏内部的格式了：

```
defmodule Math do
  @moduledoc """
  This module provides mathematical functions
  as sin, cos and constants like pi.

  ## Examples

      Math.pi
      #=> 3.1415...

  """
end
```

我们也提供了一个叫[ExDoc](https://github.com/elixir-lang/ex_doc)的工具，它能被用于从文档里产生HTML网页。

你可以通过查看[Module](http://elixir-lang.org/docs/stable/Module.html)的文档来找到支持的属性的完整列表。Elixir也用属性来定义[typespecs](http://elixir-lang.org/docs/stable/Kernel.Typespec.html)，通过：

* `@spec` - 提供了函数的指标
* `@callback` - 提供了行为回调函数的指标
* `@type` - 定义了在`@spec`内使用的类型
* `@typep` - 定义了在`@spec`内部使用的私有类型
* `@opaque` - 定义了在`@spec`内部使用的opaque类型

这个部分涵盖了内建的属性。然而，属性也能被开发者使用或被库扩展来支持客制化的行为。

## 14.2 作为恒量

Elixir开发者会经常用模块属性来当成恒量：

```
defmodule MyServer do
  @initial_state %{host: "147.0.0.1", port: 3456}
  IO.inspect @initial_state
end
```

> 注意，和Erlang不同，用户定义的属性默认不被存储咋模块内。它们的值只有在编译时才存在。开发者能用通过调用[`Module.register_attribute/3`](http://elixir-lang.org/docs/stable/Module.html#register_attribute/3)来使得一个属性接近Erlang中的行为。

试图去访问一个不存在的属性为导致一个警告：

```
defmodule MyServer do
  @unknown
end
warning: undefined module attribute @unknown, please remove access to @unknown or explicitly set it to nil before access
```

最后，属性能被从函数内部访问：

```
defmodule MyServer do
  @my_data 14
  def first_data, do: @my_data
  @my_data 13
  def second_data, do: @my_data
end

MyServer.first_data #=> 14
MyServer.second_data #=> 13
```

注意从函数内部读取一个属性的值，只是这个值的当前切片。也就是说，读取是在编译时而不是运行时发生的。我们将看到，这让属性能被当作编译时的溢恶存储机制。

## 14.3 作为零时存储

Elixir组织其中的一个项目是[`Plug`](https://github.com/elixir-lang/plug)，它是目标是称为Elixir中编写web库和框架的通过基础。

Plub库也允许开发者去定义它们自己的plugs，能被在web服务器内运行：

```
defmodule MyPlug do
  use Plug.Builder

  plug :set_header
  plug :send_ok

  def set_header(conn, _opts) do
    put_resp_header(conn, "x-header", "set")
  end

  def send_ok(conn, _opts) do
    send(conn, 200, "ok")
  end
end

IO.puts "Running MyPlug with Cowboy on http://localhost:4000"
Plug.Adapters.Cowboy.http MyPlug, []
```

在上面的例子中，我们用宏` plug/1`去连接那些会被当web服务器存在时调用的函数。从内部来讲，每次你调用`plug/1`， Plub库会把接受到的参数存储在属性`@plugs`内。就在模块编译之气那，Plug调用了一个定义了访法(`call/2`)的回调，用它来处理请求。这个访法将按次序运行属性`@plugs`中的所有plugs。

为了能理解底层的代码，我们需要宏，随意我们讲在有关元编程的那一章再次研究这个模式。然而当前的焦点是去用模块属性来当成一个存储来运行开发者来创建DSL。

另一个例子来自于ExUnit框架，它用模块属性来注释和存储。

```
defmodule MyTest do
  use ExUnit.Case

  @tag :external
  test "contacts external service" do
    # ...
  end
end
```

ExUnit中的标签用来注释测试，之后也能被用于过滤测试。例如，你能避免运行外部测试在你的机器上，因为它们都很慢而且依赖于其他的服务，当然你也可以在你自己的编译系统上开启它们。

我们希望这个部分让你领略了Elixir是如何支持元编程和模块属性是如何在其中扮演了一个重要的角色。

在涉及到异常处理和其他结构sigils和comprehensions之气那，我们将在下一章，探索structs和协议。
