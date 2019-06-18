# 信号处理

像命令行应用程序这样的进程需要对操作系统发送的信号作出反应。最常见的例子可能是<kbd>Ctrl键</kbd>+<kbd>C</kbd>，通常指示进程终止的信号。要在Rust程序中处理信号，您需要考虑如何接收这些信号以及如何对它们作出反应。

<aside>

**注：**如果您的应用程序不需要正常关闭，则默认处理是正常的（即立即退出并让操作系统清理资源，如打开的文件句柄）。在这种情况下：不需要做这一章告诉你的事情！

但是，对于需要自己清理的应用程序，本章非常相关！例如，如果应用程序需要正确关闭网络连接（与另一端的进程说再见），请删除临时文件或重置系统设置，然后继续阅读。

</aside>

## 操作系统之间的差异

在UNIX系统（如Linux、MacOS和FreeBSD）上，进程可以接收[信号]. 它可以以默认（操作系统提供的）方式对它们作出反应，捕获信号并以程序定义的方式处理它们，或者完全忽略信号。

[signals]: https://manpages.ubuntu.com/manpages/bionic/en/man7/signal.7.html

Windows没有信号。你可以用[控制台处理程序]定义在事件发生时执行的回调。还有[结构化异常处理]它处理所有类型的系统异常，如除数为零、无效访问异常、堆栈溢出等。

[console handlers]: https://docs.microsoft.com/de-de/windows/console/console-control-handlers

[structured exception handling]: https://docs.microsoft.com/en-us/windows/desktop/debug/structured-exception-handling

## 第一步：处理ctrl+c

这个[ctrlc]板条箱做的正是它的名字所暗示的：它允许你对用户按下<kbd>Ctrl键</kbd>+<kbd>C</kbd>以跨平台的方式。使用板条箱的主要方法是：

[ctrlc]: https://crates.io/crates/ctrlc

```rust,ignore
{{#include signals-ctrlc.rs:1:7}}
```

当然，这并没有那么有帮助：它只打印一条消息，否则不会停止程序。

在一个真实的程序中，最好在信号处理程序中设置一个变量，然后在程序中的各个地方进行检查。例如，可以设置`Arc<AtomicBool>`（线程之间可共享的布尔值）在信号处理程序中，在热循环中，或者在等待线程时，您会定期检查其值，并在该值变为真时中断。

## 处理其他类型的信号

这个[ctrlc]仅板条箱手柄<kbd>Ctrl键</kbd>+<kbd>C</kbd>或者，在UNIX系统上称为`SIGINT`（中断信号）。要对更多的Unix信号作出反应，您应该看看[信号挂钩-]. 其设计如[此博客文章][signal-hook-post]是目前社区支持最广泛的图书馆。

下面是一个简单的例子：

```rust,ignore
{{#include signals-hooked.rs:1:14}}
```

[signal-hook-post]: https://vorner.github.io/2018/06/28/signal-hook.html

## 使用频道

您可以使用通道，而不是设置一个变量并让程序的其他部分检查它：您创建一个通道，每当接收到信号时，信号处理程序就向该通道发送一个值。在应用程序代码中，您使用此通道和其他通道作为线程之间的同步点。使用[横梁通道-]它看起来像这样：

[crossbeam-channel]: https://crates.io/crates/crossbeam-channel

```rust,ignore
{{#include signals-channels.rs:1:31}}
```

## 使用未来和流

如果您正在使用[东京]，您很可能已经用异步模式和事件驱动设计编写了应用程序。您可以启用信号挂钩，而不是直接使用横梁的通道。`tokio-support`功能。这可以让你打电话[`.into_async()`]信号钩上`Signals`获取实现的新类型的类型`futures::Stream`.

[signal-hook]: https://crates.io/crates/signal-hook

[tokio]: https://tokio.rs/

[`.into_async()`]: https://docs.rs/signal-hook/0.1.6/signal_hook/iterator/struct.Signals.html#method.into_async

## 当您在处理第一个ctrl+c时收到另一个ctrl+c时要做什么？

大多数用户会按<kbd>Ctrl键</kbd>+<kbd>C</kbd>，然后给你的程序几秒钟退出，或者告诉他们发生了什么。如果那不发生，他们会按<kbd>Ctrl键</kbd>+<kbd>C</kbd>再一次。典型的行为是让应用程序立即退出。
