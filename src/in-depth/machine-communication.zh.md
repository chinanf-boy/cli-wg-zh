# 与机器通信

当您能够组合命令行工具时，它们的威力真的会大放异彩。这不是一个新想法：事实上，这是[Unix哲学]：

> 期望每个程序的输出都成为另一个程序的输入，但还不知道。

[unix philosophy]: https://en.wikipedia.org/wiki/Unix_philosophy

如果我们的程序满足这个期望，我们的用户会很高兴。为了确保这项工作良好，我们不仅应该为人类提供相当好的输出，还应该为其他程序提供一个适合需要的版本。让我们看看怎么做。

<aside>

**旁白：**确保阅读[关于CLI输出的章节][output]首先在教程中。它包括如何将输出写入终端。

[output]: ../tutorial/output.html

</aside>

## 谁在读这个？

要问的第一个问题是：我们的输出是为彩色终端前的人类，还是为另一个程序？为了回答这个问题，我们可以使用像[阿蒂]：

[atty]: https://crates.io/crates/atty

```rust,ignore
use atty::Stream;

if atty::is(Stream::Stdout) {
    println!("I'm a terminal");
} else {
    println!("I'm not");
}
```

根据谁将读取我们的输出，然后我们可以添加额外的信息。例如，如果你跑步，人类喜欢颜色`ls`在一个随机的生锈项目中，您可能会看到这样的情况：

```console
$ ls
CODE_OF_CONDUCT.md   LICENSE-APACHE       examples
CONTRIBUTING.md      LICENSE-MIT          proptest-regressions
Cargo.lock           README.md            src
Cargo.toml           convey_derive        target
```

因为这种样式是为人类设计的，在大多数配置中，它甚至会打印一些名称（例如`src`）以彩色显示它们是目录。如果您改为将此管道传输到文件或类似的程序`cat`，`ls`将调整其输出。它将在自己的行上打印每个条目，而不是使用适合我的终端窗口的列。它也不会发出任何颜色。

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

## 简单的机器输出格式

历史上，生成的唯一类型的输出命令行工具是字符串。对于那些在终端前能够阅读文本和理解其含义的人来说，这通常是很好的。但是，其他程序通常没有这种能力：它们理解类似工具输出的唯一方法`ls`如果程序的作者包含一个解析器`ls`输出。

这通常意味着输出仅限于易于解析的内容。像TSV（制表符分隔值）这样的格式非常流行，其中每个记录都在自己的行上，并且每一行包含制表符分隔的内容。这些基于文本行的简单格式允许`grep`用于输出工具，如`ls`. `| grep Cargo`不管你的台词是不是来自`ls`或者文件，它将逐行过滤。

缺点是你不能用简单的`grep`调用以筛选`ls`给了你。为此，每个目录项都需要携带额外的数据。

## 机器的JSON输出

制表符分隔值是输出结构化数据的一种简单方法，但它要求另一个程序知道期望哪个字段（以及顺序），并且很难输出不同类型的消息。例如，假设我们的程序想向消费者发送消息，告诉他们它当前正在等待下载，然后输出一条消息，描述它得到的数据。这些是非常不同的类型的消息，试图在TSV输出中统一它们需要我们发明一种方法来区分它们。同样，当我们想要打印包含两个不同长度项目列表的消息时。

不过，最好选择一种在大多数编程语言/环境中易于理解的格式。因此，在过去的几年中，许多应用程序获得了在[杰森]. 很简单，几乎每种语言都存在解析器，但其强大程度足以在许多情况下发挥作用。虽然它是一种人类可以读取的文本格式，但许多人也在处理那些在解析JSON数据和将数据序列化为JSON方面速度非常快的实现。

[json]: https://www.json.org/

在上面的描述中，我们已经讨论过程序正在编写“消息”。这是一种考虑输出的好方法：程序不一定只输出一个数据块，但实际上在运行时可能会发出许多不同的信息。在输出JSON时，支持这种方法的一个简单方法是为每条消息编写一个JSON文档，并将每个JSON文档放到新行（有时调用[行分隔JSON][jsonlines]）这可以使实现像使用常规的`println!`.

[jsonlines]: https://en.wikipedia.org/wiki/JSON_streaming#Line-delimited_JSON

下面是一个简单的例子，使用`json!`宏来自[塞尔德·杰森_]要在您的Rust源代码中快速编写有效的JSON，请执行以下操作：

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

（正在运行`cargo`具有`-q`抑制其正常输出。后面的参数`--`被传递到我们的程序。）

### 实例：ripgrep

*[瑞普瑞普]*是替代*格雷普*或*银*用铁锈写的。默认情况下，它将生成如下输出：

[ripgrep]: https://github.com/BurntSushi/ripgrep

```console
$ rg default
src/lib.rs
37:    Output::default()

src/components/span.rs
6:    Span::default()
```

但是给出`--json`它将打印：

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

如您所见，每个JSON文档都是一个包含`type`字段。这将允许我们为`rg`当这些文档进入并显示匹配项（以及它们所在的文件）时，即使在*瑞普瑞普*仍在搜索。

<aside>

**旁白：**这是Visual Studio代码使用的方式*瑞普瑞普*它的代码搜索。

</aside>

## 对人和机器输出的摘要

[传达]是一个正在开发的库，它试图使以适合人类和机器的格式输出消息更容易。您定义自己的消息类型，并实现`Render`特征（手动，在宏的帮助下，或者使用派生属性）来说明它们应该如何格式化。目前，它支持打印人工输出（包括自动检测是否应该着色）、编写JSON文档（或`stdout`或者同时指向一个文件）。

[convey]: https://crates.io/crates/convey

即使您不使用这个库，编写一个适合您的用例的类似抽象也是一个好主意。

## 如何处理流入我们的输入

<aside class="todo">

**TODO：**讨论如何使用stdin（请参见[α95](https://github.com/rust-lang-nursery/cli-wg/issues/95)）

</aside>
