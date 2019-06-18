# 更好的错误报告

我们只能接受这样一个事实：错误会发生。与许多其他语言不同的是，在使用rust时，很难不注意和处理这个事实：因为它没有异常，所有可能的错误状态通常都编码在函数的返回类型中。

## 结果

像这样的功能[`read_to_string`]不返回字符串。相反，它返回[`Result`]其中包含`String`或某种类型的错误（在本例中[`std::io::Error`]）

[`read_to_string`]: https://doc.rust-lang.org/1.31.0/std/fs/fn.read_to_string.html

[`result`]: https://doc.rust-lang.org/1.31.0/std/result/index.html

[`std::io::error`]: https://doc.rust-lang.org/1.31.0/std/io/type.Result.html

你怎么知道它是什么？自从`Result`是一个`enum`，您可以使用`match`要检查它是哪种变体：

```rust,no_run
let result = std::fs::read_to_string("test.txt");
match result {
    Ok(content) => { println!("File content: {}", content); }
    Err(error) => { println!("Oh noes: {}", error); }
}
```

<aside>

**旁白：**不知道什么是Enum，也不知道它们是如何生锈的？[查看铁锈书的这一章](https://doc.rust-lang.org/1.31.0/book/ch06-00-enums.html)以跟上速度。

</aside>

## 展开

现在，我们可以访问文件的内容，但是在`match`阻止。为此，我们需要以某种方式处理错误案例。挑战在于`match`块需要返回相同类型的内容。但有一个巧妙的方法可以解决这个问题：

```rust,no_run
let result = std::fs::read_to_string("test.txt");
let content = match result {
    Ok(content) => { content },
    Err(error) => { panic!("Can't deal with {}, just exit here", error); }
};
println!("file content: {}", content);
```

我们可以在`content`在比赛区之后。如果`result`如果是错误，字符串将不存在。但因为程序在到达使用点之前就退出了`content`很好。

这可能看起来很激烈，但很方便。如果您的程序需要读取该文件，并且如果该文件不存在，则无法执行任何操作，那么退出是一种有效的策略。甚至还有一个快捷方式`Result`S，呼叫`unwrap`：

```rust,no_run
let content = std::fs::read_to_string("test.txt").unwrap();
```

## 不必惊慌

当然，中止程序并不是处理错误的唯一方法。而不是`panic!`我们也可以很容易地写`return`：

```rust,no_run
# fn main() -> Result<(), Box<std::error::Error>> {
let result = std::fs::read_to_string("test.txt");
let _content = match result {
    Ok(content) => { content },
    Err(error) => { return Err(error.into()); }
};
# Ok(())
# }
```

但是，这会改变函数所需的返回类型。实际上，在我们的示例中一直隐藏着一些东西：这个代码所在的函数签名。在最后一个例子中，`return`它变得很重要。这是*满的*例子：

```rust,no_run
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let result = std::fs::read_to_string("test.txt");
    let content = match result {
        Ok(content) => { content },
        Err(error) => { return Err(error.into()); }
    };
    println!("file content: {}", content);
    Ok(())
}
```

我们的退货类型是A`Result`！这就是为什么我们可以写`return Err(error);`在第二个比赛臂。看看有什么`Ok(())`在底部？它是函数的默认返回值，表示“结果正常，没有内容”。

<aside>

**旁白：**为什么这不是写为`return Ok(());`？很容易——这也是完全有效的。锈块的最后一个表达是它的返回值，并且习惯上省略不必要的`return`s.

</aside>

## 问号

就像打电话一样`.unwrap()`是`match`具有`panic!`在错误臂中，我们有另一个`match`那个`return`S在错误臂中：`?`.

没错，问号。可以将此运算符附加到类型的值`Result`生锈会在内部扩展到类似于`match`我们刚刚写信。

试一试：

```rust,no_run
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let content = std::fs::read_to_string("test.txt")?;
    println!("file content: {}", content);
    Ok(())
}
```

非常简洁！

<aside>

**旁白：**这里还发生了一些不需要理解的事情。例如，我们的`main`函数是`Box<dyn std::error::Error>`. 但我们已经看到了`read_to_string`返回[`std::io::Error`]. 这是因为`?`扩展到代码*转换*错误类型。

`Box<dyn std::error::Error>`也是一种有趣的类型。这是一个`Box`可以包含*任何*实现标准的类型[`Error`][`std::error::error`]特质。这意味着基本上所有的错误都可以放入这个框中，所以我们可以使用`?`返回的所有常规函数`Result`s.

[`std::error::error`]: https://doc.rust-lang.org/1.31.0/std/error/trait.Error.html

</aside>

## 提供上下文

使用时出现的错误`?`在你的`main`功能正常，但不太好。例如：当你跑步时`std::fs::read_to_string("test.txt")?`但是文件`test.txt`不存在，您将得到以下输出：

> 错误：OS代码：2，种类：未找到，消息：“无此类文件或目录”

如果代码中没有包含文件名，就很难判断哪个文件是`NotFound`. 有多种方法可以解决这个问题。

例如，我们可以创建自己的错误类型，然后使用它来构建自定义错误消息：

```rust,ignore
{{#include errors-custom.rs}}
```

现在，运行此命令，我们将收到自定义错误消息：

> 错误：customError（“错误读取`test.txt`：没有此类文件或目录（操作系统错误2）“）

不是很漂亮，但稍后我们可以很容易地为我们的类型调整调试输出。

这种模式实际上很常见。但它有一个问题：我们不存储原始错误，只存储它的字符串表示。常用的[`failure`]图书馆有一个很好的解决方案：类似于我们的`CustomError`类型，它有一个[`Context`]包含说明和原始错误的类型。图书馆也带来了一个扩展特性。（[`ResultExt`]）加上[`context()`]和[`with_context()`]方法`Result`.

[`failure`]: https://docs.rs/failure

[`context`]: https://docs.rs/failure/0.1.3/failure/struct.Context.html

[`resultext`]: https://docs.rs/failure/0.1.3/failure/trait.ResultExt.html

[`context()`]: https://docs.rs/failure/0.1.3/failure/trait.ResultExt.html#tymethod.context

[`with_context()`]: https://docs.rs/failure/0.1.3/failure/trait.ResultExt.html#tymethod.with_context

为了将这些打包的错误类型转换为人类真正想要读取的内容，我们可以进一步添加[`exitfailure`]板条箱，并使用其类型作为我们的返回类型`main`功能。

让我们先通过添加`failure = "0.1.5"`和`exitfailure = "0.5.1"`到`[dependencies]`我们的部分`Cargo.toml`文件。

完整的示例如下：

[`exitfailure`]: https://docs.rs/exitfailure

```rust,ignore
{{#include errors-exit.rs}}
```

这将打印一个错误：

> 错误：无法读取文件`test.txt`\
> 信息：由于没有这样的文件或目录（OS错误2）
