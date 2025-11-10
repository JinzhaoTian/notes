Draw Call 是 CPU 调用图像编程接口，如 OpenGL 中的 glDrawElements 命令或者 DirectX 中的 DrawlndexedPrimitive 命令，以命令 GPU 进行渲染的操作。

在每次调用 Draw Call 之前， CPU 需要向 GPU 发送很多内容，包括数据、状态和命令等。在这一阶段， CPU 需要完成很多工作，例如检查渲染状态等。而一旦 CPU 完成了这些准备工作， GPU 就可以开始本次的渲染。GPU 的渲染能力是很强的，渲染 200 个还是 2000 个三角网格通常没有什么区别，因此渲染速度往往快于 CPU 提交命令的速度。如果 Draw Call 的数量太多， CPU 就会把大量时间花费在提交 Draw Call 上，造成 CPU 的过载。