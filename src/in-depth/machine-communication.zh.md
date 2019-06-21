# 与机器交互

当您能够组合命令行工具时，它们的威力真的会很闪耀。这不是一个新想法：事实上，这是[Unix 哲学][unix philosophy]：

> 期望每个程序的输出，都成为另一个程序的输入，这正是你无法想像的程序。

[unix philosophy]: https://en.wikipedia.org/wiki/Unix_philosophy

如果我们的程序满足这个期望，我们的用户会很高兴。为了确保这项工作良好，我们不仅应该为人类提供相当好的输出，还应该为其他程序提供一个适合的版本。让我们看看怎么做。

<aside>

**旁白：** 确保先阅读在教程中的[关于 CLI 输出的章节][output]。它包括如何将输出写入终端。

[output]: ../tutorial/output.zh.html

</aside>

## 谁在读取？

要问的第一个问题是：我们的输出是给彩色终端前的人类，还是给另一个程序？为了回答这个问题，我们可以使用像[atty]这样的箱子：

[atty]: https://crates.io/crates/atty

```rust,ignore
use atty::Stream;

if atty::is(Stream::Stdout) {
    println!("I'm a terminal");
} else {
    println!("I'm not");
}
```

根据谁读取我们的输出，我们之后就可以添加额外的信息。人类喜欢颜色，例如，如果你在一个随机的 Rust 项目中运行`ls`，您可能会看到这样的情况：

```console
$ ls
CODE_OF_CONDUCT.md   LICENSE-APACHE       examples
CONTRIBUTING.md      LICENSE-MIT          proptest-regressions
Cargo.lock           README.md            src
Cargo.toml           convey_derive        target
```

因为这种样式是为人类设计的，在大多数配置中，它甚至会打印一些名称（例如`src`），并以彩色显示它们是目录。如果您改为将这个输出，经过管道传输到文件或类似`cat`的程序，`ls`会调整其输出。它将在单行上，打印每个条目，而不是使用适合我的终端窗口的列数。它也不会发出任何颜色。

```console
$ ls | cat
CODE_OF_CONDUCT.md
CONTRIBUTING.md
Cargo.lock
Cargo.toml
LICENSE-APACHE
LICENSE-MIT
README.md
convey_derive
examples
proptest-regressions
src
target
```

## 机器的简单输出格式

历史上，命令行工具生成输出的唯一类型，就是字符串。对于那些在终端前，能够阅读文本和理解其含义的人来说，这通常是很好的。但是，其他程序通常没有这种能力：它们理解类似`ls`工具输出的唯一方法，就在于程序的作者是否包含一个`ls`输出的解析器。

这通常意味着，输出仅限于易于解析的内容。像 TSV（Tab 分隔值）这样的格式非常流行，其中每个记录都在自己的行上，并且每一行包含 tab 分隔的内容。这些基于文本行的简单格式，允许`grep`能用在像`ls`这样会输出的工具上。`| grep Cargo`才不管你的文本是不是来自`ls`或者文件，它都会逐行过滤。

缺点是你不能用简单调用`grep`，就筛选`ls`给你的所有目录。因为，每个目录项，都需要携带额外的数据。

## 机器的 JSON 输出

tab 分隔值是输出结构化数据的一种简单方法，但它要求另一个程序知道要预想哪个字段（以及顺序），并且很难输出不同类型的消息。例如，假设我们的程序，想向用户发送消息，告诉他们，它当前正在等待下载，然后输出一条消息，描述它得到的数据。这些都可能是非常不同的类型的消息，试图在 TSV 输出中统一它们，需要我们发明一种方法来区分它们。同样的情况还有，当我们想要打印包含两个不同长度项的列表的消息时。

不过，最好选择一种在大多数编程语言/环境中，都易于解析的格式。因此，在过去的几年中，许多应用程序都发展[JSON]的解析能力。JSON 很简单，几乎每种语言都存在它的解析器，但其强大程度，足以在许多情况下发挥作用。虽然它是一种人类可以读取的文本格式，但许多人也在，解析 JSON 数据，和将数据序列化为 JSON 方面做了许多工作，现已是速度非常快的实现了。

[json]: https://www.json.org/

在上面的描述中，我们已经讨论过程序要编写“消息”。这是一种考虑输出的好方法：其实程序不一定只输出一个数据块，而实际上，在运行的时候可能会发出许多不同(类型)的信息。在输出 JSON 时，支持这种方法的一个简单方法，是为每条消息编写一个 JSON 文档，并将每个 JSON 文档放到新的一行（有时，调用下[行-分隔 JSON][jsonlines]）。这可以让(信息)实现像使用常规的`println!`一样简单。

[jsonlines]: https://en.wikipedia.org/wiki/JSON_streaming#Line-delimited_JSON

下面是一个简单的例子，使用来自[serde_json]的`json!`宏，用来在您的 Rust 源代码中，快速编写有效的 JSON：

[serde_json]: https://crates.io/crates/serde_json

```rust,ignore
{{#include machine-communication.rs:1:22}}
```

下面是输出：

```console
$ cargo run -q
Hello world
$ cargo run -q -- --json
{"content":"Hello world","type":"message"}
```

（运行`cargo`带`-q`，能抑制其正常输出。`--`后面的参数被传递到我们的程序。）

### 实例：ripgrep

*[ripgrep]*是 _grep_ 或 _ag_ 的替代品，用 Rust 写的。默认情况下，它将生成如下输出：

[ripgrep]: https://github.com/BurntSushi/ripgrep

```console
$ rg default
src/lib.rs
37:    Output::default()

src/components/span.rs
6:    Span::default()
```

但是给出`--json`，它将打印：

```console
$ rg default --json
{"type":"begin","data":{"path":{"text":"src/lib.rs"}}}
{"type":"match","data":{"path":{"text":"src/lib.rs"},"lines":{"text":"    Output::default()\n"},"line_number":37,"absolute_offset":761,"submatches":[{"match":{"text":"default"},"start":12,"end":19}]}}
{"type":"end","data":{"path":{"text":"src/lib.rs"},"binary_offset":null,"stats":{"elapsed":{"secs":0,"nanos":137622,"human":"0.000138s"},"searches":1,"searches_with_match":1,"bytes_searched":6064,"bytes_printed":256,"matched_lines":1,"matches":1}}}
{"type":"begin","data":{"path":{"text":"src/components/span.rs"}}}
{"type":"match","data":{"path":{"text":"src/components/span.rs"},"lines":{"text":"    Span::default()\n"},"line_number":6,"absolute_offset":117,"submatches":[{"match":{"text":"default"},"start":10,"end":17}]}}
{"type":"end","data":{"path":{"text":"src/components/span.rs"},"binary_offset":null,"stats":{"elapsed":{"secs":0,"nanos":22025,"human":"0.000022s"},"searches":1,"searches_with_match":1,"bytes_searched":5221,"bytes_printed":277,"matched_lines":1,"matches":1}}}
{"data":{"elapsed_total":{"human":"0.006995s","nanos":6994920,"secs":0},"stats":{"bytes_printed":533,"bytes_searched":11285,"elapsed":{"human":"0.000160s","nanos":159647,"secs":0},"matched_lines":2,"matches":2,"searches":2,"searches_with_match":2}},"type":"summary"}
```

如您所见，每个 JSON 文档都是一个包含`type`字段的对象(map)。这将允许我们为`rg`编写一个简单前端，读取它们所在的文档，并显示匹配项（以及它们所在的文件）时，即便*ripgrep*仍在搜索。

<aside>

**旁白：** 这就是 Visual Studio Code 的代码搜索，使用的是*ripgrep*。

</aside>

## 对人和机器输出的摘要

[convey]是一个正在开发的库，它试图让输出消息更容易，以适合人类和机器格式。您定义自己的消息类型，并实现一个`Render`trait（可在宏的帮助下，手动编写，或者使用派生属性）来说明它们应该如何格式化。目前，它支持打印人类输出（包括，自动检测是否应该上色）、写 JSON 文档（可以是`stdout`或者指向一个文件）或者是兼顾两者。

[convey]: https://crates.io/crates/convey

即使您不使用这个库，编写一个适合您用例的相仿抽象，也是一个好主意。

## 如何处理流入我们的输入

<aside class="todo">

**TODO：** 讨论如何使用 stdin（请参见[α95](https://github.com/rust-lang-nursery/cli-wg/issues/95)）

</aside>
