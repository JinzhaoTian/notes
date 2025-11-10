
WebGL（Web Graphics Library）和OpenGL（Open Graphics Library）在本质上是相似的，都是用于渲染图形的API，但它们之间有一些关键的区别，主要涉及到它们的设计、应用场景和使用方式：

1. **平台和环境：**
    - **OpenGL：** OpenGL是一个跨平台的图形API，可以在各种操作系统上使用，包括Windows、macOS和Linux。OpenGL通常用于本地应用程序的图形渲染，如游戏和计算机辅助设计等。
    - **WebGL：** WebGL是Canvas元素，并允许使用JavaScript调用底层的OpenGL渲染功能，从而在Web中创建交互式专门为在Web浏览器中进行图形渲染而设计的，是OpenGL ES 2.0的一个子集。它基于HTML5的的3D图形。
2. **语言和接口：**
    - **OpenGL：** 通常使用C或C++编写，尽管有其他语言的绑定和接口。
    - **WebGL：** 主要使用JavaScript，因为它是Web浏览器中的脚本语言。WebGL通过在浏览器中的JavaScript脚本中使用特定的API调用，与底层的OpenGL ES 2.0通信。
3. **上下文初始化：**
    - **OpenGL：** 在本地应用程序中，OpenGL的上下文初始化通常是由操作系统提供的。
    - **WebGL：** 在WebGL中，上下文初始化是由浏览器提供的，通过在HTML文档中创建一个Canvas元素并请求WebGL上下文来实现。
4. **版本和功能：**
    - **OpenGL：** OpenGL具有多个版本，包括OpenGL 1.x、OpenGL 2.x、OpenGL 3.x等。较新版本通常提供更多的功能和优化。
    - **WebGL：** WebGL是基于OpenGL ES 2.0的，这是OpenGL的一个嵌入式系统版本。虽然相对较简化，但仍支持许多现代图形渲染功能。
5. **调试和工具：**
    - **OpenGL：** 本地应用程序通常可以使用丰富的调试工具和性能分析工具，以帮助优化和调试图形渲染。
    - **WebGL：** 浏览器提供了一些工具，例如浏览器的开发者工具，可以用于调试WebGL应用程序，但相对于本地应用程序来说可能更有限。

总体而言，OpenGL和WebGL之间的主要区别在于它们的应用场景和使用环境。OpenGL用于本地应用程序的图形渲染，而WebGL使得在Web浏览器中进行高性能的3D图形渲染成为可能。

### Demo

[su7 (gamemcu.com)](https://gamemcu.com/su7/)


[node-3d/node-3d: Guidlines and common information (github.com)](https://github.com/node-3d/node-3d)



## 优化方向

[WebGL渲染引擎优化方向 -- 加载性能优化 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/695167970)

[WebGL渲染引擎优化方向——渲染帧率的优化 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/698896567)

