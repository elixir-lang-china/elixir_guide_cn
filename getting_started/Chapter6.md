# 二进制，字符串和字符列表

在“基础数据类型”一章，我们学习了关于字符串以及如何用函数`is_binary/1`作检查：

```
iex> string = "hello"
"hello"
iex> is_binary string
true
```

在这章，我们将理解什么是二进制，它们和字符串的关系以及一个单引号的值，比如`like this`在Elixi中的含义。

## 6.1 UTF-8和Unicode

字符串是用UTF-8编码的二进制。如果要彻底地理解这句话的意思，我们需要理解字节和codepoint之间的不同。

Unicode给每个我们熟知的字符都赋予了一个codepoint。例如，字母`"a"`的codepoint是`97`而字母`"ł"`的codepoint是322.当把字符串`"hełło"`写道磁盘上的时候，我们需要把这些codepoint转换为字节。我们假定一个字节对应一个codepoint，那我们就没法写`"hełło"`了，因为`"ł"`的codepoint是`322`,而一个字节只能表`0`到`255`.不过当然，既然你可以读取字符串`"hełło"`在你的屏幕上，它一定用了某种方法。这种方法就是编码（encoding）。

当用字节代表codepoint，我们需要用某种方法将之编码。Elixir选择UTF-8编码作为自己的默认编码。当我们说字符串是用UTF-8的字符串，是指字符串是按照UTF-8标准将字节组织起来用以表示codepoing的。

既然我们有字符`"ł"`这样的`322`的codepoint，我们需要一个以上的字节来表示。这也是为什么我们看到计算字符串的字节大小和字符的长度的结果式不同的：

```
iex> string = "hełło"
"hełło"
iex> byte_size string
7
iex> String.length string
5
```

UTF-8能用一个字节表示codepoint`h`，`e`和`o`，但两个字节才能表示`ł`。在Elixir里，有可以用`?`得到字符的codepoint：

```
iex> ?a
97
iex> ?ł
322
```

你也可以用[字符串模块](http://elixir-lang.org/docs/stable/String.htm)l中的函数把字符串分解成codepoint：

```
iex> String.codepoints("hełło")
["h", "e", "ł", "ł", "o"]
```

你将会看到Elixir对字符串有着极好的支持。它也支持许多的Unicode操作。事实上，Elixir通过了[“The string type is broken”](http://mortoray.com/2013/11/27/the-string-type-is-broken/)中的所有测试。

然而，字符串仅仅式故事的一部分。如果字符串是一串二进制，而且我们可以用函数``，那Elixir必定在底层有某种数据类型在支撑字符串。事实就是如此，让我们谈谈二进制吧！

## 6.2 二进制（和比特串）

在Elixir里，你可以用`<<>>`定义二进制：

```
iex> <<0, 1, 2, 3>>
<<0, 1, 2, 3>>
iex> byte_size <<0, 1, 2, 3>>
4
```

一个二进制仅仅式一个字节序列。当然，这些字节可以用任何方式解释，甚至式非法的字符串：

```
iex> String.valid?(<<239, 191, 191>>)
false
```

字符串合并操作实际上是一个二进制合并操作符：

```
iex> <<0, 1>> <> <<2, 3>>
<<0, 1, 2, 3>>
```

一个Elixir中常用的窍门就是用空二进制`<<0>>`同一个字符串合并来获取字符串的内部表示：

```
iex> "hełło" <> <<0>>
<<104, 101, 197, 130, 197, 130, 111, 0>>
```

二进制中的每个数字代表一个比特，所以最大值只能是255。但同时二进制也可以通过添加新修改符（modifier）来表示大于255的数字或者把一个codepoint转换成对应的UTF-8表示：

```
iex> <<255>>
<<255>>
iex> <<256>> # truncated
<<0>>
iex> <<256 :: size(16)>> # use 16 bits (2 bytes) to store the number
<<1, 0>>
iex> <<256 :: utf8>> # the number is a codepoint
"Ā"
iex> <<256 :: utf8, 0>>
<<196, 128, 0>>
```

既然一个字节有8比特，如果我们把大小设为1个比特，会怎么样？

```
iex> <<1 :: size(1)>>
<<1 :: size(1)>>
iex> <<2 :: size(1)>> # truncated
<<0>>
iex> is_binary(<< 1 :: size(1)>>)
false
iex> is_bitstring(<< 1 :: size(1)>>)
true
iex> bit_size(<< 1 :: size(1)>>)
1
```

这样的话它的值就不再是二进制，而是一个比特串（bitstring）了 -- 只不过式一堆的比特。所以二进制只不过是长度除以8的比特串而已！

我们也可以对二进制和比特串进行模式匹配：

```
iex> <<0, 1, x>> = <<0, 1, 2>>
<<0, 1, 2>>
iex> x
2
iex> <<0, 1, x>> = <<0, 1, 2, 3>>
** (MatchError) no match of right hand side value: <<0, 1, 2, 3>>
```

注意上面的每一个二进制都要求匹配正好8比特。然而，我们也可以用修改符来匹配剩余的二进制：

```
iex> <<0, 1, x :: binary>> = <<0, 1, 2, 3>>
<<0, 1, 2, 3>>
iex> x
<<2, 3>>
```

上面的模式仅仅在二进制在`<<>>`尾部的情况下才有效。相似的情况也出现在字符串合并操作符`<>`上：

```
iex> "he" <> rest = "hello"
"hello"
iex> rest
"llo"
```

到这里，关于比特串，二进制和字符串就结束了。一个字符串是UTF-8编码的二进制，二进制是长度除以8的比特串。虽然这显示了Elixir在如何处理bite和字节上的灵活性，但在99%的情况下你只会和二进制， 以及函数`is_binary/1`和`byte_size/1`打交道。

## 6.3 字符列表

一个字符列表只不过是一个字符的列表：

```
iex> 'hełło'
[104, 101, 322, 322, 111]
iex> is_list 'hełło'
```

你可以看到，它没有用字节而是用了包含单引号之间的字符的codepoint的列表。所以，双引号表示的是字符串（二进制），单引号表示的式字符列表（列表）。

在实际中，字符列表最常见的引用是在和Erlang代码的时候，特别式一些比较老的库没法用接受二进制的参数。你能用函数`to_string/1`和`to_char_list/1`，在字符列表和二进制之间互相转换。

```
iex> to_char_list "hełło"
[104, 101, 322, 322, 111]
iex> to_string 'hełło'
"hełło"
iex> to_string :hello
"hello"
iex> to_string 1
"1"
```

注意这些函数式多态的。它们不仅仅能把字符列表转成字符串，而且也能转整数和原子等其他类型。

有了二进制，字符串和字符列表，是时候谈一下键-值对数据结构了。
