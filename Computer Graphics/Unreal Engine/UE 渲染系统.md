在 Unreal Engine 中，可以从 3 个线程去理解渲染：**Game Thread**、**Render Thread**、**RHI Thread**。

**Game Thread** 是整个游戏系统的心脏，可以控制渲染系统的执行，
```cpp
// 源码位置：RenderingThread.cpp 第235行
/** The RHI thread runnable object. */
class FRHIThread : private FRunnable
{
    virtual uint32 Run() override
    {
        // RHI Thread的主循环
        FTaskGraphInterface::Get().AttachToThread(ENamedThreads::RHIThread);
        FTaskGraphInterface::Get().ProcessThreadUntilRequestReturn(ENamedThreads::RHIThread);
        return 0;
    }
};

// 源码位置：RenderingThread.cpp 第290行
/** The rendering thread main loop */
void RenderingThreadMain(FEvent* TaskGraphBoundSyncEvent)
{
    // Render Thread的主循环
    FTaskGraphInterface::Get().AttachToThread(RenderThread);
    FTaskGraphInterface::Get().ProcessThreadUntilRequestReturn(RenderThread);
}
```

在一个 GameLoop 开始的时候，先会读取玩家的输入和输出，更新 `FScene` 中的世界位置，`SceneRendering` 是负责一整帧各阶段串起来的核心控制层，[`Renderer`](UE%20Renderer.md) 会按照 `SceneRendering` 的要求执行，其可以分为两类：`DeferredShadingRenderer` 和 `MobileSceneRenderer`，前者是默认的延迟 `RenderingPath`，后者是备选的移动端的 `ForwardRenderingPath`。

在 Render Thread 内，RDG 主要执行 `RenderGraphBuilder` 里面的任务，Renderer 把 Build 好的 RDG 转化为 RHI 命令。
 



## Rendering Pipeline

![](_imgs/Pasted%20image%2020251110175347.png)![](_imgs/Pasted%20image%2020251110175354.png)![](_imgs/Pasted%20image%2020251110175408.png)




















## 参考

1. [虚幻引擎渲染系统入门笔记 - 知乎](https://zhuanlan.zhihu.com/p/1970825479113114681)

