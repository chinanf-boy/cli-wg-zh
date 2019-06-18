# 项目设置

如果你还没有准备好,[安装生锈]在你的电脑上(它应该只需要几minut

[install rust]: https://www.rust-lang.org/tools/install

先运行`cargo new grrs`在目录中存储您的编程)按`grrs`目录,你会发现一个典型的设置为一个生锈

-   一个`Cargo.toml`为我们的项目文件,其中包含元数据,包括
-   一个`src/main.rs`文件的入口点(主要)binar

如果你可以执行`cargo run`在`grrs`目录和得到一个“Hello World”,你都准备好了

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
