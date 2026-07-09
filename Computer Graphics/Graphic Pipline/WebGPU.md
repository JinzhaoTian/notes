WebGPU 是一种现代的 Web 图形与计算 API，旨在为网页应用（如游戏、数据可视化、机器学习推理等）提供高性能、低开销且安全地访问 GPU（图形处理器）的能力。

可以把它理解为 WebGL 的下一代继任者。

## 核心特点

1. **性能接近原生**
    - 它直接映射了现代图形 API（如 Vulkan、Direct3D 12、Metal）的设计思路，减少了 CPU 与 GPU 之间的瓶颈和开销，能充分发挥 GPU 的并行计算能力。
    - 相比 WebGL，在复杂 3D 场景、大量粒子、后期特效等场景下性能提升显著。
2. **通用计算（GPGPU）**
    - WebGL 主要用于图形渲染，计算能力很弱。
    - WebGPU 原生支持通用计算着色器，允许你在 GPU 上运行通用计算任务（如矩阵运算、物理模拟、图像处理、轻量级机器学习推理），而无需绕路图形渲染管线。
3. **更现代的功能**
    - 支持 Compute Shader（计算着色器）
    - 支持 Pipeline 对象（图形管线、计算管线）
    - 支持多线程（通过 `Worker` 共享 GPU 资源）
    - 显式控制资源布局和同步，减少隐式开销
4. **跨平台与安全性**
    - 由 W3C 标准化，运行在现代浏览器（Chrome、Edge、Firefox、Safari）中。
    - 经过沙箱设计，不能直接访问 GPU 硬件底层，避免恶意代码破坏系统。

## 使用

开发基于 WebGPU 的程序，核心路径主要分为 Web 端（JavaScript）和原生端（如 C++、Rust ）。

### Web 端

这里我们聚焦于更常见的 Web 端开发，带你从环境搭建到绘制第一个图形，梳理出完整的开发脉络。

#### 环境准备

开发 WebGPU 程序，首先需要一个支持它的浏览器和基本的开发工具。

- **浏览器**：推荐使用 **Chrome 113+**、**Edge 113+** 或 **Firefox 121+** 的最新版本[](https://developer.baidu.com/article/detail.html?id=5129697)。你可以在 `chrome://version` 检查Chrome版本。
    - **特别提醒**：如果使用旧版Chrome（如113），可能需要手动开启实验性功能。在地址栏输入 `chrome://flags`，搜索并启用 `#enable-unsafe-webgpu` 后重启浏览器。
- **代码编辑器**：推荐 **VS Code**，可以安装 `webgpu-language-features` 等插件来获得语法高亮支持。
- **本地服务器**：WebGPU API需要在安全上下文（如 `localhost` 或 `https`）中运行，**不能**直接用浏览器打开HTML文件。你可以使用 `http-server`、`live-server` 或 `Vite` 等工具在本地启动一个静态服务器。
- **Node.js环境**：如果你打算使用构建工具（如Vite、Webpack），需要安装Node.js。

#### 核心开发步骤

开发一个典型的 WebGPU 应用，通常遵循以下七个步骤。

1. **检测浏览器兼容性**

在JavaScript代码中，通过检查 `navigator.gpu` 对象来判断浏览器是否支持WebGPU。
```javascript
if (!navigator.gpu) {
    alert('你的浏览器不支持WebGPU，请使用最新版Chrome、Edge或Firefox。');
}
```

2. **获取 GPU 适配器（Adapter）与设备（Device）**
	- **适配器 (Adapter)**：代表物理GPU（如独立显卡或集成显卡）。
	- **设备 (Device)**：是开发者与GPU交互的主要入口，几乎所有资源的创建（如缓冲区、纹理、管线）都通过它来完成。
```javascript
// 请求适配器，可以指定性能偏好
const adapter = await navigator.gpu.requestAdapter({
    powerPreference: 'high-performance' // 优先使用独立显卡
});
if (!adapter) { throw new Error('找不到GPU适配器'); }
// 通过适配器请求设备
const device = await adapter.requestDevice();
```

3. **配置Canvas上下文**

获取HTML中 `<canvas>` 元素的WebGPU上下文，并配置其格式。
```javascript
const canvas = document.getElementById('myCanvas');
const context = canvas.getContext('webgpu');
// 获取浏览器对Canvas纹理的最佳格式（如 'bgra8unorm' 或 'rgba8unorm'）
const presentationFormat = navigator.gpu.getPreferredCanvasFormat();
// 配置Canvas上下文
context.configure({
    device: device,
    format: presentationFormat,
    alphaMode: 'premultiplied' // 可选，透明通道处理方式
});
```

4. **创建着色器模块 (Shader Module)**

WebGPU使用 **WGSL (WebGPU Shading Language)** 作为着色器语言[](https://developer.baidu.com/article/detail.html?id=7432620)。你需要将WGSL代码作为字符串传入`device.createShaderModule()`来创建着色器模块。
```javascript
const shaderCode = `
    @vertex
    fn vs_main(@builtin(vertex_index) in_vertex_index: u32) -> @builtin(position) vec4f {
        // 一个简单的三角形顶点数据
        var pos = array<vec2f, 3>(
            vec2f(0.0, 0.5),
            vec2f(-0.5, -0.5),
            vec2f(0.5, -0.5)
        );
        return vec4f(pos[in_vertex_index], 0.0, 1.0);
    }
    @fragment
    fn fs_main() -> @location(0) vec4f {
        return vec4f(1.0, 0.0, 0.0, 1.0); // 红色
    }
`;
const shaderModule = device.createShaderModule({ code: shaderCode });
```

5. **创建渲染管线 (Render Pipeline)**

渲染管线描述了顶点数据如何被处理、如何被光栅化。创建时需要指定着色器、图元类型等。
```javascript
const pipeline = device.createRenderPipeline({
    vertex: {
        module: shaderModule,
        entryPoint: 'vs_main', // 顶点着色器入口函数
    },
    fragment: {
        module: shaderModule,
        entryPoint: 'fs_main', // 片元着色器入口函数
        targets: [{ format: presentationFormat }] // 指定渲染目标格式
    },
    primitive: {
        topology: 'triangle-list' // 图元类型：三角形列表
    }
});
```

6. **编码与提交GPU命令**

所有发给GPU的指令都通过`GPUCommandEncoder`来编码。你需要创建一个编码器，用它开始一个渲染通道（Render Pass），执行绘制命令，最后将命令提交到GPU的命令队列（Command Queue）。

```javascript
// 1. 创建命令编码器
const commandEncoder = device.createCommandEncoder();
// 2. 获取当前帧的纹理视图 (从Canvas上下文)
const currentTexture = context.getCurrentTexture();
const view = currentTexture.createView();
// 3. 开始编码渲染通道
const renderPassDescriptor = {
    colorAttachments: [{
        view: view,
        clearValue: { r: 0.2, g: 0.3, b: 0.3, a: 1.0 }, // 背景色
        loadOp: 'clear',    // 在绘制前清空颜色
        storeOp: 'store',   // 绘制后存储结果
    }]
};
const passEncoder = commandEncoder.beginRenderPass(renderPassDescriptor);
// 4. 在渲染通道中设置管线并绘制
passEncoder.setPipeline(pipeline);
passEncoder.draw(3); // 绘制3个顶点，即一个三角形
// 5. 结束渲染通道
passEncoder.end();
// 6. 提交所有命令到GPU
device.queue.submit([commandEncoder.finish()]);
```

7. **实现渲染循环**

为了制作动画或持续渲染，通常会使用 `requestAnimationFrame` 来创建一个循环，在每一帧执行步骤6的编码和提交操作。

```javascript
function frame() {
    // ... (步骤6的所有命令编码和提交代码)
    requestAnimationFrame(frame);
}
requestAnimationFrame(frame);
```


### 原生端

WebGPU 并非只能在浏览器环境运行，它从一开始就被设计为一个跨平台的图形与计算 API。除了浏览器，它完全可以在桌面端、移动端等原生环境中运行。

#### 核心实现

1. **Dawn**：由 Google 主导开发，是 Chrome 浏览器内置的 WebGPU 实现。它同时也作为一个独立的 C++ 库，供开发者构建原生应用。
2. **[wgpu](wgpu.md)**：由 Mozilla 主导，用 Rust 语言实现。它被设计为“**原生优先**”，即在桌面端运行是它的首要目标，对 Web 的支持反而次之。

#### 开发步骤

1. **C++ 开发**：直接使用 Dawn 库提供的 `webgpu.h` C API 或 `webgpu_cpp.h` C++ 封装。Chrome 官方有文档指导如何用 C++ 编写一个同时运行在 Web 和桌面端的应用，并且有对应的跨平台示例项目可供参考。
2. **Rust 开发**：使用 `wgpu` 库，它能在 Windows、Linux、macOS 等桌面系统上，基于 Vulkan、Metal、DirectX 12 等现代图形 API 原生运行。`wgpu` 生态成熟，是 Rust 社区进行 GPU 开发的首选。
3. **Node.js 开发**：通过 NPM 包（如 `@kmamal/gpu` 或 `node-dawn`）在 Node.js 环境中使用 WebGPU。这使得 JavaScript 开发者无需浏览器也能进行 GPU 计算或渲染。例如，`@kmamal/gpu` 包就提供了与浏览器 `GPU` 对象类似的 API。