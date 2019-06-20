# 打包和分发一个 Rust 工具

如果你相信你的项目已经准备好了给人用了，那么是时候打包和发布了。

我们会有三种方法，并从”最快设置的“开始说明，直到”用户最方便的“。

## 最快的:`cargo publish`

发布您的应用程序的最简单方法就是用 Cargo。还记得如何在项目添加外部依赖吗？Cargo 会从箱子仓库[crates.io]下载它们。通过`cargo publish`，你也可以轻松把箱子发布到[crates.io]。对所有的箱子都适用，还包括那些二进制目标文件。

发布一个箱子到[crates.io]很简单：如果你还没有试过，那要先在[crates.io]创建一个账号。目前，可以通过 Github 账号授权完成，所以你会需要一个 Github 账号(并登陆)。下一步，通过在本地机器，使用 cargo 登陆，而要做到这一步，你要去到[crates.io 账户页面][crates.io account page]创建一个新的令牌，然后运行`cargo login <your-new-token>`。每台电脑只需要做这一次。你可以在 cargo 的[发布指南][publishing guide]上，了解更多。

现在，你知道了 cargo 还有 crates.io。那你就准备好发布箱子了。在你匆忙发布一个新箱子之前，请确保你为你的箱子，添加了必要的元信息。你可以在[Cargo 的清单格式][cargo's manifest format]上，找到所有你能设置的字段。下面有个常用字段的快速预览。

```toml
[package]
name = "grrs"
version = "0.1.0"
authors = ["Your Name <your@email.com>"]
license = "MIT OR Apache-2.0"
description = "A tool to search files"
readme = "README.md"
homepage = "https://github.com/you/grrs"
repository = "https://github.com/you/grrs"
keywords = ["cli", "search", "demo"]
categories = ["command-line-utilities"]
```

<aside class="note">

**注：**这个例子包括强制的许可证字段，是给 Rust 项目的常见选择：该许可证还用于 Rust 编译器本身。它还指向一个`README.md`文件。它应该包括对您的项目是关于什么的一个快速描述，并且不仅会在您箱子的 crates.io 页面上，而且还包括 Github 默认情况下在存储库页面上显示的。

</aside>

[crates.io]: https://crates.io/
[crates.io account page]: https://crates.io/me
[publishing guide]: https://doc.rust-lang.org/1.31.0/cargo/reference/publishing.html
[cargo's manifest format]: https://doc.rust-lang.org/1.31.0/cargo/reference/manifest.html

### 如何从 crates.io 安装二进制文件

我们已经看到了如何将箱子发布到 crates.io，您可能想知道如何安装它。与库不同的是，当你运行`cargo build`（或类似的命令）的时候， Cargo 会为你下载和编译哪些，您需要显式告诉它安装二进制文件。

这是用`cargo install <crate-name>`完成的。 默认情况下，它将下载箱子，编译它包含的所有二进制目标文件（“发布”模式，因此可能需要一段时间），并将它们复制到`~/.cargo/bin/`目录。（确保您的 shell ，知道在那里查找二进制文件！）

也可以从 Git 存储库安装箱子，只安装一个箱子的特定二进制文件，并指定一个备选目录来安装它们。查看`cargo install --help`有关详细信息。

### 何时使用

`cargo install`是发布二进制箱子的简单方法。Rust 开发人员使用起来非常方便，但也有一些明显的缺点：因为它总是从头开始编译源代码，所以您的工具的用户将需要拥有 Rust、Cargo 以及您的项目需要的所有其他系统依赖项，都要安装在用户机器上。编写大型 Rust 代码库也需要一些时间。

此外，没有简单的方法来更新用 cargo 安装的工具：用户需要在某个时刻运行`cargo install`，并通过`--force`覆盖旧二进制文件的标志。这是一个[missing 功能][cargo-issue-2082]，不过还有[像这个一样][cargo-update]子命令，您可以通过安装来添加它。

[cargo-issue-2082]: https://github.com/rust-lang/cargo/issues/2082
[cargo-update]: https://crates.io/crates/cargo-update

最好将其用于分发，针对其他 Rust 开发人员的工具。例如：有很多 Cargo 的子命令 `cargo-tree`或`cargo-outdated`，可以和它一起安装。

## 分发二进制文件

Rust 是一种编译为本机代码的语言，默认情况下静态链接所有依赖项。当你在`grrs`项目中运行`cargo build`，那么它会(构建)一个`grrs`二进制文件。尝试一下：使用`cargo build`，构建在`target/debug/grrs`。而当你运行`cargo build --release`，则会构建在`target/release/grrs`。除非使用显式需要在目标系统上，安装外部库的箱子（如使用系统的 OpenSSL 版本），否则此二进制文件将仅依赖于公共系统库。这就是说，你只需要把一个文件，发送给和你运行相同操作系统的人，他们就可以运行它了。

这已经很强大了！它可以解决我们刚才在`cargo install`上看到的两个缺点：不需要在用户的机器上安装 Rust，他们可以立即运行二进制文件，而不需要花一分钟来编译。

所以，正如我们所看到的，`cargo build` *已经*为我们构建二进制文件。唯一的问题是，它们不能保证在所有平台上都能工作。如果`cargo build`在您的 Windows 机器上，默认情况下，就不会得到在 Mac 上工作的二进制文件。有没有一种方法，可以为所有感兴趣的平台自动生成这些二进制文件？

### 在 CI 上构建二进制发布

如果您的工具是开源的，并且托管在 GitHub 上，那么很容易建立一个免费的 CI（持续集成）服务，比如[Travis CI]。（还有其他服务也可以在其他平台上工作，但 Travis 非常流行。）这基本上是，在每次将更改推送到存储库时，就在虚拟机中，运行所设置命令。这些命令是什么，以及它们运行的机器类型都是可配置的。例如：一个好主意是在机器上运行`cargo test`，配有 Rust 和一些常用的工具。如果失败，您就知道最近的更改中存在问题。

[travis ci]: https://travis-ci.com/

我们还可以使用它，来构建二进制文件，并将它们上载到 Github！事实上，如果我们运行`cargo build --release`，并把二进制文件上传到某个地方，那么我们应该都设置好了，对吗？不完全是。我们仍然需要确保我们构建的二进制文件，与尽可能多的系统兼容。例如，在 Linux 上，我们可以不设为当前系统编译，而是为`x86_64-unknown-linux-musl`目标，不依赖默认系统库。在 MacOS 上，我们可以设置`MACOSX_DEPLOYMENT_TARGET`到`10.7`，仅依赖于 10.7 及更高版本中的系统功能。

您可以看到使用这种方法构建二进制文件的一个示例，[这是][wasm-pack-travis]针对 Linux 和 MacOS ，以及[这是][wasm-pack-appveyor]针对 Windows（使用 AppVeyor）。

[wasm-pack-travis]: https://github.com/rustwasm/wasm-pack/blob/51e6351c28fbd40745719e6d4a7bf26dadd30c85/.travis.yml#L74-L91
[wasm-pack-appveyor]: https://github.com/rustwasm/wasm-pack/blob/51e6351c28fbd40745719e6d4a7bf26dadd30c85/.appveyor.yml

另一种方法是使用预构建（Docker）映像，其中包含构建二进制文件所需的所有工具。这也使我们能够轻松瞄准，更具异国情调的平台。这个[trust]Project 包含的脚本，可以 include 你的项目，以及有关如何设置的说明。它还包括对使用 AppVeyor 的 Windows 的支持。

如果您希望在本地设置它，并在您的计算机上生成发布文件，那么您仍然可以查看 trust。它在内部使用[cross]，它的工作原理类似于 Cargo，但将命令转发给 Docker 容器内的 Cargo 处理。镜像的定义也在[cross 的存储库][cross]可用。

[trust]: https://github.com/japaric/trust
[cross]: https://github.com/rust-embedded/cross

### 如何安装这些二进制文件

你把你的用户指向你的发布页面，[像这个一样][wasm-pack-release]。他们就可以下载我们刚刚创建的工件。我们刚刚生成的发布工件，没有什么特别的：最后，它们只是包含我们的二进制文件的存档文件！这意味着，工具的用户可以用浏览器下载它们，提取它们（通常是自动执行的），并将二进制文件复制到他们喜欢的地方。

[wasm-pack-release]: https://github.com/rustwasm/wasm-pack/releases/tag/v0.5.1

这确实需要一些手动“安装”程序的经验，因此您需要在 REAMDE 文件中，添加有关如何安装此程序的部分。

<aside class="note">

**注：**如果你用[trust]构建二进制文件，并将其添加到 GitHub 版本中，还可以告诉人们运行`curl -LSfs https://japaric.github.io/trust/install.sh | sh -s -- --git your-name/repo-name`，如果你认为这样更容易的话。

</aside>

### 何时使用

一般来说，使用二进制发布是一个好主意，几乎没有任何缺点。当然，它不能解决用户必须手动安装，和更新您的工具的问题，但他们可以快速获得最新版本，而无需安装 rust。

### 除了二进制文件外，还要打包什么

现在，当用户下载我们的发布版本时，他们将获得`.tar.gz`，一个只包含二进制文件的(压缩)文件。所以，在我们的示例项目中，他们只会得到一个他们可以运行的`grrs`文件。但我们的存储库中，已经有了一些他们可能想要的更多文件。如：告诉他们如何使用这个工具的 REAMDE 文件，还有许可证文件。因为我们已经有了它们，所以它们很容易添加。

不过，还有一些更有趣的文件，特别是对于命令行工具来说更为合理：除了这个 REAMDE 文件之外，我们还提供了一个手册，以及向 shell 添加补全可能标志的配置文件如何？你可以用手写，但是*鼓掌吧(clap)*，我们的参数解析库（也是 structopt 的基础库）有一种为我们生成所有这些文件的方法。见[深入的章节][clap-man-pages]，了解更多详细信息。

[clap-man-pages]: ../in-depth/docs.zh.html

## 将应用程序放入软件包存储库

到目前为止，我们看到的两种方法，都不是您常在计算机上安装软件的方法。尤其是在大多数操作系统上，会使用全局包管理器安装命令行工具。用户的优势是显而易见的：如果可以像安装其他工具一样安装程序，就不必考虑如何安装程序。这些，包管理器还允许用户在新版本可用时，更新其程序。

不幸的是，支持不同的系统意味着，你必须看看这些不同的系统是如何工作的。对于某某来说，这可能跟往存储库内，添加文件一样简单（例如，添加一个 Formula 文件，如给 MacOS`brew`使用的[这个][rg-formula]文件），但对于其他的，您通常需要自己发送补丁，并将您的工具，添加到他们的存储库中。有一些有用的工具，比如[cargo-rpm](https://crates.io/crates/cargo-rpm)和[cargo-deb](https://crates.io/crates/cargo-deb)。但是，描述它们是如何工作的，以及如何为这些不同的系统，正确地打包您的工具超出了本章的范围。

[rg-formula]: https://github.com/BurntSushi/ripgrep/blob/31adff6f3c4bfefc9e77df40871f2989443e6827/pkg/brew/ripgrep-bin.rb

相反，让我们来看一个用 Rust 编写的工具，它可以在许多不同的包管理器中使用。

### 例如：ripgrep

[ripgrep]是用 Rust 编写的`grep`/`ack`/`ag`替代。它非常成功，适用于许多操作系统：请看它 REAMDE 文件的[“安装”部分][rg-install]！

请注意，它列出了几个不同的选项，如何去安装它：它从一个指向包含二进制文件的 GitHub 发行版的链接开始，这样您就可以直接下载它们；然后它列出了，如何使用一组不同的包管理器安装它；最后，您还可以使用`cargo install`。

这似乎是一个很好的主意：不去选择这里介绍的方法，而是从`cargo install`开始，加上二进制版本，最后开始使用(各个)系统包管理器，来分发工具。

[ripgrep]: https://github.com/BurntSushi/ripgrep
[rg-install]: https://github.com/BurntSushi/ripgrep/tree/31adff6f3c4bfefc9e77df40871f2989443e6827#installation
