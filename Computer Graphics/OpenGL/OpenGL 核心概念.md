

## 渲染模式

- **立即渲染模式（Immediate Mode）**：早期的 OpenGL 使用立即渲染模式，即**固定渲染管线**。这种模式下绘制图形很方便，OpenGL 的大多数功能都被库隐藏起来，是一种配置化的管线，开发者很少有控制 OpenGL 如何进行计算的自由。而随着需求场景变的多样和复杂，开发者迫切希望能有更多的灵活性。
- **核心模式（Core-profile）**：在核心模式，OpenGL迫使我们使用现代的函数。现代函数的优势是更高的灵活性和效率，然而也更难于学习。

随着时间推移，规范越来越灵活，开发者对绘图细节有了更多的掌控，现代 OpenGL 转变为可编程（Programmable）渲染管线，而这里的编程语言就是 GLSL 语言，它是一种类 C 的语言，专为图形计算量身定制，包含了一些针对向量和矩阵操作的有用特性，我们用它编写我们自己的顶点着色器和片段着色器。

## 状态机

OpenGL 自身是一个巨大的状态机（State Machine）：一系列的变量描述 OpenGL 此刻应当如何运行。OpenGL 的状态通常被称为 OpenGL 上下文。

我们通常使用如下途径去更改 OpenGL 状态：设置选项，操作缓冲。最后，我们使用当前 OpenGL 上下文来渲染。假设当我们想告诉 OpenGL 去画线段而不是三角形的时候，我们通过改变一些上下文变量来改变 OpenGL 状态，从而告诉 OpenGL 如何去绘图。一旦我们改变了 OpenGL 的状态为线段绘制模式，下一个绘制命令就会画出线段而不是三角形。  
当使用 OpenGL 的时候，我们会遇到一些状态设置函数，这类函数将会改变上下文。以及状态使用函数，这类函数会根据当前 OpenGL 的状态执行一些操作。只要你记住 OpenGL 本质上是个大状态机，就能更容易理解它的大部分特性。

## 对象

OpenGL 的对象可以理解为 OpenGL 驱动内部管理的一种资源（Resource）的抽象。当你创建一个对象时，你实际上是在 GPU 端或驱动端申请了一块内存，并得到一个用于操作这块内存的句柄（Handle），也就是一个 ID。

### 核心特点

1. **句柄（Handle）**：每个对象都有一个唯一的、无符号整型的 ID（例如 `GLuint`），通过这个 ID 来引用和操作对象。
2. **状态（State）**：对象内部包含特定的数据或设置。例如，一个缓冲区对象存储着顶点数据，而一个纹理对象存储着图像数据和采样参数。
3. **绑定点（Binding Point/Target）**：这是 OpenGL 对象模型中最核心的概念。为了使用一个对象，你必须将它绑定到一个特定的**绑定点**上。这个绑定点就像一个全局的插槽，后续的 OpenGL 操作如果指向这个插槽，就会影响当前绑定到该插槽的对象。
    - 例如：`glBindBuffer(GL_ARRAY_BUFFER, vbo)` 将缓冲区对象 `vbo` 绑定到 `GL_ARRAY_BUFFER` 目标。之后调用 `glVertexAttribPointer` 就会从这个缓冲区中读取数据。
4. **生成和删除**：使用 `glGen*`（如 `glGenBuffers`）来生成对象 ID，使用 `glDelete*`（如 `glDeleteBuffers`）来销毁对象。


比如对一个对象的创建和使用，就可以使用如下这种函数接口的方式进行设置：
```c++
{
	// 创建对象 
	unsigned int objectId = 0;
	glGenObject(1, &objectId);
	// 绑定对象至上下文 
	glBindObject(GL_WINDOW_TARGET, objectId); 
	// 设置当前绑定到 GL_WINDOW_TARGET 的对象的一些选项 
	glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_WIDTH, 800); 
	glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_HEIGHT, 600); 
	
	// 将上下文对象设回默认 
	glBindObject(GL_WINDOW_TARGET, 0);

}
```

这是一个样例，有助于理解 OpenGL 的相关接口。使用对象的一个好处是在程序中，我们不止可以定义一个对象，并设置它们的选项，每个对象都可以是不同的设置。在我们执行一个使用OpenGL状态的操作的时候，只需要绑定含有需要的设置的对象即可。


### 对象类型

1. **缓冲区对象（Buffer Objects）**：用于在 GPU 上存储任意类型的二进制数据块。
	- **作用**：存储顶点数据、顶点索引、变换矩阵、着色器变量等。
	- **常见类型（通过绑定目标区分）**：
	    - **顶点缓冲区对象（VBO）**：绑定到 `GL_ARRAY_BUFFER`，用于存储顶点属性（位置、颜色、法线、纹理坐标等）。
	    - **索引缓冲区对象（EBO / IBO）**：绑定到 `GL_ELEMENT_ARRAY_BUFFER`，用于存储顶点的索引，实现顶点复用。
	    - **统一缓冲区对象（UBO）**：绑定到 `GL_UNIFORM_BUFFER`，用于在着色器之间高效地共享大块的只读数据（如变换矩阵、光源参数）。
	    - **着色器存储缓冲区对象（SSBO）**：绑定到 `GL_SHADER_STORAGE_BUFFER`，功能比UBO更强大，允许着色器对其进行读写操作，常用于通用计算（GPGPU）。

![](imgs/Pasted%20image%2020240817112644.png)

2. **顶点数组对象（Vertex Array Object，VAO）**：VAO 是 OpenGL 对象状态的记录器或容器。
	- **作用**：它记录了当 VBO 和 EBO 被绑定时，你设置的顶点属性指针（Vertex Attribute Pointer） 状态，简单来说，它记录了如何从 VBO 中解释出顶点数据。
	- 在渲染时，你只需要绑定对应的 VAO，所有关于顶点数据来源和格式的状态就都恢复了，无需重新设置 VBO 和属性指针，大大简化了渲染代码。

3. **纹理对象（Texture Objects）**：用于存储和采样图像数据，是渲染中视觉细节的来源。
	- **作用**：存储 1D、2D、3D 纹理、立方体贴图等，并提供采样参数（过滤方式、环绕模式等）。

4. **帧缓冲区对象（Framebuffer Object，FBO）**：FBO 是离屏渲染（Off-screen Rendering）的核心，可以把它想象成一个自定义的画布或画框。
	- **作用**：它本身不存储数据，而是作为一个**附件集合**，可以附着颜色、深度和模板缓冲区。**默认的帧缓冲区就是你的窗口**（由窗口系统提供，如 GLFW 创建的窗口）。
	- **附件类型**：
	    - **颜色附件**：通常附着纹理对象（ `GL_COLOR_ATTACHMENT0` 等），用于存储颜色信息。
	    - **深度附件**：附着纹理或渲染缓冲区对象，存储深度信息。
	    - **模板附件**：附着纹理或渲染缓冲区对象，存储模板信息。
	- **用途**：后期处理、阴影映射、反射、 picking 等所有需要先渲染到纹理而不是直接渲染到屏幕的场景。

5. **渲染缓冲区对象（Renderbuffer Object，RBO）**：一种优化过的、用于离屏渲染的图像缓冲区。
	- **作用**：专门用作 FBO 的附件，用于存储深度和模板数据。与纹理对象不同，渲染缓冲区的数据格式是 OpenGL 内部优化的，不能被着色器直接采样，因此在某些情况下（特别是只用做深度/模板测试时）效率更高。
	- **用途**：主要作为 FBO 的深度和模板附件。

6. **着色器程序对象（Shader Program Object）**：这是着色器管线（Shader Pipeline）的最终链接产物。
	- **作用**：将编译好的着色器对象链接成一个完整的、可在 GPU 上执行的程序。
	- **工作流**：
		1. 创建着色器对象（`glCreateShader`），附加源代码并编译。
		2. 创建程序对象（`glCreateProgram`）。
		3. 将编译好的着色器对象附加到程序对象上。
		4. 链接程序（`glLinkProgram`）。
		5. 使用 `glUseProgram` 来激活这个程序进行渲染。

7. **采样器对象（Sampler Object）**：将纹理的采样参数从纹理对象中分离出来。
	- **作用**：独立地封装过滤（Filtering）和环绕（Wrapping）等状态。这样，同一个纹理可以被不同的采样器对象以不同的方式采样，增加了灵活性。
	- **用法**：创建采样器对象，设置参数，然后在绑定纹理的同时绑定对应的采样器对象。

8. **查询对象（Query Object）**：用于异步获取 GPU 内部的信息。
	- **作用**：查询诸如遮挡查询（有多少片段通过了深度测试）、时间戳查询（GPU操作耗时）、变换反馈查询（捕获的图元数量）等信息。
	- **用法**：开始查询 -> 执行渲染操作 -> 结束查询 -> 在之后获取查询结果。

9. **变换反馈对象（Transform Feedback Object）**：用于捕获顶点着色器（或几何着色器）的输出。
	- **作用**：将处理后的顶点数据流直接捕获到缓冲区对象中，可以用于粒子系统、动画骨骼处理等。


### 对象间如何协同工作

```cpp
// 初始化阶段
GLuint vao, vbo, ebo, shaderProgram, texture;

glGenVertexArrays(1, &vao);
glGenBuffers(1, &vbo);
glGenBuffers(1, &ebo);

// 1. 设置VAO
glBindVertexArray(vao);

// 2. 设置VBO (绑定并上传数据)
glBindBuffer(GL_ARRAY_BUFFER, vbo);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

// 3. 设置EBO (绑定并上传数据)
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, ebo);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);

// 4. 告诉OpenGL如何解析顶点数据 (此状态被记录在VAO中)
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);

// 5. 创建着色器程序 (省略编译链接细节)
shaderProgram = createShaderProgram(...);

// 6. 创建纹理
glGenTextures(1, &texture);
glBindTexture(GL_TEXTURE_2D, texture);
// ... 设置纹理参数和图像数据

// 渲染循环
while (!glfwWindowShouldClose(window)) {
    // 7. 使用着色器程序
    glUseProgram(shaderProgram);
    
    // 8. 绑定纹理到纹理单元
    glActiveTexture(GL_TEXTURE0);
    glBindTexture(GL_TEXTURE_2D, texture);
    
    // 9. 绑定VAO (这自动绑定了VBO和EBO，并恢复了顶点属性状态)
    glBindVertexArray(vao);
    
    // 10. 绘制！
    glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
    
    glfwSwapBuffers(window);
    glfwPollEvents();
}
```

