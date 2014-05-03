# 5 `case`，`cond`和`if`

在这一章中我们将学习`case`，`cond`和`if`的流程控制结构。

## 5.1 `case`

`case`允许我们对很多模式的值进行比较，知道找到匹配的：

```
iex> case {1, 2, 3} do
...>   {4, 5, 6} ->
...>     "This clause won't match"
...>   {1, x, 3} ->
...>     "This clause will match and bind x to 2 in this clause"
...>   _ ->
...>     "This clause would match any value"
...> end
```

如果你想对一个现存的变量进行模式匹配，你需要`^`操作符：

```
iex> x = 1
1
iex> case 10 do
...>   ^x -> "Won't match"
...>   _  -> "Will match"
...> end
```

字句也允许用附加调教作为守护：

```
iex> case {1, 2, 3} do
...>   {1, x, 3} when x > 0 ->
...>     "Will match"
...>   _ ->
...>     "Won't match"
...> end
```

在上面的例子中，第一个字句只有当`x`的值为正的时候才匹配。Erlang虚拟机只允许有限的一些表达式作为守护：

* 比较操作符(`==`, `!=`, `===`, `!==`, `>`, `<`, `<=`, `>=`)
* 布尔值操作符(`and`, `or`) and negation operators (`not`, `!`)
* 运算操作符 (`+`, `-`, `*`, `/`)
* 只要左侧是一个literal的时候都能用`<>` 和 `++`
* `in` 操作符
* 下面的这些类型检查hanshu：

  * `is_atom/1`
  * `is_binary/1`
  * `is_bitstring/1`
  * `is_boolean/1`
  * `is_float/1`
  * `is_function/1`
  * `is_function/2`
  * `is_integer/1`
  * `is_list/1`
  * `is_map/1`
  * `is_number/1`
  * `is_pid/1`
  * `is_port/1`
  * `is_reference/1`
  * `is_tuple/1`
* 外加这些函数：

  * `abs(number)`
  * `bit_size(bitstring)`
  * `byte_size(bitstring)`
  * `div(integer, integer)`
  * `elem(tuple, n)`
  * `hd(list)`
  * `length(list)`
  * `map_size(map)`
  * `node()`
  * `node(pid | ref | port)`
  * `rem(integer, integer)`
  * `round(number)`
  * `self()`
  * `size(tuple | bitstring)`
  * `tl(list)`
  * `trunc(number)`
  * `tuple_size(tuple)`

谨记，守护函数中的错误是不可见的，它只能让守护函数失效：

```
iex> hd(1)
** (ArgumentError) argument error
    :erlang.hd(1)
iex> case 1 do
...>   x when hd(x) -> "Won't match"
...>   x -> "Got: #{x}"
...> end
"Got 1"
```

如果没有任何一个子句匹配，会导致一个错误：

```
iex> case :ok do
...>   :error -> "Won't match"
...> end
** (CaseClauseError) no case clause matching: :ok
```

注意匿名函数也能用好几个子句和守护：

```
iex> f = fn
...>   x, y when x > 0 -> x + y
...>   x, y -> x * y
...> end
#Function<12.71889879/2 in :erl_eval.expr/5>
iex> f.(1, 3)
4
iex> f.(-1, 3)
-3
```

每个匿名函数的子句所接受的参数量必须相同，不然也会报错。

## 5.2 `cond`

当你需要匹配不同的值的时候`case`会非常有用。然而，在很多情况下，我们要检查不同的状况并找出第一个为真的。在这种情况下，可以考虑使用`cond`：

```
iex> cond do
...>   2 + 2 == 5 ->
...>     "This will not be true"
...>   2 * 2 == 3 ->
...>     "Nor this"
...>   1 + 1 == 2 ->
...>     "But this will"
...> end
"But this will"
```

它等价于很多其他语言中的`else if`子句，Elixir也有`if`和`else`不过用的较少。

如果所有的条件都不为真，会导致一个错误。因此，有必要把最后一个条件设置为`true`，这样就总能匹配了：

```
iex> cond do
...>   2 + 2 == 5 ->
...>     "This is never true"
...>   2 * 2 == 3 ->
...>     "Nor this"
...>   true ->
...>     "This is always true (equivalent to else)"
...> end
```

最后，请注意在`cond`眼里，除了`nil`和`false`之外的所有值都为真：

```
iex> cond do
...>   hd([1,2,3]) ->
...>     "1 is considered as true"
...> end
"1 is considered as true"
```

# 5.3 `if`和`unless`

除了`case`和`cond`，当你值需要检查一个条件的时候，Elixir也提供了两个有用的宏`if/2`和`unless/2`：

```
iex> if true do
...>   "This works!"
...> end
"This works!"
iex> unless true do
...>   "This will never be seen"
...> end
nil
```

在上面的例子中，如果提供给`if/2`发挥`false`或者`nil`，`do/end`之间的代码就不执行，只返回`nil`。反之就是`unless/2`的情况。

它们也支持`else`：

```
iex> if nil do
...>   "This won't be seen"
...> else
...>   "This will"
...> end
"This will"
```
> `if/2`和`unless/2`有趣的地方在于它们在语言中是用宏的方式实现的，和在其他语言中一样它们是语言的特殊构件。你可以在[kernel模块文档](http://elixir-lang.org/docs/stable/Kernel.html)里查阅`if/2`的文档和代码。类似`+/2`这样的操作符和`is_function/2`之类的函数都位于`Kernel`模块，当程序启动的时候它们会自动被载入。


# 5.4 `do` 块

到目前为止，我们已经学习了四种控制结构：`case`，`cond`，`if`和`unless`，并且它们都被`do`/`end`块所包含。其实`if`也能写成这样：

```
iex> if true, do: 1 + 2
3
```

在Elixir中，`do`/`end`块是一种把一堆表带式传给`do:`的方便表示。下面两个是等价的：

```
iex> if true do
...>   a = 1 + 2
...>   a + 10
...> end
13
iex> if true, do: (
...>   a = 1 + 2
...>   a + 10
...> )
13
```

我们说第二中语法用了**关键字列表**。`else`也可以用同样的表达：

```
iex> if false, do: :this, else: :that
:that
```

牢记一个细节非常重要，当我们用`do`/`end`块，它们总是和最远的那个函数调用绑定。例如，下面的表达式：

```
iex> is_number if true do
...>  1 + 2
...> end
```

会被解析成：

```
iex> is_number(if true) do
...>  1 + 2
...> end
```

这会导致Elixir试图去调用函数`if/1`， 当然会抛出一个未定义函数错误。加上圆括号可以避免这样的歧义：

```
iex> is_number(if true do
...>  1 + 2
...> end)
true
```

关键字列表在这个语言中扮演了一个重要的角色，在很多的函数和宏中都广泛地应用。我们将未来的章节中对它们进行进一步的的探索。现在是时候谈谈关于“二进制，字符串和字符列表”了。
