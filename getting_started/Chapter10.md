# 10 可枚举类和流

## 10.1 可枚举类

Elixir中提供了可枚举类，和用于处理其的[Enum模块](http://elixir-lang.org/docs/stable/Enum.html)。我们已经见到过两种可枚举类了，列表和表单：

```
iex> Enum.map([1, 2, 3], fn x -> x * 2 end)
[2, 4, 6]
iex> Enum.map(%{1 => 2, 3 => 4}, fn {k, v} -> k * v end)
[2, 12]
```

`Enum`模块提供了众多的函数用以转换，排序，组织，过滤和获取可枚举类中的元素。在Elixir中它是开发者最经常使用的模块之一。

Elixir也支持`range`：

```
iex> Enum.map(1..3, fn x -> x * 2 end)
[2, 4, 6]
iex> Enum.reduce(1..3, 0, &+/2)
6
```

因为Enum模块一设计开始就是着眼于能和不同的数据类型打交道，它的API只包含那些能通用于不同数据类型的函数。对一些特殊的操作，你就可能需要针对那个数据类型的模块了。例如，如果你要把一个元素插入到一个列表的特定位置，你应该使用`List`模块中的`List.insert_at/3`。插入一个值到一个范围中可能就意义不大。

我们说Enum模块中的函数是多态的，是指它们能够通用在不同类型的数据上。实际上，Enum模块的函数可以用在任何实现了[可枚举协议](http://elixir-lang.org/docs/stable/Enumerable.html)的任何数据类型上。我们将在下面的章节中讨论协议，现在我们要讨论可枚举类中的一个特例，流。

## 10.2 即刻 vs 懒惰

Enum模块中的所有函数都是饥渴的。许多函数接受一个可枚举类，并且返回一个列表。

```
iex> odd? = &(rem(&1, 2) != 0)
#Function<6.80484245/1 in :erl_eval.expr/5>
iex> Enum.filter(1..3, odd?)
[1, 3]
```

这意味着当进行连续的函数调用是，每一个调用都会产生一个列表，直到最后结果为止。

```
iex> 1..100_000 |> Enum.map(&(&1 * 3)) |> Enum.filter(odd?) |> Enum.sum
7500000000
```

上面是一个管道操作的例子。我们从一个管道开始，对其中的每一个元素都乘以`3`。第一个操作会返回一个包含`1000_000`个成员的列表。接着我们返回由其中所有的奇数成员组成的列表。最后我们在加总这个有`50_000`个成员的列表。

初次之外，Elixir还有[流模块](http://elixir-lang.org/docs/stable/Stream.html)，提供了懒惰操作：

```
iex> 1..100_000 |> Stream.map(&(&1 * 3)) |> Stream.filter(odd?) |> Enum.sum
7500000000
```

流模块并不产生中间性的列表，相反所有的操作都只有达到Enum模块的时候才会被触发。流模块在处理非常大，甚至_可能无限_长的集合的时候非常有用。

## 10.3 流

流是懒惰的，能降解的可枚举类。

说它懒惰是因为，就像在上面的例子中显示的，`1..100_000 |> Stream.map(&(&1 * 3))`返回一个数据类型，是一个流，它表示对返回`1..100_000`的`map`计算。

```
iex> 1..100_000 |> Stream.map(&(&1 * 3))
#Stream<1..100_000, funs: [#Function<34.16982430/1 in Stream.map/2>]>
```

进一步说，它是可降解的因为我们能通过管道连接很多流：

```
iex> 1..100_000 |> Stream.map(&(&1 * 3)) |> Stream.filter(odd?)
#Stream<1..100_000, funs: [...]>
```

许多流模块中的函数接受任何一种可枚举类并返回一个流作为结果。它同时也提供了创建普通的，甚至是无限的流的函数。例如，`Stream.cycle/1`能被用于创建一个在给定的可枚举类上无限往复的流。注意不要对这样的流使用类似`Enum.map/2 `这样的函数，这回导致一个死循环：

```
iex> stream = Stream.cycle([1, 2, 3])
#Function<15.16982430/2 in Stream.cycle/1>
iex> Enum.take(stream, 10)
[1, 2, 3, 1, 2, 3, 1, 2, 3, 1]
```

在另一方面，`Stream.unfold/2`能被用于从一个初始值产生一个流：

```
iex> stream = Stream.unfold("hełło", &String.next_codepoint/1)
#Function<15.16982430/2 in Stream.cycle/1>
iex> Enum.take(stream, 3)
["h", "e", "ł"]
```

另一个有趣的函数是`Stream.resource/3`，它能被用来对资源进行处理，确保即时在失败的情况下，资源也能在枚举开始之前被打开，结束之后在关闭。例如，我们可以用它来流化一个文件：

```
iex> stream = File.stream!("path/to/file")
#Function<18.16982430/2 in Stream.resource/3>
iex> Enum.take(stream, 10)
```

上面的例子会你选中的文件的前10行。这显示了流在处理大文件或网络资源之类的慢资源的时候会非常有用。

不要被[Enum模块](http://elixir-lang.org/docs/stable/Enum.html)和[Stream模块](http://elixir-lang.org/docs/stable/Stream.html)中的函数的个数和复杂性吓坏了，随着用的越多，你很多就会熟悉它们。我们建议你，首先聚焦在Enum模块中，只有当你在和慢资源或非常巨大，近乎无限的资源打交道，懒惰对你非常重要是才选择流。

下面，我们将研究Elixir中的一个核心特性，进程。它允许我们用一个简单而又容易理解的方式编写并行，分布式的程序。
