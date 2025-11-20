不应该一个文件一个文件的看，应该是首先能成功编译，在运行中摸索运行逻辑。

## 源码下载

获取：[EpicGames/UnrealEngine: Unreal Engine source code](https://github.com/EpicGames/UnrealEngine)

## 构建

### Windows

依次点击：
1. `Setup.bat`
2. `GenerateProjectFiles.bat`

会生成 `UE5.sln` 文件，用 `VS 2022` 版本打开，点击安装额外组件，
![](_imgs/Pasted%20image%2020251120093042.png)

将 UE5 设为启动项，
![](_imgs/Pasted%20image%2020251120092912.png)


设置正确的调试模式，如 `Development Editor and your solution platform to Win64` 等待**编译 40 分钟左右**即可。


