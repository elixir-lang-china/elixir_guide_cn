# 2 基础数据结构

在这章中我们将学习Elixir中的一些基础的数据结构：整数，浮点数，原子，列表和字符串。它们是：

```
iex> 1          # integer
iex> 0x1F       # integer
iex> 1.0        # float
iex> :atom      # atom / symbol
iex> "elixir"   # string
iex> [1, 2, 3]  # list
iex> {1, 2, 3}  # tuple
```

## 2.1 基础运算

打开`iex`， 输入下面的表达式：

```
iex> 1 + 2
3
iex> 5 * 5
25
iex> 10 / 2
5.0
```

你可能注意到了，在Elixir中调用函数时圆括号并不是必须的。

Elixir同时也支持二进制，八进制和十六进制的数字：

```
iex> 0x1F
31
iex> 0o777
511
iex> 0b1010
10
```

浮点数表达式需要在一个点之后跟随至少一位数字，同时也支持`e`的乘方的形式：

```
iex> 1.0
1.0
iex> 1.0e-10
1.0e-10
```

在Elixir中，浮点数都是64位双精度。

## 2.2 布尔值

Elixir支持两种布尔值，`true`和`false`。

```
iex> true
true
iex> true == false
false
```

Elixr提供了一些谓语函数用来检查值的类型。比如，函数`is_boolean/1`能被用来检查一个值是不是布尔值：

> 注意：在Elixir里，用函数名加参数量的组合来识别一个函数。因此，`is_boolean/1`代表一个函数的名字是`is_boolean`，接受的参数量是一个。而`is_boolean/2`（并非真实存在于Elixir里）是另一个函数，虽然名字和之前那个相同，但是接受不同参数的参数。

```
iex> is_boolean(true)
true
iex> is_boolean(1)
false
```

你也可以用`is_integer/1`来检查参数是否是整数，用`is_float/1`检查参数是否是浮点数或者用`is_number/1`检查参数是否是上面两个其中之一。

> 注意：在任何时候你都可以在控制台中键入`h`来打印出帮助信息。`h`函数也能被用来访问函数的文档。例如，输入`h is_integer/1`会打印出函数`is_integer/1`的文档。它也能用在操作符和其他的构件上（试试`h ==/2`）。

## 2.3 原子

原子是一些以名字为值的恒量。在其他的一些语言中，也称为符号（Symbols）。

```
iex> :hello
:hello
iex> :hello == :world
false
```
事实上，布尔值`true`和`false`就是原子：

```
iex> true == :true
true
iex> is_atom(false)
true
```
## 2.4 字符串

在Elixir中字符串必须用双引号来表达，并且用UTF-8来编码：

```
iex> "hellö"
"hellö"
```

Elixir也支持字符串嵌套：

```
iex> "hellö #{:world}"
"hellö world"
```
结束符可以直接放置在字符串中，也可以用escape sequences：

```
iex> "hello
...> world"
"hello\nworld"
iex> "hello\nworld"
"hello\nworld"
```

你能用`IO`模块中的函数`IO.puts/1`来打印字符串：

```
iex> IO.puts "hello\nworld"
hello
world
:ok
```

注意函数在打印完之后用原子作为返回值。

在Elixir中，字符串在底层是用二进制，也就是字节来实现的：

```
iex> is_binary("hellö")
true
```

我们也能得到一个字符串的字节数：

```
iex> byte_size("hellö")
6
```

请注意在上面的例子中，虽然字符串只有5个字符，但它的字节数是6。这是因为在UTF-8中字符"ö"需要占用两个字节。要得到字符串的实际字符长度，我们可以用函数`String.length/1`：

```
iex> String.length("hellö")
5
```

Elixir标准库中的[字符串模块](http://elixir-lang.org/docs/stable/String.html)包含了一些用来对字符串进行Unicode标准操作的函数：

```
iex> String.upcase("hellö")
"HELLÖ"
```

谨记，在Elixir里_双引号字符串_和_单引号字符串_是不一样的，它们的底层实现的数据类型是不同的：

```
iex> 'hellö' == "hellö"
false
```

我们会在第六章`二进制，字符串和字符列表`中学习到更多关于Unicode支持和单双引号字符串之间的不同之处。

## 2.5 匿名函数

函数用关键值`fn`和`end`来表达：

```
iex> add = fn a, b -> a + b end
#Function<12.71889879/2 in :erl_eval.expr/5>
iex> is_function(add)
true
iex> is_function(add, 2)
true
iex> is_function(add, 1)
false
iex> add.(1, 2)
3
```

在Elixir中函数是“第一等公民”，这意味着它们能被想整数和字符串一样当成参数传给别的函数。在上面的例子里，我们把变量指向的函数`add`传给了另一个函数`is_function/1`，并且返回了正确的结果，`true`。我们也能用`is_function/2`来检查一个函数接受的函数数量。

注意当调用一个匿名函数时，在指向这个匿名函数的变量名和圆括号之间需要有一个点号（`.`）。

匿名函数本身也是一个闭包，因此它们能够访问在被定义时的同视域的其他变量：

```
iex> add_two = fn a -> add.(a, 2) end
#Function<6.71889879/1 in :erl_eval.expr/5>
iex> add_two.(2)
4
```

牢记，在一个函数内部的变量赋值， 并不影响函数外部的环境：

```
iex> x = 42
42
iex> (fn -> x = 0 end).()
0
iex> x
42
```

## 2.6 （链接）列表

Elixir用方括号来表示一个列表，列表内元素的类型是随意的：

```
iex> [1, 2, true, 3]
[1, 2, true, 3]
iex> length [1, 2, 3]
3
```

两个列表可以用函数`++/2 `，`--/2`进行合并或相异操作：

```
iex> [1, 2, 3] ++ [4, 5, 6]
[1, 2, 3, 4, 5, 6]
iex> [1, true, 2, false, 3, true] -- [true, false]
[1, 2, 3, true]
```

贯穿整个教程，我们会不停地谈到列表的头（head）和尾（tail）。头是列表中的第一个元素，尾是剩下的。它们可以被分别用函数`hd/1`和`tl/1`得到。让我们创建一个列表，试着获取头和尾：

```
iex> list = [1,2,3]
iex> hd(list)
1
iex> tl(list)
[2, 3]
```

试图获取一个空列表的头将会导致一个错误：

```
iex> hd []
** (ArgumentError) argument error
```

Oops！

## 2.7 元组（Tuples）

Elixir用花括号来表示元组。和列表一样，元组能包含任何类型的元素：

```
iex> {:ok, "hello"}
{:ok, "hello"}
iex> size {:ok, "hello"}
2
```

元组中的元素在内存中是连续的。也就是说，用索引来访问元组中的元素或者获取元组的大小这样的操作是非畅快的。（元组的索引是从0开始的）：

```
iex> tuple = {:ok, "hello"}
{:ok, "hello"}
iex> elem(tuple, 1)
"hello"
iex> tuple_size(tuple)
2
```

用函数`set_elem/3`可以把一个元素放置在一个特定的索引上：

```
iex> tuple = {:ok, "hello"}
{:ok, "hello"}
iex> set_elem(tuple, 1, "world")
{:ok, "world"}
iex> tuple
{:ok, "hello"}
```

注意`set_elem/3`返回一个新的元组。因为在Elixri中的数据类型都是不可变的，所以变量`tuple`中的那个老的元组并没有变化。不可变量带来的一个好处是，它使得Elixir的代码变得相对简单，因为你不需要担心有一些代码会破坏某处的一个数据。

同时，不可变量也对避免一些常见的问题有帮助，比如并行代码中的race conditions，当两个以上的实体在统一时间试图修改同一个数据结构。

## 2.8 列表还是元组

列表和元组的不同之处在哪里？

在内存中列表是互相链接的形式存在的。这意味着列表中的每一个元素都指向下一个元素，直到到达列表的最后。我们管每一个这样的组对**cons cell**：

```
iex> list = [1|[2|[3|[]]]]
[1, 2, 3]
```

这也意味着获取一个类别的长度是一个线性操作：我们必须贯穿真个列表来弄清楚真个列表的大小。用把新元素插入列表头部的方式，来更新列表是一个比较快的操作：

```
iex> [0] ++ list
[0, 1, 2, 3]
iex> list ++ [4]
[1, 2, 3, 4]
```

在上面的例子中，第一个操作是比较快的，因为我们只是加入了一个新的元组，并且让它指向旧的列表。第二个列子就比较慢了，因为我们必须重建整个列表然后才能加入新元素。

从另一方面来说，元组在内存中是一个整体。这意味着获取它的大小或通过索引来访问元素是非常快的。然而，修改或增减元组中的元素是非常昂贵的操作，因为它首先需要在内存中复制整个元组。

更具这些性能上的特点，我们可以来决定两者的不同用途。一个非常常见的例子是用函数返回一个元组来表示附加的信息。比如，`File.read/1`用来读取一个文件的内同，它返回的就是一个元组：

```
iex> File.read("path/to/existing/file")
{:ok, "... contents ..."}
iex> File.read("path/to/unknown/file")
{:error, :enoent}
```

在Elixir中，当我们计算一个数据结构的大小时，需要遵循一个简单的原则：当这个操作所需的时间是一个恒量（也就是说，这个数值是事先已经计算好了的）的时候，函数应该用`size`来命名，否这的话应该用`length`。

例如，迄今我们遇到过4个此类的函数：`byte_size/1`（获取字符串中的字节数），`tuple_size/1`（获取元组的大小），`length/1`（获取列表的大小），`String.length/1`（获取字符串中的字符数）。可见，当我们用`byte_size/1`去获得字符串中的字节数的时候，这个操作是相当简单的，但当我们希望用`String.length/1`得到其中的unicode字符的数量时是非常昂贵的，因为我们需要历遍整个字符串。

Elixir同时也支持其他的一些数据类型，`Port`，`Reperence`和`PID`，用于进程间的通许。当讲到进程相关的章节的时候，我们会具体谈到它们。现在让我们了解一下和基础数据结构相关的一些基础操作符。
