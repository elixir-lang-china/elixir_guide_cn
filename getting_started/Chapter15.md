# 15 Structs

在早期的几章中，我们已经学习了表单：

```
iex> map = %{a: 1, b: 2}
%{a: 1, b: 2}
iex> map[:a]
1
iex> %{map | a: 3}
%{a: 3, b: 2}
```

Structs是在表单上的扩展，带来了默认值，编译时保证，和Elixir中的多态。

定义一个struct，我们只需要去调用在一个模块内调用`defstruct/1`

```
iex> defmodule User do
...>   defstruct name: "jose", age: 27
...> end
{:module, User,
 <<70, 79, 82, ...>>, {:__struct__, 0}}
```

现在我们用语法`%User{}`来能创建这个stuct的一个“实例”：

```
iex> %User{}
%User{name: "jose", age: 27}
iex> %User{name: "eric"}
%User{name: "eric", age: 27}
iex> is_map(%User{})
true
```

struct提供了编译时的某一个数值存在于struct中的保证：

```
iex> %User{oops: :field}
** (CompileError) iex:3: unknown key :oops for struct User
```

当我们谈论表单是，我们演示了我们才能访问和更新表单中的一个域。这同样适用于structs：

```
iex> jose = %User{}
%User{name: "jose", age: 27}
iex> jose.name
"jose"
iex> eric = %{jose | name: "eric"}
%User{name: "eric", age: 27}
iex> %{user | oops: :field}
** (ArgumentError) argument error
```

有了更新的语法，虚拟机就能直到没有新的键会被加入表单和struct，运行表单在内存中共享它们的结构。在上面的例子中，`jose`和`eric`共享了通过一个内存中的键结构。

Struct也能被用于模式匹配，它们保证了两边的struct都是统一类型的：

```
iex> %User{name: name} = jose
%User{name: "jose", age: 27}
iex> name
"jose"
iex> %User{} = %{}
** (MatchError) no match of right hand side value: %{}
```

匹配之所以能完成，是因为struct在表单内部存储了一个叫`__struct__`的域：

```
iex> jose.__struct__
User
```

总的来说，一个struct只不过是一个带有默认域的普通表单。注意我们说它是普通的表单因为实现的表单的所有协议，没有一个能被用于structs。例如，你不能对一个sturct进行枚举：

```
iex> user = %User{}
%User{name: "jose", age: 27}
iex> user[:name]
** (Protocol.UndefinedError) protocol Access not implemented for %User{age: 27, name: "jose"}
```

struct也不同于字典，所以模块``也对它们无效：

```
iex> Dict.get(%User{}, :name)
** (ArgumentError) unsupported dict: %User{name: "jose", age: 27}
```

我们讲在下一章谈到struct如何同协议互动。
