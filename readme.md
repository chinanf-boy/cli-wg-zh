# rust-lang-nursery/cli-wg [![explain]][source] [![translate-svg]][translate-list]

<!-- [![size-img]][size] -->

[explain]: http://llever.com/explain.svg
[source]: https://github.com/chinanf-boy/Source-Explain
[translate-svg]: http://llever.com/translate.svg
[translate-list]: https://github.com/chinanf-boy/chinese-translate-list
[size-img]: https://packagephobia.now.sh/badge?p=Name
[size]: https://packagephobia.now.sh/result?p=Name

「 Rust 命令行工作组之书 」

[中文](./readme.md) | [english](https://github.com/rust-lang-nursery/cli-wg)

---

## 校对 ✅

<!-- doc-templite START generated -->
<!-- repo = 'rust-lang-nursery/cli-wg' -->
<!-- commit = '3b2578bf05bcc5b52183e506ac7d5aa17d9e3b27' -->
<!-- time = '2019-06-08' -->

| 翻译的原文 | 与日期        | 最新更新 | 更多                       |
| ---------- | ------------- | -------- | -------------------------- |
| [commit]   | ⏰ 2019-06-08 | ![last]  | [中文翻译][translate-list] |

[last]: https://img.shields.io/github/last-commit/rust-lang-nursery/cli-wg.svg
[commit]: https://github.com/rust-lang-nursery/cli-wg/tree/3b2578bf05bcc5b52183e506ac7d5aa17d9e3b27

<!-- doc-templite END generated -->

- [x] readme.md
  <!-- - [ ] [./logs/2018-05-03-packaging.md](./logs/2018-05-03-packaging.zh.md) -->
  - [x] [Rust CLI 调研](./survey-results/Readme.zh.md)
- [x] [./src/SUMMARY.md](./src/SUMMARY.md)
- [x] [开始](./src/README.zh.md)
- [x] [15 分钟内完成一个命令行应用程序](./src/tutorial/README.zh.md)
  - [x] [项目设置](./src/tutorial/setup.zh.md)
  - [x] [对命令行参数解析](./src/tutorial/cli-args.zh.md)
  - [x] [第一次实现](./src/tutorial/impl-draft.zh.md)
  - [x] [更好的错误报告](./src/tutorial/errors.zh.md)
  - [x] [给人和机器的输出](./src/tutorial/output.zh.md)
  - [x] [测试](./src/tutorial/testing.zh.md)
  - [x] [打包和分发一个 Rust 工具](./src/tutorial/packaging.zh.md)
- [x] [深入的主题](./src/in-depth/README.zh.md)
  - [x] [信号处理](./src/in-depth/signals.zh.md)
  - [x] [使用配置文件](./src/in-depth/config-files.zh.md)
  - [x] [退出代码](./src/in-depth/exit-code.zh.md)
  - [x] [与人交流](./src/in-depth/human-communication.zh.md)
  - [x] [与机器交互](./src/in-depth/machine-communication.zh.md)
  - [x] [CLI 应用程序呈现的文档](./src/in-depth/docs.zh.md)

### 贡献

欢迎 👏 勘误/校对/更新贡献 😊 [具体贡献请看](https://github.com/chinanf-boy/chinese-translate-list#贡献)

## 生活

[hIf help, **buy** me coffee —— 营养跟不上了，给我来瓶营养快线吧! 💰](https://github.com/chinanf-boy/live-need-money)

---

# CLI 工作组

本存储库用于协调 Rust CLI 工作组的工作，也称为“Rust CLIQuE”（Rust CLI 质量增强）。

它还包含 CLAiR ，[Rust 中的命令行应用程序][clair]之书。

[clair]: https://rust-lang-nursery.github.io/cli-wg/

- [工作组?](https://internals.rust-lang.org/t/announcing-the-2018-domain-working-groups/6737)
- [工作组通告](https://internals.rust-lang.org/t/announcing-the-cli-working-group/6872/1)
- 和我们聊天
  - [Discord](https://discord.gg/dwq4Zme)
  - [Gitter](https://gitter.im/rust-lang/WG-CLI)

## 我们的目标

在这里，我们做一个真实声明：

Rust 会使得编写跨平台，测试的，现代命令行应用程序变得丝滑无比，同时结合了行业最佳实践，并提供了出色的文档。

## 什么是 CLI？

对我们自己的想法和目的来说，CLI 是任何程序，只要它

- 在终端发射
- 接受来自各种源的配置，例如命令行参数，环境变量或配置文件
- 使用最少/无用户交互，即可完成运行
- 接受来自`stdin`，文件或网络的输入
- 对某些输入（文件，网络，`stdin`），可基于某指定配置
- 通过标准输出（文件，网络，`std{out,err}`）交互

（我们[特别指出][i4]，现在并不想专注于“TUI”应用程序。）

[i4]: https://github.com/rust-lang-nursery/cli-wg/issues/4
