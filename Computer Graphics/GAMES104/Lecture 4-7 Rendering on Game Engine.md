绝大部分游戏都是需要绘制的，理论上是**需要做到实时绘制**（Realtime - 30 FPS）。

## Challenges

1. **挑战一**：**游戏的画面是十分复杂的**，一帧画面可能会出现成千上万个游戏对象。![](_imgs/Pasted%20image%2020251013102730.png)
2. **挑战二**：**需要深度适配当代的硬件（不同的电脑配置）**![](_imgs/Pasted%20image%2020251013102857.png)
3. **挑战三**：**追求稳定的帧率，面对大场景和小场景，高分辨率低分辨率，都能有同样的帧率**，因此绘制算法要在一个固定的时间里面。同样，也**追求尽可能高的帧率**，给的时间越来越小，要求越来越高。![](_imgs/Pasted%20image%2020251013103154.png)
4. **挑战四**：**CPU 带宽限制，一般只能吃掉 $10\%$ ~ $20\%$，剩下的都是给到 Gameplay 系统**。![](_imgs/Pasted%20image%2020251013103624.png)
## Outline of Rendering

![](_imgs/Pasted%20image%2020251013144440.png)

游戏引擎并不只是渲染引擎，但是渲染是游戏中的一个重要环节。游戏引擎中的渲染和计算机图形学的渲染侧重点是不同的：
1. **计算机图形学中的渲染**：
	- 为了明确的**解决某单一问题**，比如透明物体或者是水面等明确的需求
	- **关注算法或数学上的正确性**，并不关注硬件如何实现。
		- 比如辐射度算法是一个通过模拟光学理论得到的模型，计算机需要计算几天才能得到一帧漂亮结果。
	- **没有性能的要求**，图形学中 30 帧即为 Real-Time，10 帧为 interactive。
2. **游戏引擎中的渲染**：
	- **游戏场景复杂**，需要对庞大数目的物体进行渲染，物体种类繁多，且融合了大量渲染效果，复杂度很高，比如游戏中我们需要渲染水体，角色，植被，云彩等等，我们需要运用不同的图形学算法对其物体进行渲染，以及光照运算和各种后处理，游戏引擎的渲染模块中需要包含这么多东西，是一个 all in one 的组合。
	- 游戏需要面对硬件处理问题，因此**需要深度去适配当代硬件**
	- 游戏在不同场景和不同质量的显示器硬件上运行需要有稳定的帧率，即场景切换时仍**要求保证稳定帧率**，即使显示器等硬件质量的不断提升，也需要在其上拥有稳定的帧率
	- **实时性要求高**，游戏 CPU 端仅有 10%-20% 分配给了渲染，剩下的分配给 GamePlay 系统。




### Basics of Game Rendering

#### 绘制流程

现代游戏渲染是通过 CPU + GPU 合作处理模式，CPU 准备好数据渲染数据后将其提交到 GPU，GPU 设置好渲染状态后开始处理 CPU 所提交的数据。

![](_imgs/Pasted%20image%2020251017214557.png)

#### 渲染中的计算

![](_imgs/Pasted%20image%2020251017214507.png)

1. **投影和光栅化**：主要是关于矩阵的计算
	- 透视投影
	- 正交投影
	- 屏幕空间的三角形离散成一系列像素

2. **着色计算**
	- 常量访问，比如需要知道屏幕的长宽，像素个数，需要访问常数
	- 数学计算（加减乘除），比如 Blinn-Phong 模型需要知道法线，光线，眼睛，并计算光有衰减百分比
	- 纹理采样

3. **纹理采样**：
	- 纹理采样其实是渲染过程中非常复杂/昂贵的一个环节，假设我们现在在 3D 空间内有一个砖墙，当我们离砖墙十分近时，可以看到其上面的一个个像素；但当我们离墙十分远时，我们在屏幕空间上的一个像素，可能包含了砖墙上的很多像素。



#### 计算硬件

硬件底层为了加速运算，集成了指令并行的机制：

![](_imgs/Pasted%20image%2020251017220853.png)

1. **SIMD**：单指令多数据，比如处理一个 vector4 的加法，在计算时，将 $x$，$y$，$z$，$w$ 同时进行加法运算，即一个指令完成四个加法运算，即 $x$，$y$，$z$，$w$ 的加法运算。
2. **SIMT**：单指令多线程，其核心是将 core 做的足够小，从而能够内置很多个core，这样在执行 $c=a+b$ 这条指令时，各个 core 上都执行这条指令，但是我们每个 core 上的 $a$ 和 $b$ 的数据是不同的。

##### CPU

在 CPU 中，广泛使用 SIMD，


##### GPU

以 Fermi 架构举例（第一个完整的 GPU 计算架构），

![](_imgs/Pasted%20image%2020251017224850.png)

GPU 中放置了很多个内核，但又将其分成了一组一组的形式，一组内核称为 GPC（Graphics Progressing Cluster，图形处理集群）。在 GPC 中可以看到很多的 SM（Streaming Multiprocessor），SM 中存在很多小的内核，这些内核是指令的直接执行者，如果是 N 卡这些核叫做 CUDA（Compute Unified Device Architecture，统一计算设备架构），给 SM 一条指令，CUDA 核们就开始工作。

##### From CPU To GPU

> [!tip] 
> Always minimize data transfer between CPU and GPU when possible.

现代 CPU 的架构是冯诺依曼架构（数据与计算分离），这种架构的问题就是计算需要准备好数据，因为找数据是特别慢的，数据在不同 Units 之间搬来搬去也是特别慢的。

![](_imgs/Pasted%20image%2020251020214736.png)

CPU 和 GPU 可以看做是独立的机器，两个机器之间的数据传递成本很高。如果进行一个计算，先让 CPU 将数据传到 GPU 中进行计算，等 GPU 计算完后 CPU 再将结果读取回来，再基于这个结果进行判断，之后再告诉 CPU 如何绘制，这个过程叫数据的 back-force。

##### Cache

![](_imgs/Pasted%20image%2020251020221550.png)


#### Renderable

游戏世界中有很多类型的 GameObject，比如 FPS 游戏的车，船，飞机，士兵等，他们可以移动，拥有 HP 值，开火，死亡等，但这些都是逻辑上的描述，是无法绘制出来的，逻辑上表达的游戏对象和真正绘制的东西是两个东西。

在游戏引擎里，可绘制的数据一般包括：
1. **Mesh**：网格数据，是由一个个顶点数据组成的三角面的集合
	- **顶点**：
		- 位置
		- 法向量
		- 纹理坐标
		- 颜色
		- ...
	- **三角面片**：每三个顶点可以形成一个三角面片
		- Triangle List：不对顶点数据进行处理，因此 $n$ 个三角形需要 $n \times 3$ 个顶点数据。
		- Triangle Strip：顶点列表中，连续三个顶点表示一个三角面，这样就省去了索引数据，并且对缓存友好。
		- Triangle Fan![](_imgs/Pasted%20image%2020251020221648.png)
> [!tip] 要在顶点上存储法向量方向
> 因为如果通过临近三角形的法向量来求三角形上顶点法向量方向的话，当你在渲染像正方形这样的物体时，会发现折线部分上的顶点法向量方向是错误的。

2. **Material**：在渲染里面定义的材质只是视觉材质（也就是看起来像塑料、金属等）而不是物理材质（弹性、摩擦等）。

3. **Texture**：表达材质时，texture 起到了很重要的作用，因为大多数时候我们判断一个物体的材质，第一时间是通过其 texture 来判断的，而不是根据材质的参数。

4. **Shader**：有了 mesh，material，texture 等，需要通过 shader 才能将这些真正给绘制出来。
	- shader 是 source code，但是在引擎中又会被当做数据进行处理。


#### Render Objects in the Engine

1. **Coordinate System and Transformation**：模型是基于自己的坐标系，需要讲这些转换到屏幕空间坐标。![](_imgs/Pasted%20image%2020251020224929.png)
2. **Object with Many Materials**：一个物体可能不同部分会有不同的材质，GPU 作为一个状态机，只会保留最后材质所提交的状态进行渲染。
	- **Submesh**：对于存在多个材质的对象，会根据材质对 mesh 进行切分为不同的 submesh，每个 submesh 有对应的 Material、Texture、Shader，并且把 Vertex 和 Triangle 放在一个大的 buffer 里进行管理，至此一个完整的复杂对象渲染就处理完成了。![](_imgs/Pasted%20image%2020251020225429.png)
	- **缺点**：如果我们需要绘制大量这样的复杂 GameObject，如果每个单位都独立存储一份完整的渲染数据，这样的开销太过巨大。![](_imgs/Pasted%20image%2020251020225638.png)
3. **Resource Pool**：这些 GameObject 的 Mesh、Material、Texture 都有重复部分，因此较好的数据组织方式是对渲染资源数据创建资源池，相同的 Mesh、Material、Texture 等保证不会重复存储。![](_imgs/Pasted%20image%2020251020230105.png)
4. **Instance**：每个 GameObject 使用 Resource Pool 中的 Handle 来索引实际的 Mesh、Material、Texture、Shader 等数据。此概念也可以进一步引申为游戏引擎中的**实例化**概念，将实际 GameObject 的定义和实例分离。![](_imgs/Pasted%20image%2020251116115402.png)

5. **Sort by Material**：将场景中的物体按照材质进行排序，将相同材质物体一起计算更新，从而只需要设置一次材质，大大减少了 GPU 设置渲染状态的耗时从而提升了速度。

> [!tip] GPU Batch Rendering
> 游戏场景中，很多物体都是重复的，在一次绘制中设置 VB、IB 也是很浪费的。在使用 Compute Shader 时可以一次 Draw Call 把成千上百个 GameObject 一次绘制出来，这就是 GPU Batch Rendering 思想。

> [!tip]
> 现代的游戏引擎架构中，会尽可能的把绘制运算交给 GPU 而不是 CPU 。


#### Visibility Culling

对于一个拥有很多 GameObject 的游戏场景，不能将每个角色都绘制出来，这样硬件的负荷大，因此需要 visibility culling，它是引擎的渲染模块中的一个基础底层系统。

1. **根据视锥体进行剔除**：给每个物体定义一个包围盒或者包围球，这样问题就简化为如何判断包围盒或者包围球和视锥体的内外关系。![](_imgs/Pasted%20image%2020251020231023.png)
2. **根据场景划分进行剔除**：通过对场景中的 GameObject 进行划分管理，比如经典的四叉树、BVH 划分等，预先剔除摄像机覆盖范围外的对象。![](_imgs/Pasted%20image%2020251020230952.png)
	- **PVS（Potential Visibility Set）**：将一个大的游戏场景划分为一系列的子场景，如图，相邻的子场景之间设置 Portal（对应真实世界中的门或窗），当你站在一个子场景时，通过 Portal 只能看见有限的子场景。![](_imgs/Pasted%20image%2020251020231337.png)

3. **在 GPU 中剔除**：通过 GPU 进行 Culling 操作，比如 early-Z。
	- 利用了 GPU 高效的并行化能力，用比较低的成本形成一群遮挡物的深度图，然后通过比较从而节省掉不必要的计算过程，对于大型场景很有用。![](_imgs/Pasted%20image%2020251020231620.png)
	- **Early-Z（z-buffer）**：在绘制对象时，靠前的物体会挡住靠后的物体，在进行真正绘制之前，Camera 会对空间对象生成一张深度图（z-buffer）。在之后绘制对象时，就可以判断像素的深度是否符合要求，以此来判断是否进行绘制。

#### Texture Compression

纹理压缩，可以节省数据传输的带宽。
- 日常使用的图片压缩格式（如 PNG、JPEG 等），有很好的压缩或显示效果，但在游戏引擎中无法直接使用这些压缩算法，因为无法快速随机访问像素。
- **Block Compression**：在游戏引擎中，一般将纹理划分为多个小块然后进行压缩。
	- 以 DXTC 举例，对于每个划分的小块，取得其中最亮和最暗的像素点，那么我们就可以通过插值处理从而求得二者中间一系列的颜色。
	- **常见算法**：
		- PC：BC7、DXTC
		- Mobile：ASTC

#### Authoring Tools of Modeling

1. Polymodeling![](_imgs/Pasted%20image%2020251020232000.png)
2. Sculpting![](_imgs/Pasted%20image%2020251020231945.png)
3. Scanning![](_imgs/Pasted%20image%2020251020231933.png)
4. Procedural Modeling![](_imgs/Pasted%20image%2020251020231922.png)

#### Rendering Pipeline

> [!info] 新概念的渲染管线
> 随着建模工具的不断进步，我们得到的模型也变得更加细节具体，数据量也不断增大，游戏和影视有很大的重合部分，但由于游戏的实时渲染以及硬件存储要求，通常一个模型的面片数不会超过 1 万，而影视级的模型通常是千万级的。

有个重要的发展方向是 **Cluster-Based Mesh Pipeline**，将模型分成多个 Cluster（每个 Cluster有 32\64个 triangle），根据这些 Cluster 与摄像机的远近来展示不同的细节。

> [!info] Meshlet
> Meshlet 是现代图形学中用于高效渲染复杂 3D 模型的一种核心技术，它通过将大型 Mesh 分割成更小、更易于管理的块（即 Meshlet），并结合新一代硬件特性（如 Mesh Shader），极大地提升了渲染性能。

现代 GPU 已经可以基于数据动态地生成几何细节（曲面细分 Tessellation），而不是像原先的管线将 mesh 数据上传，因此当将每个 Cluster 大小确定好后，由于它的计算都是高效一致的，以相同的 Cluster 结构让 GPU 来并行处理时，提高了效率。![](_imgs/Pasted%20image%2020251020232855.png)

1. **传统管线流程**：
```mermaid
graph LR
    A[顶点输入] --> B[顶点着色器]
    B --> C[图元组装]
    C --> D[光栅化]
    D --> E[像素着色器]
```
2. **Mesh Shader 管线流程**：
```mermaid
graph LR
    A[任务着色器 <br />（可选）] --> B[网格着色器]
    B --> C[光栅化]
    C --> D[像素着色器]
```

> [!info] [Unreal Engine 5 的 Nanite](../Unreal%20Engine/UE5%20Nanite.md)
> 
> 1. 无缝边界的层次化 LOD Clusters
> 2. 无需硬件特殊支持，而是通过 GPU 上的持久线程（计算着色器）在预计算的 BVH 树上实现分层集群剔除，替代任务着色器方案。



### Lighting，Materials and Shaders

我们是通过光来看到这个世界的，光线赋予了斑斓的世界，同样的在虚拟世界中也需要绘制出光照效果。

#### Rendering Equation

$$
L_o(p, w_o) = L_e(p, w_o) + \int_{\Omega^+}L_i(p, w_i)f_r(p, w_i, w_o)(n \cdot w_i)\mathrm{d}w_i
$$
**核心思想**：出射光亮度 $L_o$ = 物体自发光亮度 $L_e$ + 反射光亮度 $L_r$

其中，
- $\int_{\Omega^+} ... d\omega_i$ 代表了来自点 $p$ 上方整个半球（$\Omega^+$）所有可能方向的光线对最终出射光亮度的总贡献。
- $L_i(p, \omega_i)$ 为入射光亮度，表示从 $\omega_i$ 方向到达点 $p$ 的光的能量；
- $f_r(p, \omega_i, \omega_o)$ 为**双向反射分布函数**（Bidirectional Reflectance Distribution Function，**BRDF**），BRDF 的具体形式决定了材质的质感（例如，漫反射、镜面反射、金属感等），用于计算对于从 $\omega_i$ 方向射入的光，有多少能量会被反射到 $\omega_o$ 方向；
- $(n \cdot \omega_i)$ 为余弦项（Lambert’s Cosine Law），这个点积项描述了入射光亮度的接收效率，其中 $n$ 是表面法线。


#### Complexity of Real Rendering

渲染方程在理论上是完美的，但直接求解它来渲染一个真实的游戏场景，在计算上是不可行的，因为真实世界的光路传播极其复杂。

> [!question] **挑战一**：**如何获取任意给定入射方向的入射辐射量（光亮度）**

渲染方程中的入射光 $L_i$ 隐含了一个能见度项，即光是否能照到这个点 $p$，因此判断入射光 $L_i$ 是否为零（即是否被遮挡）是计算光照的第一步，称为**可见性**（Visibility）问题。

可见性问题在游戏中最直观、最普遍的表现就是**阴影**（Shadowing）。为了确定一个点是否在阴影中，引擎必须知道光源与该点之间是否有遮挡，其计算量巨大，也容易产生锯齿（Aliasing）、不精确（Peter Panning）、自遮挡错误（Shadow Acne）等各种视觉瑕疵，而且不存在一种能完美解决所有场景、所有光照情况的阴影技术，开发者需要根据具体需求，在多种充满妥协（Hacks）的技术中进行选择和组合。

> [!tip] 术语
> 为了精确描述光，图形学中有两个基础且核心的“黑话”：
> - **Radiance**（辐射度）：指的是从一个表面发出或反射出去的光的能量和分布
> - **Irradiance**（辐照度）：指的是一个表面接收到的所有入射光能量的总和，是对半球空间上所有方向的 Incoming Radiance 进行积分的结果。

此外，**真实世界的光源形态各异**，远比简单的数学模型复杂。简单光源如方向光（Directional Light）、点光源（Point Light）和聚光灯（Spotlight），这些在实时渲染中相对容易处理。

但是复杂光源如面光源（Area Light）的引入会使问题复杂度急剧升高，因为光源自身的旋转和形态都会极大地影响着色结果，产生柔和的阴影（软阴影），这对于简单的可见性测试来说非常困难。


> [!question] **挑战二**：**对光照和散射函数在半球面上的积分计算量大**

渲染方程的核心是一个对半球面积分的计算：
$$
L_r = \int_{\Omega^+}L_i(p, w_i)f_r(p, w_i, w_o)(n \cdot w_i)\mathrm{d}w_i
$$
这个积分在计算上是极其昂贵的。对于离线渲染，可以使用蒙特卡洛积分通过发射数千条射线并取平均值来逼近结果。但是对于实时渲染，逐点、逐像素地进行暴力数值积分是完全不可行的。因此，如何快速、近似地求解这个积分，是实时渲染中被称为**着色**（Shading）的核心难题。

> [!tip] 蒙特卡洛积分（Monte Carlo Integration）
> 


> [!question] **挑战三**：**渲染方程是递归的**

在真实世界中，光线会在物体表面之间多次反弹（Bouncing），意味着场景中的每一个物体，一旦被照亮，其自身就变成了一个新的光源，所以一个点的光照不仅来自主光源（直接光照），还大量来自于周围环境反射过来的光（间接光照 / 全局光照 Global Illumination）。

因此在渲染方程中，入射光 $L_i$ 不仅来自直接光源，还来自其他物体的反射光 $L_o$ ，这意味着方程是递归的：**为了算 A 点的亮度，你必须先知道 B 点的亮度；而要计算 B 点的光照，你又可能需要知道 A 点反射出去的光，无限递归。**

早期的光线追踪（Ray Tracing）算法试图通过递归追踪光线来模拟这一过程，但这会导致光线数量爆炸性增长，即使在离线渲染中也难以承受。


> [!quote] 补充：复杂材质的数学拟合（BRDF Complexity）
> 渲染方程中的 $f_r$（BRDF）决定了物体看起来像金属、塑料还是皮肤，真实的 BRDF 模型（如 Microfacet 理论）非常复杂，包含 $D$（法线分布）、$G$（几何遮挡）、$F$（菲涅尔项）三项。
> 
> 游戏引擎在处理粗糙度极高的物体或多层材质时，计算量会激增。

#### Rendering in the Engine

求解渲染方程如此困难，因此在游戏引擎中，一般不直接求解复杂的积分和递归，而是将问题分解成几个可以被硬件快速处理的、简化的部分。

##### Simplify Light

在渲染方程中，$L_i$ 是来自半球上方所有方向的入射光。游戏引擎通过以下方式将其离散化和预计算：
1. **分析光源（Analytic Lights）**：将复杂的光源简化为数学点（点光源、方向光、聚光灯）。这使得积分运算变成了简单的加法计算。
2. **环境光（Environment Lighting）**：模拟来自天空或远景的光。
    - **环境光（Ambient Light）**：最简单的方式就是一个恒定的颜色值/强度，假设所有间接光都是均匀的。
    - **球谐函数（Spherical Harmonics）**：用极少量的系数来近似低频的环境光（通常用于漫反射）。
    - **预过滤环境贴图（Prefiltered Env Maps）**：提前计算好不同粗糙度下的光照结果，运行时通过查表直接获取。

##### Simplify Material

在渲染方程中，材质项 $f_r$（BRDF）描述了光线如何与表面交互。游戏引擎的发展过程中，出现过这些技术：
###### Empirical Models

早期游戏引擎（如 DirectX 9 时代）的主流方案有 Lambert（漫反射）、Phong / Blinn-Phong（高光）。Blinn-Phong 模型是一种经验模型，并不完全符合物理规律，但计算极其简单。其基于如下假设：（**光可叠加原理**，Principle of Superposition）来自不同光源的光在某一点产生的效果，等于每个光源单独作用时效果的线性总和。

Blinn-Phong 模型将材质的视觉表现分解，

$$
\begin{aligned}
L &= L_a + L_d + L_s \\
  &= k_aI_a + k_d\left(\frac{I}{r^2}\right)\max(0, \vec{n} \cdot \vec{l}) + k_s\left(\frac{I}{r^2}\right)\max(0, \vec{n} \cdot \vec{h})^p
\end{aligned}
$$

**局限性**：

1. **能量不守恒（Not Energy Conserving）**：更准确的说法是**能量不保守**（Not Energy Conservative），物理正确的光照模型应该是**能量保守**的，即出射能量必须**小于等于**入射能量（因为部分能量会被表面吸收），而不是严格相等（守恒）。
	- **问题**：在某些参数组合下（如过高的 $k_d$ 和 $k_s$），模型计算出的出射光能量可能会超过入射光能量。
	- **表现**：在单次渲染中问题不明显，但在需要多次光线反弹的算法中（如光线追踪），这种能量的微小增长会逐次累积放大，导致渲染结果出现严重错误（如，一个封闭黑盒内部因为能量凭空增加而变得异常明亮）。

2. **缺乏物理真实感（Lacks Physical Realism）**：  
	- **问题**：Blinn-Phong 的参数（$k_d$，$k_s$）与真实的物理材质属性没有直接对应关系，美术师只能依靠经验和试错来调节，难以稳定地创作出各种不同质感的材质。由于其模型的局限性，无论如何调节参数，用 Blinn-Phong 渲染出的物体（无论是木头、石头还是金属）往往都带有一种挥之不去的塑料感。
	- **对比**：现代的 PBR 材质模型通过引入粗糙度（Roughness）、金属度（Metallic）等与物理属性直接挂钩的参数，能够更精准、更真实地表现大千世界的各种材质。

###### Physically Based Models

基于物理的渲染（PBR）是一套渲染准则，要求材质必须遵循物理世界的三个基本原则：

1. **能量守恒**：反射的光不能多于入射光。
2. **微表面理论**：认为所有物体表面都是由无数微小的平面镜组成的。
3. **菲涅尔效应**：视线越平齐（掠射角），物体的反射率越高。

PBR 将繁杂的材质参数统一为 BaseColor（基础色）、Roughness（粗糙度）、Metallic（金属度） 等标准化输入。对于引擎来说，这意味着无论在什么光照下，材质都能表现出一致的真实感。

在实现 PBR 时，主流 BRDF 模型是 Cook-Torrance 模型，其通过一个复杂的公式描述了光线在微表面上的高光反射：

1. **$D$**：法线分布函数（Normal Distribution Function），描述微表面法线的集中程度（决定高光点的大小和亮感）。
2. **$G$**：几何遮挡函数（Geometry Function），描述微表面之间互相遮挡的情况。
3. **$F$**：菲涅尔方程（Fresnel Equation），描述不同观察角度下的反射比，常用 Schlick 逼近法（将复杂的指数运算简化为简单的 5 次方乘法）。

Cook-Torrance 模型将真实物理世界中杂乱无章的表面散射，浓缩成了三个可计算的数学项，使得 GPU 能够并行处理。

###### Pre-computation & Approximation

在计算环境光（IBL）时，根据 Cook-Torrance 公式，每个像素都需要对整个半球环境贴图进行积分采样，这在实时渲染中计算量太大，因此 Epic Games 的 Brian Karis 在普及 PBR 时提出了最重要的工程简化：**拆分求和近似**（Split Sum Approximation），将渲染方程中的材质项与光照项拆开，通过预计算一张 2D Look-up Table（LUT），让复杂的积分在运行时变成一次纹理采样。

在运行时，GPU 只需要**采样两次纹理**并进行简单的乘加运算，就能得到原本需要几千次采样才能算出的环境反射积分结果。

针对 Diffuse（漫反射）部分，使用 Spherical Harmonics 将环境光压成几个系数，彻底消灭积分。


###### Special-purpose Simplification

1. **次表面散射简化（SSS）**：
    - **方案**：Pre-integrated Skin Shading（预积分皮肤着色）。
    - **逻辑**：不去模拟光线在皮肤内部的散射，而是根据物体的厚度或曲率，提前计算好光照衰减图。
2. **多层材质简化（Clear Coat）**：
    - **方案：** 双层 BRDF 叠加。
    - **逻辑：** 将车漆、碳纤维等结构简化为底层（Base）和顶层（Coating）两次高光计算。
3. **布料材质（Cloth）**：
    - **方案**：Silk/Velvet 模型（使用不同的 $D$ 项，如 Charlie 或 Ashikhmin）。

##### Simplify Visibility to Light 

在渲染方程中，计算光照时最昂贵的操作之一是**判断光线是否被遮挡**，其外在表现就是阴影，阴影是提升场景真实感和空间感的关键元素。

在图形学早期，涌现了大量阴影算法，如 Shadow Volume（阴影锥）、Perspective Shadow Maps 等。但在过去十几二十年的游戏行业发展中，Shadow Map（阴影贴图）凭借其相对简单的实现、稳定的效果和对硬件的友好性，成为了事实上的行业标准，并在此基础上衍生出无数的改进技术。
###### Shadow Map

Shadow Map 放弃了物理上精确的光线追踪，而是从光源视角渲染一张深度图，然后在运行时对比深度值来判断是否在阴影中。

**算法流程**：
1. **生成阶段（Light Pass）**：
	- 将摄像机放置在光源位置，朝向场景进行渲染。
	- 此过程不关心颜色、材质等信息，只将离光源最近的物体的深度值写入一张纹理中。
2. **渲染阶段（Camera Pass）**：
	- 正常从玩家的摄像机视角渲染场景。
	- 对于场景中的每一个需要计算光照的片元（Fragment），将其坐标反向投影（Reproject）到光源的裁剪空间中。
	- 计算出该片元到光源的距离 $d_{current}$。
	- 查询 Shadow Map 中对应位置存储的深度值 $d_{shadowmap}$（即该方向上离光源最近的遮挡物的距离）。
	- **深度比较**：
		- 如果 $d_{current} > d_{shadowmap}$，意味着当前片元在遮挡物之后，处于阴影中。
		- 如果 $d_{current} \leq d_{shadowmap}$，意味着当前片元就是那个最近的物体（或在它之前），处于光照中。

**主要问题**：
1. **走样（Aliasing）**：Shadow Map 分辨率有限，导致阴影边缘呈现锯齿状。
2. **自遮挡（Self-Shadowing）**/ **阴影粉刺（Shadow Acne）**：由于浮点数精度限制，一个表面上的点在进行深度比较时，可能会被误判为在自己产生的阴影之下，导致表面出现不正确的条纹状阴影。
3. **透视走样（Perspective Aliasing）**：当一个很大的多边形以很小的角度朝向光源时，它在 Shadow Map 中只占很小的区域，导致阴影精度严重不足。


**解决方案**：
1. **阴影偏移（Shadow Bias）**：在进行深度比较时，给从 Shadow Map 中采样到的深度值 $d_{shadowmap}$ 增加一个微小的偏移量（Bias），人为地将表面推离阴影，以避免自遮挡。
	- **副作用**：Bias 会导致一个新问题 Peter Panning，即物体与其阴影分离，看起来像是“浮”在空中，最常见的就是角色脚底的阴影与脚分离。



##### Simplify Computing（Option）

即使渲染方程被简化了，如果场景中有成千上万个物体和光源，计算量依然爆炸。

1. **渲染管线进化**：
    - **Forward Rendering（前向渲染）**：简单但光源多了会非常慢。
    - **Deferred Rendering（延迟渲染）**：将几何信息（G-Buffer）与光照计算解耦。先画物体，再统一算光照，避免了被遮挡像素的无效计算。
    - **Clustered Shading（集群渲染）**：将空间切成小块（Clusters），每个像素只计算影响它的那一小撮灯光。
2. **LOD（Level of Detail）**：远处的物体用更简单的模型和更简单的 Shader 计算。



#### Pre-computed Global Illumination

为了在游戏中实现逼真的光照效果，我们需要模拟光线的多次反弹。

![](_imgs/Pasted%20image%2020251024174424.png)

实时计算全局光照太过困难，因此可以预先计算全局光照的结果，以空间换时间。对于场景中，其实大部分物体都是静止不动的，比如假设场景中 $90 \%$ 的物体是静止不动的，而且设置好了场景中的光源角度，其实就可以通过预计算来提前算出来全局光照。

假设预计算出了全局光照，此时对于间接光需要面临两个挑战：
1. **数据量**：对于场景中的任意一点，其接收到的间接光来自四面八方（一个球面空间的采样），相当于将球面像地图一样展开，但是如果我们存这样的数据的话，数据量会十分的大，因为场景中每个像素点都要存一个。
2. **计算量**：即使存下了整个场景的光照，在运行时，要计算某个像素的最终颜色，需要将该点接收到的球面光照函数与其材质的BRDF进行复杂的积分运算，**计算量大**。

虽然可以用蒙特卡洛积分去进行采样，但在绘制时，如果每一个 fragment 上都进行这么一个球状的光照函数采样再累加的这么一个卷积运算，那么计算量太大了。

因此我们需要想出一种能让积分快速进行的方法，接下来就需要傅里叶变换和一些其他的数学知识了。

##### Spherical Harmonics

> [!tip] 傅里叶变换（Fourier Transform）
> 傅里叶变换（Fourier Transform）可以将一个信号（如图像、声音）从其原始域（如空间域、时域）转换到频域（Frequency Domain），将复杂的信号分解为一系列不同频率的正弦/余弦波的叠加。
> $$
> f(x) = \frac{A}{2} + \frac{2A \cos(t\omega)}{\pi} - \frac{2A \cos(3t\omega)}{3\pi} + \frac{2A \cos(5t\omega)}{5\pi} - \frac{2A \cos(7t\omega)}{7\pi} + \cdots
> $$
> ![](_imgs/Pasted%20image%2020251231170128.png)
> 
> **频域思想**带来的两大优势：
> 1. **高效的数据压缩**（Efficient Data Compression）：大部分自然信号（包括光照）的能量都集中在低频部分，通过在频域中仅保留少数低频部分的系数，我们就可以得到原始信号的一个很好的近似。
> 	- 如：一张 200x200 的图像可能需要数万个像素数据来存储，但我们可能只需要几十个频域系数就能大致还原出图像的轮廓。这是一种非常高效的压缩方式。
> 2. **简化的卷积运算**（Simplified Convolution）：两个函数在空间域中的卷积，等价于它们在频域中的逐元素相乘（点积）。这意味着，原本极其耗时的积分运算，在转换到频域后，可以变成一个非常简单的乘法运算，计算复杂度大大降低。

Spherical Harmonics（球极谐波）将分布在球面上的函数（如来自四面八方的光照）拆解为一系列基础形状（基函数）的组合。基函数中的每个函数都是二维函数，并且每个二维函数都是定义在球面上的，相互正交。

在图形学中，球面函数 $f(\omega)$（如环境光）通常被近似表示为一系列基函数的加权之和：

$$f(\omega) \approx \sum_{l=0}^{n} \sum_{m=-l}^{l} c_{l}^{m} y_{l}^{m}(\omega)$$

其中，**$c_{l}^{m}$** 为存储在顶点或探针中的 SH 系数（通常是 RGB 三元组）；$y_{l}^{m}(\omega)$ 为实值基函数。为了消除复数，我们根据 $m$ 的正负定义不同的基函数：

$$y_{l}^{m}(\theta, \phi) = \begin{cases} \sqrt{2} N_{l}^{m} P_{l}^{m}(\cos \theta) \cos(m\phi) & \text{if } m > 0 \\ N_{l}^{0} P_{l}^{0}(\cos \theta) & \text{if } m = 0 \\ \sqrt{2} N_{l}^{|m|} P_{l}^{|m|}(\cos \theta) \sin(|m|\phi) & \text{if } m < 0 \end{cases}$$

其中， $P_l^m(x) = \frac{(-1)^m}{2^l l!} (1-x^2)^{m/2} \frac{d^{l+m}}{dx^{l+m}} (x^2-1)^l$ 是**伴随勒让德多项式**（Associated Legendre Polynomials），公式的核心部分；$N_l^m = \sqrt{\frac{(2l+1)}{4\pi} \frac{(l-m)!}{(l+m)!}}$ 是归一化常数，确保基函数在球面上是正交归一的。


**公式参数**：
- $l$：阶数（Degree），决定了波动的复杂程度（类似傅里叶级数中的频率）
	- 取值范围：$l \ge 0$ 且为整数。
- $m$：次数（Order），决定了绕 $z$ 轴的旋转对称性。
	- 取值范围： $-l \le m \le l$。
- $\theta$：球坐标中的极角（Colatitude）
- $\phi$：球坐标中的方位角（Azimuth）。


在 Shader 代码中，为了性能，我们通常不使用 $\theta, \phi$（角度计算慢），而是直接使用单位方向向量 $(x, y, z)$，图形学中最常用的前两阶（Order 2，9个系数）的展开形式：

| **阶数 (l,m)** | **索引 i** | **基函数 $y_i​(x,y,z)$ 的简写形式**  | **物理意义**      |
| ------------ | -------- | ---------------------------- | ------------- |
| $l=0, m=0$   | 0        | $0.282095$                   | **DC (平均亮度)** |
| $l=1, m=-1$  | 1        | $0.488603 \cdot y$           | 纵向梯度          |
| $l=1, m=0$   | 2        | $0.488603 \cdot z$           | 深度梯度          |
| $l=1, m=1$   | 3        | $0.488603 \cdot x$           | 横向梯度          |
| $l=2, m=-2$  | 4        | $1.092548 \cdot xy$          | 象限对称          |
| $l=2, m=-1$  | 5        | $1.092548 \cdot yz$          | -             |
| $l=2, m=0$   | 6        | $0.315392 \cdot (3z^2 - 1)$  | 轴向挤压          |
| $l=2, m=1$   | 7        | $1.092548 \cdot xz$          | -             |
| $l=2, m=2$   | 8        | $0.546274 \cdot (x^2 - y^2)$ | 蝶形对称          |

###### In the Engine

游戏中的环境光，尤其是移除了太阳等主光源后的间接光照，其主要特征是低频的：光照变化缓慢、柔和，没有硬朗的阴影。球谐函数正是为此而生，可以用极少的几个系数就准确地捕捉到这种低频光照的整体感觉。

**实践**：
1. 在现代很多引擎中，为了极致的性能和存储效率，通常只使用到一阶 SH（即 $l = 0$ 和 $l = 1$，共 4 个基函数）来编码环境光。
	- **效果**：即便只用一阶 SH，重建出的环境光贴图虽然模糊，但它准确地保留了**光线的主要方向和强度**，对于计算漫反射效果已经完全足够。
2. **存储压缩**：SH 只存几个数字（系数）就可以知道每个点四周的光是从哪儿来的，**为了计算 RGB 三色光照，每一阶都需要为 R、G、B 通道各存储一组系数**。
	- $l = 0$（零阶）：基础环境光，代表这个点的平均亮度。
		- **系数数量**：1个（$m=0$）。
		- **RGB 存储**：$1 \times 3 = 3$ 个数值。
		- **数学意义**：$Y_0^0 = 0.282095$（一个常数）。
		- **物理意义**：平均亮度（DC Component），它代表了球面所有方向光照的平均值。如果只用零阶，物体看起来会像被均匀的漫反射光包围，没有任何明暗变化。
		- **存储特性**：这是光照能量最集中的部分，通常是 HDR（高动态范围）格式。
	- $l = 1$（一阶）：方向性梯度，代表光波动的主要方向（左边亮还是右边亮）。
		- **系数数量**：3个（$m = -1, 0, 1$）。
		- **RGB 存储**：$3 \times 3 = 9$ 个数值。
		- **数学意义**：分别对应 $y, z, x$ 轴方向的正弦/余弦变化。
		- **物理意义**：光照的主要方向（Directional Bias），描述了光是从哪个“半球”过来的。
		    - 比如：如果 $x$ 轴系数很大，说明左边比右边亮。
		- **存储特性**：通常表示相对变化，范围较小，常作为 LDR（低动态范围）压缩存储。
	- $l = 2$（二阶）：光照细节（曲率）
		- **系数数量**：5 个 ($m = -2, -1, 0, 1, 2$)。
		- **RGB 存储**：$5 \times 3 = 15$ 个数值。
		- **数学意义**：二阶多项式，如 $xy, yz, 3z^2-1$ 等。
		- **物理意义**：光照的“形状”（Highlight Shaping），能表现出光照在球面上更复杂的分布，比如“相对的两侧亮，中间暗”或者更尖锐的光照过渡。
		- **实际现状**：很多移动端游戏为了省内存只存到一阶，高端 PC / 主机游戏会用到二阶。
3. **计算简化**：
	- **真实的物理过程（卷积）**：想要计算一个物体表面的颜色，需要计算：“来自四面八方的光” $\times$ “表面对每个方向光的反射率”，然后全部加起来（积分），这在实时游戏中是根本算不出来的，计算量太恐怖。
	- **SH 的数学魔法（点积）**：如果你把“光照”和“材质反射”都转换成 SH 系数，那么上面的复杂计算就变成了：系数 A · 系数 B = 最终颜色，也就是简单的向量点积（对应位相乘再相加）。

Spherical Harmonics 在引擎中主要负责漫反射（Diffuse）的间接光，而镜面反射通常交给反射探针（Reflection Probes）或光线追踪。


##### Lightmap

Lightmap 是将场景中静态物体接收到的光照效果（包括直接光、阴影、间接反弹光）提前计算好，并存储在纹理贴图中。在游戏运行时，显卡只需要简单地采样这张图，就能显示出极其真实的全局光照效果。

> [!tip] Lightmap 的演进
> 1. **早期（Quake时代）**：由 John Carmack 提出，最初的 Lightmap 主要用于预计算静态阴影，存储的是一个标量光照值。
> 2. **现代（GI时代）**：Peter-Pike Sloan 提出将 SH 应用于光照，Lightmap 的每个纹素（texel）不再存储一个简单的颜色值，而是存储一组 SH 系数，这使得 Lightmap 能够记录下每个点的方向性光照信息，从而可以正确地计算出法线对光照的影响，得到高质量的漫反射甚至部分高光效果。

**工作流程**：
1. **离线计算**：在开发阶段，引擎（如 Unity 或 Unreal）会使用高精度的光线追踪算法（如光能传递 Radiosity）计算光线在场景中的反复弹射。
2. **生成贴图**：计算结果被保存为一张或多张纹理图片。
3. **采样叠加**：在游戏运行时，物体的基础颜色（Albedo）会与 Lightmap 的颜色相乘，合成最终画面。

**关键技术点**：UV2 通道
1. **纹理 UV（UV0）**：用于平铺基础材质（如砖墙、木纹），为了节省空间，通常会有大量重叠（Overlapping）。
2. **光照 UV（UV2）**：专门为 Lightmap 设计，必须满足两个严苛条件：
    - **不可重叠**：场景中每个点的光影都是唯一的，所以 UV 必须平展开。 
    - **必须在 $[0, 1]$ 空间内**：不能像普通纹理那样平铺。


**优点**:  
1. **极高的运行时性能**：运行时，渲染一个 Lightmap 的成本约等于多采样一张纹理，**开销极低**。
2. **极高的视觉质量**：离线计算没有时间限制，可以运行非常复杂和精确的光线追踪算法，从而产生极其细致、微妙（subtle）的光影效果，深受美术喜爱。
  
**缺点**:  
1. **超长的预计算时间**：烘焙过程耗时极长，是开发流程中的一个巨大瓶颈。
2. **完全静态**：Lightmap 只能处理静态物体（Static Objects）和静态光源（Static Lights）。一旦场景中的物体或光源发生移动，之前烘焙的所有光照信息都会失效，必须重新烘焙。
3. **对动态物体的融合难题**：动态物体无法拥有预烘焙的 Lightmap。传统的 Hack 方法（如在物体脚下采样 Lightmap 颜色）效果很差，经常出现“角色走进烘焙好的阴影里突然变黑”的穿帮问题。
4. **巨大的存储开销**：Lightmap 本质是纹理，会占用大量的显存和磁盘空间（通常为几十到几百MB），这正是其空间换时间策略的代价。


##### Light Probes

> [!tip] 为什么需要它？（弥补 Lightmap 的缺陷）
> Lightmap 只能用于静态物体（如墙壁、地面）。
> - **问题**： 如果你的主角（动态物体）走进了一个只有 Lightmap 的小黑屋，而屋子里由于烘焙有一束很亮的间接光，主角身上通常还是黑的，因为他没法使用墙上的贴图。
> - **解决方案**： 在空间中放置很多点（探针），提前记录下这些位置的光照。当主角移动到这些点附近时，就从这些点“借”一点光。

Light Probes 是游戏引擎中用于解决“动态物体如何接收静态全局光照”的技术，每个 Light Probe 实际上存储的就是一组 SH 系数（通常是一阶或二阶）。

**核心机制**：插值（Interpolation），玩家在场景中是连续移动的，但探针是离散的点。为了保证光照平滑过渡，引擎会进行插值：
1. **寻找邻居**：引擎会找到离动态物体最近的 4 到 8 个探针（通常通过四面体插值 Tetrahedral Interpolation）。
2. **混合系数**：根据物体与这些探针的距离，加权计算出一组全新的“混合 SH 系数”。
3. **实时渲染**：将这组混合后的系数传给物体的 Shader，在顶点或像素阶段计算出最终的漫反射光照。



#### Physically Based Rendering

基于物理的渲染（Physically Based Rendering，PBR）是现代游戏引擎（如 UE5，Unity，Frostbite）渲染管线的基石，其理论基础是**微平面理论**（Microfacet Theory）。

##### Microfacet Theory 

微平面理论（Microfacet Theory）是现代物理建模渲染的基石，其核心思想是：在宏观上看起来粗糙的物体表面，在微观尺度下其实是由无数个微小的、完美的镜面组成的。

> [!tip] 反射定律
> 只有当平面的法线 $m$ 正好位于入射光方向 $l$ 和观察方向 $v$ 的正中间时，观察者才能看到反射光。

半程向量（Half-way Vector） $h$ 是理解微平面理论最重要的数学技巧，定义

$$
h = \frac{l + v}{||l + v||}
$$
在计算某个像素的亮度时，实际上是在问：“在这一块宏观表面内，有多少比例的微平面法线 $m$ 正好等于 $h$？”


##### Cook-Torrance BRDF

Cook-Torrance BRDF（双向反射分布函数）是描述物体表面**高光反射**最核心的数学模型，在渲染方程中，BRDF 项 $f_r$ 通常被拆分为**漫反射**（Diffuse）和**高光**（Specular）两个部分：

$$
f_r = k_{d}f_{lambert} + k_{s}f_{specular}
$$

其中， $k_d$​ 是漫反射比例（通常由 Fresnel 项决定），$k_s$​ 是反射比例，且满足能量守恒 $k_d​+k_s​=1$ 。漫反射项（Lambertian） $f_{lambert}$ 通常采用最简单的 Lambert 模型，假设光线向四周均匀散射：

$$
f_{lambert} = \frac{c}{\pi}
$$

其中，$c$ 是物体的反射率（Albedo/Base Color）。高光项（Cook-Torrance Specular）$f_{specular}$ 公式如下：

$$
f_{specular}​= \frac{D⋅G⋅F​}{4(n⋅l)(n⋅v)}
$$

其中，$n$ 是宏观表面法线；$v$ 是观察方向向量；$l$ 是入射光线方向向量；为了在实时引擎中计算这个公式，需要定义三个关键函数：$D$、$G$、$F$ ：

1. $D$：法线分布函数（Normal Distribution Function，NDF），描述了微表面法线 $h$ 与宏观表面法线 $n$ 的一致程度，**决定了高光的形状和集中度**。其主流实现是 **GGX**（Trowbridge-Reitz）：

$$
D(h)=\frac{\alpha^2}{π((n⋅h)^2(α^2−1)+1)^2}
$$
其中，$\alpha \in [0, 1]$ 是粗糙度 $r$（Roughness）的平方（ $\alpha = r^2$ ）， $\alpha$ 越小，法线越集中（表面越光滑），$\alpha$ 越大，法线越分散（表面越粗糙）。

> [!tip] GGX 核心思想
> GGX 比作一个高品质的音响系统：
> 1. **高音要清脆**（High-frequency Peak）：GGX 的高光核心部分非常尖锐和明亮，能够表现出金属或光滑表面上那种“清脆”的亮点。
> 2. **低音要浑厚**（Low-frequency Tail）：GGX 的高光衰减非常平缓和宽广，形成一个长长的“拖尾”。这使得高光向周围环境的过渡非常柔和，避免了传统模型中常见的“贴膏药”式的生硬边缘。
> 
> 这种中心尖锐，边缘柔和的特性，使得 GGX 能够更真实地模拟现实世界中各种材质的高光反射。

2. $G$：几何函数（Geometry Function），描述了微表面之间产生的相互遮挡（Shadowing）和掩蔽（Masking）现象，**决定了高光的亮度衰减**。其主流实现是 Smith 结合 Schlick-GGX：

$$
\begin{aligned}
G(v,l,n) &= G_1(v)G_1​(l) \\
\\
G_1​(v) &= \frac{n⋅v}{(n⋅v)(1−k)+k}​
\end{aligned}
$$
其中，对于实时渲染的光源，通常取 $k = \frac{(r + 1)^2}{8}$。

> [!tip] Smith 模型的简化理解
> 可以这样通俗地理解其工作原理（以各项同性材质为例）：
> 1. 假设有 100% 的光能射入材质，根据粗糙度 $r$ 和入射角度 $l$，$G$ 函数计算出有 $30\%$ 的光被遮挡（Shadowing），剩下 $70\%$ 的有效光能参与反射。
> 2. 这 70% 的光能向视线方向反射，由于材质是各项同性的，微观结构是随机均匀分布的，因此视线方向同样会发生遮挡。$G$ 函数再次计算，这 $70\%$ 的能量中又有 $30\%$ 被遮蔽（Masking）。
> 3. 最终，到达视线的光能是 $70\% * 70\% = 49\%$。
> 
> 这个过程确保了能量守恒，粗糙的表面不会反射出比入射光更多的能量。

3. $F$：菲涅尔方程（Fresnel Equation），描述了在不同入射角度下，表面反射光线与折射光线的比例，**决定了高光的强度**。其主流实现是 Schlick 逼近法（Schlick's Approximation）：

$$
F(h,v)=F_0​+(1−F_0​)(1−(h⋅v))^5
$$

其中，$h$ 是半程向量，即 $v + l$ 的归一化向量；$F_0​$ 是基础反射率（垂直观察时的反射率），非金属的 $F_0$​ 通常很低（如 0.04），而金属的 $F_0$​ 很高且带有颜色。这个 $5$ 次方是一个经过大量实验和拟合得出的经验值，它能很好地模拟真实世界中反射率随角度变化的曲线。

> [!tip] 菲涅尔效应
> 菲涅尔方程描述了一个众所周知的物理现象：当你的视线以掠射角度（接近平行于表面）观察一个物体时，其反射率会急剧增加。

4. 分母 $4(n⋅l)(n⋅v)$ ：是一个数学上的修正因子，源于从微观几何面积向宏观立体角转换时的坐标系变换 Jacobian 决定式。在实际代码编写中，为了防止除以零，通常会给分母加上一个极小的偏移量（如 0.0001）。

通过将 $D$、$G$、$F$ 三个组件相乘，Cook-Torrance BRDF 模型就构建完成了。

其**成功在于**：
1. **物理 plausible**：基于微表面理论，比传统经验模型（如 Blinn-Phong）更符合物理规律和能量守恒。
2. **参数直观**：艺术家不再需要调整晦涩的 Power 值，而是通过 Roughness（粗糙度）、Metallic (金属度) 和 Albedo（基础色）等直观参数来定义材质。
3. **表现力强**：仅用少数几个参数，就能生动地表达出从粗糙的混凝土到光滑的金属等各种截然不同的材质。

> [!tip] 从理论到实践：测量 BRDF 数据
> 为了让艺术家能够准确地设置 PBR 参数，图形学界付出了巨大的努力来测量真实世界材质的 BRDF 数据。
> 
> 像 MERL（Mitsubishi Electric Research Laboratories）这样的机构，使用精密设备（测角反射计）扫描了上百种真实材质，并将其 BRDF 数据公开，形成了著名的 MERL 数据库。
> 
> 这些测量数据为引擎开发者和艺术家提供了宝贵的参考。我们可以知道，真实的木头、塑料、黄金等材质，其 $Roughness$ 和 $F_0$ 值大概在什么范围，从而创建出更可信的数字资产。这为 PBR 工作流的标准化和普及奠定了坚实的基础。
> 
> ![](_imgs/Pasted%20image%2020260101161810.png)

##### Disney Principled BRDF

Disney Principled BRDF 是由迪士尼动画工作室的 Brent Burley 在 2012 年 SIGGRAPH 会议上提出的材质模型，其目标是**将复杂的物理方程转化为艺术家直观可调的“参数化”系统**。这一模型后来直接演变成了现代游戏引擎中 Metallic/Roughness 工作流的行业标准。

在它出现之前，图形学界虽然有了物理正确的 Cook-Torrance 模型，但参数极其复杂（如折射率 $n$，$k$ 等），艺术家很难上手。

**设计准则**：
1. **直观而非物理**：参数应该是艺术家容易理解的（如粗糙度），而不是物理公式里的系数（如 $F_0$​）。
2. **参数尽量少**：能够用最少的滑块覆盖绝大多数材质。
3. **参数范围标准化**：所有参数都应该在 $[0,1]$ 之间。
4. **鲁棒性**：无论参数如何组合，结果都应该是物理上合理的（即不会出现不符合能量守恒的发光情况）。

**核心参数**：迪士尼最初提出了 11 个参数，虽然游戏引擎为了性能简化了其中一部分，但最核心的几项已经成为所有 PBR 系统标配：
1. **Base Color（基础色）**：表面的反射率，对于金属来说是镜面反射颜色，对于电介质来说是漫反射颜色。
2. **Subsurface（次表面）**：控制光线在表面下的散射（模拟皮肤、玉石）。
3. **Metallic（金属度）**：0 为电介质（木头、塑料），1 为纯金属。它会改变 F0​ 的计算逻辑。
4. **Specular（高光度）**：调节非金属表面的入射反射量。
5. **Roughness（粗糙度）**：控制微平面的散射程度（即高光的模糊度）。
6. **Anisotropic（各向异性）**：模拟拉丝金属或光盘等具有方向性纹理的材质。
7. **Sheen（光泽度）**：模拟布料边缘的微小绒毛感。
8. **Clearcoat（清漆层）**：在材质表面额外增加一层半透明的透明涂层（如车漆）。

![](_imgs/Pasted%20image%2020260101164755.png)


##### Specular/Glossiness

> [!tip] 两种主流的材质工作流
> PBR 的理论模型虽然统一，但在实践中，为了方便美术师创作并保证物理正确性，业界演化出了两种主流的材质工作流（Workflow）：一套是目前成为主流的 Metallic/Roughness（MR）模型，另一套就是你提到的 Specular/Glossiness（SG）模型。
> 
> 虽然现在很多游戏引擎默认使用 MR 模型，但 SG 模型在离线渲染（如 V-Ray）和一些高性能 3A 资产制作中依然占有一席之地。

Specular/Glossiness（SG）模型是一种通过直接定义材质的基础反射率（$F_0$）和平滑度来表现物体特征的方法。SG 模型最大的特点是将材质的各个核心物理属性完全通过贴图来控制，赋予了艺术家像素级别的精确控制能力，几乎不需要手动设置零散的数值参数。

**核心贴图**（Core Maps）：在 SG 工作流中，材质的属性由以下三张关键贴图决定：
1. **Diffuse**（漫反射贴图）
	- **作用**：RGB 三通道，存储物体的固有色（不含光影信息的纯色），常被称为 Albedo。
	- **特殊规则**：在物理上，金属会吸收几乎所有的折射光。因此，在 SG 模型中，纯金属的 Diffuse 颜色必须是全黑的。
2. **Specular**（高光/反射贴图）
	- **作用**：RGB 三通道，直接定义材质的 $F_0$  (垂直入射时的反射率)。
	- **特点**：它是一张 RGB 彩色贴图。
		- **非金属**：使用灰度值（通常在 $2\%∼5\%$ 左右的反射率）。
		- **金属**：使用彩色（因为金属会吸收特定波长的光，导致反射光带有颜色，如金、铜）。
3. **Glossiness**（光泽度贴图）
	- **作用**：单通道灰度图，描述表面的平滑程度。
	- **数学关系**：它与 MR 模型中的 Roughness（粗糙度）是相反的，$Glossiness=1−Roughness$。
	- **表现**：白色（1.0）代表极度光滑，产生尖锐的高光；黑色（0.0）代表极度粗糙，高光完全弥散。

**优点**：
1. **功能强大且完整**：能够精确地表达各种材质，无论是金属还是非金属。
2. **符合物理直觉**：遵循迪士尼原则，艺术家创作出的材质效果稳定且可预测。
3. **像素级控制**：给予艺术家极高的创作自由度。

**缺点**：
1. **过于灵活**：特别是 `Specular` 贴图，艺术家需要同时理解并绘制 $F_0$ 的颜色和强度，对于区分导体（金属）和绝缘体（非金属）的物理规则需要有一定认知，否则容易出错。

**Shader 中的实现**：
1. **材质参数准备**（Material Inputs）：
```hlsl
// 1. 采样贴图
float3 albedo = pow(tex2D(BaseColorMap, uv).rgb, 2.2); // 转回线性空间
float roughness = tex2D(RoughnessMap, uv).r;
float metallic = tex2D(MetallicMap, uv).r;

// 2. 计算 F0 (基础反射率)
// 电介质 (非金属) 的 F0 默认为 0.04，金属则使用 albedo
float3 F0 = float3(0.04, 0.04, 0.04); 
F0 = lerp(F0, albedo, metallic);

// 3. 计算中间变量
float3 N = normalize(worldNormal);
float3 V = normalize(viewPos - worldPos);
float alpha = roughness * roughness; // 采用迪士尼的平方映射
```

2. **实现 Cook-Torrance 三项**：
```hlsl
float DistributionGGX(float3 N, float3 H, float alpha) {
    float a2 = alpha * alpha;
    float NdotH = max(dot(N, H), 0.0);
    float denom = (NdotH * NdotH * (a2 - 1.0) + 1.0);
    return a2 / (PI * denom * denom);
}

float3 FresnelSchlick(float cosTheta, float3 F0) {
    return F0 + (1.0 - F0) * pow(1.0 - cosTheta, 5.0);
}

float GeometrySchlickGGX(float NdotV, float k) {
    return NdotV / (NdotV * (1.0 - k) + k);
}

float GeometrySmith(float3 N, float3 V, float3 L, float k) {
    float NdotV = max(dot(N, V), 0.0);
    float NdotL = max(dot(N, L), 0.0);
    return GeometrySchlickGGX(NdotV, k) * GeometrySchlickGGX(NdotL, k);
}
```

3. **合成渲染方程**（Assembly）：
```hlsl
// 计算单光源贡献
float3 H = normalize(V + L);
float NdotL = max(dot(N, L), 0.0);
float NdotV = max(dot(N, V), 0.0);

// 计算 D, G, F
float  D = DistributionGGX(N, H, alpha);
float3 F = FresnelSchlick(max(dot(H, V), 0.0), F0);
float  G = GeometrySmith(N, V, L, k);

// 计算 Specular 项
float3 numerator = D * G * F;
float denominator = 4.0 * NdotV * NdotL + 0.0001; // 防止除以零
float3 specular = numerator / denominator;

// 计算 Diffuse 项 (基于能量守恒)
float3 kS = F;            // 反射的比例
float3 kD = 1.0 - kS;     // 折射的比例
kD *= 1.0 - metallic;     // 金属几乎不产生漫反射

float3 diffuse = kD * albedo / PI;

// 最终颜色输出
float3 Lo = (diffuse + specular) * radiance * NdotL;
```


##### Metallic/Roughness

Metallic/Roughness（MR）模型源自 Disney Principled BRDF 的设计理念，通过将复杂的物理属性压缩为几个直观的参数，极大地降低了美术创作的门槛，同时保证了渲染的物理正确性，是目前现代游戏引擎和工业标准（如 glTF 格式）中最主流的 PBR 工作流。

**核心贴图**（Core Maps）：在 MR 工作流中，物体的材质表现主要由以下三张贴图定义：
1. **Base Color**（基础色）：
	- **非金属**（电介质）：代表物体的漫反射颜色（Albedo）。
	- **金属**：代表物体的反射颜色（Specular Color）。因为金属没有漫反射，光线进入表面后会被立即吸收。
	- **关键规则**：贴图中不能包含任何光影信息（如 AO 或阴影），必须是纯粹的颜色。
2. **Metallic**（金属度）：
	- **取值**：通常是二值化的（0 或 1）。
		- 0（黑色）：绝缘体/电介质（木头、塑料、布料）。
		- 1（白色）：纯金属（金、银、铝）。
	- **作用**：它是控制渲染引擎如何处理 $F_0$（基础反射率）和 $k_d$（漫反射比例）的开关。
3. **Roughness**（粗糙度）：
	- **取值**：0（极光滑）到 1（极粗糙）。
	- **作用**：对应微平面理论中的法线分布（$D$ 项），粗糙度越高，高光越模糊；粗糙度越低，高光越尖锐。
	- **映射**：在渲染管线内部，通常采用 $\alpha = Roughness^2$ 来获得更自然的视觉过渡。

**内部逻辑**：
1. **自动确定 $F_0$​**（基础反射率）：在 SG 模型中，你需要手动画 $F_0$​。但在 MR 模型中：
	- $Metallic = 0$：引擎自动将 $F_0$​ 设为固定的 $0.04$（绝大多数非金属的平均反射率）。 
	- $Metallic = 1$：引擎将 Base Color 作为 $F_0$ ​
	- 公式：`F0 = lerp(0.04, BaseColor, Metallic)`
2. **自动执行能量守恒**：引擎会自动根据金属度来剔除漫反射成分：
	- **纯金属**：所有的 $k_d​$（漫反射）被设为 $0$。
	- **公式**：`kD = (1.0 - Specular) * (1.0 - Metallic)`

**局限性**：虽然它很强大，但也有力所不及的地方：
1. **非金属的 $F_0​$ 调整**：有些特殊的电介质反射率不是 0.04（如宝石或水）。为此，UE4/5 引入了一个额外的 Specular 参数（默认 0.5 对应 0.04 反射率）来微调这部分。
2. **白边走样**（White Edges）：在金属和非金属的交界处（由于贴图插值），可能会出现一圈白色的像素点，这是 MR 模型天然的走样问题。

**Shader 中的实现**：
1. **材质参数准备**（Material Inputs）：
```hlsl
// 1. 颜色空间转换 (sRGB -> Linear)
// BaseColor 必须在线性空间计算
float3 albedo = pow(texBaseColor.Sample(uv).rgb, 2.2); 

// 2. 采样 MR 贴图
float metallic  = texMetallic.Sample(uv).r;
float roughness = texRoughness.Sample(uv).r;

// 3. 计算 Alpha (迪士尼平方映射)
// 让粗糙度的视觉变化更线性
float alpha = roughness * roughness;

// 4. 【核心步骤】计算 F0 (基础反射率)
// 非金属默认使用 0.04 (4% 反射率)，金属则直接使用 Albedo 的颜色作为反射率
float3 F0 = float3(0.04, 0.04, 0.04);
F0 = lerp(F0, albedo, metallic);
```

2. **实现 Cook-Torrance 三项**：
```hlsl
float DistributionGGX(float3 N, float3 H, float a) {
    float a2 = a * a;
    float NdotH = max(dot(N, H), 0.0);
    float denom = (NdotH * NdotH * (a2 - 1.0) + 1.0);
    return a2 / (PI * denom * denom);
}

float3 FresnelSchlick(float cosTheta, float3 F0) {
    return F0 + (1.0 - F0) * pow(1.0 - cosTheta, 5.0);
}

float3 FresnelSchlick(float cosTheta, float3 F0) {
    return F0 + (1.0 - F0) * pow(1.0 - cosTheta, 5.0);
}
```

3. **光照整合**（按单光源计算）：
```hlsl
// 1. 基础向量计算
float3 L = normalize(lightPos - worldPos);
float3 H = normalize(V + L);
float NdotL = max(dot(N, L), 0.0);
float NdotV = max(dot(N, V), 0.0);

// 2. 运行 Cook-Torrance 三项
float  D = DistributionGGX(N, H, alpha);
float  G = GeometrySmith(NdotV, NdotL, roughness);
float3 F = FresnelSchlick(max(dot(H, V), 0.0), F0);

// 3. 计算高光项 (Specular)
float3 numerator = D * G * F;
float denominator = 4.0 * NdotV * NdotL + 0.0001; // 防止除以0
float3 specular = numerator / denominator;

// 4. 【核心步骤】计算能量守恒下的漫反射
// kS 是反射的比例，即菲涅尔项 F
float3 kS = F;
// kD 是折射（漫反射）的比例
float3 kD = float3(1.0, 1.0, 1.0) - kS;
// 关键：如果是纯金属，则完全没有漫反射
kD *= (1.0 - metallic);

// 5. 最终输出
float3 diffuse = kD * albedo / PI;
float3 directLight = (diffuse + specular) * lightColor * NdotL;
```

4. **关键逻辑点**（总结）：
	- **$F_0$ 的线性插值**：`F0 = lerp(0.04, albedo, metallic)`，这行代码是整个 MR 的灵魂。它解决了电介质和金属在物理特性上的巨大差异。
	- **$k_D$ 的修正**：`kD *= (1.0 - metallic)`，保证了当你把金属度拉满时，漫反射部分会自动消失，符合金属吸收折射光的物理事实。
	- **通道打包**（Channel Packing）：在实际工程中，`metallic` 和 `roughness` 通常会被存在一张贴图的不同通道里（例如 R 通道存金属度，G 通道存粗糙度），减少显存压力。

5. **后处理**（Post-Processing）：由于 Shader 计算是在 HDR（高动态范围）下进行的，还需要在输出前进行 Tone Mapping（色调映射）和 Gamma 校正，否则画面会看起来太黑或者颜色失真。
```hlsl
// 伪代码：输出前的最后处理
color = ToneMapping(directLight + ambient); // 映射到 0-1
color = pow(color, 1.0 / 2.2);              // Gamma 2.2 校正
```


#### Image-Based Lighting

基于图像的光照（Image-Based Lighting，IBL）是一种将 360 度全景图像视为一个巨大光源的渲染技术，IBL 要解决的问题是如何让物体融入到真实的环境光照中。

在传统的渲染中，我们通过手动放置点光源、方向光来照亮场景。而在 IBL 中，全景图（通常是 HDR 格式）的每一个像素都被当作一个发射光线的发光点。这使得物体能够完美地融入其周围的环境，获得极其真实的间接光照和反射效果。

**核心逻辑**：**环境即光源**，渲染方程中的入射光 $L_i$​ 不再由简单的数学公式（如点光源）提供，而是通过对环境贴图（Environment Map）进行采样获得：

$$
L_o​(x, \omega_o) = \int_{\Omega} ​f_r(x,\omega_o, \omega_i)​ L_i​(x, \omega_i​)cos \theta_i d\omega_i 
$$

为了在实时引擎中高效运行，IBL 通常被拆分为漫反射（Diffuse）和高光（Specular）两个部分进行处理：

$$
\begin{aligned}
L_o​(x, \omega_o) &= \int_{\Omega} ​f_r(x,\omega_o, \omega_i)​ L_i​(x, \omega_i​)cos \theta_i d\omega_i \\
&= \int_{\Omega} ​(k_df_{Lambert} + f_{CookTorrance}) L_i(x, \omega_i​)cos \theta_i d\omega_i \\
&= \int_{\Omega} ​k_df_{Lambert} L_i(x, \omega_i​)cos \theta_i d\omega_i + \int_{\Omega} ​f_{CookTorrance} L_i(x, \omega_i​)cos \theta_i d\omega_i \\
&= L_d(x, \omega_o) + L_s(x, \omega_o) \\
\end{aligned}
$$

1. **漫反射 IBL**（Diffuse IBL）：辐照度贴图（Irradiance Map）
	- **目标**：模拟来自环境的低频间接光（也就是环境光）。
	- **挑战**：漫反射需要对半球上的所有像素进行积分。
	- **解决方案**：
	    - **预计算辐照度图（Irradiance Map）**：提前将环境贴图进行极致的模糊处理，存储每个方向受到的总光照。
	    - **球谐函数（Spherical Harmonics）**：如我们之前讨论的，将这张模糊的图压缩为 9 个系数，计算速度极快。
$$
\begin{aligned}
L_d(x, \omega_o) &= \int_\Omega k_d f_{Lambert} L_i(x, \omega_i) cos \theta_i d\omega_i \\
&\approx k_d^*c \frac{1}{\pi} \int_\Omega L_i(x, \omega_i) cos \theta_i d\omega_i
\end{aligned}
$$

> [!quote] 推导
> 在严谨的物理模型中，$k_d$​（即 $1−F$）其实是与入射角 $\theta_i$​ 相关的（Fresnel 效应），但在实时渲染中，为了优化性能通常会做出以下假设：
> - **预计算辐照度**（Irradiance Map）：积分部分 $\frac{1}{\pi} \int_{\Omega} L_i(x, \omega_i) cos \theta_i d\omega_i$ 被称为 Irradiance（辐照度）。在游戏引擎中，这部分通常通过预计算的环境贴图（Cube Map）或球谐函数（Spherical Harmonics）来快速获取。![](_imgs/Pasted%20image%2020260102191748.png)
> - **$k_d$​ 的近似**：将 $k_d$​ 视为一个常数并移出积分号（即 $k_d^*$ ），虽然这在物理上不完全精确（因为 Fresnel 应该在积分内计算），但在漫反射占主导的情况下，这种近似带来的视觉误差很小，且极大提升了计算效率。


2. **高光 IBL**（Specular IBL）：辐射度贴图（Radiance Map），预滤波环境贴图（Pre-filtered Environment Map）
	- **目标**：模拟物体表面的环境反射（镜面反射）。
	- **关键点**：反射的效果取决于物体的**粗糙度**。
	- **解决方案**：使用 Split Sum Approximation（拆分求和近似）。
	    -  **预过滤环境贴图（Pre-filtered Env Map）**：提前生成一系列不同模糊程度的贴图（Mipmaps）。最清晰的层级对应 0 粗糙度（镜子），最模糊的层级对应 1 粗糙度。
	    - **BRDF 查找表（LUT）**：处理材质随角度变化的反射率。

$$
\begin{aligned}
L_s(x, \omega_o) &= \int_{\Omega} ​f_{CookTorrance} L_i(x, \omega_i​)cos \theta_i d\omega_i\\
&\approx \underbrace{ \left( \frac{\int_\Omega f_{CookTorrances} L_i(x, \omega_i) cos \theta_i d\omega_i}{\int_\Omega f_{CookTorrances} cos \theta_i d\omega_i} \right)}_{Lighting\ Term} \cdot \underbrace{ \left( \int_\Omega f_{CookTorrances} cos \theta_i d\omega_i \right)}_{BRDF\ Term} \\
\\
Lighting\ Term  &\approx \frac{\sum_k^N L(\omega_i^k) G(\omega_i^k)}{\sum_k^NG(\omega_i^k)} \\
&\approx \frac{\sum_k^N L(\omega_i^k) \alpha}{\sum_k^NG(\omega_i^k)} \\
\\
BRDF\ Term  &= \int_{\Omega} f cos \theta d\omega_i \\
&\approx \int_{\Omega} (F_0 + (1 - F_0)(1-cos \theta)^5) cos \theta d\omega_i \\
&\approx F_0 \cdot A + B \\
&\approx F_0 \cdot LUT.r + LUT.g
\end{aligned}
$$

> [!quote] 推导
> 原始的镜面反射积分式将环境光 $L_i​$ 与复杂的 BRDF 项耦合在一起，目的是为了将原本无法在实时环境下完成的半球积分，转化为可以预计算的纹理查询。
> 1. **第一步**：使用 Split Sum Approximation（拆分求和近似）将积分“强行”拆分为两个独立积分的乘积，虽然它不完全等价，但在材质粗糙度较低（高光集中）或环境光颜色变化较均匀时，其结果非常接近物理真实情况。
> 2. **第二步**：对于 Lighting Term，将其离散化为蒙特卡洛采样（Monte Carlo Sampling）的求和形式，其中，
> 	- $L(\omega_i^k)$ 代表在特定采样方向 $\omega_i^k$ 上的**环境光辐射度**（Radiance），即环境贴图中对应像素的颜色；
> 	- $G(\omega_i^k)$ 代表重要性采样的权重，在推导中，它通常对应于 Cook-Torrance 模型中的 $D$（法线分布项） 和 $cos \theta$ 的乘积。为了让这一项只取决于粗糙度 $\alpha$，在预计算时假设视角方向 $V=N=R$（即视线、法线和反射方向重合）。
> 	- Lighting Term 被简化为对环境贴图（Cube Map）的预卷积采样，即预过滤环境贴图 （Pre-Filtered Environment Map）。
> 3. 第三步：对于 BRDF Term，
> 	- 引入 Schlick 菲涅尔近似将 $F$ 拆分为包含 $F_0$	（基础反射率）和 $(1−cos \theta)^5$ 的形式。
> 	- 提取变量：通过数学变换，将 $F_0$ 从积分号内提取出来
> 	- 剩下的积分部分只剩下两个变量粗糙度（$\alpha$）和入射角的余弦值（$cos \theta$）。
> 	- 引擎预计算一张 2D 查找表（Look-up Table，LUT），Shader 运行时只需采样这张图，即可得到 A（存储在 R 通道）和 B（存储在 G 通道）的值。
> 
> ![](_imgs/Pasted%20image%2020260102191152.png)
> 
> 结合以上两点，IBL Specular 方案虽然是基于大量假设的**近似解**，但它带来了革命性的视觉提升：
> 1. **真实的高光**：首次让游戏场景中的物体能够从环境中反射出柔和、模糊且带有环境色彩的高光，而不仅仅是点光源产生的生硬光斑。
> 2. **丰富的层次感**：加入 IBL 后，场景的材质质感和空间层次感都得到了极大的增强，整体视觉效果更舒适、更真实。![](_imgs/Pasted%20image%2020260102205620.png)
> 3. **行业标准**：正是由于其出色的效果和高效的性能，IBL 与 PBR 相辅相成，迅速成为过去十几年所有 3A 游戏引擎的标配技术。


#### Classic Shadow Solution

##### Shadow Mapping

Shadow Mapping 是目前工业界最主流、应用最广泛的方案，由 Lance Williams 在 1978 年提出。

**核心原理**：这是一个两趟（Two-pass）算法。
1. **第一趟**：从光源视角渲染场景，只记录深度信息到一张纹理中，称为 Shadow Map（深度图）。
2. **第二趟**：从相机视角渲染场景，将每个像素的坐标转换到光源空间，并与其对应的 Shadow Map 深度值进行比较。如果当前点比 Shadow Map 记录的点更深，则说明该点在阴影中。

**挑战**：走样（Aliasing）分辨率不足导致的锯齿。
1. **自遮挡**（Self-shadowing）：由于数值精度问题，物体表面会产生错误的黑斑（Shadow Acne）。
	- **解决方法**：引入 Bias（偏移量）来消除黑斑。

##### Cascaded Shadow Maps

传统的阴影贴图（Shadow Map）技术，是将整个场景从光源视角渲染到一张深度图上。但在大场景中，这会遇到一个不可调和的矛盾：

1. **高分辨率 Shadow Map**：可以保证近处阴影清晰，但覆盖范围有限，且性能和显存开销巨大。
2. **低分辨率 Shadow Map**：可以覆盖广阔的远景，但会导致近处物体的阴影出现严重的锯齿和失真（像素块）。

无法用一张固定精度的 Shadow Map 同时满足近处枪械的精细阴影和远处山脉的轮廓阴影。

为了解决大场景（如开放世界）中 Shadow Map 分辨率严重不足的问题，级联阴影贴图（Cascaded Shadow Maps，CSM）成为了现代引擎处理日光阴影的标准。

**核心原理**：
1. **视锥体分割**：根据与相机的距离，将视锥体（Frustum）划分为多个层级（Cascades）
2. **独立 Shadow Map**：近处使用覆盖范围小但精度高的 Shadow Map，远处使用覆盖范围大但精度低的 Shadow Map。
3. **分辨率的重新分配**：
	- **近处 Cascade**：用一张高分辨率的 Shadow Map 覆盖一小片区域，保证了近景阴影的极高质量。
	- **远处 Cascade**：用同样分辨率的 Shadow Map 覆盖一大片区域，虽然单位面积的精度下降了，但由于透视效应（近大远小），远处的阴影在屏幕上本身就占据较少像素，这种精度已经足够。
4. **渲染时采样**：在为屏幕上的某个像素着色时，首先判断该像素所代表的物体点位于哪个 Cascade 区域，然后采样对应的 Shadow Map 来计算阴影。


**挑战**：
1. **层级间的混合（Blending）**：如果不进行处理，不同 Cascade 的交界处会因为分辨率的突变而产生一条非常明显的硬边界。实际项目中需要使用各种滤波和混合技术（通常是一些非常巧妙的 dirty hacks）来平滑过渡，消除这条接缝。
2. **性能开销（Performance Cost）**：CSM 是一个非常昂贵的渲染过程。 
	- **绘制调用（Draw Calls）**：场景需要从光源视角被重复绘制多次（每个 Cascade 绘制一次）。
	- **显存占用**：需要存储多张 Shadow Map，增加了显存压力。
	- **CPU 开销**：需要为每个 Cascade 单独进行视锥体裁剪和可见性计算。
	- 在复杂的 3A 游戏中，阴影渲染的耗时通常在 2ms 到 5ms 之间，是渲染管线中最耗时的模块之一。

##### Shadow Volumes

阴影卷轴（Shadow Volumes）是一种基于几何的方案，曾在 Doom 3 中被发挥到极致。

**核心原理**：
1. 根据光源和物体的轮廓，向外延伸挤出一个封闭的几何体（Volume）。
2. 利用模板测试（Stencil Buffer）来统计，如果一个像素在阴影体内部，则该像素处于阴影中。

**优点**：阴影边缘极其锐利，且没有阴影贴图那样的分辨率问题（像素级精确）。

**缺点**：对几何体处理开销极大，且非常消耗显存带宽（Fill-rate），难以处理带透明贴图（如叶子）的物体。


##### Percentage Closer Filtering

百分比靠近过滤（Percentage Closer Filtering，PCF）是一种为了产生软阴影（Soft Shadows）而对 Shadow Mapping 进行的改进技术。

**核心原理**：
1. 在进行深度比较时，不再只采样一个点，而是采样目标像素周围的一个邻域，并计算采样点中处于非阴影状态的比例，从而得到一个平滑的阴影边缘。



###### Percentage Closer Soft Shadows

百分比靠近软阴影（Percentage Closer Soft Shadows，PCSS）是一个高级变种，它能模拟出更真实的半影（Penumbra）效果，即阴影会随着遮挡物与接收物距离的增加而变得更加模糊。PCSS 通过动态计算 PCF 的滤波范围来实现这一点，是目前许多引擎中标配的高质量软阴影方案。

**核心原理**：让阴影的柔和程度与遮挡物和接收物之间的距离相关联。
1. 首先，通过一次采样（Sample 0）来估算遮挡物（Blocker）的平均深度。
2. 然后，根据这个平均深度以及光源的大小，计算出合适的滤波范围（Penumbra Size）。
3. 最后，在这个动态计算出的范围内进行 PCF 滤波。

PCSS 是一种非常成熟且效果出色的技术，在许多现代游戏引擎中都是软阴影的标配方案，能有效缓解阴影的锯齿（Aliasing）问题。




##### Variance Shadow Maps

变体阴影贴图（Variance Shadow Maps，VSM）是一种基于统计学思想的软阴影技术，它通过存储深度的均值和方差来快速估算阴影的遮蔽百分比，从而实现非常高效的模糊效果。

**核心原理**：
1. 在 Shadow Map 中不仅存储深度 $d$，还存储深度的平方 $d^2$。
2. 利用切比雪夫不等式（Chebyshev's Inequality）通过均值和方差直接推算出阴影比例。

尽管 VSM 的数学推导在某些情况下并不完全精确（被称为 hacks），但它在实践中效果极佳且性能很高，因此也成为了许多引擎中的常用选项。


#### Moving Wave of High Quality

近些年，渲染技术正经历一场剧烈的变革，其根本驱动力源于硬件和图形 API 的飞速发展。

##### Real-Time Ray Tracing

![](_imgs/Pasted%20image%2020260102215221.png)

实时光线追踪（Real-Time Ray Tracing，RTRT）被视为现代引擎渲染的“圣杯”，随着 NVIDIA RTX 硬件和 DXR (DirectX Raytracing) / Vulkan RT 接口的普及，渲染管线正在从传统的纯光栅化向混合渲染（Hybrid Rendering）和全光追（Path Tracing）演进。

**核心加速架构**：BVH 与硬件加速，实时光追的核心挑战在于如何快速在数百万个三角形中找到光线的交点。
1. **加速结构（Acceleration Structures）**：现代引擎使用两级结构：
    - **BLAS（Bottom-Level AS）**：存储具体的模型几何数据。
    - **TLAS（Top-Level AS）**：存储场景中物体的实例及其变换信息。
2. **硬件求交（RT Cores）**：硬件厂商（NVIDIA/AMD）将求交运算（Ray-Box，Ray-Triangle）固化到芯片中，性能比软件模拟提升了数十倍。

**当前主流应用**：**实时反射（Real-time Reflections）**，这是光追技术最直观、效果最显著的应用之一，已成为许多现代游戏的标配特性。

###### ReSTIR

ReSTIR（Reservoir-based Spatiotemporal Importance Resampling，时空重采样）是近五年 GI 领域最重要的学术突破，彻底解决了“多光源、多反弹”下采样效率低下的问题。

**核心原理**：传统的 GI 每一帧都在“瞎猜”光线该往哪射。ReSTIR GI 会记录周围像素和上一帧像素找到的“优质光路”（即光照贡献大的路径），并在当前像素进行**时空重采样**。

**优势**：能够以极低的每像素采样数（如 $1spp$）实现非常纯净、多反弹的间接光照，且支持海量动态光源。

###### Denoising & Reconstruction

由于实时渲染中每个像素的采样数（spp）极低，原始输出布满了噪点。降噪技术是 RTRT 能否落地的关键。

1. **时空滤波（Spatiotemporal Filtering）**：如 ASVGF，利用运动矢量（Motion Vectors）在时间维度累积光照能量。
2. **AI 降噪与重建**：
    - **NVIDIA DLSS 3.5（Ray Reconstruction）**：使用训练好的神经网络直接替代传统的人工降噪器，能够在补帧的同时，修复光追反射中的细节缺失。



##### Real-Time Global Illumination

![](_imgs/Pasted%20image%2020260102220324.png)

在游戏引擎渲染中，实时全局光照（Real-Time Global Illumination，RTGI）被认为是挑战渲染方程的最后堡垒。它的目标是在 $16.6ms$ 内模拟光线在场景中多次反弹后的效果（Indirect Lighting），包括漫反射反弹（Color Bleeding）和环境遮蔽（AO）。

###### Unreal Engine 5 Lumen

Lumen 是目前工业界影响力最大的 RTGI 方案，其核心在于多层次混合追踪（Multi-level Hybrid Tracing）。

**技术路径**：
1. **近处/精细**：使用硬件光线追踪（Hardware RT）。
2. **中距离**：使用 Mesh Distance Fields（网格距离场）进行软件追踪，绕过了复杂的 BVH 遍历。
3. **远景**：使用高度场（Heightfields）或预缩放纹理。

**表面缓存**（Surface Cache）：为了避免每帧重复计算复杂的着色，Lumen 将场景物体的光照信息缓存到特定的 Atlas 空间中，显著降低了光追开销。

###### Dynamic Diffuse GI

动态探针方案 DDGI（Dynamic Diffuse GI）是由 NVIDIA 推动的、基于 RTX 硬件加速的探针方案，是传统静态光照探针的进化版。

**工作流**：
1. 在场景中布置一个 3D 探针网格（Grid of Probes）。
2. 每个探针每帧利用光线追踪向四周发射少量射线，更新周围环境的亮度（Irradiance）和深度（Distance）。

**优势**：通过对探针数据的统计学过滤（防止漏光），实现了完全动态的间接光照，性能开销非常可预测，非常适合 3A 游戏的动态昼夜系统。

###### Neural Radiance Caching

神经辐射缓存（Neural Radiance Caching，NRC）是 AI 与图形学结合的前沿领域，目前 NVIDIA 等厂商正在积极探索。

**原理**：使用一个极其轻量级的神经网络在运行时实时训练。

**逻辑**：渲染器不需要追踪完整的光路，只需追踪一小段，然后“询问”神经网络：在这个位置、这个方向上的剩余光照是多少？神经网络通过实时学习场景的光照分布来回答。

**前景**：它可以大幅减少路径追踪的递归深度，是解决“无限反弹”GI 的终极思路。


#### Shader Management

现代游戏引擎面临的一个巨大工程挑战是**管理数量庞大的着色器**，随着渲染功能和材质复杂度的提升，着色器的组合数量会呈指数级增长，形成所谓的“着色器爆炸”（Shader Explosion），必须采用系统化的方法进行管理。

##### Uber Shader and Variants

编写一个功能全面、包含所有可能性的“超级着色器”模板，在这个模板内部，使用宏定义（Preprocessor Macros）（如 `#ifdef`，`#if`）来包裹不同的功能代码块。

编译器会根据不同的宏定义组合，从这个 Ubershader 模板中编译生成出成千上万个具体、优化后的着色器版本。这些版本被称为着色器变体（Shader Variants）或排列组合（Permutations）。

##### Cross Platform Shader Compile

![](_imgs/Pasted%20image%2020260102222726.png)

不同图形 API 和平台使用不同的着色器语言，现代解决方案 SPIR-V（Standard Portable Intermediate Representation）是使用一个标准的、可移植的中间语言（Intermediate Representation，IR）作为桥梁，实现“一次编写，到处运行”的着色器管线。




### Special Rendering

本节探讨如何在虚拟世界中，重现我们所处的美丽、复杂的自然世界。
#### Terrain

地形（Terrain）在图形学和游戏引擎中，特指地表、山脉、峡谷等自然地貌的几何表示，是构成虚拟世界的基础骨架，其表现力直接决定了世界的真实感和沉浸感。

地形渲染（Terrain Rendering）被视为一个典型的“海量数据处理”问题，地形的挑战在于其巨大的空间尺度与极高的细节要求之间的矛盾。

##### Heightmap-based

高度场（Height Field）是实现地形渲染最经典、最基础，且至今仍在广泛使用的方法。简单来说，它是一种用 2D 数据来表示 3D 表面的数学模型，其本质上是一张灰度图，其中每个像素的亮度值（或颜色通道值）直接对应于地形上某个 $(x, y)$ 坐标点的高 $z$ 坐标（注意：世界坐标和图形学的坐标体系的 $x$，$y$，$z$ 不一样），

$$
z = f(x, y)
$$

在数据存储上，表现为：
1. **2D 数组**：每一个元素存储一个浮点数高度值。
2. **高程图（Height Map）**：一张灰度图，像素的亮度（0~255 或更高精度的 16-bit）直接映射为地形的物理高度。

**渲染流程**：将一张高程图（Height Map）转换为 3D 模型的过程非常直观：
1. **创建基础网格（Grid）**：在 $xy$ 平面上创建一个均匀分布的、由大量顶点组成的平面网格，网格的精细度决定了地形的细节程度。
2. **顶点位移（Vertex Displacement）**：遍历网格上的每一个顶点，将其 $(x, y)$ 坐标映射到高程图的对应像素位置。
3. **采样高度**：读取该像素的亮度值，并将其作为该顶点的 $z$ 坐标。
4. **应用材质与光照**：在生成的地形模型上应用纹理、材质（如草地、岩石）和光照，最终渲染出逼真的效果。

**优势**：
1. **存储极度紧凑**：相比于存储海量的三角形（每个顶点需要 $x, y, z$ 坐标和索引），高度场只需要存储一个 $z$ 值。$x$ 和 $y$ 坐标可以根据像素在网格中的位置隐式计算出来。这使得引擎可以处理数十公里的巨大地图。
2. **碰撞检测极快**：在物理引擎中，判断一个物体是否撞到了地面，只需要根据物体的 $(x, y)$ 坐标去高度场里查一下高度 $z$。这比复杂的“射线-三角形”碰撞检测快出好几个数量级。
3. **易于实现 LOD**（Level of Detail）：因为高度场是规则的网格结构，它非常适合使用四叉树或 CDLOD 算法进行空间切分，从而实现远处精简、近处细腻的渲染方案。

**致命局限**：
1. **可扩展性差**：对于一个固定大小的均匀网格，如果要表达的世界范围非常大（例如，几千平方公里的开放世界），或者要求地形精度非常高，那么所需的顶点和三角形数量将会是天文数字。
2. **单值性**：因为一个 $(x, y)$ 坐标只能对应一个高度，所以你无法用高度场做出“山洞”或者“底下能走人的桥”，更无法做出垂直甚至向内凹进的悬崖。


**性能瓶颈**：游戏场景的尺度可能达到几十甚至上百公里，如果以每厘米的精度去渲染整个地形，三角形的数量将达到数千亿个，远远超出了任何 GPU 的处理能力。


**解决方案**：观察现实世界，我们发现人眼对近处细节敏感，对远处细节不敏感。这为性能优化提供了思路，即引入 LOD（Level of Detail）机制。

地形 LOD 是游戏引擎中处理超大规模地形的核心技术，但地形 LOD 有其特殊性，不能像独立物体那样简单地切换模型，因为地形是一个连续的表面，如果不同 LOD 级别的地块之间处理不当，就会在接缝处产生裂缝或孔洞。

> [!tip] T形接缝（T-Junction）
> T-Junction（T形接缝）这是指一个高精度地块的边上有顶点，而与之相邻的低精度地块的对应边上没有这个顶点，形成一个 “T” 字形的断裂，导致视觉上的裂缝。

> [!tip] 裂缝（Cracks）
> 相邻两个块的深度（LOD 等级）可能不同，边缘的三角形无法对齐，因此产生裂缝（Cracks）。

> [!tip] 跳变（Popping）
> 当相机移动时，地形块会突然从低精度切换到高精度，玩家会看到山头突然“蹦”出来一下。


##### LOD Metric

在游戏引擎中，地形网格的细分程度（Tessellation Level）并不是固定的，而是由一套评价函数（LOD Metric）动态计算得出的。

1. **距离因数**（Distance-based Factor）：相机离地形越近，细分程度越高。
	- **逻辑**：引擎会将地形划分为多个块（Patches）。每一帧，CPU 或 GPU 会计算相机到每个块中心（或边缘）的欧几里得距离。
	- **实现**：在 CDLOD 算法中，通常会定义一系列半径范围（LOD Ranges）。如果地形块落在 $R_0$ 范围内，则使用最高细分；在 $R_1$ 范围内则减半，以此类推。
2. **屏幕空间误差**（Screen Space Error）：这是目前 3A 引擎（如《地平线》、《荒野大镖客》）最常用的标准。
	- **逻辑**：引擎会预先计算：**“如果我把这个地形块简化，它在屏幕上造成的像素位移是多少？”**
	- **计算**：如果简化导致的视觉差异（几何体边缘的跳变）在屏幕上小于 1 个像素（或者预设的阈值 $\tau$），那么引擎就认为可以降低细分程度。
	- **优点**：它考虑了相机的焦距（FOV）。当玩家开镜（Zoom in）时，虽然距离没变，但屏幕空间误差变大，地形会自动变得更精细。
3. **地形曲率/复杂度**（Curvature / Ruggedness）：并非所有的地形都需要同样密度的三角形。
	- **逻辑**：
		* **平原/沙漠**：即使离得很近，因为表面很平坦，用两个大三角形就能表现，不需要过度细分。
	    - **峭壁/乱石**：表面起伏剧烈，需要极高的细分程度才能还原高度图里的细节。
	- **实现**：引擎会根据高度图的拉普拉斯算子（计算二阶导数）或方差来判断该区域的复杂度。复杂度越高，分配的三角形越多。
4. **视锥体与遮挡**（Frustum & Occlusion）：严格来说，这决定了细分是否为“零”。
	- **视锥体剔除**：如果地形块不在相机的视野范围内，其细分程度直接降为最低或完全不渲染，从而节省开销。
	- **遮挡剔除**：如果一座大山完全挡住了后面的地形，被挡住的部分也会降低细分或停止渲染。
5. **性能预算与硬件限制**（Performance Budget）
	- **三角形预算（Triangle Budget）**：引擎通常会设置一个上限（例如：地形总面数不能超过 200 万个三角形）。如果当前场景太复杂，系统会整体调低所有块的细分级别。
	- **硬件细分因子（Tessellation Factor）**：如果使用 GPU 硬件细分（Hardware Tessellation），细分程度受限于显卡的特定寄存器值（通常最大为 64）。

##### Triangle-Based Subdivision

基于二叉树的三角形剖分（Binary Triangle Subdivision）是一个非常经典的技术方案，其代表性算法是著名的 ROAM（Real-Time Optimally Adapting Meshes）。

![](_imgs/Pasted%20image%2020260104164939.png)

**核心原理**：基于对等腰直角三角形（Isosceles Right Triangle）的递归剖分。
1. **初始状态**：将地形的一个正方形区域对角切开，分为两个基础三角形。
2. **二分过程**：
	- 找到三角形最长的那条边（斜边）。
	- 从对角顶点向斜边中点画一条线。 
	- 原始三角形（父节点）就分裂成了两个完全相同的子三角形（子节点）。
3. **递归**：这个过程可以无限递归下去，形成一个深度的**二叉树结构**。![](_imgs/Pasted%20image%2020260104170445.png)

**解决裂缝**：
1. **钻石结构**：共享同一条斜边的两个三角形被称为一个钻石（Diamond）。
2. 如果你要分裂一个三角形的斜边，而与之相邻的三角形（共用这条斜边的那个三角形）还没有分裂，那么你必须先强制分裂相邻的三角形。![](_imgs/Pasted%20image%2020260104170350.png)


**缺点**：
1. **资源格式不匹配**：游戏引擎中的资源，如高度图（Height Field）、纹理贴图（Texture）等，天然就是以矩形（或正方形）网格存储的。如果使用三角形来管理，会导致近一半的存储和内存空间被浪费，并且处理逻辑也变得异常复杂。
2. **数据管理不直观**：其数据结构如同“七巧板”，由各种不规则的三角形拼凑而成。这对于地形数据的存储、流式加载和编辑都带来了巨大的复杂性。

##### QuadTree-Based Subdivision

**核心原理**：基于正方形网格对地形递归地切分为四个子块。从根节点（整个地形）开始，如果某个块距离相机足够近，就将其分裂为四个子块。这个过程一直持续到满足精度要求或达到叶子节点。

**优势**：
1. **直观性**：“切豆腐块”的方式非常符合人类对地理区域划分的直觉。
2. **数据对齐**：正方形的地块与纹理、高度图等资源格式完美对齐，数据管理和访问效率极高。
3. **资源管理**：以 Block 为单位进行资源打包和流式加载，逻辑清晰，易于实现。当摄像机进入某个 Block 的范围时，引擎便加载对应的数据包。
4. **可扩展性**：这种结构天然支持虚拟纹理（Virtual Texturing）等高级技术，因为它们都依赖于类似的基于网格的页面管理思想。

###### GeoMipmapping

GeoMipmapping 是地形 LOD 的经典方案，来源于纹理的 Mipmapping。

**核心思想**：将地形划分为多个块（Patches），每个四叉树节点预先生成并存储几组不同精度的索引缓冲（Index Buffers），渲染时，根据相机与该块的距离选择合适的精细度。

**裂缝处理**：通常使用裙边（Skirts）技术，即在每个块的四周向下延伸一圈三角形来遮挡裂缝。这样即使有缝隙，玩家也只能看到“墙”而看不到地底。


###### Continuous Distance-Dependent LOD（CDLOD）

Continuous Distance-Dependent LOD（CDLOD）是目前工业界最广泛采用的地形方案，解决了网格切换时的跳变（Popping）问题。

**核心思想**：
1. **共享网格**：引擎预先生成一个固定分辨率的方形网格（例如 $33 \times 33$ 或 $65 \times 65$ 顶点的网格）。
	- 所有的地形块在渲染时都使用这同一个网格模型。对于高 LOD（近处）的块，网格在世界空间缩放得较小；对于低 LOD（远处）的块，网格被拉伸得很大。
2. **节点选择**：每一帧在 CPU 端遍历四叉树，根据距离计算每个节点是否可见、处于哪一级 LOD。
3. **平滑过渡**（Morphing）： 为了解决“跳变”问题，CDLOD 在顶点着色器中使用一个 $f \in [0, 1]$ 的因子。当相机靠近时，顶点会从低精度位置平滑地“位移”到高精度位置。

**数学表达**：
$$
V_{final} = \text{lerp}(V_{low\_res}, V_{high\_res}, f)
$$

**优点**：内存占用极低（所有块共用一套网格），过渡极其丝滑。


###### Runtime Virtual Texturing

运行时虚拟纹理（Runtime Virtual Texturing，RVT）是将复杂的、多图层混合的材质结果，在运行时动态地绘制到一张巨大的、虚拟的缓存贴图（Atlas）中，渲染时只需进行一次简单的纹理采样。

> [!tip] Texture Splatting
> 在没有 RVT 之前，地形渲染主要依靠 Texture Splatting（纹理溅射）。
> 1. **性能瓶颈**：如果一个地形块混合了 8 种材质（草地、泥土、碎石、雪地等），GPU 在渲染该像素时必须进行 8 次以上的纹理采样（甚至更多，因为每种材质包含 Albedo，Normal，Roughness），并进行复杂的数学混合。
> 2. **采样风暴**：随着材质种类增加，Shader 指令数和带宽开销会呈线性爆炸。
> 3. **融合难题**：想要让场景中的石头、房屋与地形表面产生自然的“软接触”或“青苔覆盖”非常困难且昂贵。

**核心原理**：
1. **虚拟空间（Virtual Space）**：定义一张逻辑上极其巨大的贴图（例如 $128k \times 128k$），覆盖整个地形。
2. **物理缓存（Physical Cache）**：在显存中开辟一块实际的、有限大小的贴图空间，用来存放当前视角下可见的贴图块（Tiles/Pages）。
3. **页表（Page Table）**：记录虚拟坐标到物理缓存坐标的映射关系。
4. **按需渲染（Render on Demand）**：
    - 当相机移动到新区域时，GPU 发现某个 Tile 在缓存中不存在（Cache Miss）。
    - **关键点：** 引擎会在运行时，将这个 Tile 对应的所有复杂材质混合逻辑只计算一次，渲染进物理缓存中。
    - **最终绘制：** 渲染地形网格时，Shader 只需要根据页表查到缓存位置，进行 **1 次** 采样即可。

###### GPU Driven Quadtree

由 CPU 统筹全局，由 GPU 负责细节的遍历、评估和绘制决策。

**核心原理**：
1. **CPU**：仅将最顶层的四叉树结构或地形参数（如高度图、视点位置）打包成 Buffer 送往 GPU。
2. **GPU**（Compute Shader）：
	- **并行遍历**：使用 Compute Shader 并行处理四叉树的所有潜在节点。
	- **动态评估**：在 GPU 上计算每个块的 LOD、进行精细的视锥体剔除，甚至是基于 HZB (Hierarchical Z-Buffer) 的**遮挡剔除 (Occlusion Culling)**。
	- **生成指令**：将通过测试的节点数据写入一个 Append Buffer（即间接参数缓冲区）。
	- **间接绘制**（Indirect Drawing）：最终调用 `DrawIndexedIndirect`。

**挑战**：
1. **数据一致性**：确保 Compute Shader 生成的网格索引和顶点数据在逻辑上是连续的，不产生裂缝（通常结合 Skirts 或特定的 Morphing 逻辑）。
2. **复杂性**：需要非常深底层的 GPU 编程知识。例如，如何处理树的层级递归（GPU 本质上不支持真正的递归，通常用多级 Dispatch 或位运算模拟）。
3. **调试困难**：GPU 端的错误极难排查，通常需要使用 NSight 或 RenderDoc 等专业工具查看 Buffer 里的原始数值。



##### Solving T-Junctions
###### Stitching

Stitching（缝合技术）的目的就是将这些“参差不齐”的边缘强行连接在一起。

**核心原理**：不改变任何一边的网格拓扑结构，而是通过修改索引缓冲（Index Buffer）来重新定义边缘三角形的连接方式。

**算法流程**：
1. 假设 A 地块（高LOD）和 B 地块（低LOD）相邻。A 地块的共享边界上有 5 个顶点，而 B 地块只有 3 个。
2. 在生成 A 地块的索引缓冲（Index Buffer）时，将 A 地块边界上多出来的“中间点”（例如第 2 和第 4 个顶点）在顶点着色器（Vertex Shader）中“吸附”到 B 地块边界上对应的顶点位置。
3. 这样一来，原本用于连接这些中间点的三角形，其两个顶点会重合在一起，形成一个**面积为零**的三角形。

![](_imgs/Pasted%20image%2020260104171400.png)


###### Skirts

Skirts（裙边技术）是指在每个地形块（Patch）的边缘，额外增加一圈垂直向下延伸的几何面片。

**核心原理**：
1. **网格扩展**：在生成地形块的网格（Grid）时，除了正常的 $N \times N$ 的水平网格，在最外圈的边缘顶点处，额外生成一圈索引，连接到一组位置稍微下移（沿负 Y 轴偏移）的顶点。
2. **高度偏移**：这些裙边顶点的水平坐标 $(x, y)$ 与边缘顶点一致，但其高度值 $z$ 会比正常的高度图采样值更低。
3. **遮挡缝隙**：当相邻块产生裂缝时，由于每个块都有这圈向下的“墙”，这堵“墙”会挡住视线，使得玩家看到的是下垂的地形表面，而不是地底的真空。

**优点**：
1. **简单粗暴**：不需要知道邻居是谁，也不需要知道邻居的 LOD 等级。每个块管好自己的裙子就行。
2. **GPU 友好**：所有的地形块可以使用完全相同的索引缓冲区，不需要因为缝合而频繁切换数据，非常适合**实例化渲染（Instancing）**。
3. **性能恒定**：额外的几何开销在现代 GPU 面前几乎可以忽略不计。

**局限性**：
1. **重复绘制（Overdraw）**：裙边会产生少量的像素浪费。
2. **低视角穿帮**：如果玩家的相机极度贴近地面且面向缝隙，理论上能看到裙边产生的几何重叠，但在实际 3A 游戏中，配合地表的植被和贴图，这种情况极难被察觉。



|**特性**|**Stitching (缝合)**|**Skirts (裙边)**|
|---|---|---|
|**做法**|修改索引，强行连接边缘顶点。|在边缘向下挤出一段“围栏”遮挡缝隙。|
|**视觉效果**|**完美吻合**，完全不存在缝隙。|仍有重叠，但在俯视角下看不出来。|
|**复杂度**|较高（需维护多种 Index Buffer 组合）。|极低（只需在网格生成时多画一圈）。|
|**性能开销**|增加 Draw Call 切换或索引切换压力。|略微增加顶点和像素着色开销。|
|**物理一致性**|几何体是连续的，适合高精度物理。|几何体不连续，可能影响极精细的射线检测。|


##### Triangulated Irregular Network

Triangulated Irregular Network (不规则三角网，简称 TIN) 是与高度场（Height Field）并列的一种核心几何表示方法，其基于向量，通过一组不规则分布的顶点（称为特征点或离散点）以及连接这些顶点的三角形来表示 3D 表面。

对于包含大面积平坦区域的地形（如沙漠、平原），可以预先使用**网格简化**（Mesh Simplification）算法生成 TIN，用极少的三角形来表达平坦地表，同时保留关键的地貌特征。

> [!tip] 网格简化（Mesh Simplification）
> **流程**：从一个超高精度的原始地形网格开始，通过算法迭代地移除那些对整体形状贡献最小的顶点（通常位于平坦区域）。
> **目标**：在尽可能减少顶点和三角面数量的同时，最大程度地保留地形的宏观特征（如山脊、沟壑的轮廓线）。简化后的顶点会巧妙地分布在这些特征线上。

**关键算法**：Delaunay Triangulation（德劳内三角化），其遵循两个核心准则：
1. **空圆特性**：任何一个三角形的外接圆内部都不包含网格中的其他任何顶点。
2. **最大化最小角**：算法倾向于产生接近等边的三角形，尽量避免出现极度细长的“针状”三角形。




##### Content-Specific Static Mesh Simplification

与其在运行时动态计算 LOD，不如在开发阶段就通过算法，为一块巨大的地形生成一个“最优”的、LOD 已经固化了的**静态网格**，其目标是在大幅减少三角形数量的同时，最大限度地保留物体的视觉特征（如轮廓、关键细节和材质表现）。


**核心算法**：二次误差度量（Quadric Error Metrics，QEM）
1. **边坍缩**（Edge Collapse）：通过不断地将一条边的两个顶点合并为一个新顶点，来减少三角形数量。
2. **误差判定**：为了决定“坍缩哪条边损失最小”，QEM 为每个顶点维护一个二次方对称矩阵 $Q$。当顶点 $v$ 到周围平面的距离平方和最小时，认为误差最小：

$$Error(v) = v^T Q v$$

引擎会优先选择 $Error$ 值最小的边进行坍缩，从而在简化时完美保留平滑表面，并锁死尖锐边缘。


##### GPU-Driven Real-Time Tessellation

现代游戏引擎普遍采用在 GPU 上实时生成和细分几何体的方式来渲染地形，这提供了前所未有的细节和灵活性。

1. **传统细分管线**（Traditional DX11 Pipeline）：包含三个核心阶段：
	- **Hull Shader**：接收控制点，计算细分因子（Tess Level）。
	- **Tessellator**：硬件单元，根据因子全自动切分三角形。
	- **Domain Shader**：计算新生成的顶点在 3D 空间的位置，通常采样 Displacement Map（位移贴图）。![](_imgs/Pasted%20image%2020260104180027.png)![](_imgs/Pasted%20image%2020260104180043.png)
2. **GPU-Driven 管线**（DirectX 12 Mesh Shader Pipeline）：
	- **GPU Culling**：在 GPU 上进行簇级（Cluster）或三角形级的视锥体/遮挡剔除。
	- **Dynamic LOD**：根据屏幕空间误差（Screen Space Error）在 GPU 上直接生成一个包含细分参数的缓冲数据。
	- **Execute Indirect**：使用间接绘制指令，让 GPU 根据自己计算出的结果来决定画多少面。![](_imgs/Pasted%20image%2020260104180417.png)

**优势**：
1. **高度统一与灵活**：一个 Shader 干了过去多个 Shader 的活，逻辑更内聚。
2. **编程模型更简洁**：摆脱了 DX11 管线令人困惑的多阶段交互。
3. **性能潜力巨大**：允许开发者实现更高效的剔除和几何生成算法。


> [!tip] 技术选型与硬件普及
> 作为引擎开发者，选择使用哪种技术不仅是技术问题，更是现实问题。
> 1. **API与硬件依赖**： 
> 	- Mesh Shader 是 DirectX 12 的核心特性
> 	- DirectX 12 需要 Windows 10 或更高版本的操作系统支持。
> 2. **开发者的困境**:  
> 	- 尽管 Mesh Shader 在技术上远比 DX11 的细分管线优秀，但如果仍有大量玩家使用不支持 DX12 的旧系统（如Windows 7），引擎就必须向下兼容，维护两套甚至多套渲染路径
> 	- 因此，引擎团队会密切关注 Steam 硬件调查（Steam Hardware Survey）等数据，以判断新技术的普及率，决定何时可以放弃对旧 API 的支持，全面拥抱下一代技术。
> 
> **结论**: 引擎开发是一个在前沿技术、硬件普及、玩家基数之间不断权衡和演进的过程。地形渲染技术的变迁，完美地体现了图形API、硬件厂商和游戏开发者三者之间紧密协作、共同推动行业进步的关系。




###### GPU-Driven Dynamic Terrain Deformation

###### Beyond Height Fields



##### Procedural Content Generation





#### Sky




#### Atmosphere




#### Ambient Occlusion




#### Anti-Aliasing


### Post-Processing



#### Bloom


#### Tone Mapping





### Rendering Pipeline


![](_imgs/Pasted%20image%2020251024173001.png)


#### Forward Rendering





#### Deferred Rendering




### Render Graph


