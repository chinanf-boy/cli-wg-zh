# 与人交流

确保先在教程中，阅读[关于 CLI 输出的章节][output]。它包括如何将输出写入终端，而本章将讨论*什么*是输出。

[output]: ../tutorial/output.zh.html

## 当一切都好的时候

即使一切正常，报告应用程序的进度也很有用。在这些信息中，尽量做到大众性和简洁性。不要在日志中，使用过多的技术术语。记住这一准则：应用程序没有崩溃，那么，用户就没有理由查找错误。

最重要的是，保持交流风格的一致性。使用相同的前缀和句式结构，使日志易于浏览。

尝试让应用程序的输出，讲述它正在做什么，以及它如何影响用户。这可能涉及到，显示所涉及步骤的时间线，甚至是长期运行操作的进度条和指示器。用户在任何时候，都不应该感觉到应用程序，在做一些他们无法理解的神秘事情。

## 当很难知道发生了什么事时

当交流不可名状时，保持一致是很重要的。不遵循严格日志记录级别的，且要大量日志记录的应用程序，所提供的信息量，与非日志记录应用程序的相同，甚至更少。

因此，重要的是，定义与之相关的事件和消息的严重性，然后对它们使用一致的日志级别。这样用户就可以通过`--verbose`标志或环境变量（如`RUST_LOG`）选择哪堆日志。

常用`log`箱子[定义][log-levels]的以下级别（按严重性，增序）：

- trace
- debug
- info
- warning
- error

好主意是，把 *info*作为默认日志级别。信息的输出。（一些倾向于更安静输出样式的应用程序，在默认情况下，可能只显示警告和错误。）

此外，在日志消息中，使用类似的前缀和句式结构总是好的，这样，就可以使用类似`grep`来过滤它们。消息本身应该提供足够的上下文，以便在筛选日志中有用，但同时不要*太*冗长。

[log-levels]: https://docs.rs/log/0.4.4/log/enum.Level.html

### 日志语句示例

```console
error: could not find `Cargo.toml` in `/home/you/project/`
```

```console
=> Downloading repository index
=> Downloading packages...
```

以下日志输出来自 [wasm-pack]：

```console
 [1/7] Adding WASM target...
 [2/7] Compiling to WASM...
 [3/7] Creating a pkg directory...
 [4/7] Writing a package.json...
 > [WARN]: Field `description` is missing from Cargo.toml. It is not necessary, but recommended
 > [WARN]: Field `repository` is missing from Cargo.toml. It is not necessary, but recommended
 > [WARN]: Field `license` is missing from Cargo.toml. It is not necessary, but recommended
 [5/7] Copying over your README...
 > [WARN]: origin crate has no README
 [6/7] Installing WASM-bindgen...
 > [INFO]: wasm-bindgen already installed
 [7/7] Running WASM-bindgen...
 Done in 1 second
```

## 当恐慌时

一个经常被遗忘的方面是，当程序崩溃时，它也会输出一些东西。在 Rust 中，“崩溃”通常是“恐慌(panic)”（即，“崩溃控制”，与“操作系统杀死了进程”有所不同）。默认情况下，当发生紧急情况时，“崩溃处理程序”将向控制台打印一些信息。

例如，如果使用`cargo new --bin foo`创建一个新的二进制项目，并用`panic!("Hello World")`替代`fn main`中的内容，运行程序时会得到：

```console
thread 'main' panicked at 'Hello, world!', src/main.rs:2:5
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

这对你和开发者来说，都是有用的信息。（意外：程序因您的`main.rs`文件的第2行而崩溃）。但是对于一个，甚至不访问源代码的用户来说，这并不是很有价值。事实上，它很可能只是令人困惑。这就是为什么添加一个定制的崩溃处理程序是一个好的主意，它提供了一个更加注重最终用户的输出。

有一个箱子就是这么做的，它叫做[human-panic]。要将其添加到 CLI 项目中，请导入它，并在`main`函数种调用`setup_panic!()`宏：

```rust,ignore
use human_panic::setup_panic;

fn main() {
   setup_panic!();

   panic!("Hello world")
}
```

这将显示一条非常友好的消息，并告诉用户他们可以做什么：

```console
Well, this is embarrassing.

foo had a problem and crashed. To help us diagnose the problem you can send us a crash report.

We have generated a report file at "/var/folders/n3/dkk459k908lcmkzwcmq0tcv00000gn/T/report-738e1bec-5585-47a4-8158-f1f7227f0168.toml". Submit an issue or email with the subject of "foo Crash Report" and include the report as an attachment.

- Authors: Your Name <your.name@example.com>

We take privacy seriously, and do not perform any automated error collection. In order to improve the software, we rely on people to submit reports.

Thank you kindly!
```

[human-panic]: https://crates.io/crates/human-panic
[wasm-pack]: https://crates.io/crates/wasm-pack
