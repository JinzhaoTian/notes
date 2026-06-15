GPU（Graphics Processing Unit，图形处理器）由 NVIDIA 在 1999 年定义，指集成硬件变换与光照（T&L，Transform and Lighting）引擎的单芯片图形处理器，能够将 3D 图形计算从 CPU 转移至专用硬件。

## 发展历史

### （1999年以前）前 GPU 时代

早期图形处理依赖CPU执行软件渲染，或使用具备固定功能管线的加速芯片。这些芯片只能执行预设的图形操作（如绘制三角形、填充像素），无法编程，开发者无法自定义渲染效果[](https://developer.baidu.com/article/detail.html?id=3796077)。

### （1999-2000年）GPU 诞生与固定管线时代

1999 年，NVIDIA 发布 GeForce 256，首次提出 GPU 概念。其核心技术突破是集成硬件T&L引擎，将顶点变换与光照计算从 CPU 卸载至专用硬件。此时 GPU 采用固定功能管线（Fixed-Function Pipeline），渲染流程的各阶段由硬件预设，开发者无法干预。

竞争对手 ATI（2006 年被 AMD 收购）同期推出 Radeon 系列，采用类似架构。

### （2001-2006年）可编程着色器时代

2001年，NVIDIA GeForce 3（NV20）引入可编程着色器（Programmable Shaders）。开发者可使用类汇编语言编写顶点着色器（Vertex Shader）和像素着色器（Pixel Shader），自定义顶点变换与像素颜色计算逻辑，标志着图形渲染进入可编程时代。

此时着色器类型仍分离——顶点着色器与像素着色器为独立的硬件单元，数量和功能不对等。

### （2006-2017年）统一着色器与通用计算革命

2006-2007年，两大里程碑事件并行发生：
- **统一着色器架构（Unified Shader Architecture）**：NVIDIA G80 架构与微软 DirectX 10 同步推出，将顶点、像素、几何着色器统一为通用的流处理器（Stream Processor），任意着色器类型均可使用全部计算资源，消除了硬件资源闲置。
- **GPGPU 与 CUDA 生态奠基**：NVIDIA 在 G80 架构中引入 CUDA（Compute Unified Device Architecture，统一计算设备架构），首次允许开发者使用 C 语言编写 GPU 通用计算程序，GPU 从专用图形处理器升级为通用并行计算加速器。

2010年，Fermi 架构发布，是首个为通用计算完整设计的 GPU 架构，支持 ECC 内存（Error Correction Code Memory，错误校正码内存），双精度性能大幅提升，GPU 正式进入科学计算与数据中心领域。

2017年，Volta 架构发布，引入张量核心（Tensor Core），专门加速深度学习的矩阵乘加运算，FP16 性能较 Pascal 提升 3 倍，标志着 GPU 正式成为 AI 计算的硬件基石。

### （2018年至今）实时光追与AI融合时代

2018年，Turing 架构发布，首次在消费级 GPU 中集成 RT Core（Ray Tracing Core，光线追踪核心），实现实时光线追踪硬件加速，同时引入第二代 Tensor Core 支持 INT8 推理。

2020年，Ampere 架构发布，首次引入结构化稀疏性支持和多实例 GPU（MIG）技术，同时大幅提升各核心单元的并行吞吐能力。

2022年，Hopper 架构发布，首次引入 Transformer Engine，专为大语言模型训练与推理优化设计。采用台积电 4N 工艺，集成 800 亿晶体管。

2024年，Blackwell 架构发布，集成 2080 亿晶体管，AI 推理性能较 Hopper 提升 30 倍。采用定制化台积电 4NP 工艺，是 NVIDIA 首款双芯粒（Chiplet）设计的 GPU 架构——两个光罩尺寸极限的裸片通过 10 TB/s 片间互联连接为统一 GPU。


## 核心技术

![](_imgs/Pasted%20image%2020260615112819.png)


### SIMT 执行模型

GPU采用 SIMT（Single Instruction, Multiple Threads，单指令多线程）执行模型，是 SIMD（Single Instruction, Multiple Data，单指令多数据）的线程级变体。

- **Warp/Wavefront**：32个（NVIDIA）或64个（AMD）线程组成一个调度单元，所有线程在同一时钟周期执行相同指令
- **分支发散（Branch Divergence）**：warp内线程若执行不同指令路径，硬件需串行执行各分支路径，性能下降
- **线程束级调度（Warp-level Scheduling）**：SM内包含多个warp调度器，可并发执行不同warp以隐藏内存延迟

### 流式多处理器

SM（Streaming Multiprocessor，流式多处理器）是 GPU 的核心计算单元，各代架构通过调整 SM 内组成实现性能跃升：

|架构|年份|CUDA核心/SM|张量核心/SM|制程|代表产品|
|---|---|---|---|---|---|
|Fermi|2010|32|无|40/28nm|Quadro 7000|
|Kepler|2012|192|无|28nm|K80|
|Maxwell|2014|128|无|28nm|GTX 980|
|Pascal|2016|64|无|16nm|GTX 1080|
|Volta|2017|64|8|12nm|V100|
|Turing|2018|64|8|12nm|RTX 2080 Ti|
|Ampere|2020|64|4|7nm|A100|
|Hopper|2022|128|4|4nm|H100|
|Blackwell|2024|—|4 (五代)|4NP|B200|

### 内存层次结构

GPU 采用分层内存架构，容量与延迟权衡：

- **寄存器文件（Register File）**：每个线程私有，单周期访问，容量最小
- **共享内存（Shared Memory）/ L1 缓存**：同一 SM 内线程共享，用户可控（可编程缓存），延迟低，Volta 架构将 L1 与共享内存统一为 96KB 可配置空间
- **L2 缓存**：全 GPU 共享，容量数 MB 至数十 MB
- **全局显存（VRAM）**：GDDR 或 HBM（High Bandwidth Memory，高带宽内存），容量最大（数 GB 至数十 GB），延迟最高

### 专用硬件单元

1. **张量核心（Tensor Core）**：自 Volta 起引入，每个时钟周期执行 4×4 矩阵乘加运算（D = A×B + C）。
	- 五代演进：
		- 一代（Volta）：FP16累加到FP32
		- 二代（Turing）：增加INT8/INT4/Binary支持
		- 三代（Ampere）：增加TF32/BF16、结构稀疏性
		- 四代（Hopper）：增加FP8、Transformer引擎
		- 五代（Blackwell）：增加FP4/FP6、第二代Transformer引擎

2. **光线追踪核心（RT Core）**：自 Turing 起引入，硬件加速 BVH（Bounding Volume Hierarchy，包围体层次结构）遍历与光线-三角形相交测试。

