# 13 `alias`, `require`和`import`

为了达成软件的复用性，Elixir提供了三个命令。我将在下面几章看到，它们之所以被称为指令是因为它们有作用域。

## 13.1 `alias`

`alias`允许你给任何模块设置别名。想象一下我们的`Math`模块使用了一个特别的列表实现用于一些特殊的数学计算：

```
defmodule Math do
  alias Math.List, as: List
end
```

那么从现在开始，任何对`List`的引用都会被扩展成`Math.List`。如果你想要使用原版的`List`模块，它还是可以在命名控件`Elixir`里找到：

```
List.flatten             #=> uses Math.List.flatten
Elixir.List.flatten      #=> uses List.flatten
Elixir.Math.List.flatten #=> uses Math.List.flatten
```

> 注意：Elixir中的所有模块都定义在一个主命名空间`Elixir`下。然而，在引用的时候，你可以忽略它。

别名经常被用于定义快捷方式。实际上，不用选项`as`调用`alias`会自动指向模块名的最后一个部分，比如：

```
alias Math.List
```

等同于：

```
alias Math.List, as: List
```

注意`alias`是有作用域的。你可以只在某个函数内部设置别名：

```
defmodule Math do
  def plus(a, b) do
    alias Math.List
    # ...
  end

  def minus(a, b) do
    # ...
  end
end
```

在上面的例子中， 因为我们在函数`plus/2`内部启用了`alias`，这个别名只在函数`plus/2`内部有效。`minus/2`则不会被影响。

## 13.2 `require`

Elixir提供了宏作为元编程（用代码产生代码）的机制。

宏是在编译时被执行和扩展的代码。这意味着，为了能使用宏，我们需要保证这些模块和实现是在编译时可用的。这可以用`require`来达成：

```
iex> Integer.is_odd(3)
** (CompileError) iex:1: you must require Integer before invoking the macro Integer.is_odd/1
iex> require Integer
nil
iex> Integer.is_odd(3)
true
```

在Elixir中，`Integer.is_odd?/1`是一个可以被当作守护使用的宏。这意味着，为了调用`Integer.is_odd?/1`，我们需要首先引入`Integer`模块。

总的来说，一个模块在被使用之前无需被引入，除非我们需要这个模块中的宏。调用一个不可得的宏会导致一个错误。注意和`alias`一样，`require`也是有作用域的。我们将在更详细地讨论宏。

## 13.3 `import`

用`import`可以非常方便地直接引入其他模块中的函数和宏。例如，如果我们需要反复用到`List`模块中的`duplicate`函数，我们能简单地引入它：

```
iex> import List, only: [duplicate: 2]
nil
iex> duplicate :ok, 3
[:ok, :ok, :ok]
```

在这个例子中，我们仅仅从`List`里引入了函数`duplicate`（参数量2）。虽然`only:`并非是必须的，还是推荐使用。另一个可选项是`except`。

`import`也支持`:macros`和`:functions`和`:only`一起使用。例如，要引入所有的宏，可以这么写：

```
import Integer, only: :macros
```

或这引入全部的函数：

```
import Integer, only: :functions
```

注意`import`也是有作用域的，这意味着我们能在特定函数内引入特定的宏：

```
defmodule Math do
  def some_function do
    import List, only: [duplicate: 2]
    # call duplicate
  end
end
```

在上面的例子中，引入的`List.duplicate/2`只在那个函数内部才可见。`duplicate/2`对这个模块内的其他函数都是不可用的（更不用说其他的模块了）


注意引入一个模块会自动requires它

## 13.4 别名

到这里，也许你在好奇Elixir中的别名到底是什么？它们究竟是如何存在的？

Elixir中的别名首先是一个首字母大写的识别符（像`String`，`Keyword`之类），会在编译时转换成原子。例如，别名`String`默认翻译成`:"Elixir.String"`：

```
iex> is_atom(String)
true
iex> to_string(String)
"Elixir.String"
iex> :"Elixir.String"
String
```

使用了`alias/2`，我们就能改变一个别名最后翻译的结果。

之所以这样是因为在Erlang虚拟机（随之在Elixir）中，模块是用原子表现的。例如，下面是我们如何调用Erlang的模块的：

```
iex> :lists.flatten([1,[2],3])
[1, 2, 3]
```

这套机制同时也允许我们动态地调用模块内的一个函数：

```
iex> mod = :lists
:lists
iex> mod.flatten([1,[2],3])
[1,2,3]
```

也就是说，我们直接在原子`:lists`上调用函数`flatten`。

## 13.5 嵌套

讨论完了别名，我们可以接着谈谈嵌套，以及它是如何工作的。考虑一下下面的代码：

```
defmodule Foo do
  defmodule Bar do
  end
end
```

上面的这个例子定义了两个模块`Foo`和`Foo.Bar`。只要它们一直在同一个作用域内，第二个模块都可以作为`Foo`内部的`Bar`被访问到。如果之后开发者决定把`Bar`移到另一个文件，引用它就需要全名（`Foo.Bar`）了，要么像我们之前讨论的一样，用`alias`设置一个别名。

也就是说，上面的代码等价于：

```
defmodule Elixir.Foo do
  defmodule Elixir.Foo.Bar do
  end
  alias Elixir.Foo.Bar, as: Bar
end
```

随后几章我们将看到，别名也在宏里扮演了关键的角色，用来保证它们的同构性。到这里我们差不多完成了Elixir中的模块部分，下一章是关于模块的属性的。
