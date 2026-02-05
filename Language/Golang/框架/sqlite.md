在 macOS 上开发，在 Linux 上部署，这是非常普遍的工作流。但只要你的 Go 项目依赖 CGO（比如官方的 SQLite 驱动），简单的交叉编译就会变成一场噩梦。

[modernc.org/sqlite](https://pkg.go.dev/modernc.org/sqlite) 采用纯 Go 实现的 SQLite 驱动，完全摆脱了 CGO 的束缚。虽然性能略有损失，但换来的是无痛的交叉编译和极致的部署便利性。


