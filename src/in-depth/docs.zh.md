# CLI 应用程序呈现的文档

CLIS 的文档通常包括`--help`命令和手册中的一节（`man`）第页。

两者都可以在使用时自动生成`clap`v3（在未发布的 alpha 中，在编写时），通过`man`后端。

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

其次，您需要使用`build.rs`在编译时根据应用程序的代码定义生成手动文件。

有一些事情需要记住（例如您希望如何打包二进制文件），但现在我们只需将`man`我们旁边的文件`src`文件夹。

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

现在编译应用程序时`head.1`项目目录中的文件。

如果你把它打开`man`您将能够欣赏您的免费文档。
