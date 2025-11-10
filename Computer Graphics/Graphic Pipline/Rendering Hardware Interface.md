RHI（Rendering Hardware Interface，或者说 Graphics Interface）是一种用于抽象图形渲染硬件的编程接口，常见于图形渲染引擎中。RHI 提供了一层抽象，使得渲染引擎可以与不同的图形 API 以及底层的硬件设备（如 GPU ）进行交互，而无需直接针对每种 API 或硬件编写不同的代码。

很多 RHI 实现会在设计阶段、维护过程中犯下和积累各种各样的设计错误（尤其是 U 开头的那两个引擎），甚至形成通往功能的屏障，会给程序员带来很大的困扰。要封装一个良好的 RHI 实现，需要穿透具体的平台 API 定义，透视到 GPU 的硬件执行原理和功能上。



## 封装库

1. 基础：
	- [andrejnau/FlyCube](https://github.com/andrejnau/FlyCube)：最早支持mesh shader的一批，优点是结构清晰，适合教学。
2. 薄：
	- [NVIDIA-RTX/NVRHI](https://github.com/NVIDIA-RTX/NVRHI)
	- [ConfettiFX/The-Forge](https://github.com/ConfettiFX/The-Forge)
3. 厚：
	- [gpuweb](https://github.com/gpuweb/gpuweb) 
	- slang-gfx
4. 额外：
	- d3d12lite
	- sokol

## 参考

1. [Render Hardware Interface - Open 3D Engine](https://www.docs.o3de.org/docs/atom-guide/dev-guide/rhi/)
