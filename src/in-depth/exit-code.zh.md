# 退出代码

程序并不总是成功的。当发生错误时，您应该确保正确地发出必要的信息。除了在大多数系统上[告诉用户错误](human-communication.zh.html)，当进程退出时，它也会发出退出代码（0 到 255 之间的整数，与大多数平台兼容）。您应该尝试为程序的状态发出正确的代码。例如，在理想情况下，当程序成功时，它应该的退出代码为`0`。

但是，当一个错误发生时，它会变得更加复杂。在真实世界，许多工具(的退出代码)会是`1`当发生常见故障时。目前当进程恐慌时，Rust 设置的退出代码为`101`。除此之外，人们在他们的程序中，还做了很多事情。

那么，该怎么办？BSD 生态系统收集了它们退出代码的通用定义（您可以[在这里][`sysexits.h`]找到它们）。Rust 箱子[`exitcode`]提供这些相同的代码，可以在应用程序中使用。有关可能使用的值，请参阅其 API 文档。

一种使用方法是这样的：

```rust,ignore
fn main() {
    // ...actual work...
    match result {
        Ok(_) => {
            println!("Done!");
            std::process::exit(exitcode::OK);
        }
        Err(CustomError::CantReadConfig(e)) => {
            eprintln!("Error: {}", e);
            std::process::exit(exitcode::CONFIG);
        }
        Err(e) => {
            eprintln!("Error: {}", e);
            std::process::exit(exitcode::DATAERR);
        }
    }
}
```

[`exitcode`]: https://crates.io/crates/exitcode
[`sysexits.h`]: https://www.freebsd.org/cgi/man.cgi?query=sysexits&apropos=0&sektion=0&manpath=FreeBSD+11.2-stable&arch=default&format=html
