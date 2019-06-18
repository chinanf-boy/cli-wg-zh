# 第一次实施*grrs*

在关于命令行参数的最后一章之后，我们有了输入数据，可以开始编写实际的工具。我们的`main`函数当前只包含此行：

```rust,ignore
{{#include impl-draft.rs:17:17}}
```

我们先打开我们得到的文件。

```rust,ignore
{{#include impl-draft.rs:18:19}}
```

<aside>

**旁白：**看到了吗？[`.expect`]这里的方法？这是一个要退出的快捷函数，当无法读取值（在本例中是输入文件）时，该函数将使程序立即退出。它不是很漂亮，在下一章[更好的错误报告]我们将研究如何改进这一点。

[`.expect`]: https://doc.rust-lang.org/1.31.0/std/result/enum.Result.html#method.expect

[nicer error reporting]: ./errors.html

</aside>

现在，让我们对这些行进行迭代，并打印每个包含我们的模式的行：

```rust,ignore
{{#include impl-draft.rs:21:25}}
```

试一试：`cargo run -- main src/main.rs`现在应该工作了！

<aside class="exercise">

**读者练习：**这不是最好的实现：它将把整个文件读取到内存中——不管文件有多大。找到一种方法来优化它！（一个想法可能是使用[`BufReader`]而不是`read_to_string()`）

[`bufreader`]: https://doc.rust-lang.org/1.31.0/std/io/struct.BufReader.html

</aside>
