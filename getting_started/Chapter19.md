# 19 Sigils

我们已经学习到了Elixir提供了双引号的字符串和单引号的字符列表。然而，这只是这个语言中那些具有文字表现形式的结构的表层现象。比如，原子，是另一种常常通过``形式创建的结构。

Elixir的其中一个目标是可扩展性：开发者应该能够在某个领域扩展语言。计算机科学已经变得如此宽广的范围，以至于已经不可能仅仅用一种语言的核心就能解决所有的问题。我们最佳的赌注是使得语言可扩展，那样开发者，公司或者社区能在相关的领域内扩展语言。

在这一章，我们将探索sigil，它是语言层面提供的能用于字面呈现的机制之一。

# 19.1 正则表达式

Sigils由一个约等于号（`~`）开始，跟着一个字母，再跟着是一个分隔符。最常见的sigil是`~r`，用于[正则表达式](https://en.wikipedia.org/wiki/Regular_Expressions)：

```
# A regular expression that returns true if the text has foo or bar
iex> regex = ~r/foo|bar/
~r/foo|bar/
iex> "foo" =~ regex
true
iex> "bat" =~ regex
false
```

Elixir提供了Perl兼容的正则表达式（regrexs），类似于在[PCRE](http://www.pcre.org/)库中实现的。正则也支持修饰符。例如，修饰符`i`会使得一个正则表达式无关大小写：

```
iex> "HELLO" =~ ~r/hello/
false
iex> "HELLO" =~ ~r/hello/i
true
```

在[`Regex`](http://elixir-lang.org/docs/stable/Regex.html)模块的文档中有更详细的有关正则表达式的其他修饰符和所支持的其他操作的内容。

到目前为止，所有的例子都使用了`/`来分割正则表达式。然而sigil支持8中不同的分隔符：

```
~r/hello/
~r|hello|
~r"hello"
~r'hello'
~r(hello)
~r[hello]
~r{hello}
~r<hello>
```

只所以支持这么多不同的操作符是因为不同的分隔符能更方便地在不同的sigils中使用。例如，在正则中使用圆括号也许就不合适，因为这会和正则内部的圆括号混淆。然而， 圆括号在其他的sigil中就非常适用，我们将在下面看到例证。

## 19.2 字符串，字符列表和其他sigils

除了正则，Elixir默认提供了其他三中sigils。

`~s`sigils被用于产生字符串，同双引号类似。

```
iex> ~s(this is a string with "quotes")
"this is a string with \"quotes\""
```

而`~c`sigil用于产生字符列表：

```
iex> ~c(this is a string with "quotes")
'this is a string with "quotes"'
```

`~w`sigil用于产生一个用空格分割的词汇的列表：

```
iex> ~w(foo bar bat)
["foo", "bar", "bat"]
```

`~w`sigil也能接受修饰符`c`，`s`和`a`，来选择结果的格式：

```
iex> ~w(foo bar bat)a
[:foo, :bar, :bat]
```

除了小写字母的sigils， Elixir也支持大写形式的sigils。虽然`~s`和`~S`都返回字符串，前者能允许代码忽略和融合，而后者不能：


```
iex> ~s(String with escape codes \x26 interpolation)
"String with escape codes & interpolation"
iex> ~S(String without escape codes and without #{interpolation})
"String without escape codes and without \#{interpolation}"
```

下面这些忽略符号能被用于字符串和字符列表：
* `\"` - 双引号
* `\''` - 单引号
* `\\` - 单斜杠
* `\a` - bell/alert
* `\b` - backspace
* `\d` - 删除
* `\e` - 忽略
* `\f` - form feed
* `\n` - 换行符
* `\r` - 回车
* `\s` - 空格
* `\t` - tab
* `\v` - vertial tab
* `\DDD`，`\DD`，`\D` - 用八进制表示的字符 DDD，DD或D（例如`\377`）
* `\xDD` - 用十六进制表示的字符 DD， 例如（`\x37`）
* `\x{D...}` - 用十六禁止表示的字符，至少含有一个十六进制字符（例如， `/x{abc123}`）

Sigil也支持heredocs，用三双引号或三单引号来作为分隔符：

```
iex> ~s"""
...> this is
...> a heredoc string
...> """
```

最常见的heredocs sigil是用于编写文档。例如，如果你需要在你的文档中包含忽略字符，会很麻烦，因为它需要对某些字符做双重忽略：

```
@doc """
Converts double-quotes to single-quotes.

## Examples

    iex> convert("\\\"foo\\\"")
    "'foo'"

"""
def convert(...)
```

但有了`-S`， 我们能彻底避免这个问题：

```
@doc ~S"""
Converts double-quotes to single-quotes.

## Examples

    iex> convert("\"foo\"")
    "'foo'"

"""
def convert(...)
```

# 19.3 定制sigil

正如在开头暗示的那样，sigil是Elixir中可扩展的。实际上，sigil`sigil_r`相当于用两个参数调用函数`sigil_r`：

```
iex> sigil_r(<<"foo">>, 'i')
~r"foo"i
```

这就是说，我们通过函数`sigil_r`来得到sigil`~r`的文档：


```
iex> h sigil_r
...
```

我们也能通过实现何时的函数，我们也能实现自己的sigil。例如，让我们实现sigil`~i(13)`来回返一个整数：

```
iex> defmodule MySigils do
...>   def sigil_i(binary, []), do: binary_to_integer(binary)
...> end
iex> import MySigils
iex> ~i(13)
13
```

sigil也能在宏的帮助下，在编译时发挥作用。例如，在Elixir中正则表达式在编译时会被编译成高效的代码形式，随后就无需在运行时做这些事情了。如果你对这个主题感兴趣，我们推荐你学习宏并且查阅这些sigils是如何在`Kernel`模块中实现的。
