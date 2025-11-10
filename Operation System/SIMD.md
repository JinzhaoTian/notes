SIMD（Single Instruction Multiple Data，单指令多数据）是一种并行计算技术，允许一个指令同时处理多个数据元素，能够显著提高数据处理效率。

> [!tip] 比喻
> - **传统方式**：你有一个收银台，顾客排成一列长队，你一个一个地结账。
> - **SIMD方式**：你同时开了 4 个或 8 个收银台，一次可以为一组（比如 4 个）顾客同时结账。


## 常见的 SIMD 指令集

在硬件层面，CPU 和 GPU 都拥有特殊的 SIMD 寄存器和指令集，这些寄存器比普通寄存器宽得多，可以一次性装入多个数据。

1. **CPU**：Intel 的 SSE、AVX， ARM 的 NEON。
    - 例如，一个 256 位的 AVX 寄存器可以同时存放 8 个 32 位的单精度浮点数。

2. **GPU**：GPU 的整个架构就是围绕大规模的 SIMD/SIMT 理念设计的。


> [!info] GPU：将 SIMD 发挥到极致
> 虽然 CPU 有 SIMD 单元，但 **GPU 才是为 SIMD 而生的怪物**，GPU 采用了 **SIMT（Single Instruction Multiple Threads）** 架构：
> - SIMT 与 SIMD 思想同源，但更灵活，其让多个线程（每个线程处理一个数据，如一个顶点或一个像素）以锁步的方式在同一时刻执行相同的指令。
> - 一个 GPU 核心包含大量（例如 64 个）这样的线程，组成一个 Warp 或 Wavefront，当一个 Warp 执行时，它就相当于一个超大规模的 SIMD 处理器。
> 
> 这就是为什么 GPU 在图形处理和通用并行计算（GPGPU）上如此强大的根本原因——图形学的核心工作负载**天然就是数据并行的**。


## 代码示例

假设你要把两个数组中的每个对应元素相加，
```c
// 传统方式 (SISD - Single Instruction, Single Data)
for (int i = 0; i < 4; i++) {
    c[i] = a[i] + b[i]; // 执行4次加法指令
}

// SIMD 方式 (伪代码)
// 1. 一次性将 a[0]~a[3] 加载到 SIMD 寄存器 VectA
// 2. 一次性将 b[0]~b[3] 加载到 SIMD 寄存器 VectB
// 3. 执行一条加法指令： VectC = VectA + VectB // 这条指令一次性完成4个加法
// 4. 将结果 VectC 一次性存回内存 c[0]~c[3]
```


## 主要用途

### 图形学中

图形学是计算密集型任务的典型代表，其中充满了大量需要同时对大量数据执行相同操作的情况，这使得 SIMD 成为图形性能优化的核心利器。

以下是 SIMD 在图形学中的几个关键应用领域：
1. **顶点变换**：
	- 在 3D 图形中，每个模型都由成千上万个顶点组成，要将一个顶点从模型空间转换到屏幕空间，需要将其与一个 $4 \times 4$ 的模型-视图-投影矩阵相乘。这个操作包含大量的乘法和加法。使用 SIMD，可以一次性处理 4 个顶点（或者一个顶点的所有 4 个分量 $x$，$y$，$z$，$w$ ），极大地加速了整个顶点处理管线，现代图形 API 的着色器编译器会自动将这类矩阵运算编译成高效的 SIMD 指令。
2. **片段/像素着色器操作**：在像素着色器中，很多操作都是并行进行的
	- **光照计算**：对每个像素计算漫反射、镜面反射。这涉及到向量点积、归一化等操作，这些都可以用 SIMD 并行计算。
	- **纹理采样**：虽然纹理采样本身是内存访问，但围绕它的地址计算、插值计算可以利用 SIMD
	- **颜色混合**：对 RGBA 四个通道同时进行操作（例如，同时乘以一个透明度 Alpha）。
3. **物理模拟和动画**：图形学不仅仅是绘制静态画面，还包括动态世界
	- **粒子系统**：烟雾、火焰、魔法效果等由数万个粒子组成。每个粒子的位置、速度、寿命更新都是相同的计算，是 SIMD 的完美应用场景。
	- **刚体动力学**：计算多个物体的受力、速度和位置。
	- **骨骼动画**：每个顶点的蒙皮计算需要混合多个骨骼变换的影响，这也是大量的矩阵/向量运算。
4. **图像后处理**：屏幕空间的特效处理通常是对整个纹理（图像）的每个像素执行相同的操作：
	- **模糊**：需要对周围像素进行加权平均。
	- **色调映射**：调整颜色和亮度。
	- **边缘检测**：使用卷积核（如 Sobel 算子）遍历图像。
5. **几何处理**
	- **视锥体剔除**：判断多个物体或包围盒是否在摄像机视野内。
	- **层次包围盒的遍历**：在碰撞检测或光线追踪中，需要同时测试多条射线与多个包围盒的相交。


## 使用

> GLM is an excellent library with which to learn. And, DirectXMath is an excellent library with which to ship. But, it's difficult to anticipate and design the systems that can get those 2-20x speed ups from SIMD without some knowledge of how to use it yourself.


Fun projects to learn SIMD:
1. Implement a basic 3D math library using SSE4 even if you plan to toss it and use DirectXMath in your shipping product.
2. Use your SSE4 math lib to make a real-time CPU-only ray tracer. How many triangles per frame per core can you squeeze into a 1024x1024 render? I had fun writing one like that which can orbit around in [this million-poly gallery model](https://sketchfab.com/3d-models/the-picture-gallery-231fdb3e9e354c6faaa3c250f8c9988f) at 36ms per frame. Just triangles in a BVH4 AOSOA tree. Primary rays only. Triangle IDs, depth and barycentric only.
3. Write a software decompressor for BCn texture formats.

