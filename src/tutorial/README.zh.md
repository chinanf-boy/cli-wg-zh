# 通过在15分钟内编写命令行应用程序学习Rust

本教程将指导您在[锈病]. 你大概要花15分钟的时间才能达到你有一个运行程序的目的（大约在1.3章）。在那之后，我们将继续调整我们的程序，直到我们到达一个可以运送我们的小工具的点。

[rust]: https://rust-lang.org/

您将学习如何开始以及在哪里找到更多信息的所有要点。随时跳过你现在不需要知道的部分，或者在任何时候跳进去。

<aside>

**先决条件：**本教程不取代编程的一般介绍，希望您熟悉一些常见的概念。您应该习惯使用命令行/终端。如果你已经知道一些其他语言，这可能是一个良好的第一次接触生锈。

**获取帮助：**如果您在任何时候对所使用的特性感到不知所措或困惑，请看一下Rust附带的大量官方文档，首先是本书《Rust编程语言》。它配有大多数的防锈装置（`rustup doc`，可在线获取。[文档rust.lang-org.].

[doc.rust-lang.org]: https://doc.rust-lang.org

我们也非常欢迎您提出问题——众所周知，铁锈行业是友好和乐于助人的。看看[社区页面]看看人们讨论生锈的地方的清单。

[community page]: https://www.rust-lang.org/community

</aside>

你想写什么样的项目？我们从一些简单的事情开始吧：让我们写一个小的`grep`克隆。这是一个工具，我们可以给出一个字符串和一个路径，它将只打印包含给定字符串的行。我们称之为`grrs`（读作“草”）。

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

**注：**这本书是为[铁锈2018]. 代码示例也可以在Rust 2015上使用，但您可能需要稍微调整一下；添加`extern crate foo;`例如，调用。

确保运行rust 1.31.0（或更高版本），并且`edition = "2018"`设置在`[package]`您的部分`Cargo.toml`文件。

[rust 2018]: https://rust-lang-nursery.github.io/edition-guide/

</aside>
