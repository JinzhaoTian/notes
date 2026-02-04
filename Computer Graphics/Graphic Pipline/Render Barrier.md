Render Barrier（渲染屏障）是计算机图形学中的一个概念，通常指在 GPU 渲染管线中，由于某些操作需要等待之前的渲染任务完成才能继续，从而导致 GPU 流水线暂停或性能下降的情况。


## 主要场景

Render Barrier 常见于以下几种情况：

1. **资源依赖**：例如后一个渲染操作需要读取前一个操作写入的纹理（如后期处理、屏幕空间反射等），这时 GPU 必须等待前面所有片段着色器完成写入，才能开始下一个阶段，形成**纹理读取屏障**。
2. **渲染目标切换**：当 GPU 需要将渲染结果从一个 Render Target 切换到另一个 Render Target，并且后续操作依赖于前一个 Render Target 的内容时，会产生屏障。
3. **CPU-GPU 同步**：如果 CPU 需要读取 GPU 刚刚渲染的内容（例如截图、 occlusion query 结果），必须插入一个 GPU Finish 屏障，导致 CPU 等待 GPU 所有任务完成。
4. **深度/模板测试依赖**：例如延迟渲染中，先渲染 GBuffer，再渲染光照，中间可能需要一个屏障确保 GBuffer 完全就绪。


## 具体实现

在现代图形 API（如 Vulkan、DirectX 12）中，Render Barrier 被显式控制：
1. **Vulkan** ：
	- `VkPipelineStageFlag` 和 `VkImageMemoryBarrier` / `VkBufferMemoryBarrier`。
2. **DirectX 12**：
	- `D3D12_RESOURCE_BARRIER` ：开发者需要手动设置资源状态转换（如 `COLOR_ATTACHMENT` → `PIXEL_SHADER_RESOURCE`），以告知驱动何时需要屏障。