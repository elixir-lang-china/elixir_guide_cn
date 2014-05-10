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

这个协议期待实现一个单个参数的函数的`blank?`。我们能为不同的Elixir数据类型实现这个协议：

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

现在用了协议和它的实现在手，我们能调用了：

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

## 16.1 现已和structs

Elixir的可扩展性的力量只有当协议和struct结合在一起的时候才显露出来。

在之前的一章，我们已经学习到了虽然struct就是表单，但它们并没有和表单共享协议的实现。然我们同在前面一章一样，定义一个`User`struct：

```
iex> defmodule User do
...>   defstruct name: "jose", age: 27
...> end
{:module, User,
 <<70, 79, 82, ...>>, {:__struct__, 0}}
```

然后


```
iex> Blank.blank?(%{})
true
iex> Blank.blank?(%User{})
** (Protocol.UndefinedError) protocol Blank not implemented for %User{age: 27, name: "jose"}
```

由于没有能和表单分享协议实现，struct需要它们自己的实现：


```
defimpl Blank, for: User do
  def blank?(_), do: false
end
```

如果需要，你可以使用你自己的对用户是否为空的定义。不仅如此，你能用struct来编写更健壮的数据类型，比如请求，和为这个数据实现所有的相关协议，例如`Enumerable`和甚至`Blank`。

在许多的实际应用中，开发者也许希望为struct提供一个默认的实现，因为为每一个struct都实现这个协议会很无聊。这时就是falling back to any显示威力的时候：

# 16.2 Falling back to Any

如果我们能为所有的类型提供一个默认的实现，将是非常方便的。这可以通过在协议定义中讲设置`@fallback_to_any`为`true`来实现：

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

现在所有的那些我们还没有实现`Blank`协议的的数据类型（包括struct），都会被视为不空。

# 16.3 内建协议

Elixir包含了一个内建的协议。在之前的几章中，我们已经讨论了`Enum`模块就提供了许多的能通用于任何实现了`Enumerable`协议的数据类型：

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

注意Elixir中的字符串解析就调用了`to_string`函数：

```
iex> "age: #{25}"
"age: 25"
```

上的片段能运行因为数字实现了协议`String.Chars`。如果传递一个元组，会导致一个错误：

```
iex> tuple = {1, 2, 3}
{1, 2, 3}
iex> "tuple: #{tuple}"
** (Protocol.UndefinedError) protocol String.Chars not implemented for {1, 2, 3}
```

当有需要去“打印”更复杂的数据结构的时候，简单调用`inspect`函数就行，它是基于协议`Inspect`：

```
iex> "tuple: #{inspect tuple}"
"tuple: {1, 2, 3}"
```

`Inspect`协议用于把任何数据结构转换成可都的文字呈现。这也是类似IEx的工具打印的结果：

```
iex> {1, 2, 3}
{1,2,3}
iex> %User{}
%User{name: "jose", age: 27}
```

谨记，作为一个约定，当打印出的值以`#`开头，它是在用Elixir中非法的语法来呈现一个数据结构。这说明，inspect协议是不可逆的，因为在这个过程中有些信息丢失了。

```
iex> {1, 2, 3}
{1,2,3}
iex> %User{}
%User{name: "jose", age: 27}
```

除此之外，Elixir中还有一些其他的协议， 但这一章涵盖了最常见的几个。在下一章我们讲学习一点Elixir的异常和错误处理。
