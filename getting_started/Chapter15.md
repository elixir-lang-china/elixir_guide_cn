# 15 结构体 Struct

在第七章中，我们已经学习了表单：

```
iex> map = %{a: 1, b: 2}
%{a: 1, b: 2}
iex> map[:a]
1
iex> %{map | a: 3}
%{a: 3, b: 2}
```

结构体是基于表单的扩展，提供了编译时的检查和默认值的功能。

## 定义结构体

使用 `defstruct` 定义一个结构体：

```
iex> defmodule User do
...>   defstruct name: "John", age: 27
...> end
```

以上使用 `defstruct` 的代码定义了结构体的字段以及属于各自的默认值。

结构体根据他们所定义模块的名字作区分。在上面的例子，我们定义了一个名叫 User 的结构体

现在我们用一个类似于创建表单的语法来创建一个 User 结构体实例：

```
iex> %User{}
%User{age: 27, name: "John"}
iex> %User{name: "Meg"}
%User{age: 27, name: "Meg"}
```

在编译阶段，结构体保证了只有通过 `defstruct` 定义的字段才被允许存在于一个结构体实例中：

```
iex> %User{oops: :field}
** (CompileError) iex:3: unknown key :oops for struct User
```

## 访问和更新结构体

当我们谈论表单时，我们演示了怎样才能访问和更新表单中的属性。这同样适用于结构体：

```
iex> john = %User{}
%User{age: 27, name: "John"}
iex> john.name
"John"
iex> meg = %{john | name: "Meg"}
%User{age: 27, name: "Meg"}
iex> %{meg | oops: :field}
** (ArgumentError) argument error
```

当使用更新语法（`|`），虚拟机就知道不会有新的健会被加到结构体中，允许表单内部在内存中共享它们的结构。在上面的例子里，`john` 和 `meg` 在内存中共享相同的键结构。

结构体也能被用于模式匹配，它们保证了两边的结构体的键值都是统一类型的：

```
iex> %User{name: name} = john
%User{age: 27, name: "John"}
iex> name
"John"
iex> %User{} = %{}
** (MatchError) no match of right hand side value: %{}
```

## 结构体只是一个空表单

在上面的例子，模式匹配之所以有效是因为在结构体内部是一个有许多修改过属性的空表单。作为表单，结构体 存储了一个特殊的属性，名为 `__struct__`。它维持着这个结构体的名字。

```
iex> is_map(john)
true
iex> john.__struct__
User
```

注意我们把结构体当做一个空表单，是因为结构体并没有实现表单的协议。比如，你不能枚举，也不能直接访问一个结构体。

```
iex> john = %User{}
%User{age: 27, name: "John"}
iex> john[:name]
** (Protocol.UndefinedError) protocol Access not implemented for %User{age: 27, name: "John"}
iex> Enum.each john, fn({field, value}) -> IO.puts(value) end
** (Protocol.UndefinedError) protocol Enumerable not implemented for %User{age: 27, name: "John"}
```

结构体也不是一个字典，所以 `Dict` 模块里的函数也不能使用：

```
iex> Dict.get(%User{}, :name)
** (UndefinedFunctionError) undefined function: User.fetch/2
```

然而，由于结构体是个表单, 它们可以使用 `Map` 模块中的函数：

```
iex> kurt = Map.put(%User{}, :name, "Kurt")
%User{age: 27, name: "Kurt"}
iex> Map.merge(kurt, %User{name: "Takashi"})
%User{age: 27, name: "Takashi"}
iex> Map.keys(john)
[:__struct__, :age, :name]
```

我们讲在下一章谈到结构体如何同协议交互。
