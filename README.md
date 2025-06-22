# MSPing

> 一个基于 MiaoSpeed 的节点批量测活工具

## 使用方法

首先安装依赖

```
pip install -r requirements.txt
```

然后在你要使用的测活机上部署 [MiaoSpeed](https://github.com/MiaoMagic/miaospeed)

修改 `main.py` 中的 `slave_cfg` 变量

其中 `address` 格式为 `1.2.3.4:5678` 是链接地址

`token` 是后端设定的链接密钥

`path` 是后端设定的 websocket 链接路径

`invoker` 是后端设定的 `whitelist id`

如后端开启 `-mtls` 则将 `tls` 和 `skipCertVerify` 设置为 `True`

`option` 配置为测试相关内容 不懂的人别动

`buildtoken` 也是 不懂的人别动

将要测活的节点写入 `Clash.yaml` 中 注意要使用 `Clash` 格式的配置 节点名称不能重复

运行脚本 脚本运行结束后端 `Clash.yaml` 就会剩下测活可用的节点了

## 抽象设计

如果您想贡献 miaospeed 和 MSPing ，您可以参考以下 miaospeed 的抽象设计:

- **Matrix**: 数据矩阵 [interfaces/matrix.go]。即用户想要获取的某个数据的最小颗粒度。例如，用户希望了解某个节点的 RTT 延迟，则 TA 可以要求 miaospeed 对 `TEST_PING_RTT` [例如: service/matrices/httpping/matrix.go] 进行测试。
- **Macro**: 运行时宏任务 [interfaces/macro.go]。如果用户希望批量运行数据矩阵，他们往往会做重复的事情。例如 `TEST_PING_RTT` 与 `TEST_PING_HTTP` 大多数时间都在做相同的事情。如果将两个 _Matrix_ 独立运行，则会浪费大量资源。因此，我们定义了 _Macro_ 最为一个最小颗粒度的执行体。由 _Macro_ 并行完成一系列耗时的操作，随后，_Matrix_ 将解析 _Macro_ 运行得到的数据，以填充自己的内容。
- **Vendor**: 服务提供商 [interfaces/vendor.go]。miaospeed 本身只是一个测试工具，**它不具备任何代理能力**。因此，_Vendor_ 作为一个接口，为 miaospeed 提供了链接能力。

然后再参考上游依赖库 `https://github.com/AirportR/miaolib` 的更抽象设计:

- **dataclasses**: 使用 `dataclasses` 把 `MiaoSpeed` 的数据结构体全部重新构建
- **冗余的代码**: 由于依赖库大概率是从开发者自己的测速Bot中提出来的 还有一堆 `dataclasses` 是绘图和Bot相关的总之完全用不到但是我估计删了会报错所以还是留着吧
- **完全不明所以的代码**: `dataclasses` 的设计及其复杂，想要看懂结构体到底都有啥你可能需要翻阅所有文件才能成功拼凑起来 **这得益于miaospeed的抽象设计**
- **一改就爆炸的代码**: 由于语言的操作不同而且后端分支也改的不一样 很有可能你的签名代码一改就爆炸完全过不去后端验证 这是由于不同语言反序列化的操作不一样而且还有各种编码问题而且数据结构体超级复杂根本不知道是哪里的问题反正你不懂就别动了 **这得益于miaospeed的抽象设计**
- **需要自己重写的代码**: 默认的后端链接启动函数 `start()` 测试结束后完全不断开 `websocket` 链接你需要自己重写但是完全没有可用模板而且自带的 `start` 有一坨看不懂在干嘛的东西