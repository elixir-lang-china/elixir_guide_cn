# 模块

在Elixir中，相关的函数会被组合到一起，称为模块。在之前的章节中，我们已经用到了很多模块，例如[字符串模块](http://elixir-lang.org/docs/stable/String.html)。

```
iex> String.length "hello"
5
```

要在Elixir中创建我们自己的模块，我需要使用宏`defmodule`。我们使用另一个宏`def`在模块中定义函数：

```
iex> defmodule Math do
...>   def sum(a, b) do
...>     a + b
...>   end
...> end

iex> Math.sum(1, 2)
3
```

在接下去的部分中，我们的例子将会变得更加复杂，并且可能不太容易手动输入进iex里。不过乘此机会，我们正好可以学习一下如何编译Elixir代码和如执行Elixir脚本。

## 8.1 编译

在大部分的时候最好将模块写入文件，以便于编译和重用。这里我们假定我们已经有了一个文件`math.ex`，包含如下的代码：

```elixir
defmodule Math do
  def sum(a, b) do
    a + b
  end
end
```

我们可以用`elixirc`来编译这个文件：

```bash
elixirc math.ex
```

这会产生一个对应模块的字节码文件的`Elixir.Math.beam`。如果在这个文件所在的目录，我们重新开始`iex`， 我们先前定义的模块就可以使用了。

```
iex> Math.sum(1, 2)
3
```

Elixir的项目通常会包含至少以下三个目录：
* ebin - 包含编译后的字节码
* lib - 包含Elixir源代码 （通常是`.ex`文件）
* test - 包含测试（通常是 `.exs`文件）

在实际的项目中，用到的编译工具是`mix`，它负责设置正确的路径并编译。为了方便学习，Elixir也提供了一个更加灵活的脚本模式，无需编译可以之间运行。

## 8.2 脚本模式

除了常见的Elixir源码文件扩展名`.ex`，Elixir还支持`.exs`文件作为脚本。Elixir对两种文件是一视同仁的，唯一的区别在于`.ex`文件必须编译的，而`.exs`无需编译就可以之间运行。举例来说，我们可以创建一个叫`math.exs`的文件：

```
defmodule Math do
  def sum(a, b) do
    a + b
  end
end

IO.puts Math.sum(1, 2)
```

然后执行文件：

```
elixir math.exs
```

这个文件将会被在内存中编译并执行，然后打印出结果“3”。它不产生字节码。对于下面的例子，我们建议你把你的代码写入脚本中，然后按照上面的方式执行。

## 8.3 有名函数

在模块内部，我们可以用`def/2`定义函数，用`defp/2`定义私有函数。用`def/2`定义的函数能够外部的模块调用而私有函数只能被从模块内部使用。

```
defmodule Math do
  def sum(a, b) do
    do_sum(a, b)
  end

  defp do_sum(a, b) do
    a + b
  end
end

Math.sum(1, 2)    #=> 3
Math.do_sum(1, 2) #=> ** (UndefinedFunctionError)
```

函数申明同时也支持守护和多子句。如果一个函数有多个子句，Elixir会尝试每一个子句直到发现匹配的那个。下面的例子实现了一个函数去检查输入的数字是否是零：

```
defmodule Math do
  def zero?(0) do
    true
  end

  def zero?(x) when is_number(x) do
    false
  end
end

Math.zero?(0)  #=> true
Math.zero?(1)  #=> false

Math.zero?([1,2,3])
#=> ** (FunctionClauseError)
```

如果一个参数无法匹配任何一个子句，会导致一个错误。

## 8.4 函数捕捉

在这篇教程中，我们一直都用`函数名/参数量`的方式指向函数。用这种方式也可以用来获取模块中的有名函数。让我们重新打开`iex`，并运行之前定义的`math.exs`脚本：

```
$ iex math.exs
```

```
iex> Math.zero?(0)
true
iex> fun = &Math.zero?/1
&Math.zero?/1
iex> is_function fun
true
iex> fun.(0)
true
```

本地函数或者已经引入的其他模块的函数，比如`is_function/1`， 没有模块也可以被捕捉。

```
iex> &is_function/1
&:erlang.is_function/1
iex> (&is_function/1).(fun)
true
```

注意捕捉语法也是一种定义函数的快捷方式：

```
iex> fun = &(&1 + 1)
#Function<6.71889879/1 in :erl_eval.expr/5>
iex> fun.(1)
2
```

上面例子中`&1`是传给函数的第一个参数。`&(&1 + 1)`等价于`fn x -> x + 1 end`。上面的语法适合于定义短小的函数。更多的关于函数捕捉操作符`&`，请参考[Kernel.SpecialForms文档](http://elixir-lang.org/docs/stable/Kernel.SpecialForms.html)。

## 8.5 默认参数

Elixir中的有名函数也支持默认参数：

```
defmodule Concat do
  def join(a, b, sep \\ " ") do
    a <> sep <> b
  end
end

IO.puts Concat.join("Hello", "world")      #=> Hello world
IO.puts Concat.join("Hello", "world", "_") #=> Hello_world
```

默认参数值可以是任何一个表达式，但不会在函数定义时执行。它只是被存储在哪里。每次函数被调用的时候，所有的默认参数值都会被使用，代表参数值的表达式就会被执行：

```
defmodule DefaultTest do
  def dowork(x \\ IO.puts "hello") do
    x
  end
end
```

```
iex> DefaultTest.dowork 123
123
iex> DefaultTest.dowork
hello
:ok
```

在一个一个带有默认参数的函数有多个子句，我们建议创建一个函数头（无需实际的函数体），专用于申明默认参数：

```
defmodule Concat do
  def join(a, b \\ nil, sep \\ " ")

  def join(a, b, _sep) when nil?(b) do
    a
  end

  def join(a, b, sep) do
    a <> sep <> b
  end
end

IO.puts Concat.join("Hello", "world")      #=> Hello world
IO.puts Concat.join("Hello", "world", "_") #=> Hello_world
IO.puts Concat.join("Hello")               #=> Hello
```

在使用默认参数值的时候，注意避免覆盖函数定义。考虑下面的例子：

```
defmodule Concat do
  def join(a, b) do
    IO.puts "***First join"
    a <> b
  end

  def join(a, b, sep \\ " ") do
    IO.puts "***Second join"
    a <> sep <> b
  end
end
```

如果我们把上面的代码保存到一个文件“concat.ex”，并编译。Elixir会发出下面的警告：

```
concat.exs:7: this clause cannot match because a previous clause at line 2 always matches
```

编译器在告诉我们当用两个参数代用函数`join`只会选择`join`的第一个定义，而第二个定义只有在三个参数的时候才会被选中。

```
$ iex concat.exs
```

```
iex> Concat.join "Hello", "world"
***First join
"Helloworld"
```

```
iex> Concat.join "Hello", "world", "_"
***Second join
"Hello_world"
```

到这里我们对模块的简介就结束了。在下面的几章，我们将学习如何用函数递归，Elixir中的可以从别的模块中引入函数的语法工具，以及讨论模块的属性。
