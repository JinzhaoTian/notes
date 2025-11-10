GLFW（Graphics Library Framework）是一个专门针对 OpenGL 的 C 语言库，它提供了一些渲染物体所需的最低限度的接口。它允许用户创建 OpenGL 上下文，定义窗口参数以及处理用户输入。


## 使用


```c++
void glfwSetWindowUserPointer(GLFWwindow * window, void * pointer);
void glfwGetWindowUserPointer(GLFWwindow * window);
```
设置了与窗口 `window` 关联的用户数据指针，它的值通过 `pointer` 传入，**只能保存一个指针**。也可以传入一个结构体，这样就可以间接保存多个指针或数据。

### 窗口提示

窗口提示（Window Hint）用于在创建 GLFW 窗口之前设置窗口的属性。

```cpp
glfwWindowHint(GLFW_RESIZABLE, true);
```

常见的窗口提示如下：
1. **`GLFW_RESIZABLE`**：窗口是否可由用户调整大小
	- `GLFW_TRUE`：可调整大小（默认）
	- `GLFW_FALSE`：不可调整大小
2. **`GLFW_DECORATED`**：控制窗口是否有标题栏、边框等装饰
	- `GLFW_TRUE`：有装饰（默认）
	- `GLFW_FALSE`：无装饰（无边框窗口）
3. **`GLFW_FOCUSED`**：控制窗口创建后是否获得输入焦点
	- `GLFW_TRUE`：获得焦点（默认）
	- `GLFW_FALSE`：不获得焦点
4. **`GLFW_MAXIMIZED`**：控制窗口是否以最大化状态创建
	- `GLFW_TRUE`：最大化创建
	- `GLFW_FALSE`：正常大小创建（默认）
5. **`GLFW_FLOATING`**：控制窗口是否浮动（置顶）于其他窗口之上
	- `GLFW_TRUE`：窗口将始终位于顶部，不会被其他窗口遮挡
	- `GLFW_FALSE`：窗口遵循正常层级关系（默认）
6. **`GLFW_VISIBLE`**：控制窗口创建后是否立即可见
	- `GLFW_TRUE`：立即可见（默认）
	- `GLFW_FALSE`：隐藏窗口，需要调用 `glfwShowWindow` 来显示
7. **`GLFW_AUTO_ICONIFY`**：控制全屏窗口在失去焦点时是否自动最小化
	- `GLFW_TRUE`：自动最小化（默认）
	- `GLFW_FALSE`：不自动最小化
8. **`GLFW_SAMPLES`**：设置多重采样抗锯齿的采样数
	- `0`：禁用多重采样（默认）
	- `4`：通常使用的值，提供较好的抗锯齿效果
	- 更高值: 更好的质量但性能开销更大



### 窗口图标

GLFW 提供了 `glfwSetWindowIcon` 函数来设置窗口图标，需要使用`GLFWimage`结构体来加载图像数据。

```cpp
#define STB_IMAGE_IMPLEMENTATION
#include "stb_image.h"

// 创建窗口
GLFWwindow* window = glfwCreateWindow(1280, 720, "My GLFW Window", nullptr, nullptr);
if (window == nullptr) {
    // 处理窗口创建失败的情况
    return -1;
}

// 设置程序图标
GLFWimage icon;
// 使用stbi_load加载图片文件。最后一个参数"4"表示强制转换为RGBA四通道
icon.pixels = stbi_load("assets/icon.png", &icon.width, &icon.height, nullptr, 4);
if (icon.pixels != nullptr) {
    glfwSetWindowIcon(window, 1, &icon);
    stbi_image_free(icon.pixels); // 及时释放图像数据，避免内存泄漏
} else {
    // 可选：处理图片加载失败的情况（例如文件不存在）
    std::cerr << "Failed to load icon image." << std::endl;
}

// 继续后续的OpenGL上下文设置等...
glfwMakeContextCurrent(window);
```

多尺寸图标：
```cpp
// 准备多个尺寸的图标
const int iconCount = 2;
GLFWimage icons[iconCount];

const char* iconPaths[iconCount] = {"assets/icon_64x64.png", "assets/icon_16x16.png"};
for (int i = 0; i < iconCount; ++i) {
    icons[i].pixels = stbi_load(iconPaths[i], &icons[i].width, &icons[i].height, nullptr, 4);
    if (icons[i].pixels == nullptr) {
        // 处理某个图标加载失败
        std::cerr << "Failed to load icon: " << iconPaths[i] << std::endl;
    }
}

// 设置图标数组（即使某个加载失败，也会使用成功的部分）
glfwSetWindowIcon(window, iconCount, icons);

// 记得释放所有加载成功的图像数据
for (int i = 0; i < iconCount; ++i) {
    if (icons[i].pixels != nullptr) {
        stbi_image_free(icons[i].pixels);
    }
}
```





## 函数式调用

```C++
int main() {
	// 实例化GLFW窗口
	glfwInit();
	glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
	glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
	glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
	
	// 创建窗口
	GLFWwindow* window = glfwCreateWindow(SCR_WIDTH, SCR_HEIGHT, "OpenGL Test", NULL, NULL);
	if (window == NULL) {
		glfwTerminate();
		return -1;
	}
	glfwMakeContextCurrent(window);
	glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);
	
	/* load OpenGL... */
	
	// 渲染循环
	while (!glfwWindowShouldClose(window)) {
		processInput(window);
		
		/* rendering... */
		
		glfwSwapBuffers(window);
		glfwPollEvents();
	}
	
	glfwTerminate();
	return 0;
}

void processInput(GLFWwindow* window)
{
    if (glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
        glfwSetWindowShouldClose(window, true);
}

void framebuffer_size_callback(GLFWwindow* window, int width, int height) {
	// 每当窗口改变大小，GLFW会调用这个函数并填充相应的参数
	glViewport(0, 0, width, height);
}
```
