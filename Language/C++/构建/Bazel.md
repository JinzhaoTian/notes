[Bazel](https://bazel.google.cn/?hl=zh-cn) 是 Google 开源的一个编译构建工具，它可以协调构建各个模块，并且可以运行单元测试。它的扩展语言使它可以构建任何类型的计算机语言，并且原生支持 Java，Ｃ，Ｃ++ 和 Python。

![](../_imgs/Pasted%20image%2020240729113650.png)

Bazel 采用**增量构建**的方式，使得 Bazel 在进行重复构建时可以重复使用未经修改且已缓存的构建，因此重复构建速度特别快。

Bazel 支持以下平台：
- Ubuntu Linux（16.04，15.10，和 14.04）
- Mac OS X
- ~~Windows（试验）：不推荐在 Windows 上使用 Bazel~~


## 安装

### Ubuntu

1. 添加 Bazel 的分发包地址作为源
```bash
echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list
curl https://bazel.build/bazel-release.pub.gpg | sudo apt-key add -
```

如果你想安装测试版的 Bazel，把上面命令中的 stable 替换成 testing

2. 安装和升级 Bazel
```bash
sudo apt-get update && sudo apt-get install bazel
```


