# 递归

由于在函数式编程语言中数据是不可变的，Elixir中的循环的写法和普通的命令式语言是不同的。例如，在某种命令式语言中，可能是这样的：

```
for(i = 0; i < array.length; i++) {
  array[i] = array[i] * 2
}
```
在上面的例子中，我们改变了数组和变量`i`。在Elixir中这是不行的。事实上，Elixir这样的函数式语言依赖于递归：一个函数被反复地调用直到达到一个界限。下面的这个例子中，用来多次打印一个字符串：

```
defmodule Recursion do
  def print_multiple_times(msg, n) when n <= 1 do
    IO.puts msg
  end

  def print_multiple_times(msg, n) do
    IO.puts msg
    print_multiple_times(msg, n - 1)
  end
end

Recursion.print_multiple_times("Hello!", 3)
# Hello!
# Hello!
# Hello!
```

和例子类似，一个函数可能有多个子句。某一个子句只有在接受到的参数匹配自己的模式而且它的守护返回`true`的时候才会被执行。

在上面的例子中，当函数`print_multiple_times/2 `被初次调用的时候，参数`n`的值是`3`。

第一个子句有一个守护，只有当`n`小于等于`1`的时候才满足条件。显然这个条件目前无法满足，所以Elixir选择执行第二个子句。

第二个定义满足定义而且没有守护，所以它被执行了。首先它打印出`msg`，然后用`n - 1（2）`作为第二个参数再调用自身。我们的`msg`再次被打印了，然后函数`print_multiple_times/2 `被调用，这一次第二个参数的值是`1`。

因为现在`n`为`1`， 函数``的第一个定义的守护返回`true`，这个定义就被执行了。`msg`再一次被打印，完成整个过程。

根据我们对`print_multiple_times/2 `的定义，不管第二个参数是什么，要么执行第一个定义（又称“基本例子”）或者执行我们的第二个定义，它的每次调用都是我们离基本定义更进一步。

现在让我们看看如何利用递归的威力来加总一个列表中的数字：

```
defmodule Math do
  def sum_list([head|tail], accumulator) do
    sum_list(tail, head + accumulator)
  end

  def sum_list([], accumulator) do
    accumulator
  end
end

Math.sum_list([1, 2, 3], 0) #=> 6
```

我们用列表`sum_list`和初始值0作为参数来调用`[1,2,3]`.我们会尝试每一个子句直到找到匹配模式规则的那个。在这个例子中，列表`[1,2,3]`匹配模式`[head|tail]`，于是`head = 1`和`tail = [2,3]`， 而累计值`accumulater`被设为`0`；

接着我们列表的有和累计值相加`head + accumulator`，然后再次调用`sum_list`，同时把列表的尾作为第一个参数。因为尾也是一个列表，所以可以一次又一次匹配`[head|tail]`，直到列表为空，然我们看一看下面的过程：

```
sum_list [1, 2, 3], 0
sum_list [2, 3], 1
sum_list [3], 3
sum_list [], 6
```

当列表为空的时候，最后一个子句就被匹配了，它返回最终的结果，`6`。

这样的把一个列表的中元素个数递减到一个的过程，称为“递减”算法，它是函数式编程的基础之一。

如果我们要把一个列表中的数值都倍增，该怎么办？

```
defmodule Math do
  def double_each([head|tail]) do
    [head * 2| double_each(tail)]
  end

  def double_each([]) do
    []
  end
end

Math.double_each([1, 2, 3]) #=> [2, 4, 6]
```

在这里我们用递归遍历了整个列表并且把每个元素都加倍，然后返回一个新的列表。这种把历遍列表中每个元素的的过程被称为“映射”算法。

递归和尾调优化是Elixir重要的组成部分，常用于创建循环。然而，在实际的Elixir编程中你其实并不需要想上面的例子那样来做。

在我们下一章就要讲到的[Enum](http://elixir-lang.org/docs/stable/Enum.html)模块中，已经为处理列表提供了很多的方便。比如，上面的例子可以被写成这样：

```
iex> Enum.reduce([1, 2, 3], 0, fn(x, acc) -> x + acc end)
6
iex> Enum.map([1, 2, 3], fn(x) -> x * 2 end)
[2, 4, 6]
```
或者，使用捕捉语法：

```
iex> Enum.reduce([1, 2, 3], 0, &+/2)
6
iex> Enum.map([1, 2, 3], &(&1 * 2))
[2, 4, 6]
```

接下去让我们仔细的研究一下可枚举类和流。
