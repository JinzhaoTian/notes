OpenGL 是一个由 Khronos 组织制定并维护的**图形接口规范**（Graphics API Specification），定义了一套处理图形图像的统一规则，严格规定了每个函数该如何执行、应该做什么、如何工作，以及它们的输出值。

OpenGL 内部具体每个函数是如何实现的，将由 OpenGL 库的开发者自行决定，通常是显卡的生产商（如 NVIDIA、AMD、Intel），并位于你的显卡驱动中。当使用 Apple 系统时，OpenGL 库是由 Apple 自身维护的。在 Linux 系统中，有显卡生产商提供的 OpenGL 库，也有一些爱好者改编的版本。

> [!info] 编译时的不确定性
> 不同的操作系统（Windows, Linux, macOS）提供了不同版本的 OpenGL 头文件和库文件。例如，Windows 的 `opengl32.lib`（系统自带的）只支持到非常古老的 OpenGL 1.1 版本。
> 
> 这意味着，如果你的代码直接调用一个 OpenGL 1.2 以上的函数（比如 `glBindBuffer`），在 Windows 上编译时就会报错，因为编译器在 `opengl32.lib` 和头文件里根本找不到这个函数的声明。

> [!info] 运行时扩展机制
> 为了支持新特性，OpenGL 引入了扩展（Extension）机制，显卡厂商可以在新驱动中提供全新的函数和功能，而无需等待标准的更新。
> 
> 要使用这些新函数，你必须在程序运行时，向当前活动的 OpenGL 上下文（Context）查询并获取这些函数的内存地址。


## OpenGL "库加载器"

> [!tip] 引入原因
> 由于 OpenGL 驱动版本众多，它大多数函数的位置都无法在编译时确定下来，需要在运行时查询。所以任务就落在了开发者身上，**开发者需要在运行时获取函数地址并将其保存在一个函数指针中供以后使用**。取得地址的方法因平台而异，代码非常复杂，而且很繁琐，我们需要对每个可能使用的函数都要重复这个过程。

1. **GLEW（The OpenGL Extension Wrangler Library）**：
	- 老牌、经典的库，存在时间很长，非常稳定。
	- **工作方式**：在初始化时（`glewInit()`），它会**一次性查询并加载**当前 OpenGL 上下文支持的所有标准函数和扩展函数。

```cpp
#include <GL/glew.h>
// 通常需要先包含 GLFW 等窗口库的头文件

// ... 初始化窗口，创建 OpenGL 上下文 ...

GLenum err = glewInit();
if (err != GLEW_OK) {
    // 处理错误
    fprintf(stderr, "Error: %s\n", glewGetErrorString(err));
    exit(-1);
}

// 现在可以直接使用现代OpenGL函数了
glCreateShader(GL_VERTEX_SHADER);
```


2. **GLAD（Multi-Language Vulkan/GL/GLES/EGL/GLX/WGL Loader-Generator）**：
	- 更现代、更受欢迎的库，被认为是 GLEW 的继任者。
	- **核心特点**：它是一个在线生成器，访问 GLAD 官方网站，选择你需要的精确的 OpenGL 版本和特定的扩展，它会为你生成一个定制的库。
	- **优势**：
	    - **按需加载**：只加载你明确要求的功能，生成的代码更小、更高效。
	    - **避免冲突**：不会加载你不需要的函数，减少了因函数指针错误导致的潜在问题。
	    - **API最新**：对最新的 OpenGL 特性支持更好。
	    - **多语言支持**：也支持 Vulkan 等。
```cpp
#include <glad/glad.h>
// 注意：glad.h必须放在其它OpenGL相关头文件（如GLFW）之前

// ... 初始化窗口，创建OpenGL上下文 ...

if (!gladLoadGL()) {
    // 处理错误：加载失败
    exit(-1);
}

// 或者使用更通用的加载器，与具体窗口库解耦
// if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress)) { ... }

// 现在可以直接使用现代OpenGL函数了
glCreateShader(GL_VERTEX_SHADER);
```



## OpenGL "窗口/上下文管理库"

> [!tip] 引入原因
> OpenGL本身只是一个图形API，它不关心窗口、输入或操作系统。它只负责在给定的"画布"上绘制图形。但为了能看到绘制的效果，你需要：
> 1. **创建一个窗口**
> 2. **在这个窗口中创建一个 OpenGL 渲染上下文**
> 3. **处理用户输入**（键盘按键、鼠标移动等）
> 4. **处理窗口事件**（窗口大小改变、最小化、关闭等）
>
> 如果不用这些库，你就需要直接调用操作系统的原生 API ：
> 1. **在 Windows 上**：使用 Win32 API 创建窗口，然后用 `wglCreateContext` 等函数创建 OpenGL 上下文。
> 2. **在 Linux 上**：使用 X11 API 创建窗口，然后用 `glXCreateContext` 等函数创建 OpenGL 上下文。
> 3. **在 macOS 上**：使用 Cocoa API 创建窗口，然后用 `NSOpenGLContext` 创建 OpenGL 上下文。
>
> **直接使用原生 API 创建 OpenGL 窗口和上下文非常痛苦，且难以维护**。

1. **Freeglut（Free OpenGL Utility Toolkit）**：
	- 是古老的 GLUT 库的一个开源、维护更活跃的替代品。
	- **设计哲学**：
		- **简单、易用**，采用**事件驱动/回调函数**模型。
		- 你注册一些函数（如 `display`, `keyboard`, `reshape`），然后进入主循环，库会在相应事件发生时自动调用你的函数。
	- **特点**：
	    - API非常简洁，学习曲线平缓。
	    - 除了窗口和上下文管理，还提供了一些简单的UI功能，如弹出菜单。
	    - 主要面向教学、原型开发和简单的演示程序。

```cpp
#include <GL/freeglut.h>

void display() {
    // 你的渲染代码在这里
    glClear(GL_COLOR_BUFFER_BIT);
    // ... 绘制图形 ...
    glutSwapBuffers(); // 交换缓冲区，显示图像
}

void keyboard(unsigned char key, int x, int y) {
    // 键盘输入处理
    if (key == 27) // ESC键
        exit(0);
}

int main(int argc, char** argv) {
    // 1. 初始化GLUT
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGB); // 双缓冲模式
    glutInitWindowSize(800, 600);
    
    // 2. 创建窗口
    glutCreateWindow("My First FreeGLUT Program");
    
    // 3. 注册回调函数
    glutDisplayFunc(display);
    glutKeyboardFunc(keyboard);
    
    // 4. 进入主事件循环
    glutMainLoop();
    
    return 0;
}
```

2. **GLFW（Graphics Library Framework）**：
	- 一个更现代、更轻量、功能更强大的库，被设计用来替代 GLUT/Freeglut，是现代 OpenGL 开发（尤其是游戏和图形应用）的**事实标准**。
	- **设计哲学**：
		- **灵活、可控**。
		- 它不强制使用回调函数，而是提供了一个更过程化的 API。你可以在自己的主循环中主动查询输入、处理事件。
	- **特点**：
	    - 对多窗口、高 DPI 显示、游戏手柄输入等现代特性支持更好。
	    - 与 Vulkan 的集成非常好。
	    - 设计更符合现代应用程序的结构（比如游戏循环）。
	    - 社区活跃，更新频繁。

```cpp
#include <GLFW/glfw3.h>

int main() {
    // 1. 初始化GLFW
    if (!glfwInit()) {
        return -1;
    }
    
    // 2. 创建窗口和OpenGL上下文
    GLFWwindow* window = glfwCreateWindow(800, 600, "My First GLFW Program", NULL, NULL);
    if (!window) {
        glfwTerminate();
        return -1;
    }
    
    // 3. 将上下文设为当前
    glfwMakeContextCurrent(window);
    
    // 4. 初始化GLAD（加载OpenGL函数指针）
    // gladLoadGL();
    
    // 5. 渲染循环
    while (!glfwWindowShouldClose(window)) {
        // 渲染指令
        glClear(GL_COLOR_BUFFER_BIT);
        // ... 绘制图形 ...
        
        // 交换缓冲区
        glfwSwapBuffers(window);
        
        // 检查并处理事件（输入、窗口事件等）
        glfwPollEvents();
    }
    
    // 6. 清理资源
    glfwTerminate();
    return 0;
}
```


## 图形渲染管线

OpenGL 的代码逻辑大部分在渲染循环中，在 OpenGL 中，任何事物都在 3D 空间中，而屏幕和窗口却是 2D 像素数组，这导致 OpenGL 的大部分工作都是关于把 3D 坐标转变为适应你屏幕的 2D 像素。

3D 坐标转为 2D 坐标的处理过程是由 OpenGL 的**图形渲染管线**（Graphics Pipeline）管理的，图形渲染管线实际上指的是一堆原始图形数据途经一个输送管道，期间经过各种变化处理最终出现在屏幕的过程，其可以被划分为两个主要部分：
- 第一部分把你的 3D 坐标转换为 2D 坐标。
- 第二部分是把 2D 坐标转变为实际的有颜色的像素。


**在 OpenGL 中所有的数据都要放在显存中**，我们通过一定的手段去管理它，既要提供地方存放它，还要提供方法去正确地提取它们，去使用它们，OpenGL 通过 VAO，VBO，EBO 这些手段来解决这些问题。


图形渲染管线接受一组 3D 坐标，然后把它们转变为你屏幕上的有色 2D 像素输出。图形渲染管线可以被划分为几个阶段，每个阶段将会把前一个阶段的输出作为输入。所有这些阶段都是高度专门化的（它们都有一个特定的函数），并且很容易并行执行。正是由于它们具有并行执行的特性，当今大多数显卡都有成千上万的小处理核心，它们在 GPU 上为**每一个阶段**运行**各自的小程序**，从而在图形渲染管线中快速处理你的数据，这些**小程序叫做着色器（Shader）**。

![](imgs/Pasted%20image%2020230726161405.png)

图形渲染管线的每个阶段的抽象展示，要注意蓝色部分代表的是我们可以注入自定义的着色器的部分。





## 参考资料

1. [LearnOpenGL CN](https://learnopengl-cn.github.io/)
2. [OpenGL 4 Reference Pages (khronos.org)](https://registry.khronos.org/OpenGL-Refpages/gl4/)



