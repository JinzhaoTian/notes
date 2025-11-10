GPU Driven Pipeline（GPU 驱动渲染管线） 是一种渲染架构，其核心思想是：将尽可能多的渲染决策从 CPU 转移到 GPU 上，并最大限度地减少 CPU 与 GPU 之间的通信。

## 与传统渲染管线的对比

1. **传统CPU驱动管线**
	1. **CPU 端（主线程）**：
	    - **遍历场景：** CPU 遍历所有场景对象（如树木、石头、角色）。
	    - **视锥体剔除：** CPU 计算相机视锥，判断哪些物体在视野内。
	    - **设置渲染状态：** 对于每个需要渲染的物体，CPU 设置其材质、着色器、纹理、顶点缓冲区等。
	    - **提交Draw Call：** CPU 向 GPU 发送一个命令，说：“嘿，GPU，请用这个状态画这个网格。” 这就是一个 **Draw Call**。
	2. **GPU端**：
	    - 接收命令和状态。
	    - 执行顶点着色、光栅化、片元着色等标准图形管线步骤。
	- **传统管线的瓶颈**：
		- **CPU瓶颈**：当场景中有成千上万个物体时（如茂密的森林、城市），CPU 需要逐一遍历、剔除、准备和提交成千上万个 Draw Call，这会消耗大量 CPU 时间，成为性能瓶颈。
		- **CPU-GPU通信瓶颈**：大量的 Draw Call 意味着 CPU 需要不断地打扰 GPU，命令队列可能饱和，导致 GPU 等待。

2. **GPU Driven Pipeline**：
	1. **CPU端（轻量级工作）**：
	    - 将场景的原始数据（如所有物体的变换矩阵、包围盒、网格数据、材质 ID）一次性上传到 GPU 的存储缓冲区（如 SSBO/Storage Buffer）中。
	    - 只发起一个或极少数几个间接 Draw Call，这个 Call 本身不包含具体的绘制信息，它更像是一个授权，告诉 GPU：“你可以开始绘制了，具体画什么，你自己看数据决定。”
	2. **GPU端（核心工作）**：
	    - **计算着色器登场**：在正式绘制之前，先启动 **Compute Shader**。
	    - **GPU 执行剔除**：Compute Shader 并行地对成千上万个物体进行视锥体剔除、遮挡剔除（Hierarchical Z-Buffer occlusion culling）等，每个物体一个线程，效率极高。
	    - **构建间接绘制参数**：剔除后，存活的物体会被 Compaction（紧凑化排列）。然后， Compute Shader 会向一个**间接参数缓冲区**中写入最终需要执行的绘制命令。每个命令包含顶点数、实例数等信息。
	    - **GPU执行绘制**：图形管线使用之前发起的那个间接 Draw Call，读取由 Compute Shader 填充的间接参数缓冲区，自动执行所有经过筛选的绘制命令。

## 关键技术与组件

1. **Compute Shader**：这是实现这一切的基石，它允许 GPU 进行通用计算，而不仅仅是图形处理。
2. **间接渲染**：
    - `glDrawElementsIndirect`（OpenGL）
    - `vkCmdDrawIndexedIndirect`（Vulkan）
    - `ExecuteIndirect`（DirectX）
    - `DrawMeshInstancedIndirect`（Unity）
3. **Shader Storage Buffer Object** / **Storage Buffer**：用于在 CPU 和 GPU 之间，以及 GPU 内部不同着色器阶段之间传递大量结构化数据（如物体矩阵、材质列表）。
4. **GPU 上的场景数据**：整个场景的描述（层次结构、空间结构如 BVH /八叉树）都需要被组织成 GPU 友好的格式。


## 挑战与复杂性

1. **开发难度大**：需要深入理解图形 API（如 Vulkan、DX12）和 GPU 硬件架构，调试非常困难，因为核心逻辑运行在 GPU 上。
2. **数据驱动**：整个渲染流程由数据缓冲区驱动，需要精心设计数据结构和管线。
3. **CPU-GPU同步**：虽然通信减少，但管理好 CPU 和 GPU 对数据的读写顺序（避免竞争条件）变得至关重要。
4. **工具链支持弱**：传统调试器和性能分析工具对 GPU 计算部分的洞察力有限。

## 实际应用

1. **《战神》（2018）** 和 **《赛博朋克2077》** 等 3A 大作都广泛使用了 GPU Driven Pipeline 来渲染其复杂的开放世界。
2. **UE5 Nanite 虚拟化几何系统**：这是 GPU Driven Pipeline 的一个极致体现，它将数百万个多边形模型的细节和可见性判断完全交给了 GPU。