# 正在分析命令行参数

对cli工具的典型调用如下所示：

```console
$ grrs foobar test.txt
```

我们希望我们的计划`test.txt`打印出包含`foobar`. 但是我们如何得到这两个值呢？

程序名后的文本通常称为“命令行参数”或“命令行标志”（尤其是当它们看起来像`--this`）在内部，操作系统通常将它们表示为字符串列表，大致来说，它们由空格分隔。

有很多方法可以考虑这些参数，以及如何将它们解析为更容易处理的内容。您还需要告诉程序的用户他们需要给出哪些参数，以及期望的格式。

## 获取参数

标准库包含函数[`std::env::args()`]这给了你一个[迭代器]给出的参数。第一个条目（在索引处`0`）将是您的程序被称为的名称（例如`grrs`，下面的是用户随后编写的。

[`std::env::args()`]: https://doc.rust-lang.org/1.31.0/std/env/fn.args.html

[iterator]: https://doc.rust-lang.org/1.31.0/std/iter/index.html

以这种方式获得原始参数非常容易：

```rust,ignore
{{#include cli-args-struct.rs:10:11}}
```

## 作为数据类型的CLI参数

与其把它们看作一堆文本，不如把cli参数看作表示程序输入的自定义数据类型。

看看`grrs foobar test.txt`：有两个参数，第一个是`pattern`（要查找的字符串），然后`path`（要查找的文件）。

我们还能对他们说些什么呢？嗯，首先，两者都是必需的。我们还没有讨论任何默认值，因此我们希望用户始终提供两个值。此外，我们可以稍微介绍一下它们的类型：模式应该是一个字符串，而第二个参数应该是一个文件的路径。

在Rust中，围绕所处理的数据构建程序是非常常见的，因此这种查看CLI参数的方式非常适合。让我们从这个开始：

```rust,ignore
{{#include cli-args-struct.rs:3:7}}
```

这定义了一个新的结构（a[`struct`]）它有两个字段用于存储数据：`pattern`和`path`.

[`struct`]: https://doc.rust-lang.org/1.31.0/book/ch05-00-structs.html

<aside>

**旁白：**
[`PathBuf`]就像一个[`String`]但对于跨平台工作的文件系统路径。

[`pathbuf`]: https://doc.rust-lang.org/1.31.0/std/path/struct.PathBuf.html

[`string`]: https://doc.rust-lang.org/1.31.0/std/string/struct.String.html

</aside>

现在，我们仍然需要得到我们的程序进入这个表单的实际参数。一种选择是手动解析从操作系统获得的字符串列表，并自己构建结构。它看起来像这样：

```rust,ignore
{{#include cli-args-struct.rs:10:15}}
```

这行，但不太方便。你将如何处理支持的要求`--pattern="foo"`或`--pattern "foo"`？你将如何实施`--help`？

## 使用structopt分析CLI参数

一个更好的方法是使用许多可用的库中的一个。最流行的用于分析命令行参数的库称为[`clap`]. 它具有您所期望的所有功能，包括对子命令的支持、shell完成和伟大的帮助消息。

这个[`structopt`]图书馆建立在`clap`并提供要生成的“派生”宏`clap`代码`struct`定义。这很好：我们所要做的就是注释一个结构，它将生成将参数解析到字段中的代码。

[`clap`]: https://clap.rs/

[`structopt`]: https://docs.rs/structopt

我们先进口吧`structopt`通过添加`structopt = "0.2.10"`到`[dependencies]`我们的部分`Cargo.toml`文件。

现在，我们可以写了`use structopt::StructOpt;`在我们的代码中，并添加`#[derive(StructOpt)]`就在我们的上面`struct Cli`. 同时，我们还将编写一些文档注释。

看起来像这样：

```rust,ignore
{{#include cli-args-structopt.rs:3:14}}
```

<aside class="node">

**注：**有很多自定义属性可以添加到字段中。例如，我们添加了一个来告诉structopt如何解析`PathBuf`类型。要说您想在后面的参数中使用此字段`-o`或`--output`，您可以添加`#[structopt(short = "o", long = "output")]`. 有关详细信息，请参阅[结构选择文档][`structopt`].

</aside>

就在`Cli`结构我们的模板包含`main`功能。当程序启动时，它将调用此函数。第一行是：

```rust,ignore
{{#include cli-args-structopt.rs:15:18}}
```

这将尝试将参数解析为`Cli`结构。

但如果失败了呢？这就是这种方法的好处：Clap知道期望哪个字段，以及它们期望的格式是什么。它可以自动生成一个`--help`信息，以及一些重大错误建议您通过`--output`当你写的时候`--putput`.

<aside class="note">

**注：**这个`from_args`方法用于`main`功能。当失败时，它将打印出一个错误或帮助消息，并立即退出程序。不要在其他地方使用它！

</aside>

## 这就是它的样子

在没有任何参数的情况下运行它：

```console
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 10.16s
     Running `target/debug/grrs`
error: The following required arguments were not provided:
    <pattern>
    <path>

USAGE:
    grrs <pattern> <path>

For more information try --help
```

我们可以在使用时传递参数`cargo run`直接写在后面`--`：

```console
$ cargo run -- some-pattern some-file
    Finished dev [unoptimized + debuginfo] target(s) in 0.11s
     Running `target/debug/grrs some-pattern some-file`
```

如您所见，没有输出。很好：这意味着没有错误，我们的程序结束了。

<aside class="exercise">

**读者练习：**使此程序输出其参数！

</aside>
