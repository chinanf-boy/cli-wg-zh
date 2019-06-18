# 测试

经过几十年的软件开发，人们发现了一个事实：未经测试的软件很少工作。（许多人甚至会说：“大多数测试过的软件也不能工作。”但我们都是乐观主义者，对吧？）因此，为了确保您的程序完成您期望它做的，测试它是明智的。

一个简单的方法是写一个`README`描述程序应执行的操作的文件。当您准备好发布新版本时，请通过`README`确保行为仍如预期。您也可以通过写下程序对错误输入的反应，使这成为一个更严格的练习。

还有一个好主意：写下来`README`在你写代码之前。

<aside>

**旁白：**看看[test-driven development](TDD)如果你没听说过的话。

[test-driven development]: https://en.wikipedia.org/wiki/Test-driven_development

</aside>

## 自动化测试

现在，这一切都很好，很花哨，但要手工做这些吗？这可能需要很多时间。与此同时，许多人开始喜欢告诉计算机为他们做事情。我们来谈谈如何自动化这些测试。

Rust有一个内置的测试框架，所以我们从编写第一个测试开始：

```rust,ignore
#[test]
fn check_answer_validity() {
    assert_eq!(answer(), 42);
}
```

你可以把这段代码放到几乎所有的文件中，`cargo test`会找到并运行它。这里的钥匙是`#[test]`属性。它允许构建系统发现这些函数，并将它们作为测试运行，以验证它们不会惊慌失措。

<aside class="exercise">

**读者练习：**使此测试工作。

您应该以如下输出结束：

```text
running 1 test
test check_answer_validity ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

</aside>

现在我们看到了*怎样*我们可以编写测试，我们还需要弄清楚*什么*测试。正如您所看到的，编写函数断言相当容易。但是，CLI应用程序通常不止一个函数！更糟糕的是，它经常处理用户输入、读取文件和写入输出。

## 使代码可测试

测试功能有两种互补的方法：测试构建完整应用程序的小单元，这些方法称为“单元测试”。还有“从外部”测试最终应用程序，称为“黑盒测试”或“集成测试”。让我们从第一个开始。

为了弄清楚我们应该测试什么，让我们看看我们的程序特性是什么。主要是，`grrs`应该打印出与给定图案匹配的线条。那么，让我们编写单元测试*正是这个*：我们希望确保我们最重要的一段逻辑能够正常工作，并且我们希望这样做的方式不依赖于我们周围的任何设置代码（例如处理cli参数）。

回到我们的[first implementation](impl-draft.md)属于`grrs`，我们将此代码块添加到`main`功能：

```rust,ignore
// ...
for line in content.lines() {
    if line.contains(&args.pattern) {
        println!("{}", line);
    }
}
```

遗憾的是，这并不容易测试。首先，它在主函数中，所以我们不能轻易地调用它。通过将这段代码移动到函数中，可以很容易地解决这一问题：

```rust,no_run
fn find_matches(content: &str, pattern: &str) {
    for line in content.lines() {
        if line.contains(pattern) {
            println!("{}", line);
        }
    }
}
```

现在我们可以在测试中调用这个函数，看看它的输出是什么：

```rust,ignore
#[test]
fn find_a_match() {
    find_matches("lorem ipsum\ndolor sit amet", "lorem");
    assert_eq!( // uhhhh
```

或者……我们可以吗？马上，`find_matches`直接打印到`stdout`即终端。我们不能轻易地在测试中捕捉到它！这是在实现之后编写测试时经常出现的一个问题：我们已经编写了一个函数，它与使用它的上下文紧密集成在一起。

<aside class="note">

**注：**在编写小型CLI应用程序时，这完全可以。没有必要让所有东西都可以测试！但是，重要的是要考虑代码的哪些部分可能需要编写单元测试。虽然我们将看到很容易将此函数更改为可测试的，但情况并非总是如此。

</aside>

好吧，我们怎样才能让这个可以测试？我们需要以某种方式捕获输出。Rust的标准库有一些处理I/O（输入/输出）的简洁抽象，我们将使用一个[`std::io::Write`]. 这是一个[特质][trpl-traits]它抽象了我们可以写的东西，包括字符串`stdout`.

[trpl-traits]: https://doc.rust-lang.org/book/ch10-02-traits.html

[`std::io::write`]: https://doc.rust-lang.org/1.31.0/std/io/trait.Write.html

如果这是你第一次在生锈的情况下听到“性状”，那你就要接受治疗了。性状是锈病最强大的特征之一。你可以把它们想象成Java中的接口，或者在Haskell中键入类（无论你熟悉什么）。它们允许您抽象不同类型可以共享的行为。使用特性的代码可以非常通用和灵活的方式表达思想。这意味着它也很难阅读。不要让这种情况吓倒你：即使是多年来一直使用生锈的人也不总是能立即得到通用代码的功能。在这种情况下，它有助于思考具体的用途。例如，在我们的例子中，我们抽象的行为是“写给它”。实现（“impl”）的类型的示例包括：终端的标准输出、文件、内存中的缓冲区或TCP网络连接。（向下滚动[文件`std::io::Write`][`std::io::write`]查看“实现者”列表。）

有了这些知识，让我们改变函数来接受第三个参数。它应该是实现`Write`. 这样，我们就可以在测试中提供一个简单的字符串，并对其进行断言。以下是我们如何编写此版本的`find_matches`：

```rust,ignore
{{#include testing/src/main.rs:25:31}}
```

新参数是`mut writer`也就是说，我们称之为“作家”的易变事物。它的类型是`impl std::io::Write`，可以将其读取为“实现`Write`特点”。还要注意我们如何替换`println!(…)`我们以前用过`writeln!(writer, …)`. `println!`工作原理与`writeln!`但总是使用标准输出。

现在我们可以测试输出：

```rust,ignore
{{#include testing/src/main.rs:33:38}}
```

要在我们的应用程序代码中使用它，我们必须将调用更改为`find_matches`在里面`main`通过添加[`&mut std::io::stdout()`][stdout]作为第三个参数。下面是一个主要功能的例子，它建立在我们在前几章中所看到的基础上，并使用我们提取的`find_matches`功能：

```rust,ignore
{{#include testing/src/main.rs:15:23}}
```

[stdout]: https://doc.rust-lang.org/1.31.0/std/io/fn.stdout.html

<aside class="note">

**注：**自从`stdout`需要字节（不是字符串），我们使用`std::io::Write`而不是`std::fmt::Write`. 因此，在我们的测试中，我们将空向量作为“编写器”（其类型将推断为`Vec<u8>`）中的`assert_eq!`我们使用`b"foo"`. (The `b`前缀使其成为*字节字符串文本*所以它的类型是`&[u8]`而不是`&str`）

</aside>

<aside class="note">

**注：**我们还可以使这个函数返回`String`但这会改变它的行为。它不再直接写入终端，而是将所有内容收集到一个字符串中，并在末尾一次性转储所有结果。

</aside>

<aside class="exercise">

**读者练习：**
[`writeln!`]返回[`io::Result`]因为写入可能会失败，例如当缓冲区已满且无法扩展时。向添加错误处理`find_matches`.

[`writeln!`]: https://doc.rust-lang.org/1.31.0/std/macro.writeln.html

[`io::result`]: https://doc.rust-lang.org/1.31.0/std/io/type.Result.html

</aside>

我们刚刚看到了如何使这段代码易于测试。我们有

1.  确定了我们应用程序的核心部分之一，
2.  把它放在自己的功能里，
3.  使其更加灵活。

尽管我们的目标是让它成为可测试的，但最终得到的结果实际上是一段非常惯用和可重用的生锈代码。太棒了！

## 将代码拆分为库和二进制目标

我们可以在这里再做一件事。到目前为止，我们已经把我们写的所有东西`src/main.rs`文件。这意味着我们当前的项目生成一个二进制文件。但我们也可以将代码作为库提供，如下所示：

1.  把`find_matches`函数转换为新的`src/lib.rs`.
2.  添加一个`pub`在`fn`（所以`pub fn find_matches`）使它成为我们图书馆的用户可以访问的东西。
3.  去除`find_matches`从`src/main.rs`.
4.  在`fn main`，提前呼叫`find_matches`具有`grrs::`，所以现在`grrs::find_matches(…)`. 这意味着它使用了我们刚刚编写的库中的函数！

Rust处理项目的方式是相当灵活的，尽早考虑将什么放入您的板条箱的库中是一个好主意。例如，您可以考虑先为特定于应用程序的逻辑编写一个库，然后像其他库一样在CLI中使用它。或者，如果您的项目有多个二进制文件，您可以将公共功能放在板条箱的库部分。

<aside class="note">

**注：**说到把所有的东西`src/main.rs`如果我们继续这样做，就会变得难以阅读。这个[模块化系统]可以帮助您构造和组织代码。

[module system]: https://doc.rust-lang.org/1.31.0/book/ch07-00-packages-crates-and-modules.html

</aside>

## 通过运行CLI应用程序来测试它们

到目前为止，我们已经竭尽全力测试*业务逻辑*我们的申请结果是`find_matches`功能。这是非常有价值的，也是向经过良好测试的代码库迈出的第一步。（通常，这类测试称为“单元测试”。）

但是，我们没有测试很多代码：我们为处理外部世界而编写的所有代码！假设您编写了主函数，但意外地留在硬编码字符串中，而不是使用用户提供的路径的参数。我们也应该为此编写测试！（这一级别的测试通常称为“集成测试”，或“系统测试”。）

在其核心，我们仍然在编写函数并用`#[test]`. 这只是我们在这些函数中做什么的问题。例如，我们希望使用项目的主二进制文件，并像常规程序一样运行它。我们还将把这些测试放入新目录中的新文件中：`tests/cli.rs`.

<aside>

**旁白：**按照惯例，`cargo`将在中查找集成测试`tests/`目录。同样，它将在`benches/`和中的示例`examples`。这些约定还扩展到您的主要源代码：库具有`src/lib.rs`文件，主二进制文件是`src/main.rs`或者，如果有多个二进制文件，则cargo希望它们位于`src/bin/<name>.rs`. 遵循这些约定将使阅读rust代码的人更容易发现您的代码库。

</aside>

回想起来，`grrs`是一个在文件中搜索字符串的小工具。我们之前已经测试过可以找到匹配项。让我们考虑一下我们可以测试哪些其他功能。

这是我想出来的。

-   当文件不存在时会发生什么？
-   当没有匹配项时，输出是什么？
-   当我们忘记一个（或两个）参数时，程序是否会出错退出？

这些都是有效的测试用例。此外，我们还应该为“快乐路径”包含一个测试用例，即我们找到至少一个匹配项并打印出来。

为了使这些测试更简单，我们将使用[`assert_cmd`]机箱。它有一堆整洁的助手，允许我们运行我们的主二进制文件并查看它的行为。此外，我们还将添加[`predicates`]有助于我们写断言的板条箱`assert_cmd`可以测试（并且有很大的错误消息）。我们将不将这些依赖项添加到主列表，而是添加到我们的`Cargo.toml`. 只有在开发板条箱时才需要它们，而不是在使用板条箱时。

```toml
{{#include testing/Cargo.toml:12:14}}
```

[`assert_cmd`]: https://docs.rs/assert_cmd

[`predicates`]: https://docs.rs/predicates

这听起来像是很多设置。不过，让我们直接投入并创造`tests/cli.rs`文件：

```rust,ignore
{{#include testing/tests/cli.rs:1:15}}
```

您可以使用运行此测试`cargo test`只是我们上面写的测试。第一次可能需要更长的时间，比如`Command::main_binary()`需要编译主二进制文件。

## 生成测试文件

测试我们刚刚看到的只是检查食物

我们需要有一个文件的内容我们知道,s*应该*返回并检查这个期望在我们的代码。

在[`tempfile`]板条箱。`dev-dependencies`让我们将它添加到`Cargo.toml`:

```toml
{{#include testing/Cargo.toml:15}}
```

[`tempfile`]: https://docs.rs/tempfile/3/tempfile/

这是一个新的测试用例,你可以写在下面`file`超出范围(最后的函数),t

```rust,ignore
{{#include testing/tests/cli.rs:17:34}}
```

<aside class="exercise">

**读者的练习:**为传递一个空字符串添加集成测试

</aside>

## 测试什么?

尽管它当然可以写集成很有趣

总的来说这是一个好主意写集成t

这也是一个好主意不要重点测试`--help`因为它是为您生成。

相反,你可能居[`proptest`].[如果你有一个程序使用任意f]在边界情况发现bug。

[`proptest`]: https://docs.rs/proptest

[fuzzer]: https://fuzz.rs/book/introduction.html

<aside>

**旁白:**你可以找到完整的、可运行的源代码[在这本书的库中][src]。

[src]: https://github.com/rust-lang-nursery/cli-wg/tree/master/src/tutorial/testing

</aside>
