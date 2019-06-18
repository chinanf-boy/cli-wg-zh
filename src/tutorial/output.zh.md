# 产量

## 打印“Hello World”

```rust
println!("Hello World");
```

嗯，那很简单。很好，下一个话题。

## 使用println

你可以打印你喜欢的所有东西`println!`宏。这个宏有一些非常惊人的功能，但也有一个特殊的语法。它期望您编写一个字符串文字作为第一个参数，其中包含将由后面作为进一步参数的参数值填充的占位符。

例如：

```rust
let x = 42;
println!("My lucky number is {}.", x);
```

将打印

```console
My lucky number is 42.
```

大括号（`{}`）在上面的字符串中是这些占位符之一。这是默认的占位符类型，尝试以人类可读的方式打印给定值。对于数字和字符串，这很好地工作，但并非所有类型都能做到这一点。这就是为什么还存在一个“调试表示法”，您可以通过这样填充占位符的大括号来获得：`{:?}`.

例如，

```rust
let xs = vec![1, 2, 3];
println!("The list is: {:?}", xs);
```

将打印

```console
The list is: [1, 2, 3]
```

如果您希望自己的数据类型可以打印用于调试和日志记录，那么在大多数情况下，您可以添加`#[derive(Debug)]`在他们的定义之上。

<aside>

**旁白：**“用户友好”打印是使用[`Display`]特性，调试输出（人类可读，但面向开发人员）使用[`Debug`]特质。您可以找到有关您可以在中使用的语法的更多信息`println!`在[文件`std::fmt`模块][std::fmt].

[`display`]: https://doc.rust-lang.org/1.31.0/std/fmt/trait.Display.html

[`debug`]: https://doc.rust-lang.org/1.31.0/std/fmt/trait.Debug.html

[std::fmt]: https://doc.rust-lang.org/1.31.0/std/fmt/index.html

</aside>

## 打印错误

打印错误应通过`stderr`使用户和其他工具更容易将输出传输到文件或更多工具。

<aside>

**旁白：**在大多数操作系统上，程序可以写入两个输出流，`stdout`和`stderr`. `stdout`是程序的实际输出,而`stderr`允许错误和其他消息保持separ`stdout`。

</aside>

这样,可以存储输出文件或管道`println!`和`eprintln!`前打印`stdout`而后者,`stderr`。

```rust
println!("This is information");
eprintln!("This is an error! :(");
```

<aside>

**当心**:印刷[转义代码]可能是危险的,把用户的终端int吗

[escape codes]: https://en.wikipedia.org/wiki/ANSI_escape_code

理想情况下你应该使用一个板条箱`ansi_term`当处理原始转义代码让你(

</aside>

## 对印刷性能报告

打印到终端非常缓慢!`println!`如果

首先,您可能想要减少文书`println!`告诉系统冲洗到终端*每一个*一次,因为它是常见的打印每个新行`stdout`处理在一个[`BufWriter`]默认的缓冲区8 kB。`.flush()`(你可以sti`BufWriter`当你想要立即打印。)

```rust
use std::io::{self, Write};

let stdout = io::stdout(); // get the global stdout entity
let mut handle = io::BufWriter::new(stdout); // optional: wrap that handle in a buffer
writeln!(handle, "foo: {}", 42); // add `?` if you care about errors here
```

其次,它有助于获得一个锁`stdout`(或`stderr`),用`writeln!`直接打印到它。`stdout`这可以防止系统

```rust
use std::io::{self, Write};

let stdout = io::stdout(); // get the global stdout entity
let mut handle = stdout.lock(); // acquire a lock on it
writeln!(handle, "foo: {}", 42); // add `?` if you care about errors here
```

你也可以把两种方法结合起来。

[`bufwriter`]: https://doc.rust-lang.org/1.31.0/std/io/struct.BufWriter.html

## 显示一个进度条

一些CLI应用程序运行不到一秒,其他

使用[indicatif]箱,你可以添加进度条和小spinn

```rust,ignore
{{#include output-progressbar.rs:1:9}}
```

看到[文档][indicatif docs]和[例子][indicatif examples]为更多的信息。

[indicatif]: https://crates.io/crates/indicatif

[indicatif docs]: https://docs.rs/indicatif

[indicatif examples]: https://github.com/mitsuhiko/indicatif/tree/master/examples

## 日志记录

让它更容易理解正在发生的事情`println`,但您可以指定的重要性*错误*, *警告*, *信息*, *调试*,*跟踪* (*错误*具有最高优先级,*跟踪*最低的)。

将简单的日志记录添加到您的应用程序,你会[日志]箱(包含宏日志命名的l*适配器*其实写日志输出的地方使用[syslog],或中央日志服务器。

[syslog]: https://en.wikipedia.org/wiki/Syslog

因为我们现在只关心写作[env_logger]。`log`它叫做“env”记录器,因为您可以使用一个

[log]: https://crates.io/crates/log

[env_logger]: https://crates.io/crates/env_logger

这里有一个简单的例子:

```rust,ignore
{{#include output-log.rs}}
```

假设你有这个文件`src/bin/output-log.rs`,

在Linux和macOS,您可以运行它是这样的:

```console
$ env RUST_LOG=output_log=info cargo run --bin output-log
    Finished dev [unoptimized + debuginfo] target(s) in 0.17s
     Running `target/debug/output-log`
[2018-11-30T20:25:52Z INFO  output_log] starting up
[2018-11-30T20:25:52Z WARN  output_log] oops, nothing implemented!
```

在Windows PowerShell,您可以运行它是这样的:

```console
$ $env:RUST_LOG="output_log=info" //set the env var for the current session
$ cargo run --bin output-log
    Finished dev [unoptimized + debuginfo] target(s) in 0.17s
     Running `target/debug/output-log.exe`
[2018-11-30T20:25:52Z INFO  output_log] starting up
[2018-11-30T20:25:52Z WARN  output_log] oops, nothing implemented!
```

在Windows CMD,您可以运行它是这样的:

```console
$ rem set the env var for the current session
$ set RUST_LOG=output_log=info
$ cargo run --bin output-log
    Finished dev [unoptimized + debuginfo] target(s) in 0.17s
     Running `target/debug/output-log.exe`
[2018-11-30T20:25:52Z INFO  output_log] starting up
[2018-11-30T20:25:52Z WARN  output_log] oops, nothing implemented!
```

`RUST_LOG`是环境变量的名字你可以你吗`env_logger`还包含一个构建器可以programmatical*信息*默认级别的消息。

有很多替代品或者日志适配器`log`。

<aside>

**如果你知道你的应用程序有很多**经验表明,即使是轻微的有用的CLI`--verbose`额外的日志输出可以均[clap-verbosity-flag]包含一个快速的方法来添加`--verbose`一个项目使用`structopt`。

[clap-verbosity-flag]: https://crates.io/crates/clap-verbosity-flag

</aside>
