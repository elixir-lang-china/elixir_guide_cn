# 16 协议

协议是Elixir中实现多态的一套机制。任何数据类型，只要实现了相关的协议，都能使用这个协议。让我们看一个例子：

在Elixr中，只有`false`和`nil`被当成非真值。其他的都被认为是真值。对某些应用来说，也许实现一个`blank?`协议是非常重要的，用它来在其他数据类型上返回一个布尔值。例如，一个空列表或空二进制可以被认为是空的。

我们用下面的访法定义这个协议：

```
defprotocol Blank do
  @doc "Returns true if data is considered blank/empty"
  def blank?(data)
end
```

这个协议期待实现一个单个参数的函数`blank?`。我们能为不同的Elixir数据类型实现这个协议：

```
# Integers are never blank
defimpl Blank, for: Integer do
  def blank?(_), do: false
end

# Just empty list is blank
defimpl Blank, for: List do
  def blank?([]), do: true
  def blank?(_),  do: false
end

# Just empty map is blank
defimpl Blank, for: Map do
  # Keep in mind we could not pattern match on %{} because
  # it matches on all maps. We can however check if the size
  # is zero (and size is a fast operation).
  def blank?(map), do: map_size(map) == 0
end

# Just the atoms false and nil are blank
defimpl Blank, for: Atom do
  def blank?(false), do: true
  def blank?(nil),   do: true
  def blank?(_),     do: false
end
```

同时我们也能效法于其他的内置数据类型上。它们是：

* `Atom`
* `BitString`
* `Float`
* `Function`
* `Integer`
* `List`
* `Map`
* `PID`
* `Por`
* `Reference`
* `Tuple`

现在定义了协议以及它的具体实现，我们就能调用它了：

```
iex> Blank.blank?(0)
false
iex> Blank.blank?([])
true
iex> Blank.blank?([1, 2, 3])
false
```

传递一个还没有实现协议的数据类型，会导致一个错误：

```
iex> Blank.blank?("hello")
** (Protocol.UndefinedError) protocol Blank not implemented for "hello"
```

## 16.1 协议和结构体

Elixir的可扩展性的威力只有当协议和结构体共同使用的时候才显露出来。

在之前的一章，我们已经学习到了虽然结构体就是表单，但它们并没有共享表单的协议实现。让我们同前一章一样，定义一个 `User` 结构体：

```
iex> defmodule User do
...>   defstruct name: "john", age: 27
...> end
{:module, User,
 <<70, 79, 82, ...>>, {:__struct__, 0}}
```

然后执行：

```
iex> Blank.blank?(%{})
true
iex> Blank.blank?(%User{})
** (Protocol.UndefinedError) protocol Blank not implemented for %User{age: 27, name: "jose"}
```

由于没有能和表单分享协议实现，结构体需要它们自己的实现：

```
defimpl Blank, for: User do
  def blank?(_), do: false
end
```

如果需要，你可以使用你自己的定义来判断用户是否为空。不仅如此，你能用结构体来编写更健壮的数据类型，比如队列，而且为这个数据类型实现所有的相关协议，例如`Enumerable`，甚至`Blank`。

在许多的实际应用中，开发者也许希望为结构体提供一个默认的实现，因为为每一个结构都实现这个协议会很乏味。这时就是 `Any` 派上用场的时候了。

## 16.2 回归 `Any`

如果我们能为所有的类型提供一个默认的实现，那将是非常方便的。这可以通过在协议定义中将`@fallback_to_any`设置为`true`来实现：

```
defprotocol Blank do
  @fallback_to_any true
  def blank?(data)
end
```

现在它可以这样被实现：

```
defimpl Blank, for: Any do
  def blank?(_), do: false
end
```

现在那些所有我们还没有实现`Blank`协议的的数据类型（包括结构体），都会被视为非空。

# 16.3 内建协议

Elixir包含了一个内建的协议。在之前的几章中，我们已经讨论了`Enum`模块就提供了许多函数，能通用于任何实现了`Enumerable`协议的数据类型：

```
iex> Enum.map [1, 2, 3], fn(x) -> x * 2 end
[2,4,6]
iex> Enum.reduce 1..3, 0, fn(x, acc) -> x + acc end
6
```

另一个有用的例子是`String.Chars`协议，它指定了如何将一个包含字符串的数据结构转换成字符串。它是通过函数`to_string`来曝光的：

```
iex> to_string :hello
"hello"
```

注意Elixir中的字符串插入就调用了`to_string`函数：

```
iex> "age: #{25}"
"age: 25"
```

以上片段能运行是因为数字实现了`String.Chars`协议。如果传递一个元组，会导致一个错误：

```
iex> tuple = {1, 2, 3}
{1, 2, 3}
iex> "tuple: #{tuple}"
** (Protocol.UndefinedError) protocol String.Chars not implemented for {1, 2, 3}
```

当有需要去“打印”一个更复杂的数据结构时，简单调用`inspect`函数就行，它是基于`Inspect`协议：

```
iex> "tuple: #{inspect tuple}"
"tuple: {1, 2, 3}"
```

`Inspect`协议用于把任何数据结构转换成可读的文字呈现。这也是类似IEx的工具打印的结果：

```
iex> {1, 2, 3}
{1,2,3}
iex> %User{}
%User{name: "jose", age: 27}
```

谨记，作为一个约定，当打印出的值以`#`开头的值时，它代表一种Elixir中无效语法的数据结构。这说明inspect协议是不可逆的，因为在这个过程中有些信息可能会丢失。

```
iex> inspect &(&1+2)
"#Function<6.71889879/1 in :erl_eval.expr/5>"
```

除此之外，Elixir中还有一些其他的协议，但这一章涵盖了最常见的几个。
