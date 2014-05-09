# 14 模块属性 Module attributes
 
在Elixir中模块属性有三个目的：

* 它们用于模块注释，用户或VM的能够使用这些信息。
* 类似常量的功能 
* 在编译时作为临时的模块存储(module storage)

## 14.1 作为注释(As annotations)

Elixir 模块属性的概念来源于Erlang，例如：

```
defmodule MyServer do
  @vsn 2
end
```

在上面的例子中，我们设置了MyServer该模块的版本属性。`@vsn`用于Erlang code reload机制中的来检查模块代码
是否更新。如果没有指定版本，该版本被设置为模块函数的MD5校验码。

Elixir 保留的属性屈指可数。下面是最常用的几个：

* `@moduledoc` - 为所在的module的文档属性
* `@doc` - 用于函数和宏的文档
* `@behaviour` - 用于指定一个OTP胡行为模式或用户定义的行为.
* `@before_compile` - 提供了一个hook，在module编译之前调用，这使得在编译这前在module注入函数成为可能。

`@moduledoc` 和`@doc` 应该是用的最多的。我们也期待你会经常使用。Elixir把文档看作是第一类(first-class)的对象，并且提供了很多函数访问文档：

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
    
Elixir提倡使用markdown与heredocs编写可读的文档。heredocs多行字符串，开始和结束都是使用三引号，这种方式可以保持内文本的原有的格式：

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
  
我们也提供了一个叫`ExDoc`的工具用来HTML格式的文档。

你可以看下 []Module](http://elixir-lang.org/docs/stable/Module.html)的文档查看完整支持的属性。Elixir也使用下面的属性来定义 [ypespecs](http://elixir-lang.org/docs/stable/Kernel.Typespec.html)

* `@spec` - 可用于描述函数输入参数的类型和返回值的类型
* `@callback` - 描述行为模式的原型定义
* `@type` - 定义类型，可用于 `@spec`
* `@typep` - 定义私有类型，用于 `@spec`
* `@opaque` - 定义opaque 类型，用于 @spec
 
这部分主要介绍的是扩展属性，其实，用户是可以自己定义属性或通过扩展库支持自定义的行为模式。

## 14.2 作为常量(As constants)

Elixir 开发都经常会模块属性作为常量使用:

```
defmodule MyServer do
  @initial_state %{host: "147.0.0.1", port: 3456}
  IO.inspect @initial_state
end
```
    
> 注意：和Erlang不一样的是，用户定义的属性默认不会存储在module内。这些值只会存在于编译时, 如果想让定义属性特性 类似Erlang的属性特性，可以能过[Module.register_attribute/3](http://elixir-lang.org/docs/stable/Module.html#register_attribute/3)

试图访问未经定义的属性会打印warning:

```
defmodule MyServer do
  @unknown
end

warning: undefined module attribute @unknown, please remove access to @unknown or explicitly set it to nil before access
```
    
最后，属性是可以在函数内访问到的：

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
  
请注意，函数里面读取属性是取当前值的快照。也就是说，值是在编译时读取的，而不是运行时。正如我们将要看到的，这使得模块属性可以非常方便的在编译期间被用作存储。

## 14.3 作为临时的存储(As temporary storage)

Elixir 组织(github organization)下有个Plug项目，是在ELixir中构建Web库和框架的通用基础。

`Plug` 允许开发都定义自己的plug，运行在自己的web server：

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


在上面的例子中，我们使用`plug/1`宏连接处理函数，当有web请求时，这些函数会被调用。在内部实现上，每次你调用 `call/1`，Plug会把给定的参数存储到`@plugs`属性。该模块被编译之前，Plug运行一个回调,定义方法`call/ 2`用于处理HTTP请求。这会根据@plugs中定义顺序运行所有的plug。

为了了解底层代码，我们需要的宏，所以我们在元编程指南将再次讨论这个模式。不过这里的重点是使用module的属性如何作为存储, 并可以让开发者创建的DSL。

另一个例子来自ExUnit框架，它使用module属性作为注解和存储:

```
defmodule MyTest do
  use ExUnit.Case

  @tag :external
  test "contacts external service" do
    # ...
  end
end
```

在ExUnit框架里，Tags是用作注解，加了tag，后边就可以过滤到这个测试，因为这个测试运行比较慢且依赖其它的服务，我们不想在开发机器上运行，只想让他在build系统运行。

希望学习了这部分你能感受到Elixir是如何支持元编程，以及module 属性在元编程中发挥的重要作用。


