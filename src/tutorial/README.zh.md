# 15 分钟编写命令行应用程序，以此学习 Rust

本教程将指导您学习[Rust]。你大概要花 15 分钟的时间，才能达到你有一个运行程序的点（大约在 1.3 章）。在那之后，我们将继续调整我们的程序，直到我们到达另一个可以输送我们的小工具出去的点。

[rust]: https://rust-lang.org/

您将学习如何开始，以及在哪里找到更多信息的所有要点。随意跳过你现在不需要知道的部分，或者挑任何一节来学习。

<aside>

**先决条件：** 本教程不会取代编程语言的一般性介绍，并希望您熟悉一些常见的概念。您应该习惯使用命令行/终端。如果你有一些其他语言的经验，这会是首次接触 Rust 的良好例子。

**获取帮助：** 如果您在任何时候对所使用的特性，感到不知所措或困惑，请看一下 Rust 附带的大量官方文档，首先是本之书《Rust 编程语言》。它配有大多数的 Rust 配件（`rustup doc`），可在线获取。[doc.rust-lang.org].

[doc.rust-lang.org]: https://doc.rust-lang.org

我们也非常欢迎您提出问题——众所周知，Rust 社区是友好且乐于助人的。看看[社区页面][community page]看看人们 Rust 讨论区的清单。

[community page]: https://www.rust-lang.org/community

</aside>

你想写什么样的项目？我们先从一些简单的事情开始，怎么样：让我们写一个小的`grep`克隆。这是一个工具，可以给它一个字符串和一个路径，它将只打印包含给定字符串的行。我们称之为`grrs`（读作“grass”）。

最后，我们希望能够像这样运行我们的工具：

```console
$ cat test.txt
foo: 10
bar: 20
baz: 30
$ grrs foo test.txt
foo: 10
$ grrs --help
[some help text explaining the available options]
```

<aside class="note">

**注：** 这本书是为[Rust 2018]版本。代码示例也可以在 Rust 2015 版本 上使用，但您可能需要稍微调整一下；例如，添加`extern crate foo;`调用。

> 译者：Rust 语法版本，几乎大多都转向 2018 版本。基本从今以后，也只是一个起到标示的意义。

确保运行的是 rust 1.31.0（或更高版本），并且`Cargo.toml`文件的在`[package]`部分，有`edition = "2018"`设置。

[rust 2018]: https://rust-lang-nursery.github.io/edition-guide/

</aside>
