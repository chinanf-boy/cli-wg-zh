# 项目设置

如果你还没有准备好，情在你的电脑上[安装螃蟹-Rust][install rust](它应该只需要几分钟)。
之后，我们打开一个终端，并`cd`到接下来教程代码(你想)放置的目录。

> 译者：螃蟹是 Rust 的吉祥物，而 Rust 也有螃蟹的意思，所以，让我们开始尝尝螃蟹吧。

[install rust]: https://www.rust-lang.org/tools/install

在目录中先运行`cargo new grrs`，**grrs**是编程存进的目录，若是查看`grrs`目录(结构)，你会发现一个典型的 Rust 项目装置。

- 一个`Cargo.toml`文件，它包含我们项目的元数据。还有我们使用的依赖项/外部 库(或者说 crate-箱子)。
- 一个`src/main.rs`文件，它是我们(主)二进制文件的入口文件。

如果你可以在`grrs`目录执行`cargo run`，你会得到一个“Hello World”，那说明你都准备好了。

## 它可能会是什么样子

```console
$ cargo new grrs
     Created binary (application) `grrs` package
$ cd grrs/
$ cargo run
   Compiling grrs v0.1.0 (/Users/pascal/code/grrs)
    Finished dev [unoptimized + debuginfo] target(s) in 0.70s
     Running `target/debug/grrs`
Hello, world!
```
