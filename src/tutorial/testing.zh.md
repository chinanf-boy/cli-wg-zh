# 测试

经过几十年的软件开发，人们发现了一个事实：未经测试的软件不会工作。（许多人甚至会说：“大多数测试过的软件也不会工作。”但我们都是乐观主义者，对吧？）因此，为了确保您的程序完成您期望它做的，测试它是明智的选择。

一个简单的方法是写一个`README`文件，描述程序应执行的操作。当您准备好发布新版本时，请通过`README`确保行为仍如预期。您也可以写下程序对错误输入的反应，把这当成一个更严格的练习。

还有一个好主意：在你写代码之前，就构思`README`。

<aside>

**旁白：** 可以看看[测试驱动开发](TDD)，如果你还没头绪的话。

[test-driven development]: https://en.wikipedia.org/wiki/Test-driven_development

</aside>

## 自动化测试

现在，这一切都变得很好，且花式多样，但要手工做这些吗？那可能需要很多时间。与此同时，许多人开始喜欢告诉计算机，帮他们做事情。我们来谈谈如何自动化这些测试。

Rust 有一个内置的测试框架，所以我们从编写第一个测试开始：

```rust,ignore
#[test]
fn check_answer_validity() {
    assert_eq!(answer(), 42);
}
```

你可以把这段代码放到几乎所有的文件中，`cargo test`会找到并运行它。这里的关键是`#[test]`属性。它允许'构建系统'发现这些函数，并将它们作为测试运行，以验证它们不会恐慌崩溃。

<aside class="exercise">

**读者练习：** 使此测试工作。

您应该以如下输出结束：

```text
running 1 test
test check_answer_validity ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

</aside>

现在我们看到了我们可以*怎样*编写测试，我们还需要弄清楚要测试*什么*。正如您所看到的，编写函数断言(assert)相当容易。但是，CLI 应用程序通常不止一个函数！更糟糕的是，它经常需要处理用户输入、读文件和写输出。

## 使代码具备可测试性

测试功能有两种互补的方法：测试构建完整应用程序的小单元，这些方法称为“单元测试”。还有“从外部”测试**最终**应用程序，称为“黑盒测试”或“集成测试”。让我们从第一个开始。

为了弄清楚我们应该测试什么，让我们看看我们的程序功能是什么。`grrs`主要的工作就是，应该打印出与给定(文本)模式匹配的行。那么，让我们为这个(功能)编写单元测试：我们希望确保我们最重要的一段逻辑代码能够正常工作，并且我们希望这，以一种不会依赖于周围的任何设置代码的方式进行（例如，处理 cli 参数的代码）。

回到我们`grrs`的[首次实现一文](impl-draft.zh.md)，我们将此代码块添加到`main`函数：

```rust,ignore
// ...
for line in content.lines() {
    if line.contains(&args.pattern) {
        println!("{}", line);
    }
}
```

遗憾的是，这并不容易测试。首先，它在 main 函数中，所以我们不能轻易地调用它。通过将这段代码移动到一个函数（本例子是`find_matches`）中，可以很容易地解决这一问题：

```rust,no_run
fn find_matches(content: &str, pattern: &str) {
    for line in content.lines() {
        if line.contains(pattern) {
            println!("{}", line);
        }
    }
}
```

现在，我们可以在测试中调用这个函数，看看它的输出是什么：

```rust,ignore
#[test]
fn find_a_match() {
    find_matches("lorem ipsum\ndolor sit amet", "lorem");
    assert_eq!( // uhhhh
```

或者… 我们可以吗？现在嘛，`find_matches`会直接打印到`stdout`，即终端上显示。我们不太能在测试中轻易捕捉它！这是在先实现，后编写测试时，经常出现的一个问题：因我们已经编写了一个函数，而它与使用它的上下文紧密集成(或是粘连)在一起。

<aside class="note">

**注：** 在编写小型 CLI 应用程序时，这完全可以。没有必要所有东西都可以测试！而且，重要的是要考虑代码的哪些部分，可能需要编写单元测试。虽然我们(似乎)看到，可以很容易将此函数更改为可测试的(闭环状态)，但情况并非总是如此。

</aside>

好吧，我们怎样才能让这个变得可测试？我们需要以某种方式捕获输出。Rust 的标准库有一些处理 I/O（输入/输出）的简洁抽象，我们将使用一个[`std::io::Write`]。这是一个[trait][trpl-traits]，它抽象了我们可以写(write)的东西，包括字符串,还有`stdout`。

[trpl-traits]: https://doc.rust-lang.org/book/ch10-02-traits.html
[`std::io::write`]: https://doc.rust-lang.org/1.31.0/std/io/trait.Write.html

如果这是接触 rust 后，你第一次听到“trait”（或特质，即行为接口），那你就要认真对待了。trait 是 Rust 最强大的功能之一。你可以把它们想象成 Java 中的接口，或者在 Haskell 中类型类（无论你熟悉什么）。它们允许您抽象不同类型可以共享的行为。使用 trait 的代码可用非常通用和灵活的方式表达思想。这意味着，它也很难（以正常人类的顺序）阅读。但不要让这种情况吓倒你：即使是多年来一直使用 rust 的人，也不总是能立即得到通用代码的行为。在这种情况下，它有助于思考具体的用途。例如，在我们的例子中，我们抽象的行为是“写它”。要实现（“impl”）这个类型的示例包括：终端的标准输出、文件、内存中的缓冲区或 TCP 网络连接。（向下滚动到[文件`std::io::Write`][`std::io::write`]，查看“实现人员”名单。）

有了这些知识，让我们改变函数，接受第三个参数。而它应该是实现`Write`的。 这样，我们就可以在测试中，提供一个简单的字符串，并对其进行断言。以下是我们编写的这一版本的`find_matches`：

```rust,ignore
{{#include testing/src/main.rs:25:31}}
```

新参数是`mut writer`也就是说，我们称之为“writer”的可变(mut)事物。它的类型是`impl std::io::Write`，可以将理解为“实现`Write`trait”。还要注意我们如何将我们以前用的`println!(…)`，替换成`writeln!(writer, …)`。`println!`工作原理与`writeln!`相同，只是输出的对象总是标准输出。

现在，我们可以测试输出：

```rust,ignore
{{#include testing/src/main.rs:33:38}}
```

要在我们的应用程序代码中使用它，我们必须在`main`中，通过添加[`&mut std::io::stdout()`][stdout]作为第三个参数，改成调用`find_matches`。下面是一个 main 函数的例子，它建立在我们在前几章中所看到的基础上，并使用我们提取的`find_matches`函数：

```rust,ignore
{{#include testing/src/main.rs:15:23}}
```

[stdout]: https://doc.rust-lang.org/1.31.0/std/io/fn.stdout.html

<aside class="note">

**注：** 因为`stdout`需要字节(byte)（不是字符串），我们使用`std::io::Write`而不是`std::fmt::Write`。 因此，在我们的测试中，我们把空向量赋给“writer”（其类型将推断为`Vec<u8>`），而在`assert_eq!`我们使用一个`b"foo"`。(`b`前缀使其成为*字节字符串文本*，以它的类型是`&[u8]`，而不是`&str`。）

</aside>

<aside class="note">

**注：** 我们还可以使这个函数返回一个`String`，但这会改变它的行为。我们选择不再直接写入终端，而是将所有内容收集到一个字符串中，并在末尾一次性转储所有结果。

</aside>

<aside class="exercise">

**读者练习：** 
[`writeln!`]返回一个[`io::Result`]，因为写入可能会失败，例如当缓冲区已满且无法扩展时。向`find_matches`添加错误处理。

[`writeln!`]: https://doc.rust-lang.org/1.31.0/std/macro.writeln.html
[`io::result`]: https://doc.rust-lang.org/1.31.0/std/io/type.Result.html

</aside>

我们刚刚看到了如何使这段代码变得容易测试。我们已经

1.  确定了我们应用程序的核心部分之一，
2.  把它放在自己的函数里，
3.  使其更加灵活。

尽管我们的目标是让它成为可测试的，但最终得到的结果，实际上是一段非常惯用和可重用的 rust 代码。真好啊！

## 将代码拆分为库和二进制目标

我们可以在这里再做一件事。到目前为止，我们已经把我们写的所有东西，都放在了`src/main.rs`文件。这意味着我们当前的项目，(构建)会生成一个二进制文件。但我们也可以将代码作为库提供，如下所示：

1.  把`find_matches`函数放到新的`src/lib.rs`。
2.  添加一个`pub`在`fn`（也就是`pub fn find_matches`）使它成为用户可以访问的东西。
3.  从`src/main.rs`中，去除`find_matches`。
4.  在`fn main`，提前调用`grrs::`的`find_matches`，所以现在变为`grrs::find_matches(…)`. 这意味着它使用了我们刚刚编写的库中的函数！

Rust 处理项目的方式是相当灵活的，尽早考虑将什么放入您的箱子的库中是一个好观念。例如，您可以考虑先为应用程序的特定逻辑编写一个库，然后像其他库一样在 CLI 中使用它。或者，如果您的项目有多个二进制文件，您可以将公共函数放在箱子的库部分。

<aside class="note">

**注：** 说到如果我们继续，把所有的东西都堆到`src/main.rs`，它就会变得难以阅读。这个[模块化系统][module system]可以帮助您构造和组织代码。

[module system]: https://doc.rust-lang.org/1.31.0/book/ch07-00-packages-crates-and-modules.html

</aside>

## 通过运行 CLI 应用程序来测试它们

到目前为止，我们已经竭尽全力测试我们应用的*业务逻辑*，结果就是搞出了`find_matches`函数。这是非常有价值的，也是向经过良好测试的代码库，迈出的第一步。（通常，这类测试称为“单元测试”。）

但是，我们还有很多代码没有测试：我们为真实世界所编写的所有处理代码！假设您编写了 main 函数，但意外地留有一个硬编码字符串，而不是使用用户提供的路径参数。我们就应该为此编写测试！（这一级别的测试通常称为“集成测试”，或“系统测试”。）

在其核心，我们编写的(测试)函数还是用`#[test]`声明。重要的只是我们在这些函数中做什么的问题。例如，我们希望使用项目的 main 二进制文件，并像常规程序一样运行它。我们还将把这些测试，放入新目录中的新文件中：`tests/cli.rs`。

<aside>

**旁白：** 按照惯例，`cargo`将在`tests/`目录中查找集成测试。同样，它也会看`benches/`中的基准测试和`examples`中的示例测试。这些(测试)约定还扩展到了您主要的源代码：比如库可以有一个`src/lib.rs`文件，main 二进制文件是`src/main.rs`或者，如果有多个二进制文件，则 cargo 希望它们位于`src/bin/<name>.rs`. 遵循这些约定将使阅读 rust 代码的人更明了您的代码库。

</aside>

回想起来，`grrs`是一个在文件中，搜索字符串的小工具。我们之前已经测试了匹配项的查找。让我们考虑一下，我们还可以测试哪些其他功能。

这是我想出来的。

- 当文件不存在时，会发生什么？
- 当没有匹配项时，输出是什么？
- 当我们忘记一个（或两个）参数时，程序是否会出错退出？

这些都是有效的测试用例。此外，我们还应该为“正确路径”包含一个测试用例，即我们找到至少一个匹配项，并打印出来。

为了使这些测试更简单，我们将使用[`assert_cmd`]箱子。它有一堆整洁的助手，允许我们运行我们的主要二进制文件，并查看它的行为。此外，我们还将添加[`predicates`]箱子，有助于`assert_cmd`的断言测试（并且有很大的错误消息）。我们不会将这些依赖项，添加到依赖主列表，而是添加到我们的`Cargo.toml`中的"dev dependencies"(开发依赖)部分。只有在开发箱子时才需要它们，而不是在使用箱子时。

```toml
{{#include testing/Cargo.toml:12:14}}
```

[`assert_cmd`]: https://docs.rs/assert_cmd
[`predicates`]: https://docs.rs/predicates

这听起来像是很多设置。不过，让我们直接上手，创建`tests/cli.rs`文件：

```rust,ignore
{{#include testing/tests/cli.rs:1:15}}
```

您可以使用`cargo test`运行此测试，命令只是我们上面写的测试。第一次可能需要更长的时间，比如`Command::main_binary()`需要编译你的主要二进制文件。

## 生成测试文件

测试我们刚刚看到的只是检查我们程序写出的一个错误信息，表示输入文件不存在。
这是个要具备的重要测试，但可能不是最重要的那个。
让我们现在测试下，在文件中，实际找到的匹配项吧。

我们需要有一个文件，它的内容是我们知道的，这样我们才知道我们程序*应该*返回的正确匹配项，并在我们的测试中检查这个期望。一个想法是添加一个具有自定义内容的文件到项目，并在测试中使用。另一个则是在测试中创建临时文件。鉴于我们教程的情况，我们会搞下后者。具体来说，因为它更灵活，还可以用在其他案例；例如说，要测试一个改变文件的程序。

要创建这些个临时文件，我们会用到[`tempfile`]箱子。让我们将它添加到`Cargo.toml`的`dev-dependencies`:

```toml
{{#include testing/Cargo.toml:15}}
```

[`tempfile`]: https://docs.rs/tempfile/3/tempfile/

这是一个新的测试用例(，你可以在下面写另一个)，先创建一个临时文件(，命名它我们才能获得它的路径)，填充些文本，然后运行我们的程序，看看是不是正确输出。当`file` 走出作用域（函数的结尾），实际的临时文件会自动删除。

```rust,ignore
{{#include testing/tests/cli.rs:17:34}}
```

<aside class="exercise">

**读者的练习:**添加一个集成测试，题目是传递一个空字符串作为匹配模式。调整程序通过测试。

</aside>

## 测试什么?

尽管，编写集成测试确实有趣，还也是要花上不少时间，还要让测试持续跟上测试的变化。为了让你时间是花得更有意义，你应该三思下，你应该测试什么。

总的来说，一个的想法是，为所有用户注意到的行为类型，编写集成测试。
这意味着，你不需要覆盖所有的边缘情况。对于不同的类型，通常有一些例子就足够了，
还可以依靠单元测试来覆盖边缘情况。

最好不要把你的测试集中在你不能主动控制的事情上。
测试`--help`的确切布局是个坏主意，
因为它是为你生成/服务的。
相反，您可能只想检查某些元素是否存在。

根据你程序的本质，你可以尝试更多的测试技术。比如说，如果你已经提取了部分程序，并且想试着为(部分程序)所有的边缘情况，编写许多示例案例作为单元测试的话，那你应该看看[`proptest`]。又或是，你有一个消化任意文件并解析它们的程序，那就试着编写一个[fuzzer]，在边缘发现 bug。

[`proptest`]: https://docs.rs/proptest
[fuzzer]: https://fuzz.rs/book/introduction.html

<aside>

**旁白:**你可以找到本章节，完整的、可运行的源代码，它[在这本书的库中][src]。

[src]: https://github.com/rust-lang-nursery/cli-wg/tree/master/src/tutorial/testing

</aside>
