# 18 Comprehensions

在Elixir中，常常用到循环历遍一个可枚举类，用于过滤结果，和映射到另一个列表的值上。

Comprehension是对这一中结构的语法糖， 把这些常见的任务用一个特殊的形式`for`组织起来。

例如，我们能用下面的方式得到一个列表值的平方：

```
iex> for n <- [1, 2, 3, 4], do: n * n
[1, 4, 9, 16]
```

一个Compreshension由三个部分组成： 产生器，过滤器和集合。

##　18.1 产生器和过滤器

在上面的表达式中，`n <- [1, 2, 3, 4]`是产生器。体产出了供后面comprehension的值。任何的可枚举类都能被放置在产生器表达式的右侧：

```
iex> for n <- 1..4, do: n * n
[1, 4, 9, 16]
```

产生器表达式也支持模式匹配，来丢弃所有不匹配的模式。想象一下假如不用范围，我们有一个关键字列表，这个列表有两种键`:good `和`:bad`。而我们只关心好的值的计算结果：

```
iex> values = [good: 1, good: 2, bad: 3, good: 4]
iex> for {:good, n} <- values, do: n * n
[1, 4, 16]
```

除此之外，过滤器能被用于过滤一些特定的元素。例如， 我们能只计算奇数的平方：

```
iex> require Integer
iex> for n <- 1..4, Integer.odd?(n), do: n * n
[1, 9]
```

一个过滤器会保留除了`false`和`nil`之外的所有值。

相比直接使用`Enum`和`Stream `模块中的函数，comprehension提供了一种简洁的多的表现形式。而且，comprehension也允许多个产生器和过滤器。这里是一个例子用来接受一个文件夹的列表，然后删除每个文件夹中的所有文件：

```
for dir  <- dirs,
    file <- File.ls!(dir),
    path = Path.join(dir, file),
    File.regular?(path) do
  File.rm!(path)
end
```

谨记，在comprehension内不赋值的变量，无论是在产生器内，还是在过滤器内，不会影响到comprehension外的环境

## 18.2 比特串产生器

比特串产生器也支持的，而且当你需要组织比特串流的时候会非常有用。下面的这个例子从一个二进制接受一个列表的像素，每个像素用红绿蓝三色的数值表示，把它们转成三元组。

```
iex> pixels = <<213, 45, 132, 64, 76, 32, 76, 0, 0, 234, 32, 15>>
iex> for <<r::8, g::8, b::8 <- pixels>>, do: {r, g, b}
[{213,45,132},{64,76,32},{76,0,0},{234,32,15}]
```

一个比特串的产生器能和“普通”的可枚举类产生器混合，并提供过滤器。

# 18.3 Into

在上面的例子中，comprehension返回一个列表作为结果。

然而，用传递`:into`选项，compreshension的结果能被插入不同的数据结果。例如，我们能用比特串产生器和`:into`选项来轻松地删除字符串中的所有空格：

```
iex> for <<c <- " hello world ">>, c != ?\s, into: "", do: <<c>>
"helloworld"
```
`Set`， `maps`和其他类型的字典也能被给予`:into`选项。总的来说，`:into`接受任何一种数据结构，只要它实现了`Collectable`协议。

例如，``模块提供的流，它们既是`Enumerable`又是`Collectable`。你能用comprehension实现一个echo控制台，它返回所有接受到的输入和大写形式。

```
iex> stream = IO.stream(:stdio, :line)
iex> for line <- stream, into: stream do
...>   String.upcase(line) <> "\n"
...> end
```

如果你在那个控制台里输入任何字符串，你将会看到同样的值会以大写的形式被打印出来。不幸的是，这个comprehension会锁死你的终端，所以你必须敲击两次``才能退出程序。：）
