图形管线（Graphic Pipline）：本质上是一种 object-order rendering，从物体 object 开始，经过一系列连续的操作，最终转化为图片上的像素。

![](_imgs/Pasted%20image%2020231214101949.png)

![](_imgs/Pasted%20image%2020231214101837.png)


连续的操作如下：
- Before Rasterization：顶点处理阶段（vertex processing），将坐标转化为屏幕坐标
- Rasterization：光栅化，主要分成两个任务：（1）枚举图元覆盖的像素点（2）根据图元的属性进行插值。
	- 直线（Line）：根据直线的隐式表示或者参数化表示的不同，对直线进行光栅化（也就是上面的两步）
		- 隐式表示（implicit）：$$y = mx + b$$使用中点算法（midpoint algorithm）或者Bresenham algorithm；同时为了提高性能，采用增量算法（incremental method）。
		- 参数化表示（parametric）：$$p(t) = p_0 + t(p_1 - p_0)$$使用数值微分算法（DDA）。
	- 三角形：三角形一般是使用三个点进行表示，然后利用顶点的属性和重心坐标对三角形内部的像素点进行插值$$c = \alpha c_0 + \beta c_1 + \gamma c_2 $$
		- 要注意处理三角形边缘
		- 对重心坐标进行**透视校正**
		- 裁剪：将这个图元裁剪到符合投影的形式。
- After Rasterization：
	- 片元处理阶段（fragment processing）
	- 混合
- Antialiasing：反锯齿
- Culling Primitives：片元剔除


## 类别

尽管利用软件编码一定可以实现图形管线，但是使用硬件加速是一个重要趋势。因此有两种完全不同的图形管线，基于硬件的和基于软件的。

### Hardware Pipeline

图形硬件 API 提供了一种抽象的 GPU 硬件访问方式，简化了计算机图形生成的各个阶段，使得开发者无需深入了解硬件细节，而专注于图形的构建。它可以纯粹在软件中完成并在 CPU 上运行，或者由 GPU 进行硬件加速。

其代表是 OpenGL和 DirectX，主要支持交互渲染，期望执行的足够快以支持实时性。

- [**OpenGL**](../OpenGL/OpenGL.md)：OpenGL 是一套跨语言、跨平台，支持 2D、3D 图形渲染接口，由 [Khronos](../OpenGL/Khronos.md) 统一发布接口标准， 其实现一般由显卡厂商提供，而且非常依赖于该厂商提供的硬件。
	- **适用平台**：Windows、macOS、Linux 和 Android

- OpenGL ES：OpenGL 的子集，是针对手机和游戏主机等嵌入式设备而设计，去除了许多不必要和性能较低的 API 接口。

- [**Vulkan**](../Vulkan/Vulkan.md)：新一代的 OpenGL，相比之下，Vulkan 更接近底层，并且能很好地分配 CPU 核心来执行并行任务。
	- **适用平台**：Windows、Linux 和 Android

- **[DirectX](../DirectX/DirectX.md) 11**、**DirectX 12**：微软公司在 Windows 系统上所开发的3D图形编程接口。
	- **适用平台**：Windows

- **Metal**：Metal API 由苹果公司提供，它旨在为 iOS、iPadOS、macOS 和 tvOS 上的应用程序提供对 GPU 硬件的低级访问来提高性能，它与 Vulkan、DX12 都属于低级别的 API 。
	- **适用平台**：macOS

### Software Pipline

其代表是 [RenderMan](https://renderman.pixar.com/)，主要是支持最高质量的视觉特效和动作，计算时间较长。

> 皮克斯动画工作室的PhotoRealistic RenderMan软件渲染器，简称为PRMan。是用于影视效果制作的三维渲染软件包。由于RenderMan的商标归皮克斯公司拥有，人们经常把PRMan等同于RenderMan，实际上**RenderMan是一个渲染器规范**，皮克斯公司的PRMan是（声称）符合这一规范的一种产品。



## [Rendering Hardware Interface](Rendering%20Hardware%20Interface.md)

