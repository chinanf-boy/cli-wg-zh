# 使用配置文件

处理配置可能很烦人，特别是如果您支持多个操作系统，这些操作系统都有自己的位置来存放短期和长期文件。

对此有多种解决方案，其中一些方案的级别较低。

最容易使用的板条箱是`confy`. 它要求您输入应用程序的名称，并要求您通过`struct`（那是`Serialize`，`Deserialize`）剩下的就解决了！

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

这是非常容易使用的，您当然会放弃可配置性。但如果一个简单的配置是你想要的，这个板条箱可能是为你！

## 配置环境

<aside class="todo">

**托多**

1.  评估存在的板条箱
2.  cli args+多个configs+env变量
3.  罐头`configure`做这些？它周围有包装纸吗？

</aside>
