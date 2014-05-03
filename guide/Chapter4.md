# 模式匹配

在这章中我们将看到`=`操作符在Elixir中实际上是一个匹配操作符，和如何用它来匹配一个数据结构中的模式，我们将学习用定位操作符`^`来访问之前绑定的值。

## 4.1 匹配操作符

我们之前在Elixir中已经多次使用`=`操作符来进行复制了：

```
iex> x = 1
1
iex> x
1
```

实际上，在`Elixir`里，`=`操作符被称为匹配操作符。这是为什么：

```
iex> 1 = x
1
iex> 2 = x
** (MatchError) no match of right hand side value: 1
```
注意`1 = x`是一个合法的表达式，它能够匹配是因为左右两侧都等于1.当两侧不相等时，会导致一个`MatchError`错误。

只有当变量位于`=`左侧的是，才能复制：

```
iex> 1 = unknown
** (RuntimeError) undefined function: unknown/0
```

在上面的例子中，变量`unknow`之前并没有被定义，Elixir于是认为以在试图调用一个函数`unknown/0`， 当然一个函数也并不存在。


## 4.2 模式匹配

匹配操作符不仅用于匹配单个变量，也能用于结构复杂的数据类型。例如，我们可以匹配元组：

```
iex> {a, b, c} = {:hello, "world", 42}
{:hello, "world", 42}
iex> a
:hello
iex> b
"world"
```

当两边不匹配的时候会报错。举个例子，当两边的元组大小不一样时：

```
iex> {a, b, c} = {:hello, "world"}
** (MatchError) no match of right hand side value: {:hello, "world"}
```

或者当试图匹配不同类型是：

```
iex> {a, b, c} = [:hello, "world", "!"]
** (MatchError) no match of right hand side value: [:hello, "world", "!"]
```

更有趣的是，我们可以匹配指定的值。下面的例子假设只有当右侧是一个一原子`:ok`开头的元组的时候，左侧才匹配：

```
iex> {:ok, result} = {:ok, 13}
{:ok, 13}
iex> result
13

iex> {:ok, result} = {:error, :oops}
** (MatchError) no match of right hand side value: {:error, :oops}
```

我们能对列表进行模式匹配：

```
iex> [a, b, c] = [1, 2, 3]
[1, 2, 3]
iex> a
1
```

列表也支持对头和尾进行匹配：

```
iex> [head | tail] = [1, 2, 3]
[1, 2, 3]
iex> head
1
iex> tail
[2, 3]
```

和函数`hd/1`和`tl/1`类似，我们无法匹配对一个空列表匹配头尾：

```
iex> [h|t] = []
** (MatchError) no match of right hand side value: []
```

`[head | tail]`不仅能用于模式匹配，而且被用于前插元素到一个列表：

```
iex> list = [1, 2, 3]
[1, 2, 3]
iex> [0|list]
[0, 1, 2, 3]
```

模式识别能让开发者非常容易地解构数据类型，不仅能用于元组和列表，也适用于表单（map）和二进制。在接下去的几章中，我们将看到，这是构成Elixir中的函数递归的基础之一。

## 4.3 定位操作符

Elixir中的变量允许重新绑定：

```
iex> x = 1
1
iex> x = 2
2
```

当你不想重新绑定一个变量，只是想匹配当前绑定的值的时候，定位操作符``就有了用武之地：

```
iex> x = 1
1
iex> ^x = 2
** (MatchError) no match of right hand side value: 2
iex> {x, ^x} = {2, 1}
{2, 1}
iex> x
2
```

注意如果一个变量在一个模式中出现了两次以上，所有的引用都必须指向同一个模式：

```
iex> {x, x} = {1, 1}
1
iex> {x, x} = {1, 2}
** (MatchError) no match of right hand side value: {1, 2}
```

在很多情况下，你不需要模式中的所有值。在这种情况下，一个常见的做法是将这些不需要的值绑定到下划线，_。例如，如果一个列表中只有头是重要的话，我们可以我尾绑定到下划线：

```
iex> [h | _] = [1, 2, 3]
[1, 2, 3]
iex> h
1
```

`_`是一个特殊的变量，它是无法被读取的。试图去读它的值会导致一个未绑定变量错误：

```
iex> _
** (CompileError) iex:1: unbound variable _
```

虽然模式匹配允许我们构建非常强大的结构，它并不是万能的。比如，你无法在匹配的左侧调用函数。下面是一个非法的例子：

```
iex> length([1,[2],3]) = 3
** (ErlangError) erlang error :illegal_pattern
```

关乎模式匹配就到此告一段落。在下一章中我们将看到模式匹配被广泛地应用在许多的语言层面的构件中。
