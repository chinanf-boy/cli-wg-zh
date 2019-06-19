# 输出

## 打印“Hello World”

```rust
println!("Hello World");
```

嗯，很简单。很好，下一个话题。

## 使用 println

你大概可以用`println!`宏打印你喜欢的所有东西。这个宏有一些非常惊人的功能，但也有一个特殊的语法。它期望您编写一个字符串文字作为第一个参数，其中包含由后面参数的参数值填充的占位符。

例如：

```rust
let x = 42;
println!("My lucky number is {}.", x);
```

将打印

```console
My lucky number is 42.
```

上面的字符串中大括号（`{}`）是这些占位符之一。这是默认的占位符类型，尝试以人类可读的方式打印值。对于数字和字符串，这很好地工作，但并非所有类型都能做到这一点。这就是为什么还存在一个“调试表示法”，就是通过在大括号填充占位符：`{:?}`。

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

**旁白：**“用户友好”的打印是使用[`Display`] trait，调试输出（人类可读，但面向开发人员）是使用[`Debug`] trait。您可以在[`std::fmt`模块的文档][std::fmt]找到，有关`println!`使用语法的更多信息。

[`display`]: https://doc.rust-lang.org/1.31.0/std/fmt/trait.Display.html
[`debug`]: https://doc.rust-lang.org/1.31.0/std/fmt/trait.Debug.html
[std::fmt]: https://doc.rust-lang.org/1.31.0/std/fmt/index.html

</aside>

## 打印错误

错误的打印应该用`stderr`，让用户和其他工具更容易将输出，传输到文件或更多工具。

<aside>

**旁白：**在大多数操作系统上，程序可以写入两个输出流，`stdout`和`stderr`。 `stdout`是程序的实际输出，而`stderr`则允许错误和其他消息，并与`stdout`分隔开来。这样的话，当是错误的事件发生，(错误)输出就能存储到一个文件或管道到其他程序。

</aside>

在 Rust 中，由`println!`和`eprintln!`完成，前者打印到`stdout`，而后者打印到`stderr`。

```rust
println!("This is information");
eprintln!("This is an error! :(");
```

<aside>

**当心**：打印[转义代码][escape codes]可能是危险的，会把用户的终端置于一个奇怪的状态，所以手动打印总要小心为妙。

[escape codes]: https://en.wikipedia.org/wiki/ANSI_escape_code

理想情况下，你应该使用一个箱子`ansi_term`，当处理原始转义代码，能让你（和你的用户）日子更简单。

</aside>

## 一份打印性能的报告

如果你尝试重复`println!`的话，你会发现打印到终端非常缓慢！很容易就成为以快速为目标的程序的瓶颈。要加速，这里有两件你能做的事。

第一，您可能想要减少终端实际的刷新（flush）。而每一个`println!`会告诉系统*每次*都刷新一下终端，因为常见功能就是要打印一个新行。如果你并不需要这些，你可以用一个默认的 8 kB 缓冲[`BufWriter`]，去包裹你的`stdout`控制器。（你仍可以调用`BufWriter`的`.flush()`，若是你想要立即打印的话。)

```rust
use std::io::{self, Write};

let stdout = io::stdout(); // 获得 stdout 实体
let mut handle = io::BufWriter::new(stdout); // 可选: 把  stdout 的 控制权 包裹进一个 buffer
writeln!(handle, "foo: {}", 42); // 可加上 `?`， 若你关心错误的话。
```

第二，获得`stdout`(或`stderr`)的一个锁也有用，并使用`writeln!`直接打印到它。这可以防止系统一遍遍重复对`stdout`上锁和解锁。

```rust
use std::io::{self, Write};

let stdout = io::stdout(); // 获得 stdout 实体
let mut handle = stdout.lock(); // 获得它的一个锁
writeln!(handle, "foo: {}", 42); // 可加上 `?`， 若你关心错误的话。
```

你也可以把两种方法结合起来。

[`bufwriter`]: https://doc.rust-lang.org/1.31.0/std/io/struct.BufWriter.html

## 显示一个进度条

一些 CLI 应用程序运行不到一秒，有些则以分钟或小时计算。
如果你编写的是后者，你可能会想要告诉你的用户，程序的进展如何。
针对这个，你应该试着打印有用的更新状态，想法是能让用户容易接受。

使用[indicatif]箱，你可以为你的程序添加进度条和小提示。
下面是个快餐示例：

```rust,ignore
{{#include output-progressbar.rs:1:9}}
```

欲了解更多的信息，请看[文档][indicatif docs]和[例子][indicatif examples]。

[indicatif]: https://crates.io/crates/indicatif
[indicatif docs]: https://docs.rs/indicatif
[indicatif examples]: https://github.com/mitsuhiko/indicatif/tree/master/examples

## 日志记录

要让我们程序正在发生的事情，更容易理解。我们会想要加上一些记录语句。这是最常用的方式了，但接下来的时间，它会却能带给超大的帮助。在某些考虑下，记录就如同`println`一样，除了你可以指定信息的重要性，常见的级别有 _error_, _warn_, _info_, _debug_, 和 _trace_ (*error*具有最高优先级，*trace*最低的)。

要将简单的日志记录添加到您的应用程序，你需要两个帮手：[log]箱子(配有日志级别命名的宏）和一个 _适配器(adapter)_，也就是把日志输出到有用的地方。日志适配器的功能是很灵活：你想想，不仅可以把日志输出到终端，还可以是[syslog]，或中央日志服务器。

[syslog]: https://en.wikipedia.org/wiki/Syslog

鉴于我们现在只关心编写一个命令行应用，适配器简单用[env_logger]就好。它的名称叫做`env`，因它可能使用环境变量，来指定你命令行想要记录的部分。（当然，还有日志的级别）它会有一个时间戳和来自哪个模块，帮你前缀化你的信息，也可以简单配置日志的输出。

[log]: https://crates.io/crates/log
[env_logger]: https://crates.io/crates/env_logger

这里有一个简单的例子:

```rust,ignore
{{#include output-log.rs}}
```

假设你有这个`src/bin/output-log.rs`文件,

在 Linux 和 macOS，您可以像这样运行它:

```console
$ env RUST_LOG=output_log=info cargo run --bin output-log
    Finished dev [unoptimized + debuginfo] target(s) in 0.17s
     Running `target/debug/output-log`
[2018-11-30T20:25:52Z INFO  output_log] starting up
[2018-11-30T20:25:52Z WARN  output_log] oops, nothing implemented!
```

在 Windows PowerShell，您可以像这样运行它:

```console
$ $env:RUST_LOG="output_log=info" //set the env var for the current session
$ cargo run --bin output-log
    Finished dev [unoptimized + debuginfo] target(s) in 0.17s
     Running `target/debug/output-log.exe`
[2018-11-30T20:25:52Z INFO  output_log] starting up
[2018-11-30T20:25:52Z WARN  output_log] oops, nothing implemented!
```

在 Windows CMD，您可以像这样运行它:

```console
$ rem set the env var for the current session
$ set RUST_LOG=output_log=info
$ cargo run --bin output-log
    Finished dev [unoptimized + debuginfo] target(s) in 0.17s
     Running `target/debug/output-log.exe`
[2018-11-30T20:25:52Z INFO  output_log] starting up
[2018-11-30T20:25:52Z WARN  output_log] oops, nothing implemented!
```

`RUST_LOG`是环境变量的名字，你可以用来设置你的日志配置。`env_logger`还包含一个构建器，所以你可以用程序调整这个配置，就比如说，默认级别信息为 _info_ 。

有很多日志适配器的替代品，当然还有`log`的替换或扩展。
如果你知道你的应用要记录很多，请确保日志清晰明了，让你的用户日子舒服些。

<aside>

**提示：**经验表明，即使是小的 CLI，但有用的话，使用寿命可能是以年计的。（即使是作为一个暂时的解决方案。）如果你的应用不能工作了，和某人（或是未来的你）需要理解应用，那么`--verbose`标志的详细日志输出，就能让这个过程缩短。[clap-verbosity-flag]箱子包含一个添加`--verbose`的方法，仅限该项目使用的是`structopt`。

[clap-verbosity-flag]: https://crates.io/crates/clap-verbosity-flag

</aside>
