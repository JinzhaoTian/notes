
## 优点

- Editor较为完善，对IMGUI 进行了封装，变成了有状态的UI框架
- SceneView, GameView, AssetView 多窗口渲染支持
- Inspector
- Hierachy
- AssetBorwser
- 甚至还有一个 Hub 启动器~
- 运行时 GameObject ，Component 小框架
- 渲染：基础的 Opengl 渲染
- 打包支持
- 目录结构也比较的清晰，注释很多
- 代码少 ，简单
- 大部分功能只有基础的实现且是单线程，
- 相对容易弄懂基本概念和 Unity 非常像，上手容易

## 缺点  

- 命名空间太深了，代码会变得特别长，写起来麻烦
- 没有资源层，加载效率太低了，每次运行都需要对原始 DCC 资源进行一次解析
- 没有 C++ 反射
- 导致需要手动注册Component类型
- 需要手动编写Inspector 界面
- 序列化代码也需要手动编写


- RHI 没有抽离出来，很多渲染相关的代码直接用Opengl接口写到了Editor中
- 没有Pipeline 概念，导致加一个渲染相关的特性会特别麻烦
- shader 没有Include 功能，导致特别多的代码冗余
- 单线程：Log, 资源加载，Editor, 渲染，逻辑 等，都放一个线程中，很卡~
- 生命期管理较为混乱，依赖关系没有处理好，没有进行统一管理，容易资源泄露，和指针越界
	- eg: 在场景 update 遍历的过程中销毁或创建Actor
