# 3 基础操作符

在之前的几章中，我们了解了Elixir提供了`+`，`-`，`*`，`/`作为基本的运算操作符，外加函数`div/2`和`rem/2`用来做整数的除法和余数运算。

Elixir也提供了`++`和`--`用以修改列表：

```
iex> [1,2,3] ++ [4,5,6]
[1,2,3,4,5,6]
iex> [1,2,3] -- [2]
[1,3]
```

字符串连接操作符`<>`：

```
iex> "foo" <> "bar"
"foobar"
```

Elixir也提供个是那个布尔值操作符`or`，`and`和`not`。这些操作符严格地指定它们一个参数必须是一个布尔值（`true`或者`false`）：

```
iex> true and true
true
iex> false or is_atom(:example)
true
```

非布尔值会导致一个意外：

```
iex> 1 and true
** (ArgumentError) argument error
```
`or`和`and`是一对快捷操作符。只有当它们左侧的表达式不足以满足条件的时候才会执行右侧的表达式：

```
iex> false and error("This error will never be raised")
false

iex> true or error("This error will never be raised")
true
```

> 注意：如果你是Erlang的开发者，Elixir中的`and`和`or`实际上映射到Erlang中的`andalso`和`orelse`操作符。

除此之外，Elixir也提供了`||`，`&&`和`!`，这些操作符接受任何类型的参数。对于这些操作，除了`false`和`nil`之外的所有值都为真：

```
# or
iex> 1 || true
1
iex> false || 11
11

# and
iex> nil && 13
nil
iex> true && 17
17

# !
iex> !true
false
iex> !1
false
iex> !nil
true
```

简单来说，当你预期参数是布尔值的时候，用`and`，`or`和`not`。如果你不确定，那就使用`||`，`&&`和`!`。

Elixir也提供了比较操作符`==`，`!=`，`===`，`!==`，`<=`，`>=`，`>`和`<`：

```
iex> 1 == 1
true
iex> 1 != 2
true
iex> 1 < 2
true
```

`==`和`===`之间的不同在于，后者在比较整数和浮点数的时候更加严格：

```
iex> 1 == 1.0
true
iex> 1 === 1.0
false
```

Elixir允许在不同类型之间进行比较：

```
iex> 1 < :atom
true
```

我们之所以可以在Elixir在不同数据类型之间比较是因为有一个内建的不同类型之间的顺序，所以算法在排序时无需担心数据类型上  的不同。下面就是内建的类型之间的顺序：

```
number < atom < reference < functions < port < pid < tuple < maps < list < bitstring
```

你并不需要去记忆这些顺序，重要的是了解的确有这么一个东西。

好了，简单介绍到这里。在下一章中，我们将讨论函数的基础，数据类型之间的转换和一点流程控制。
