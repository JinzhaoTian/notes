wgpu 是一个用 Rust 编写的、跨平台的、安全的图形 API，基于现代 WebGPU 标准，旨在为开发者提供对 GPU（图形处理器）强大能力的高效、安全且统一的访问方式。

wgpu 不是从零创造的完全新事物，而是对现代 GPU 编程接口（如 DirectX 12、Vulkan、Metal）的一次“统一封装”。其主要目标有两个：
1. **走向原生**：为 Rust 语言提供一个地道（idiomatic）、安全且高性能的图形编程接口。
2. **服务 Web**：作为 Firefox 等浏览器中 WebGPU 标准的底层核心实现，让网页也能调用 GPU 算力。

## 核心特性

1. **跨平台支持**：这是 wgpu 最强大的特性之一。无论你在哪个平台开发，都可以使用同一套 API。它能在多种后端（Backend）上原生运行：
    - **Windows**：通过 DirectX 12 或 Vulkan。
    - **macOS/iOS**：通过 Metal。
    - **Linux/Android**：通过 Vulkan。
    - **Web**：通过 WebGPU（在浏览器中）或 WebGL2（作为备选）。
2. **安全与高性能**：作为 Rust 生态的一部分，wgpu 充分利用了 Rust 语言的内存安全性和零成本抽象特性。它在提供对 GPU 底层精细控制的同时，也避免了常见的内存错误，比直接使用 C++ 编写的 Vulkan 或 OpenGL 更安全。
3. **多语言绑定**：虽然核心是 Rust 库，但通过 `wgpu-native` 项目，它也提供了 C 语言接口，这使得你可以用 Python、C++、Go 等多种语言来调用它。
