着色器（Shader）是一段运行在 GPU 中的程序，传统的 CPU 就像一个全能的大厨，能处理复杂的逻辑但一次只能做几件事；而 GPU 就像成千上万个只会切菜的帮厨，虽然逻辑简单，但能同时并行处理百万个像素，Shader 就是发给这些帮厨的指令清单。

Shader 由开发者编写，为开发者提供了很大的灵活度和可掌控度。

## 主要类型

![](../Graphic%20Pipline/_imgs/Pasted%20image%2020231214101837.png)

常见的 Shader 类型有：顶点着色器（Vertex Shader）、片元/像素着色器（Fragment / Pixel Shader）、计算着色器（Compute Shader）。

### 顶点着色器

顶点着色器主要用于**确定绘制图形的形状**，以及接收开发者传入的数据并传给后面阶段。接收外部传入的顶点数据，根据需要对顶点数据进行变换处理之后，再将顶点数据传入下一个阶段图元装配。另外顶点着色器也接收外部传进来的颜色值以及纹理采样器，然后再传递给下一个阶段进行图元装配处理。

每个顶点着色器只接收处理一个顶点坐标，有多少个顶点就会执行多少次。

### 片段着色器

片段着色器可通过编程来控制屏幕是上显示颜色，在这个阶段主要是**计算片段的颜色**。这里每个片段着色器接收一个片段数据的输入，所以有几个片段就会执行所少次，根据具体需要灵活设置该片段的颜色。

### 计算着色器

用于通用计算（如物理模拟、粒子效果），不直接参与光栅化图形管线。


## 编译

Shader 也是代码，需要从文本编译成 GPU 能懂的机器码，这个过程分为两个阶段：
1. **源代码语言**：
	- **GLSL**（`.vert`，`.frag`）： 主要用于 OpenGL 和 Vulkan。
	- **HLSL**（`.hlsl`）： 主要用于 DirectX，但现在也可用于 Vulkan。
	- **MSL**（`.metal`）：用于 Apple 的 Metal。
2. **编译流程**：由于显卡型号繁多（NVIDIA，AMD，Intel 架构各异），现代引擎通常采用离线编译（Offline Compilation）到一种中间语言（Intermediate Representation，IR），再由显卡驱动在运行时翻译成最终机器码。
	- **编写代码**：编写 HLSL 或 GLSL。
	- **编译为中间码（Bytecode）**：
	    - **Vulkan/OpenGL**：编译为 SPIR-V，这是行业标准的二进制格式。  
	    - **D3D 12**：编译为 DXIL（DirectX Intermediate Language）。
	- **驱动层编译**：程序运行时，显卡驱动读取 SPIR-V 或 DXIL，将其编译为该显卡架构（如 Ada Lovelace 或 RDNA3）专用的 ISA 指令集。

> [!tip] 最佳实践 
> 不要直接在游戏运行时解析文本源码（Online Compilation），这会导致游戏启动慢且容易出错。
> 
> 应在开发阶段使用工具（如 `glslangValidator` 或 `dxc`）将 Shader 预编译为二进制，现在也在游戏启动时编译 Shader。

### 具体处理

1. OpenGL：
```cpp
// 1. 创建Shader对象
GLuint shader = glCreateShader(GL_VERTEX_SHADER);

// 2. 加载源码并编译
glShaderSource(shader, 1, &sourceCode, NULL);
glCompileShader(shader);

// 3. 检查编译错误
glGetShaderiv(shader, GL_COMPILE_STATUS, &success);

// 4. 链接到Program
GLuint program = glCreateProgram();
glAttachShader(program, shader);
glLinkProgram(program);
```

2. Vulkan：
```cpp
// 1. 通常编译为SPIR-V离线（使用glslangValidator）
// 2. 创建Shader模块
VkShaderModuleCreateInfo createInfo{};
createInfo.codeSize = spirvCode.size() * sizeof(uint32_t);
createInfo.pCode = spirvCode.data();
vkCreateShaderModule(device, &createInfo, nullptr, &shaderModule);

// 3. 在管线创建时指定
VkPipelineShaderStageCreateInfo stageInfo{};
stageInfo.module = shaderModule;
stageInfo.pName = "main";
```

3. Direct3D 12：
```cpp
// 1. 通常使用FXC/DXC编译器编译HLSL
// 命令行：dxc -T vs_6_0 -E main shader.hlsl -Fo shader.bin

// 2. 在运行时加载
D3D12_SHADER_BYTECODE byteCode{};
byteCode.pShaderBytecode = compiledShaderData;
byteCode.BytecodeLength = shaderSize;

// 3. 在PSO中指定
D3D12_GRAPHICS_PIPELINE_STATE_DESC psoDesc{};
psoDesc.VS = byteCode;
```


## 使用

Shader 需要在运行时给它提供数据（输入）并告诉 GPU 如何执行。

1. **数据绑定（Binding Resources）**：Shader 需要两类数据：
    - **Attributes（属性）**：每个顶点独有的数据（如位置、UV 坐标、法线）。
    - **Uniforms / Constant Buffers（常量）**：全局数据（如当前时间、摄像机矩阵、光照颜色）。
    - **Textures / Samplers**：纹理图片和采样方式。
2. **管线状态（Pipeline State）**：Shader 只是渲染流程的一部分，需要配置混合模式（Blending）、深度测试（Depth Test）、剔除模式（Culling）等。
3. **Draw Call**：CPU 发送指令（如 `DrawIndexed`），GPU 启动流水线，调用 Vertex Shader 处理顶点，光栅化后调用 Pixel Shader 上色。

> [!tip] 资源绑定差异
> 1. OpenGL：全局状态机，bindless 扩展可用
> 2. Vulkan：明确的描述符集布局和绑定
> 3. D3D12：根签名和描述符表



## 管理

在大型游戏引擎（如 Unity，Unreal）中，Shader 管理是最复杂的模块之一。

1. **变体爆炸**（Shader Permutations / Variants）：
	- **挑战**：同一个 Shader 文件（例如 StandardShader），可能需要支持：
		- 有阴影 / 无阴影
		- 有法线贴图 / 无法线贴图
		- 高质量 / 低质量
	- 如果为每种情况写一个文件，你会有成千上万个 Shader。
	- **解决方案**：使用宏定义（Pre-processor Definitions），也就是 Uber Shader（超级着色器）概念。

```glsl
// 伪代码示例
void main() {
    vec3 color = baseColor;
    #ifdef USE_NORMAL_MAP
        color *= calculateNormalMapping();
    #endif
    #ifdef ENABLE_SHADOWS
        color *= calculateShadows();
    #endif
    outputColor = color;
}
```

引擎会在后台根据材质的设置，利用不同的 `#define` 组合，编译出成百上千个微小的二进制变体。

2. **材质系统**（Material System）：Shader 是代码（Class），材质是实例（Instance）。
	- Shader：定义了逻辑和需要的参数（"我需要一张贴图和一个颜色"）。
	- Material：提供了具体的数据（"贴图是 `brick.png`，颜色是红色"）。 管理系统负责将材质的数据准确地映射到 Shader 的 Uniform 槽位上。

3. 热重载（Hot-Reloading）：在开发工具中，开发者修改 Shader 代码后，引擎必须能在不重启程序的情况下，动态重新编译并替换 GPU 上的 Shader 程序，以便即时预览效果。


### 管理策略

```cpp
class ShaderManager {
private:
    // 1. 源码管理
    std::unordered_map<std::string, std::string> sourceCache;
    
    // 2. 二进制缓存
    std::unordered_map<size_t, ShaderBinary> binaryCache;
    
    // 3. 管线状态对象缓存
    std::unordered_map<PipelineKey, PipelineState> pipelineCache;
    
    // 4. 热重载系统
    void watchShaderFiles() {
        // 监控文件变化，重新编译
    }
    
public:
    // 变体管理（宏定义组合）
    ShaderVariant getVariant(const std::string& base, 
                            const std::vector<std::string>& defines);
};
```

**最佳实践**：
1. **离线编译为主**，减少运行时开销
2. **使用 SPIR-V 中间格式**（Vulkan/OpenGL）
3. **实现 Shader 热重载**，便于调试
4. **建立变体系统**处理不同渲染特性
5. **统一的 Shader 语言**（推荐 GLSL，可转译到各 API）


## 跨平台方案推荐

- **方案1**：**使用 GLSL -> 转译到各平台**
```cpp

#ifdef USE_VULKAN
    #define TARGET_SPIRV
#elif USE_D3D12
    #define TARGET_HLSL
#else
    #define TARGET_GLSL
#endif
```

- **方案2**：**使用中间工具链（如 Google 的 ShaderC ）**
```cpp
// GLSL -> SPIR-V -> 各平台后端
```

- **方案3**：**运行时选择**（`bgfx`、`The-Forge` 等引擎）



## Shading Language

着色语言是专门用于编写着色器的，常见的着色器语言有 DirectX 的 HLSL（High Level Shading Language）、OpenGL 的 GLSL（OpenGL Shading Language）以及 NVIDIA 的 CG（C for Graphics）。

| 特性       | OpenGL（传统 API）                                             | Vulkan & Direct3D 12（现代显式 API）                                                      |
| -------- | ---------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| **设计哲学** | 隐式、状态机，驱动程序做了大部分繁重的工作（内存管理、同步）。                            | 显式、底层，开发者完全控制内存、同步和执行顺序。                                                            |
| **编译方式** | 运行时编译，通常在运行时提交 GLSL 源码字符串，驱动负责编译。容易出现卡顿和兼容性问题。             | 离线编译，必须预编译为二进制中间码（SPIR-V 或 DXIL）。                                                   |
| **管线状态** | 动态状态机，可以随时单独修改某个状态（如 `glEnable(GL_DEPTH_TEST)`）。驱动在后台拼凑管线。 | 预编译 PSO（Pipeline State Object），管线状态是**不可变**的整体。如果想改一个混合模式，必须预先创建另一个完整的 PSO。         |
| **资源绑定** | 全局绑定点，简单的 Slot 绑定（`glBindTexture`）。                        | 描述符表（Descriptor Sets）+ 根签名（Root Signatures），非常复杂，需要定义资源布局、描述符堆（Descriptor Heaps）和表。 |
| **多线程**  | 极难多线程（依赖上下文 Context，通常单线程提交）。                              | 原生多线程，可以在多个 CPU 线程上录制命令缓冲（Command Buffers），最后统一提交。                                  |
| **内存管理** | 驱动管理                                                       | 显式控制                                                                                |
| **编译速度** | 慢（运行时）                                                     | 快（离线）                                                                               |
| **调试支持** | 优秀                                                         | 优秀                                                                                  |


### GLSL

GLSL（OpenGL Shading Language，也称作 GLslang），是一个以 C 语言为基础的高阶着色语言。它是由 OpenGL ARB 所建立，提供开发者对绘图管线更多的直接控制，而无需使用汇编语言或硬件规格语言。


Shader 内置函数：[Intrinsic Functions - Win32 apps | Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-intrinsic-functions) 

### HLSL

