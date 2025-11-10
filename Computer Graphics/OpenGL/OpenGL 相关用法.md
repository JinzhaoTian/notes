
> 梳理一下 OpenGL 的使用流程（假设一些相关概念都已经有了解），总结一下简要教程。

OpenGL 的主要使用流程应该是下面几个部分：
- OpenGL 环境配置：涉及到窗口，链接到 OpenGL 库函数实现
- OpenGL 对模型数据的准备：涉及对模型顶点、法向量、纹理坐标等数据的上传与绑定
- OpenGL 发起绘制命令：DrawCall


## 环境配置

既然是计算机图形学，窗口必不可少（如果只是简单的渲染一张图片，那可以没有窗口 =.= ），一般使用的窗口库是 GLFW，
```c++
{
	glfwInit();
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

    this->hwnd = glfwCreateWindow(this->wndWidth, this->wndHeight, title.c_str(), NULL, NULL);
    glfwMakeContextCurrent(this->hwnd);
}
```

然后就是**加载 OpenGL 的实现库**，
```c++
{
	// glad: load all OpenGL function pointers
    // ---------------------------------------
    if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress)) {
        std::cout << "Failed to initialize GLAD" << std::endl;
    }
}
```

一般这一步之后，就可以使用 OpenGL 的相关库了，首先要做的是设置 viewport 大小，
```c++
{
	glViewport(0, 0, this->wndWidth, this->wndHeight);
}
```

OpenGL 的相关库一般都是以 `gl*` 开头。


## 数据准备

一般涉及到 [缓冲对象](OpenGL%20核心概念.md#缓冲对象) 这个概念，几乎所有使用 OpenGL 完成的事情都用到缓冲对象数据。
### OpenGL 3.3

#### 1. 初始化顶点数组对象

首先要创建一个顶点数组对象（Vertex Array Object，VAO），这个数据接下来是要传递给顶点着色器的，
```c++
{
	GLuint vao;
	glGenVertexArrays(1, &vao);
    glBindVertexArray(vao);

	// do something

	glBindVertexArray(0);
}
```

**解析**：
```c++
void glGenVertexArrays(GLsizei n, GLuint *arrays);
```
在 OpenGL 3.0 及以上版本可用，该方法返回 n 个未使用的对象名称到数组 arrays ，在创建对象之后，**需要手动绑定 VAO** 并调用其他函数来配置它。

```c++
void glBindVertexArray(GLuint array);
```
- 如果 array 非 0 
	- 如果 array 是合法的 VAO 对象，那么会激活这个 VAO，**后续所有的操作都会作用到这个被绑定的对象**。
	- 如果 array 是非法的 VAO 对象，那么会报错。
- 如果 array 为 0，那么 OpenGL 将不再使用之前绑定的顶点对象。


#### 2. 分配缓存对象

实际存储数据的是缓冲对象，顶点数组对象会生成并管理这些缓冲对象，比如首先对顶点数据进行缓冲，
```c++
{
	GLuint vbo;
	glGenBuffers(1, &vbo);
	glBindBuffer(GL_ARRAY_BUFFER, vbo);
	glBufferData(GL_ARRAY_BUFFER, size, data, GL_STATIC_DRAW);
}
```

**解析**：
```c++
void glGenBuffers(GLsizei n, GLuint *buffers);
```
在 OpenGL 1.5 及以上版本可用，该方法返回 n 个当前未使用的缓存对象名称，并保存到 buffers 数组中，仅生成缓冲区对象名称（ID），但不实际创建或分配缓冲区对象的存储空间。生成的缓冲区对象必须绑定到一个目标（如 `GL_ARRAY_BUFFER`、`GL_ELEMENT_ARRAY_BUFFER`）之后，才能进行进一步的操作（如分配存储空间、上传数据等）。

```c++
void glBindBuffer(GLenum target, GLuint buffer);
```
- 如果 buffer 非 0，且该缓冲对象合法，那么成为当前 target 中被激活的缓冲对象。
- 如果 buffer 为 0，那么 OpenGL 将不再对当前 target 使用任何缓存对象。

```c++
void glBufferData(GLenum target, GLsizeiptr size, const void* data, GLenum usage);
```
该方法用来分配存储空间并初始化数据，调用前**需要先绑定缓冲区对象**到一个特定的目标。并且允许重新分配缓冲区的存储空间，可以在缓冲区对象已经存在的情况下再次调用该方法来重新分配或更新数据。



#### 3. 配置顶点属性

现在，顶点对象声明好了，顶点的数据传入进去了，还需要告诉 OpenGL 服务端顶点数据是如何组织的，
```c++
{
	glEnableVertexAttribArray(0);
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex), (void *)offsetof(GraphicsInterfaceVertex, Position));
    
    glEnableVertexAttribArray(1);
    glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex), (void *)offsetof(GraphicsInterfaceVertex, Normal));

    glEnableVertexAttribArray(2);
    glVertexAttribPointer(2, 2, GL_FLOAT, GL_FALSE, sizeof(Vertex), (void *)offsetof(GraphicsInterfaceVertex, TexCoords));
}
```

**解析**：
```c++
void glEnableVertexAttribArray(GLuint index);
void glDisableVertexAttribArray(GLuint index);
```
该方法用来设置是否启用与 index 索引相关联的顶点数组，这个索引与顶点着色器中的顶点属性位置相关联，如，如果在着色器中定义了 `layout (location = 0) in vec3 position;`，那么 index 就应该为 0。

```c++
void glVertexAttribPointer(GLuint index, GLint size, GLenum type, GLboolean normalized, GLsizei stride, const void* pointer);
```
该方法用来设置 index 位置对应的数据值，size 表示每个顶点需要更新的分量数量，type 表示数组中每个元素的数据类型，normalized 表示是否需要归一化，stride 表示偏移步长，pointer 表示缓存对象中从起始位置开始计算的数组数据偏移值。


#### 4. 配置 uniform 变量

一般来说，配置完前三步，再搭配上一个简单的shader，基本就可以完成显示。但是为了更多的动态效果，一些 uniform 变量必不可少。









### OpenGL 4.5

#### 1. 初始化顶点数组对象

```c++
{
	GLuint vao;
	glCreateVertexArrays(1, &vao);

	// do something

	glBindVertexArray(0);
}
```

**解析**：
```c++
void glCreateVertexArrays(GLsizei n, GLuint *arrays);
```
在 OpenGL 4.5 及以上版本引入，它是对 glGenVertexArrays 的改进版，提供了一些更简洁和直接的功能。该方法返回 n 个未使用的对象名称到数组 arrays，**直接创建并初始化 VAO**，无需在创建后立即绑定即可进行某些操作。

#### 2. 分配缓存对象

```c++
{
	GLuint vbo;
	glCreateBuffers(1, &vbo);
	glNamedBufferData(vbo, size, data, GL_STATIC_DRAW);
}
```

**解析**：
```c++
void glCreateBuffers(GLsizei n, GLuint *buffers);
```
在 OpenGL 4.5 及以上版本可用，该方法返回 n 个当前未使用的缓存对象名称，并保存到 buffers 数组中。

```c++
void glNamedBufferData(GLuint buffer, GLsizeiptr size, const void* data, GLbitfield flags);
```
在 OpenGL 4.5 及以上版本可用，该方法用来分配存储空间并初始化数据，**无需绑定缓冲区对象**，直接通过缓冲区对象名称（ID）来操作。该方法分配的存储空间是不可重新分配的，一旦分配了存储空间，就**不能改变其大小**。要修改缓冲区内容，必须使用诸如 glBufferSubData 或映射缓冲区等操作。


#### 3. 配置顶点属性

```c++
{
	glEnableVertexAttribArray(0);
	glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)0);
}
```





## 绘制

> Draw Call 是指发起绘制命令的函数调用，用于将顶点数据发送到 GPU 进行渲染。

OpenGL 真正的开始绘制其实很简单，
```c++
{
	glBindVertexArray(vao);
    glDrawElements(GL_TRIANGLES, static_cast<U32>(indices.size()), GL_UNSIGNED_INT, 0);
    glBindVertexArray(0);
}
```

**解析**：

首先，针对不同的顶点对象，需要首先使用 glBindVertexArray 绑定到上下文，

```c++
void glDrawElements(GLenum mode, GLsizei count, GLenum type, const void* indices);
```
该方法用于**从索引缓冲区中绘制元素**，count 指定要绘制的元素的数量（索引数），indices 指向索引数组的指针，如果索引数组存储在缓冲区对象中，这个参数是该缓冲区内的字节偏移量。
- `mode` ：指定要绘制的图元类型（绘制方式），常见的图元类型包括：
	- `GL_POINTS` - 绘制点。
	- `GL_LINES` - 绘制线段。
	- `GL_LINE_STRIP` - 绘制线段条带。
	- `GL_TRIANGLES` - 绘制三角形。
	- `GL_TRIANGLE_STRIP` - 绘制三角形条带。
	- `GL_TRIANGLE_FAN` - 绘制三角形扇形。
- `type` ：指定索引数组中元素的类型，常见的类型包括：
	- `GL_UNSIGNED_BYTE` - 索引用 `unsigned char` 类型表示。
	- `GL_UNSIGNED_SHORT` - 索引用 `unsigned short` 类型表示。
	- `GL_UNSIGNED_INT` - 索引用 `unsigned int` 类型表示。


其他类似的 Draw Call 有，
```c++
void glDrawArrays(GLenum mode, GLint first, GLsizei count);
```
该方法从已启用的顶点属性数组中绘制图元，mode 指绘制的图元类型，first 指从顶点数组中开始读取的第一个顶点的索引，count 指要绘制的顶点数量。


```c++
void glDrawArraysInstanced(GLenum mode, GLint first, GLsizei count, GLsizei instancecount);
```
与 glDrawArrays 类似，但支持实例化绘制，可以多次重复绘制同一几何体。

```c++
void glDrawElementsInstanced(GLenum mode, GLsizei count, GLenum type, const void* indices, GLsizei instancecount);

```
与 glDrawElements 类似，但支持实例化绘制，结合索引和实例化，可以多次重复绘制同一几何体。

```c++
void glDrawArraysIndirect(GLenum mode, const void* indirect);
```
与 glDrawArrays 类似，但通过 indirect 指向的内存块指定绘制参数，可以在不直接调用 OpenGL 函数的情况下执行 Draw Call。

```c++
void glDrawElementsIndirect(GLenum mode, GLenum type, const void* indirect);
```
与 glDrawElements 类似，但通过 indirect 指向的内存块指定绘制参数，可以在不直接调用 OpenGL 函数的情况下执行 Draw Call。

```c++
void glMultiDrawArrays(GLenum mode, const GLint* first, const GLsizei* count, GLsizei drawcount);
```
执行多个 glDrawArrays 调用，可以一次性绘制多个不同的图元。

```c++
void glMultiDrawElements(GLenum mode, const GLsizei* count, GLenum type, const void* const* indices, GLsizei drawcount);
```
执行多个 glDrawElements 调用，可以一次性绘制多个不同的索引图元。
