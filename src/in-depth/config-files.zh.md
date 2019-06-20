# 使用配置文件

处理配置可能很烦人，特别是如果您支持多个操作系统，这些操作系统都有自己存放临时和长期文件的位置。

对此有多种解决方案，其中一些方案的级别较低。

最容易使用的箱子是`confy`. 它要求您输入应用程序的名称，并要求您通过`struct`指定配置层级（那是`Serialize`序列化，`Deserialize`反序列化），剩下的交给它解决了！

```rust,ignore
#[derive(Debug, Serialize, Deserialize)]
struct MyConfig {
    name: String,
    comfy: bool,
    foo: i64,
}

fn main() -> Result<(), io::Error> {
    let cfg: ConfyConfig = confy::load("my_app")?;
    println!("{:#?}", cfg);
    Ok(())
}
```

如果你会放弃可配置性，这当然是非常容易使用的。而如果你确实就要一个简单的配置，这个箱子就是为你准备！

## 配置环境

<aside class="todo">

**TODO**

1.  评估存在的箱子
2.  cli 参数 + 多个 configs + env 变量
3.  `configure`都能做？有好的 Rust 包装吗？

</aside>
