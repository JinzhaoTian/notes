Shader内置函数：[Intrinsic Functions - Win32 apps | Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-intrinsic-functions) 
### 相关概念

着色器（Shader）是一段运行在 GPU 中的程序，这段程序由开发者编写，所以说为开发者提供了很大的灵活度和可掌控度。

![](../Graphic%20Pipline/_imgs/Pasted%20image%2020231214101837.png)

现在 OpenGL 主要有三种着色器：顶点着色器、几何着色器、片段着色器，其中顶点着色器和片段·着色器为开发者必须提供，几何着色器为可选提供。

#### 顶点着色器

顶点着色器主要用于**确定绘制图形的形状**，以及接收开发者传入的数据并传给后面阶段。接收外部传入的顶点数据，根据需要对顶点数据进行变换处理之后，再将顶点数据传入下一个阶段图元装配。另外顶点着色器也接收外部传进来的颜色值以及纹理采样器，然后再传递给下一个阶段进行图元装配处理。
每个顶点着色器只接收处理一个顶点坐标，有多少个顶点就会执行多少次。

#### 几何着色器


#### 片段着色器

片段着色器可通过编程来控制屏幕是上显示颜色，在这个阶段主要是**计算片段的颜色**。这里每个片段着色器接收一个片段数据的输入，所以有几个片段就会执行所少次，根据具体需要灵活设置该片段的颜色。

## Shading Language

着色语言是专门用于编写着色器的，常见的着色器语言有 DirectX 的 HLSL（High Level Shading Language）、OpenGL 的 GLSL（OpenGL Shading Language）以及 NVIDIA 的 CG（C for Graphics）。



### GLSL

GLSL（OpenGL Shading Language），也称作 GLslang，是一个以C语言为基础的高阶着色语言。它是由 OpenGL ARB 所建立，提供开发者对绘图管线更多的直接控制，而无需使用汇编语言或硬件规格语言。


