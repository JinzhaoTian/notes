SPIR-V（Standard Portable Intermediate Representation）是 Vulkan 和 OpenGL 标准使用的中间语言，是一个与平台和厂商无关的二进制格式。


## SPIRV-Cross

> [!tip]
> 虽然 SPIR-V 是 Vulkan 的标准，但苹果的 Metal API 并不直接接受 SPIR-V，它只接受 MSL （Metal Shading Language）。

SPIRV-Cross 是一个专门用于解析 SPIR-V 并将其反编译成可读、高效的 GLSL、MSL 或 HLSL 的库和工具。它的目标不是生成机器码，而是生成接近手写的、易于阅读的高级语言代码。例如，如果你的目标是 iOS/macOS 设备，SPIRV-Cross 就会将 SPIR-V 转换为 MSL ；如果目标是 Android/Linux 等使用 OpenGL 的平台，它就会生成 GLSL。



