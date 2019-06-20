现有的打包系统

- [cargo-deb](https://github.com/mmstick/cargo-deb)
- [cargo-wix](https://github.com/volks73/cargo-wix)
- [cargo-arch](https://crates.io/crates/cargo-arch)
- [cargo-rpm](https://github.com/iqlusion-io/crates/tree/master/cargo-rpm)
- [cargo-ebuild](https://github.com/cardoe/cargo-ebuild)
- [cargo-tarball](https://github.com/crate-ci/cargo-tarball)

相关的分发方法

- [self_update](https://github.com/jaemk/self_update)
- [trust](https://github.com/japaric/trust/)

## 打包面临的挑战

### 寻找二进制工件

挑战

- 跨平台编译，会将项目放在不同的文件夹下

可能的解决方案

- Cargo 目标是支持一个`--out-dir`，见[rust-lang/cargo#5203](https://github.com/rust-lang/cargo/pull/5203)
  - 什么时候稳定？

### 查找`build.rs`工件

例子

- (shell)补全
- 手册

挑战

- `build.rs`脚本通常写入[`OUT_DIR`](https://doc.rust-lang.org/cargo/reference/environment-variables.html#environment-variables-cargo-sets-for-build-scripts)，它具有不可预测的路径，如`target/debug/build/cargo-tarball-b8d713fa01725c21/out/`。

现有讨论

- [怎么在不运行 build 脚本的前提下，获得 OUT_DIR?](https://users.rust-lang.org/t/how-to-get-out-dir-without-running-the-build-script/17239/3)
- [非-二进制工件生成的位置](https://internals.rust-lang.org/t/location-of-generated-non-binary-artifacts/7430)

解决方法：

- 不要用`build.rs`，但产生额外的`bin`目标，将这些构建作为 CI 的一部分。

可能的解决方案

- [rust-lang/cargo#5457](https://github.com/rust-lang/cargo/issues/5457)

### 杂项

- 要知道了`bin`名称
  - 在 Windows 上，它将有一个`.exe`扩展
  - 可以用<https://doc.rust-lang.org/std/env/consts/index.html>
  - 如果你想要跨平台编译，就不能
- 签名
  - debian：在存储库上
  - windows：嵌入在 exe 中
  - mac：在 paralel 文件或扩展属性中
    - 当 tarball 进行传输时，使用的扩展属性可能会丢失
- 要知道 Ubuntu 的 多体系三元组
  - 看到[`cargo-deb`'s workaround](https://github.com/mmstick/cargo-deb/blob/master/src/manifest.rs#L697)
  - ARM 可能会变得奇怪
  - `std::env::ARCH`很含糊
- 识别目标三元组
  - `cargo-metadata`没有返回
  - 编译程序时，不可用作 env 变量
  - 解决方法：[使用一个 build 脚本](https://github.com/mmstick/cargo-deb/blob/master/build.rs)
  - 注意：用于标识默认目标三元组，可以在命令行上覆盖
- 渴望生成多个包的箱子
  - Linux 软件包：arch vs noarch
  - `-dbg`包，见<https://github.com/mmstick/cargo-deb/issues/62>
  - 工作区可能想是单个包，或每种工件类型的包

## stager

宏伟愿景：所有打包工具都使用相同的配置架构；数据和字段仅适用于个人分发需求。

如何实现这一目标

- 分层配置
  - `cargo-deb`会读取自己的配置部分，返回未指定字段的通用配置。
  - `Cargo.toml`已经提供了很多的通用配置。
- 常见，一致的模式
  - 工作区
  - 定义多个顶级包（忽略`-dbg`等等）
- 用于描述文件布局，以及元数据（如权限）以及文件来源的通用格式
  - 介绍[`stager`](https://github.com/crate-ci/stager)

或换句话说，Rust 生态系统相当于[sbt 原生包管理器](https://sbt-native-packager.readthedocs.io/en/stable/)

示例客户端：[`cargo-tarball`](https://github.com/crate-ci/cargo-tarball)，意味着要取代`trust`样板。

状态

- stager：准备集成到打包工具中，以识别 missing 功能
- cargo-tarball：几乎是自我发布，只是缺少一些功能

### 什么是 stager

Stager 定义了一种通用格式，用于描述如何在包中布局文件，并提供实现此目的的工具。

目前，该格式利用了[liquid 模版语言](https://shopify.github.io/liquid/)使用户的生活更轻松。

目前，所有数据都被暂存到一个目录中。这可以演变为支持内存暂存，其中文件直接写入压缩文件。

### 开放式问题

#### 打包配置回家

我们应该将这种打包的配置，放在一个`Cargo.toml` `[metadata]`表或单独的文件？

#### 自动生成二进制工件

为 stager 创建 Cargo 来源，运行 Cargo，并生成二进制，之后到 stager。

#### 压缩

对于`deb`，文档应该被压缩。

- 创建[crate-ci/stager#18](https://github.com/crate-ci/stager/issues/18)作为回应
- cargo-deb 想要抽象这些规则
  - 这项政策不能在 stager 生存
  - 将需要遍历 stager 配置，识别文档并对其进行变更，以启用压缩。
