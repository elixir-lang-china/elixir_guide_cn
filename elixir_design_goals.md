# Elixir 设计目标

英文原文：[Elixir Design Goals](http://elixir-lang.org/blog/2013/08/08/elixir-design-goals/)


在去年，我们已经在各种技术会议上讨论Elixir，我们一般从[介绍Elixir VM](http://vimeo.com/53221562)开始,  接着讨论Elixir的目标，抽出一些时间做现场演示，比如展示一些像在两个远程节点交换信息，甚至热更新代码。

这篇文章总结这些讲座，关注ELixir语言的目标: `兼容性`、`生产力`和`可扩展性`。 

### 兼容性(Compatibility)

Elixr 兼容Erlang VM以及现有的生态系统，当我们讨论Erlang时，一般来说主要三部分： 

* 一门叫Erlang的编程语言 
* 一系列设计原则，称为OTP 
* Erlang 虚拟机，指的是EVM和BEAM 

`Elixir`和`Erlang`运行在一样的虚拟机上，并且兼容OTP。当然不仅仅这些，所有的Erlang生态系统的工具，库
也可以用于Elixir，因为在Elixir中调用Erlang不存在性能成本，反之亦然。 

我们经常说`Erlang VM`是`Elixir`最宝贵的资产。 

所有的Elixir代码在轻量级的`进程process(actors)`中执行，每一个`进程(process)`都有自己的状态，彼此之间交换消息。
Erlang VM 将这些进程分发到多个cpu核心运行，使程序非常容易的并行。 

事实上，如果你编译过Elixir代码，包括Elixir的源代码，你会发现所有的cpu核心都在运转。随着[并行技术](http://www.parallella.org/board/) 变得更加简单和实惠，你很难忽略Erlang VM带给我们强大功能。 

最后，Erlang VM旨在构建永久运行，能够自我修复和大规模可扩展的系统。JoeArmstrong，Erlang的创建者，最近出了一个[关于OTP和Erlang VM背后的设计思想](http://www.infoq.com/presentations/self-heal-scalable-system)讲座。 

开源项目比如像CouchDB, Riak, RabbitMQ, Chef11等，还有像公司比如Ericsson, Heroku, Basho, Klarna 和 Wooga  已经从 Erlang VM的优秀特性中受益, 其中一些使用Eralng VM 已经相当长的时间

### 生产力(Productivity)

> Now we need to go meta. We should now think of a language design as being a 
pattern for language designs. A tool for making more tools of the same kind. [...] 
A language design can no longer be a thing. It must be a pattern, a pattern for 
growth. A pattern for growing a pattern, for defining the patterns that 
programmers can use for their real work and main goals.         

>    -- Guy Steele, keynote at the 1998 ACM OOPSLA conference on "Growing a Language" 

生产力很难测量，能够高效开发桌面应用的语言在数学计算方面可能会捉襟见肘。高生产力与你使用这门语言期望
从事的领域，生态系统中的可用工具，以及是否能可以方便的创造和扩展这些工具。 

基于这个原因，Elixir语言核心实现的非常简约。例如，在许多编程语言里，`if,case, try` 这些关键字都需要专
门的语法分析规则去分析，而在Elixir里，这些只是`宏(macros)`,这样做的好处就是开发者按可以照自己的需求去扩展
语言。 

这里有个例子如何在Elixir里实现一个`unless`， unless是很多语言的关键字 

```
defmacro unless(expr, opts) do 
  quote do 
    if(!unquote(expr), unquote(opts)) 
  end 
end 
 
unless true do 
  IO.puts "this will never be seen" 
end 
```


因为`宏`接收代码作为参数，我们可以在编译时非常简单的把`unless`转换在`if`

宏(Macros)也是Elixir`元编程(Meta-programming)`的构建基础：编写生成代码的代码的能力。元编程可以让开发者摆脱语言的束缚，创造强大的工具。在讲座中经常提到的一个常见例子是如何使用宏让测试框架更有表现力。让我们看一个例子： 

```
ExUnit.start 
defmodule MathTest do 
  use ExUnit.Case, async: true 
  test "adding two numbers" do 
    assert 1 + 2 == 4 
  end 
end 
```

注意到的第一件事是`async: true`选项，当你的测试代码没有任何负作用(side-effect)的时候，可以通过`async: true`  选项设定以并发运行测试。 

下一步，我们使用`assert`宏定义了一个断言。直接调用 `assert` 是一个糟糕的实践，因为错误报告不可读。在Elixir中，
推荐的方法是使用 `assertEqual` 或者 `assert_equal` 执行断言


在Elixir中，`assert`是一个宏，因此，它能够操控被断言的代码，并进行推断。这段代码，在测试运行的时候提供了详细的错误报告：


```
1) test adding two numbers (MathTest) 
   ** (ExUnit.ExpectationError) 
                expected: 3 
     to be equal to (==): 4 
   at test.exs:7 
```

这个简单的例子中，开发人员可以利用宏来提供一个简洁但功能强大的API。宏可以访问整个编译环境，能够检查导入的函数，宏, 定义变量等等。

这些例子非常简单，我们可以使用Elixir的宏实现更多的东西。例如，我们正在使用宏实现一个web应用的路由功能，可以编译成一系列被vm高度优化的模式(patterns)。最终提供一个高度的表现力，高度优化的路由算法。

宏系统也对语法有着巨大的影响， 这也是我们最后要讨论的。

### 语法 

尽管许多关于编程语言的讨论一开始就讨论语法，但是，对于Elixir，从没把提供”另一套不同的语法“作为它的目标。自从我们想在Elixir中提供一个宏系统，我们知道如果把Elixir的语法以一个简单直接的方式表达为Erlang的数据结构会是一个非常明智的做法。有了这个目标，我们设计了第一个Elixir版本，看起来是这样的：

```
defmodule(Hello, do: ( 
  def(calculate(a, b, c), do: ( 
    =(temp, *(a, b)) 
    +(temp, c) 
  )) 
)) 
```

在上面的代码片段，我们表示了一切,除了变量，函数或宏调用。像关键字参数`do:`在第一个版本已经存在。对此，我们慢慢地加入新的语法，使得一些常见的模式更优雅，同时保持相同的底层数据表示形式。我们很快就增加了运算符中缀表示法：

```
defmodule(Hello, do: ( 
  def(calculate(a, b, c), do: ( 
    temp = a * b 
    temp + c 
  )) 
)) 
```

下一步是使用括号作为可选的: 

```
defmodule Hello, do: ( 
  def calculate(a, b, c), do: ( 
    temp = a * b 
    temp + c 
  ) 
) 
```

最后，我们使用do/end 来替代 do:(...)结构： 

```
defmodule Hello do 
  def calculate(a, b, c) do 
    temp = a * b 
    temp + c 
  end 
end 
```

由于我先前的Ruby背景，很自然的会从Ruby借鉴一些。但这些只是副产品，不是语言目标。 
很多语言结构也受到Erlang的启发，像是控制流宏，操作符和容器，比如下面的Elixir代码： 


```
#  一个元组 
tuple = { 1, 2, 3 } 
 
# 两个列表合成一个 
[1,2,3] ++ [4,5,6] 
 
# Case 
case expr do 
  { x, y } -> x + y 
  other when is_integer(other) -> other 
end 
```

Erlang是这样的: 

```
% 一个元组 
Tuple = { 1, 2, 3 }. 
 
% 两个列表合成一个 
[1,2,3] ++ [4,5,6]. 
 
% Case 
case Expr of 
  { X, Y } -> X + Y; 
  Other when is_integer(Other) -> Other 
end. 
```

## 扩展性 

Elixir构建在一个非常小的的核心之上，大部分的语言结构可以通过DSL的方式替换和扩展。当然，Elixir天生
擅长某些特定的领域，比如构建并发和分布式的应用，感谢`OTP`和`Erlang VM`。 
Elixir 以标准库的形式补充了下面的领域： 

* Unicode 字符串 和 相应的 操作 
* 一个功能强大的单元测试框架 
* 更多的数据结构，包括新的集合和字典的实现 
* 多态纪录 
* 严格和惰性的枚举API 
* 便于脚本使用的函数，比如使用filesystem和path
* 可以编译和测试Elixir代码的项目管理工具 
等等 

上面大多数的特性也都提供自己的扩展机制，比如,num模块，Enum模块允许我们对`range,list,set`这些数据类型进行迭代。例如： 

```
list = [1,2,3] 
Enum.map list, fn(x) -> x * 2 end 
#=> [2,4,6] 
 
range = 1..3 
Enum.map range, fn(x) -> x * 2 end 
#=> [2,4,6] 
 
set = HashSet.new [1,2,3] 
Enum.map set, fn(x) -> x * 2 end 
#=> [2,4,6] 
```

不仅如此，任何开发人员都可以对任何数据类型实现`Enum模块`的功能，只要对这种类型实现了[Enumerable](http://elixir-lang.org/docs/stable/Enumerable.html)协议（Elixir的协议源于Clojure的协议）,这是非常方便的，因为开发者只需要知道Enum的API就可以，而不用考虑set，list，dict的特定类型的API 

语言本身还内置很多其它的协议,比如`Inspect`协议用于打印数据结构，`Access`协议可以通过key访问`key-value`类型的数据。通过良好的扩展性，Elixir确保开发人员可以方便的使用语言完成工作。

## 总结 

这篇文章的总结了Elixir的设计目标：兼容性，生产力，扩展性。通过兼容Erlang VM，我们为开发者提供了另
一套强大的工具链去构建并发，分布式，容错的系统。

多态结构的可扩展性，标准库为不同数据类型的可扩展和一致的api，包括严格和惰性的枚举，unicode的处理，一个测试框架等等。

想试下Elixir的话，可以从[getting started guide](http://elixir-lang.org/getting_started/1.html)开始 

