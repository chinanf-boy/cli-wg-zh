# CLI 应用程序呈现的文档

CLIS 的文档通常包括`--help`命令，和手册（`man`）页。

两者都可以在使用`clap`v3 时自动生成（在未发布的 alpha 中，在编写时），会用到`man`后端。

```rust,ignore
#[derive(Clap)]
pub struct Head {
    /// file to load
    #[clap(parse(from_os_str))]
    pub file: PathBuf,
    /// how many lines to print
    #[clap(short = "n", default_value = "5")]
    pub count: usize,
}
```

其次，您需要使用一个`build.rs`在编译时，根据应用程序的代码定义生成手册文件。

有一些事情需要记住（例如您希望二进制文件是怎样打包的），但现在我们只需将`man`文件放到`src`文件夹旁边。

```rust,ignore
use clap::IntoApp;
use clap_generate::gen_manuals;

#[path="src/cli.rs"]
mod cli;

fn main() {
    let app = cli::Head::into_app();
    for man in gen_manuals(&app) {
        let name = "head.1";
        let mut out = fs::File::create("head.1").unwrap();
        use std::io::Write;
        out.write_all(man.render().as_bytes()).unwrap();
    }
}
```

现在，你编译应用程序后，`head.1`文件就会你项目目录中。

如果你用`man`打开，就能够欣赏您零成本的文档。
