# 1 简介

ExUnit是Elixir自带的单元测试框架。

ExUnit很容易使用，下面是一个最简单的例子：

```
ExUnit.start

defmodule MyTest do
  use ExUnit.Case

  test "the truth" do
    assert true
  end
end
```

总的来说，我们仅仅需要调用`ExUnit.start`，用`ExUnit.Case`定义一个测试实例，以及具体的测试。大部分的测试都作为[Mix](http://elixir-lang.org/getting_started/mix/1.html)项目的一部分：

```
mix test
```

不然的话，假设你把这个文件存成一个文件`assertion_test.exs`，我们可以直接运行：

```
bin/elixir assertion_test.exs
```

在这一章，我们将讨论ExUnit中最常用的特性和如何进一步定制它。

# 1.1 开始ExUnit

ExUnit通常以`ExUnit.start`开头。这个函数接受几个选项，这个[文档](http://elixir-lang.org/docs/stable/ExUnit.html)里有更详细的内容。现在，我们只点出几个最常用的：

* `:formatter` - 当你运行测试的时候，所有的IO都是由[formatter](https://github.com/elixir-lang/elixir/blob/master/lib/ex_unit/lib/ex_unit/formatter.ex)完成的。开发者能定义它们自己的formatter， 用这个配置能让ExUnit知道去使用定制化的formatter。

* `:max_cases` - 我们很快就将看到，ExUnit能让你非常容易地并行运行测试。这可以加快你的测试进度，而且没有副作用。这个选项运行你配置ExUnit最多能同时运行的测试的个数。

# 1.2 定义一个测例

一旦ExUnit开始了，我们能定义我们自己的测例。这是由我们的模块中的`ExUnit.Case`完成的：

```
use ExUnit.Case
```

`ExUnit.Case`提供了几个函数，让我们来探查一番。

## 1.2.1 宏`test`

`ExUnit.Case`运行之中所有名字以`test`开始，并且接受单个参数的的函数：

```
def test_the_truth(_) do
  assert true
end
```

作为一种定义上面那样的测例的简便方式，`ExUnit.Case`提供了一个宏`test`，可以写成这样：

```
test "the truth" do
  assert true
end
```

这种结构可读性更高。宏`test`接受二进制或原子作为名字。


## 1.2.2 断言

另一个由`ExUnit.Case`提供的简便方式，是通过[`ExUnit.Assertions`](http://elixir-lang.org/docs/stable/ExUnit.Assertions.html)来自动引入一堆断言宏和函数。

在大部分的测试中，你只需要用到的是断言宏是`assert`和`refute`：

```
assert 1 + 1 == 2
refute 1 + 3 == 3
```

如果测试失败了，ExUnit会自动分解这些表达式，试图提供尽可能详细的信息。例如，这个失败的断言：

```
assert 1 + 1 == 3
```

以这种形式：

```
Expected 2 to be equal to (==) 3
```

然而，在一些特殊的例子中，一些另外的断言能让测试变得更容易。一个最好的例子是宏`assert_raise`：

```
assert_raise ArithmeticError, "bad argument in arithmetic expression", fn ->
  1 + "test"
end
```

所以别忘了查看[ExUnix.Assertions](http://elixir-lang.org/docs/stable/ExUnit.Assertions.html)的文档中的其他例子。

## 1.2.3 回调函数

`ExUnit.Case`定义了四种回调函数：`setup`， `teardown`， `setup_all`和`teardown_all`：

```
defmodule CallbacksTest do
  use ExUnit.Case, async: true

  setup do
    IO.puts "This is a setup callback"
    :ok
  end

  test "the truth" do
    assert true
  end
end
```

在上面的例子中，回调`setup`将会在每次测试前运行。如果定义了回调`setup_all`，它将在所有的测试之前运行一次。

一个回调必须返回`:ok`或`{ :ok, data }`。当返回的是后者的时候，`data`参数必须是一个包含测试元数据的关键字列表。这个元数据可以被其他回调函数或者测试访问到。

```
defmodule CallbacksTest do
  use ExUnit.Case, async: true

  setup do
    IO.puts "This is a setup callback"
    { :ok, from_setup: :hello }
  end

  test "the truth", meta do
    assert meta[:from_setup] == :hello
  end

  teardown meta do
    assert meta[:from_setup] == :hello
    :ok
  end
end
```

元数据用于需要手动地将状态传递个测试的时候。

## 1.2.4 异步

最后，``也运行测例并行运行。你只需要把`:async`选项的值设为`true`：

```
use ExUnit.Case, async: true
```

这将会把这个测试同其他同样是异步的测试，并行运行。其他的测试仍将以顺序的方式运行。

## 2 干吧

ExUnit仍然是一个为完工的项目。我们鼓励你访问我们的[issue tracker](https://github.com/elixir-lang/elixir/issues)，提交你希望在ExUnit中看到的新特性，
