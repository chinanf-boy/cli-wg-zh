# 包装和分配一个生锈的工具

如果你相信你的项目已经准备好了

有一些方法,我们来看看3个

## 最快的:`cargo publish`

发布您的应用程序的最简单方法是与货物。[crates.io]。`cargo publish`与[crates.io]。

这适用于所有箱,包括w[crates.io]很简单:如果你还没有吗[crates.io]。[目前.这是通过授权你G]创建一个新的令牌,然后运行`cargo login <your-new-token>`。[你只需要做这一次电脑。]你

现在货物箱。`Cargo.toml`io知道你,你[货物的清单格式']。

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

**这是一个快速概述的一些常见的条目:**这个例子包括强制许可领域`README.md`文件。它应该包括对您的项目是关于什么的快速描述，并且不仅包括在您的板条箱的ractes.io页面上，而且还包括默认情况下在存储库页面上显示的Github。

</aside>

[crates.io]: https://crates.io/

[crates.io account page]: https://crates.io/me

[publishing guide]: https://doc.rust-lang.org/1.31.0/cargo/reference/publishing.html

[cargo's manifest format]: https://doc.rust-lang.org/1.31.0/cargo/reference/manifest.html

### 如何从ractes.io安装二进制文件

我们已经看到了如何将板条箱发布到ractes.io，您可能想知道如何安装它。与图书馆不同的是，当你运行的时候，哪些货物会为你下载和编译`cargo build`（或类似的命令），您需要告诉它显式安装二进制文件。

这是用`cargo install <crate-name>`. 默认情况下，它将下载板条箱，编译它包含的所有二进制目标（在“释放”模式下，因此可能需要一段时间），并将它们复制到`~/.cargo/bin/`目录。（确保您的shell知道在那里查找二进制文件！）

也可以从Git存储库安装板条箱，只安装板条箱的特定二进制文件，并指定一个备选目录来安装它们。看看`cargo install --help`有关详细信息。

### 何时使用

`cargo install`是发布二进制板条箱的简单方法。Rust开发人员使用起来非常方便，但也有一些明显的缺点：因为它总是从头开始编译源代码，所以您的工具的用户将需要拥有Rust、Cargo以及您的项目需要安装在其机器上的所有其他系统依赖项。编写大型铁锈代码库也需要一些时间。

此外，没有简单的方法来更新用cargo安装的工具：用户需要运行`cargo install`在某个时刻，再次通过`--force`覆盖旧二进制文件的标志。这是一个[缺少功能][cargo-issue-2082]还有子命令[像这个一样][cargo-update]不过，您可以安装来添加它。

[cargo-issue-2082]: https://github.com/rust-lang/cargo/issues/2082

[cargo-update]: https://crates.io/crates/cargo-update

最好将其用于分发针对其他Rust开发人员的工具。例如：很多货物子命令`cargo-tree`或`cargo-outdated`可以和它一起安装。

## 分发二进制文件

Rust是一种编译为本机代码的语言，默认情况下静态链接所有依赖项。当你跑步时`cargo build`在包含名为`grrs`，您将得到一个名为`grrs`. 尝试一下：使用`cargo build`，会的。`target/debug/grrs`当你跑步时`cargo build --release`，会的。`target/release/grrs`. 除非使用显式需要在目标系统上安装外部库的板条箱（如使用系统的OpenSSL版本），否则此二进制文件将仅依赖于公共系统库。这就是说，你只需要把一个文件发送给和你运行相同操作系统的人，他们就可以运行它了。

这已经很强大了！它可以解决我们刚才看到的两个缺点`cargo install`：不需要在用户的机器上安装Rust，他们可以立即运行二进制文件，而不需要花一分钟来编译。

所以，正如我们所看到的，`cargo build` *已经*为我们构建二进制文件。唯一的问题是，它们不能保证在所有平台上都能工作。如果你跑步`cargo build`在您的Windows机器上，默认情况下不会得到在Mac上工作的二进制文件。有没有一种方法可以为所有有趣的平台自动生成这些二进制文件？

### 在CI上构建二进制发布

如果您的工具是开源的，并且托管在GitHub上，那么很容易建立一个免费的CI（持续集成）服务，比如[特拉维斯CI]. （还有其他服务也可以在其他平台上工作，但Travis非常流行。）这基本上是在每次将更改推送到存储库时在虚拟机中运行安装命令。这些命令是什么以及它们运行的机器类型是可配置的。例如：一个好主意是跑步`cargo test`在机器上安装有铁锈和一些常用的工具。如果失败，您知道最近的更改中存在问题。

[travis ci]: https://travis-ci.com/

我们还可以使用它来构建二进制文件并将它们上载到Github！事实上，如果我们跑步`cargo build --release`把二进制文件上传到某个地方，我们应该都设置好了，对吗？不完全是。我们仍然需要确保我们构建的二进制文件与尽可能多的系统兼容。例如，在Linux上，我们可以不为当前系统编译，而是为`x86_64-unknown-linux-musl`目标，不依赖默认系统库。在MacOS上，我们可以设置`MACOSX_DEPLOYMENT_TARGET`到`10.7`仅依赖于10.7及更高版本中的系统功能。

您可以看到使用这种方法构建二进制文件的一个示例[在这里][wasm-pack-travis]对于Linux和MacOS以及[在这里][wasm-pack-appveyor]对于Windows（使用AppVeyor）。

[wasm-pack-travis]: https://github.com/rustwasm/wasm-pack/blob/51e6351c28fbd40745719e6d4a7bf26dadd30c85/.travis.yml#L74-L91

[wasm-pack-appveyor]: https://github.com/rustwasm/wasm-pack/blob/51e6351c28fbd40745719e6d4a7bf26dadd30c85/.appveyor.yml

另一种方法是使用预构建（docker）映像，其中包含构建二进制文件所需的所有工具。这也使我们能够轻松地瞄准更具异国情调的平台。这个[信任]Project包含可包含在项目中的脚本以及有关如何设置的说明。它还包括对使用AppVeyor的Windows的支持。

如果您希望在本地设置它并在您的计算机上生成发布文件，那么您仍然可以查看信任。它使用[交叉]在内部，它的工作原理类似于货物，但将命令转发给码头集装箱内的货物处理。图像的定义也可在[交叉存储库][cross].

[trust]: https://github.com/japaric/trust

[cross]: https://github.com/rust-embedded/cross

### 如何安装这些二进制文件

你把你的用户指向你的发布页面[像这个一样][wasm-pack-release]他们可以下载我们刚刚创建的工件。我们刚刚生成的发布工件没有什么特别的：最后，它们只是包含我们的二进制文件的存档文件！这意味着工具的用户可以用浏览器下载它们，提取它们（通常是自动执行的），并将二进制文件复制到他们喜欢的地方。

[wasm-pack-release]: https://github.com/rustwasm/wasm-pack/releases/tag/v0.5.1

这确实需要一些手动“安装”程序的经验，因此您需要在自述文件中添加有关如何安装此程序的部分。

<aside class="note">

**注：**如果你用[信任]要构建二进制文件并将其添加到GitHub版本中，还可以告诉人们运行`curl -LSfs https://japaric.github.io/trust/install.sh | sh -s -- --git your-name/repo-name`如果你认为这样更容易的话。

</aside>

### 何时使用

一般来说，使用二进制发布是一个好主意，几乎没有任何缺点。它不能解决用户必须手动安装和更新您的工具的问题，但他们可以快速获得最新版本，而无需安装rust。

### 除了二进制文件外，还要打包什么

现在，当用户下载我们的发布版本时，他们将获得`.tar.gz`只包含二进制文件的文件。所以，在我们的示例项目中，他们只会得到一个`grrs`他们可以运行的文件。但我们的存储库中已经有了一些他们可能想要的更多文件。告诉他们如何使用这个工具的自述文件，例如许可证文件。因为我们已经有了它们，它们很容易添加。

不过，还有一些更有趣的文件，特别是对于命令行工具来说更为合理：除了这个自述文件之外，我们还提供了一个手册页，以及向shell添加可能标志的配置文件如何？你可以用手写，但是*鼓掌*，我们使用的参数解析库（structopt基于哪个库）有一种为我们生成所有这些文件的方法。见[这个深入的章节][clap-man-pages]了解更多详细信息。

[clap-man-pages]: ../in-depth/docs.html

## 将应用程序放入软件包存储库

到目前为止，我们看到的两种方法都不是您通常如何在计算机上安装软件。尤其是在大多数操作系统上使用全局包管理器安装的命令行工具。用户的优势是显而易见的：如果可以像安装其他工具一样安装程序，就不必考虑如何安装程序。这些包管理器还允许用户在新版本可用时更新其程序。

不幸的是，支持不同的系统意味着你必须看看这些不同的系统是如何工作的。对于某些人来说，这可能与向存储库中添加文件一样简单（例如，添加公式文件，如[这][rg-formula]对于MacOS`brew`，但对于其他人，您通常需要自己发送补丁，并将您的工具添加到他们的存储库中。有一些有用的工具，比如[cargo-rpm](https://crates.io/crates/cargo-rpm)和[cargo-deb](https://crates.io/crates/cargo-deb)但是，描述它们是如何工作的，以及如何为这些不同的系统正确地打包您的工具超出了本章的范围。

[rg-formula]: https://github.com/BurntSushi/ripgrep/blob/31adff6f3c4bfefc9e77df40871f2989443e6827/pkg/brew/ripgrep-bin.rb

相反，让我们来看一个用Rust编写的工具，它可以在许多不同的包管理器中使用。

### 例如：ripgrep

[瑞普瑞普]是替代`grep`/`ack`/`ag`用铁锈写的。它非常成功，适用于许多操作系统：请看[“安装”部分][rg-install]它的自述文件！

请注意，它列出了几个不同的选项，您可以如何安装它：它从一个指向包含二进制文件的GitHub发行版的链接开始，这样您就可以直接下载它们；然后它列出了如何使用一组不同的包管理器安装它；最后，您还可以使用`cargo install`.

这似乎是一个很好的主意：不要选择这里介绍的方法之一，而是从`cargo install`，添加二进制版本，最后开始使用系统包管理器分发工具。

[ripgrep]: https://github.com/BurntSushi/ripgrep

[rg-install]: https://github.com/BurntSushi/ripgrep/tree/31adff6f3c4bfefc9e77df40871f2989443e6827#installation
