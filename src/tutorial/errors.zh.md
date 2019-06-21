# 更好的错误报告

我们只能接受这样一个事实：错误会发生。与许多其他语言不同的是，在使用 rust 时，很难不注意和不处理这个事实：因为无一例外，所有可能的错误状态，通常都编码在函数的返回类型中。

## Result

像[`read_to_string`]这样的函数，是不返回字符串的。相反，它返回[`Result`]，其中(一个是)包含`String`或(另一种是)某类型的错误（在本例子是[`std::io::Error`]）

[`read_to_string`]: https://doc.rust-lang.org/1.31.0/std/fs/fn.read_to_string.html
[`result`]: https://doc.rust-lang.org/1.31.0/std/result/index.html
[`std::io::error`]: https://doc.rust-lang.org/1.31.0/std/io/type.Result.html

你怎么知道它是什么？因为啊，`Result`(其实)是一个`enum`(枚举)，您可以使用`match`，检查它是哪种变体：

```rust,no_run
let result = std::fs::read_to_string("test.txt");
match result {
    Ok(content) => { println!("File content: {}", content); }
    Err(error) => { println!("Oh noes: {}", error); }
}
```

<aside>

**旁白：** 不知道 Rust 的 enum 是什么，也不知道它们是如何工作？[查看 Rust 之书的有关章节](https://doc.rust-lang.org/1.31.0/book/ch06-00-enums.html)，跟上，跟上。

</aside>

## 展开(Unwrap)

现在，我们可以访问文件的内容，但在`match`区块之后我们不能肯定(它的返回类型)。为此，我们需要以某种方式处理错误案例。这里的挑战在于`match`块的所有条件语句(或是臂)，需要返回相同类型的内容。但有一个巧妙的方法可以解决这个问题：

> 译：Rust 常把 match 的条件语句，说成 手臂(arm)，看起来还挺像的。

```rust,no_run
let result = std::fs::read_to_string("test.txt");
let content = match result {
    Ok(content) => { content },
    Err(error) => { panic!("Can't deal with {}, just exit here", error); }
};
println!("file content: {}", content); // 使用 content
```

我们可以在`match`区块之后，使用`content`的字符串(String 类型)。如果`result`是错误（Err），则字符串将不存在。但因为程序在到达使用`content`点之前，就退出了，所以不会有问题。

这可能看起来很刚烈，但很方便。如果您的程序需要读取该文件，并且如果该文件不存在，无法执行任何操作，那么退出是一种有效的策略。甚至还有一个针对`Result`的快捷方式，就是调用`unwrap`：

```rust,no_run
let content = std::fs::read_to_string("test.txt").unwrap();
```

## 不必惊慌

当然，中止/崩溃程序并不是处理错误的唯一方法。不用`panic!`，我们也可以简单使用`return`：

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

但是，这会改变函数所需的返回类型。实际上，在我们的示例中一直隐藏着一些东西：这个代码所在的函数签名。而在上一个`return`例子中，它变得很重要。这是*完整*例子：

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

我们的返回类型是一个`Result`！这就是为什么我们可以在第二个 match 臂上，写`return Err(error);`。看看为啥有个`Ok(())`在底部？因它是函数的默认返回值，表示“结果(Result)正常`Ok`，且没有内容`()`”。

<aside>

**旁白：** 为什么这不是写为`return Ok(());`？很容易——这也是完全有效的。在 Rust 中，作用域内的最后一个表达式(不加`;`结尾)是它的返回值，并且习惯上省略不必要的`return`。

</aside>

## 问号`?`

就像`match`中，可在错误臂调用`.unwrap()`，作为`panic!`一样，我们的`match`有另一个能在错误臂中`return`的，就是`?`。

没错，`?`。可以将此运算符附加到类型`Result`的值上，Rust 会在内部扩展为类似我们刚刚编写的`match`语句。

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

**旁白：** 这里还发生了一些不需要理解的事情。例如，我们的`main`函数是`Box<dyn std::error::Error>`. 但我们已经看到`read_to_string`是返回[`std::io::Error`]的。这是因为`?`扩展了*转换*错误类型的代码。

`Box<dyn std::error::Error>`也是一种有趣的类型。这是一个`Box`可以包含*任何*实现标准[`Error`][`std::error::error`]trait 的类型。这意味着基本上所有的错误都可以放入这个 Box 中，所以我们可以把`?`，用在会返回`Result`的所有常规函数上。

[`std::error::error`]: https://doc.rust-lang.org/1.31.0/std/error/trait.Error.html

</aside>

## 提供上下文

在你的`main`函数使用`?`时出现错误，是可以的，但不太好。例如：当你运行`std::fs::read_to_string("test.txt")?`，但是文件`test.txt`不存在，您将得到以下输出：

> > Error: Os { code: 2, kind: NotFound, message: "No such file or directory" }

如果代码中没有包含文件名，就很难判断哪个文件是`NotFound`。有多种方法可以解决这个问题。

例如，我们可以创建自己的错误类型，然后使用它来构建自定义错误消息：

```rust,ignore
{{#include errors-custom.rs}}
```

现在，运行此命令，我们将收到自定义错误消息：

> Error: CustomError("Error reading `test.txt`: No such file or directory (os error 2)")

不是说很漂亮，但稍后我们可以为我们的类型，简单调整调试输出。

这种模式实际上很常见。但它有一个问题：我们不存储原始错误，只存储它的字符串表示。常用的[`failure`]箱子有一个很好的解决方案：类似于我们的`CustomError`类型，但它有一个[`Context`]会包含说明和原始错误的类型。箱子也带来了一个扩展 trait [`ResultExt`]），可以为`Result`加上[`context()`]和[`with_context()`]方法。

[`failure`]: https://docs.rs/failure
[`context`]: https://docs.rs/failure/0.1.3/failure/struct.Context.html
[`resultext`]: https://docs.rs/failure/0.1.3/failure/trait.ResultExt.html
[`context()`]: https://docs.rs/failure/0.1.3/failure/trait.ResultExt.html#tymethod.context
[`with_context()`]: https://docs.rs/failure/0.1.3/failure/trait.ResultExt.html#tymethod.with_context

为了将这些打包的错误类型，转换为人类真正想要读取的内容，我们可以进一步添加[`exitfailure`]箱子，并使用其类型作为我们`main`函数的返回类型。

让我们先导入这些箱子，也就是在`Cargo.toml`文件的`[dependencies]`部分，添加`failure = "0.1.5"`和`exitfailure = "0.5.1"`。

完整的示例如下：

[`exitfailure`]: https://docs.rs/exitfailure

```rust,ignore
{{#include errors-exit.rs}}
```

这将打印一个错误：

> Error: could not read file `test.txt`  
> Info: caused by No such file or directory (os error 2)
