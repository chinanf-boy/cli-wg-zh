# 对命令行参数解析

对 cli 工具的典型调用，如下所示：

```console
$ grrs foobar test.txt
```

我们希望我们的程序，可以查阅`test.txt`，并打印出包含`foobar`的那些行。但是我们如何得到这两个值呢？

(调用的)程序名，之后的文本，通常称为“命令行参数”或“命令行标志”（尤其是当它们看起来像`--this`的时候）。在内部，操作系统通常将它们表示为字符串列表，大致来说，它们由空格分隔。

这些参数有很多考虑方式，以及如何将它们解析为更容易处理的内容。您还需要告诉程序的用户，他们需要给出哪些参数，以及期望的格式。

## 获取参数

标准库包含函数[`std::env::args()`]，这给了你一个[迭代器][iterator]，含有用户给出的命令行参数。第一个条目（在索引处`0`）将是您的程序所称的名称（例如`grrs`），接下去的条目是用户随后写下的。

[`std::env::args()`]: https://doc.rust-lang.org/1.31.0/std/env/fn.args.html
[iterator]: https://doc.rust-lang.org/1.31.0/std/iter/index.html

以这种方式，获得原始参数非常容易：

```rust,ignore
{{#include cli-args-struct.rs:10:11}}
```

## 作为数据类型的 CLI 参数

与其把它们看作一堆文本，不如把 cli 参数看作表示程序输入的自定义数据类型。

瞧瞧`grrs foobar test.txt`：有两个参数，第一个是`pattern`(模式)（要查找的字符串），然后`path`(路径)（要查找的文件）。

我们还能对他们说些什么呢？嗯，首先，两者都是必需的。我们还没有讨论任何默认值，因此我们希望用户始终提供两个值。此外，我们可以稍微介绍一下它们的类型：模式应该是一个字符串，而第二个参数应该是一个文件的路径。

在 Rust 中，围绕所处理的数据，构建程序是很常见的，因此这种查看 CLI 参数的方式非常适合。让我们从这个开始：

```rust,ignore
{{#include cli-args-struct.rs:3:7}}
```

这定义了一个新的结构（一个[`struct`]）它有两个字段用于存储数据：`pattern`和`path`。

[`struct`]: https://doc.rust-lang.org/1.31.0/book/ch05-00-structs.html

<aside>

**旁白：** 
[`PathBuf`]就像一个[`String`]，但用作跨平台工作的文件系统路径。

[`pathbuf`]: https://doc.rust-lang.org/1.31.0/std/path/struct.PathBuf.html
[`string`]: https://doc.rust-lang.org/1.31.0/std/string/struct.String.html

</aside>

现在，我们仍然需要得到进入程序的实际参数。一种选择是手动解析操作系统获得的字符串列表，并自己构建结构。它看起来像这样：

```rust,ignore
{{#include cli-args-struct.rs:10:15}}
```

这样是可以的，但不太方便。但你要如何处理支持`--pattern="foo"`或`--pattern "foo"`的要求？你又如何实现`--help`？

## 使用 StructOpt 分析 CLI 参数

一个更好的方法是运用一个可用库，当然还有许多其他可用库。最流行的用于分析命令行参数的库称为[`clap`]. 它具有您所期望的所有功能，包括对，子命令的支持、shell 补全和伟大的帮助消息。

这个[`structopt`]箱子建立在`clap`之上，并提供一个“derive”宏，用来生成`struct`定义的有关`clap`代码。这很好：我们所要做的就是注释一个结构，而它将生成为，把(命令行)参数解析到字段中的代码。

[`clap`]: https://clap.rs/
[`structopt`]: https://docs.rs/structopt

我们先导入`structopt`，具体先在`Cargo.toml`文件的`[dependencies]`部分，添加`structopt = "0.2.10"`。

现在，在我们的代码中，写`use structopt::StructOpt;`，并添加`#[derive(StructOpt)]`，到我们`struct Cli`的上面。同时，我们还将编写一些文档注释。

看起来像这样：

```rust,ignore
{{#include cli-args-structopt.rs:3:14}}
```

<aside class="node">

**旁注：** （StructOpt）有很多自定义属性可以添加到字段中。例如，我们添加了一个`PathBuf`类型，让 structopt 解析。要说您想在后面的参数中，使用此字段`-o`或`--output`，您可以添加`#[structopt(short = "o", long = "output")]`。 有关详细信息，请参阅[StructOpt 文档][`structopt`]。

</aside>

就在`Cli`结构下面，我们的模板包含`main`函数。当程序启动时，它将调用此函数。第一行是：

```rust,ignore
{{#include cli-args-structopt.rs:15:18}}
```

这将尝试将(命令行)参数解析为`Cli`结构。

但如果失败了呢？下面就是这方式的好处：Clap 知道应期望哪个字段，以及它们期望的格式是什么。它可以自动生成一个`--help`信息，以及一些重大错误(信息)，建议您应把`--output`，而不是`--putput`作为传递参数。

<aside class="note">

**旁注：** 这个`from_args`方法就是给`main`函数使用的。当失败时，它将打印出一个错误或帮助消息，并立即退出程序。请不要在其他地方使用它！

</aside>

## 这就是它的样子

在没有任何参数的情况下，运行它：

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

**读者练习：** 让此程序输出其(命令行)参数！

</aside>
