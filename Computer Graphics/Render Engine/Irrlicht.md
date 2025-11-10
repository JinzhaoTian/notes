[Irrlicht](http://irrlicht.sourceforge.net/) 在国内也被叫做“鬼火”引擎，是一款用 C++ 编写的开放源代码的高性能游戏引擎。而且是跨平台的，具有很好的移植性，Irrlicht 支持 OpenGL、Direcx3D 渲染，引擎本身也实现了一套自己的渲染系统。在商业引擎中能够找到的艺术特性，Irrlicht 基本都支持。目前有很多项目中都使用到它，Irrlicht 社区也比较活跃。


目录结构

```
.
	├── bin/ ：不同平台编译Irrlicht生成的可执行文件和动态库文件放在该目录中。
	├── doc/ : 该目录中为Irrlicht API文档。
	├── examples/ ： Irrlicht官方提供的案例程序存放在该目录中。
	├── include/ ： 存放引擎所有头文件。
	├── lib/ : 该目录存放Irrlicht引擎编译过后生成的静态库。
	├── media/ ： 官方案例所需资源文件存放在此目录中。
	├── source/ ： 存放引擎源码。
	└── tools/ : 该目录下为Irrlicht引擎相关工具。如irrEdit、GUIEditor等。
```


官方案例

打开 `examples` 目录，Irrlicht 已经为目前主流开发工具 Windows 平台下 Visual Studio、macOS 下的 Xcode 以及 Linux 平台下的 GNU MAKE 做好了项目配置。