引擎采用了分层架构设计，核心模块位于 Hazel 静态库中，Hazelnut 和 Sandbox 作为客户端应用链接该核心库。

## 核心循环（Application Loop）

`Application` 类是引擎的入口点，负责管理主循环：
1. **初始化**：窗口创建、渲染器初始化、ImGui 上下文设置。
```cpp
Application::Application(const std::string& name /* = Nut App */, ApplicationCommandLineArgs args /* = ApplicationCommandLineArgs()*/)
	:m_CommandLineArgs(args)
{
    HZ_PROFILE_FUNCTION();

	HZ_CORE_ASSERT(!s_Instance, "Application already exists!");
	s_Instance = this;

	m_Window = Window::Create(WindowProps(name));
	m_Window->SetEventCallback(HZ_BIND_EVENT_FN(Application::OnEvent));//如果子类重写了OnEvent函数，则以子类为准
	
	Renderer::Init();
	
	m_ImGuiLayer = new ImGuiLayer();
	PushOverlay(m_ImGuiLayer);
}
```

2. **运行循环**：计算时间步（Timestep），更新层栈（LayerStack），渲染 ImGui，交换缓冲区。
```cpp
void Application::Run()
{
	HZ_PROFILE_FUNCTION();
	while (m_Running)
	{
		HZ_PROFILE_SCOPE("RunLoop");
		float time = (float)glfwGetTime();
		Timestep timestep = time -  m_LastFrameTime;
		m_LastFrameTime = time;
		if (!m_Minimized)
		{
			{
				HZ_PROFILE_SCOPE("LayerStack    OnUpdate");
				for (Layer* layer : m_LayerStack)  // 更新图层
					layer->OnUpdate(timestep);     // 执行图层逻辑更新(更新应用程序的逻辑状态）
			}
			m_ImGuiLayer->Begin();
			{
				HZ_PROFILE_SCOPE("LayerStack    OnImGuiRender");
				for (Layer* layer :     m_LayerStack)
					layer->OnImGuiRender();        // 进行图层实际渲染操作（逻辑更新后才能进行的渲染操作）
			}
			m_ImGuiLayer->End();
		}
		m_Window->OnUpdate();                       // 更新窗口
	}
}
```

3. **事件处理**：接收并分发窗口、输入等事件。
```cpp
void Application::OnEvent(Event& e) //Application 中创建窗口时，将此函数设为回调函数
{
	HZ_PROFILE_FUNCTION();
	EventDispatcher dispatcher(e);
	dispatcher.Dispatch<WindowCloseEvent>   (HZ_BIND_EVENT_FN  (Application::OnWindowClose));
	dispatcher.Dispatch<WindowResizeEvent>  (HZ_BIND_EVENT_FN (Application::OnWindowResize));
	//HZ_CORE_TRACE("{0}", e.ToString());
	//图层的事件处理是反向的（从尾到头），！！！反向    迭代器中的 iter 需要使用前置递增（先加后用）
	for (auto it = m_LayerStack.rbegin(); it !=     m_LayerStack.rend(); ++it)	
	{
		if (e.Handled)
		{
			break;
		}
		(*it)->OnEvent(e);
	}
}
```

## 层系统（Layer System）

引擎使用 `LayerStack` 管理渲染层和逻辑层。

1. **Layer**：基础类，定义了 `OnAttach`，`OnDetach`，`OnUpdate`，`OnEvent`，`OnImGuiRender` 等生命周期函数。
```cpp
class Layer
{
public:
    Layer(const std::string& name = "Layer");
    virtual ~Layer();
    virtual void OnAttach() {}
    virtual void OnDetach() {}
    virtual void OnUpdate(Timestep ts) {}
    virtual void OnImGuiRender() {}
    virtual void OnEvent(Event& event) {}
    inline const std::string& GetName() const { return m_DebugName; }
protected:
    std::string m_DebugName;
};
```

2. **Overlay**：覆盖层，通常用于调试 UI 或最上层的渲染，总是最后更新和渲染。

3. **执行顺序**：普通 Layer 按插入顺序更新，Overlay 最后更新；事件反向传播（从 Overlay 到底层 Layer），一旦被处理（Handled）即停止传播。

## 核心子系统

### 事件系统（Event System）

位于 `Hazel/Events`，采用阻塞式事件分发机制。

1. **事件类型（`EventType`）**：包含 `WindowClose`，`WindowResize`，`KeyPressed`，`MouseButtonPressed`，`AppTick` 等。
2. **事件分类（`EventCategory`）**：使用位掩码标记事件属性（如 `EventCategoryInput`，`EventCategoryKeyboard`），便于快速过滤。
3. **分发器（`EventDispatcher`）**：
```cpp
void Application::OnEvent(Event& e)
{
    EventDispatcher dispatcher(e);
    // 如果事件类型是 WindowCloseEvent，则调用 OnWindowClose 函数
    dispatcher.Dispatch<WindowCloseEvent>(BIND_EVENT_FN(OnWindowClose));
    dispatcher.Dispatch<WindowResizeEvent>(BIND_EVENT_FN(OnWindowResize));

    // 反向遍历层栈，将事件传递给每一层
    for (auto it = m_LayerStack.end(); it != m_LayerStack.begin(); )
    {
        (*--it)->OnEvent(e);
        if (e.Handled) // 如果事件被处理，停止传播
            break;
    }
}
```


### 渲染系统（Render System）

位于 `Hazel/Renderer`，设计为 API 无关（API Agnostic），目前主要实现 OpenGL 后端。

1. **Renderer API**：一个抽象渲染命令类，定义了基础绘图命令（`SetClearColor`，`Clear`，`DrawIndexed`）。不同的渲染API需要实现自己的 Renderer API，目前 Hazel 引擎只实现了 OpenGLRendererApi 类。
```cpp
class RendererAPI
{
public:
	enum class API
	{
		None = 0, OpenGL = 1, DirectX = 2
	};

	enum class BlendFactor
	{
		Zero = 0,
		One,
		SrcColor,
		OneMinusSrcColor,
		SrcAlpha,
		OneMinusSrcAlpha,
		DstAlpha,
		OneMinusDstAlpha,
		DstColor,
		OneMinusDstColor,
		ConstantColor,
		OneMinusConstantColor,
		ConstantAlpha,
		OneMinusConstantAlpha
	};
public:
	virtual ~RendererAPI() = default;

	static Scope<RendererAPI> Create();

	virtual void Clear() = 0;
	virtual void SetClearColor(const glm::vec4&     color) = 0;

	virtual void Init() = 0;
	virtual void SetViewport(uint32_t x,    uint32_t y, uint32_t width, uint32_t   height) = 0;

	virtual void Flush() = 0;

	virtual void SetBlendFunci(uint32_t buf,    BlendFactor src, BlendFactor dst) = 0;
	virtual void SetColorMaski(uint32_t buf,    bool r, bool g, bool b, bool a) = 0;
	virtual void SetBlendFunc(BlendFactor src,  BlendFactor dst) = 0;
	virtual void SetDepthTest(bool enabled) = 0;
	virtual void SetDepthMask(bool enabled) = 0;

	virtual void BindTexture(uint32_t   rendererID, uint32_t slot) = 0;

	virtual void DrawIndexed(const  Ref<VertexArray>& vertexArray) = 0;
	virtual void DrawIndexed(const  Ref<VertexArray>& vertexArray, uint32_t  indexCount) = 0;

	virtual void MemoryBarrierTexFetch() = 0;

	inline static API GetAPI() { return s_API; }
	inline static API SetAPI(API api) { s_API =     api; }

private:
	static API s_API;
};
```


2. **Renderer2D**：
3. **批处理（Batching）**：Renderer2D 中定义了全局静态变量 `s_Data`，其中存储了渲染 2D 物体所需的一切数据和统计数据，并且在初始化时创建了 VAO、VBO、EBO，需要绘制时直接加入 VBO；自动合并多次绘制请求（DrawQuad）到单个顶点缓冲区（VertexBuffer），减少 Draw Call。
```cpp
struct Renderer2DData
{
    static const uint32_t MaxQuads = 1000;
    static const uint32_t MaxVertices = MaxQuads * 4;
    static const uint32_t MaxIndices = MaxQuads * 6;
    uint32_t QuadIndexCount = 0;
    static const uint32_t MaxTextureSlots = 32;

    Ref<VertexArray> QuadVA;
    Ref<VertexBuffer> QuadVB;
    Ref<IndexBuffer> QuadIB;
    Ref<Shader> QuadShader;
    Ref<Texture2D> WhiteTexture;

    QuadVertex* QuadVBBase = nullptr;// 顶点   指针起始
    QuadVertex* QuadVBHind = nullptr;// 顶点   指针末尾

    Ref<VertexArray> CircleVA;
    Ref<VertexBuffer> CircleVB;
    Ref<Shader> CircleShader;

    uint32_t CircleIndexCount = 0;
    CircleVertex* CircleVBBase = nullptr;
    CircleVertex* CircleVBHind = nullptr;

    std::array<Ref<Texture2D>, MaxTextureSlots> Textures;
    uint32_t TextureSlotIndex = 1;

    // 一个方形的基础顶点
    glm::vec4 QuadVertexPosition[4]{
	    { -0.5f, -0.5f, 0.0f, 1.0f },
	    {  0.5f, -0.5f, 0.0f, 1.0f },
	    {  0.5f,  0.5f, 0.0f, 1.0f },
	    { -0.5f,  0.5f, 0.0f, 1.0f }
    };

    glm::vec4 CircleLocalPosition[4]
    {
	    { -1.0f, -1.0f, 0.0f, 1.0f },
	    {  1.0f, -1.0f, 0.0f, 1.0f },
	    {  1.0f,  1.0f, 0.0f, 1.0f },
	    { -1.0f,  1.0f, 0.0f, 1.0f }
    };

    Renderer2D::Statistics Stats;
};
static Renderer2DData s_Data;
```

4. **图元支持**：支持带颜色/纹理的四边形（Quad）、旋转四边形、圆形（Circle）。结合前面所说的批处理，可以按如下方式实现四边形的绘制：
```cpp
void Renderer2D::DrawQuad(const glm::mat4& transform, const glm::vec4& color,
	const int& entityID)
{
	HZ_PROFILE_FUNCTION();

	// If Indices more than batch rendering can include, then start new batch rendering
	if (s_Data.QuadIndexCount >= s_Data.MaxIndices) {
		FlushAndReset();
	}

	constexpr size_t quadVertexCount = 4;
	constexpr glm::vec2 texCoords[4] = { { 0.0f, 0.0f }, { 1.0f, 0.0f }, { 1.0f, 1.0f }, { 0.0f, 1.0f } };
	const float textureIndex = 0.0f;
	const float tilingFactor = 1.0f;

	for (size_t i = 0; i < quadVertexCount; i++) {
		s_Data.QuadVBHind->Position = transform * s_Data.QuadVertexPosition[i];
		s_Data.QuadVBHind->Color = color;
		s_Data.QuadVBHind->TexCoord = texCoords[i];
		s_Data.QuadVBHind->TexIndex = textureIndex;
		s_Data.QuadVBHind->TilingFactor = tilingFactor;
		s_Data.QuadVBHind->EntityID = entityID;
		s_Data.QuadVBHind++;
	}

	s_Data.QuadIndexCount += 6;

	s_Data.Stats.GraphicCount++;
} 
```

5. **统计（Stats）**：实时统计 Draw Calls, Quad Count, Vertex Count 等。
```cpp
Renderer2D::BeginScene(camera); // 开启场景，设置视图投影矩阵
Renderer2D::DrawQuad({0.0f, 0.0f}, {1.0f, 1.0f}, {0.8f, 0.2f, 0.3f, 1.0f}); // 绘制红色方块
Renderer2D::DrawQuad({1.0f, 0.0f}, {1.0f, 1.0f}, texture); // 绘制带纹理方块
Renderer2D::EndScene(); // 结束场景，提交批处理数据进行渲染
```

### 实体组件系统（ECS）与 EnTT

Hazel使用 `EnTT` 库作为 ECS 框架。核心类是 `Scene`，它持有一个 `entt::registry` 对象。

1. **创建实体**：
```cpp
// 在 Scene::CreateEntityWithUUID 中 
Entity entity = { m_Registry.create(), this };
```

2. **添加组件**：
```cpp
 // 使用 emplace 添加组件 
entity.AddComponent<TransformComponent>(glm::vec3{ 0.0f }); 
// 内部调用: m_Registry.emplace<TransformComponent>(entityHandle, args...);
```

3. **获取组件**：
```cpp
 // 获取组件引用 
auto& transform = entity.GetComponent<TransformComponent>(); 
// 内部调用: m_Registry.get<TransformComponent>(entityHandle);
```

4. **遍历实体（View）**：这是 ECS 最强大的功能，可以高效遍历拥有特定组件集合的所有实体。
```cpp
// 示例：Scene::OnUpdateRuntime 中更新物理位置
// 创建一个视图，包含所有拥有 Rigidbody2DComponent 的实体
auto view = m_Registry.view<Rigidbody2DComponent>();
for (auto e : view)
{
    Entity entity = { e, this };
    auto& tc = entity.GetComponent<TransformComponent>();
    auto& rb2c = entity.GetComponent<Rigidbody2DComponent>();

    // 同步 Box2D 物理位置到 Transform 组件
    b2Body* body = (b2Body*)rb2c.RuntimeBody;
    tc.Translation.x = body->GetPosition().x;
    tc.Translation.y = body->GetPosition().y;
    tc.Rotation.z = body->GetAngle();
}
```

5. **组（Group）**：对于需要同时拥有多个组件的遍历，使用 `group` 可以进一步优化性能 (通过排序保证内存连续性)。
```cpp
// 示例：渲染所有带 Sprite 的实体
// 获取所有同时拥有 TransformComponent 和 SpriteComponent 的实体
auto& group = m_Registry.group<TransformComponent>(entt::get<SpriteComponent>);
for (auto entity : group)
{
    // 结构化绑定获取组件
    auto [transform, sprite] = group.get<TransformComponent, SpriteComponent>(entity);
    Renderer2D::DrawQuadSprite(transform.GetTransform(), sprite, (int)entity);
}
```

### Hazelnut

位于 `Hazelnut` 项目，基于 ImGui 构建的 GUI 编辑器界面。

1. **EditorLayer**: 编辑器的核心层，负责布局管理和场景渲染。
2. **面板（Panels）**：
	- `SceneHierarchyPanel`：显示场景实体树，支持创建/销毁实体。
	- `PropertiesPanel`：（集成在 Hierarchy 中）检查和修改选中实体的组件属性。
	- `ContentBrowserPanel`：浏览项目资源 (`assets` 目录)，支持拖拽资源到场景。
	- `ToolbarPanel`：顶部工具栏，控制 播放/停止/暂停 场景。
3. **视口（Viewport）**：将 `Framebuffer` 的纹理渲染为 ImGui 图像，集成 `ImGuizmo` 实现 3D 变换操作 (移动/旋转/缩放)。
4. **鼠标拾取（Mouse Picking）**：利用 Framebuffer 的 `RED_INTEGER` 附件存储实体 ID，实现点击视口选中实体。


### 物理系统集成

引擎集成了 **Box2D** 用于 2D 物理模拟。

- **运行时**: 在 `Scene::OnRuntimeStart` 时创建 `b2World`。
- **同步**: 每一帧 `Scene::OnUpdateRuntime` 会调用 `b2World::Step` 进行物理模拟，并将 Box2D 的刚体位置同步回实体的 `TransformComponent`。
- **组件映射**: `Rigidbody2DComponent` 映射为 `b2Body`，`BoxCollider2DComponent` 映射为 `b2Fixture`。

### 脚本系统

目前支持 **Native Scripting** (原生 C++ 脚本)。

- 开发者继承 `ScriptableEntity` 类。
- 实现 `OnCreate()`, `OnDestroy()`, `OnUpdate(Timestep ts)` 方法。
- 在编辑器中通过 `NativeScriptComponent` 绑定该类。
- 运行时，引擎会自动实例化脚本对象并调用生命周期函数。


### 资源管理

1. **Texture2D**：提供抽象的纹理类和 2D 纹理类，用于取消对具体 API 的依赖，并实现了  OpenGL 2D 纹理，支持从文件加载。
```cpp
class Texture
{
public:
	virtual ~Texture() = default;

	virtual uint32_t GetWidth() const = 0;
	virtual uint32_t GetHeight() const = 0;

	virtual void SetData(void* data, uint32_t size) = 0;

	virtual void Bind(uint32_t slot = 0) const = 0;

	virtual uint32_t GetRendererID() const = 0;

	virtual bool operator==(const Texture& other) const = 0;
};



class Texture2D : public Texture
{
public:
	static Ref<Texture2D> Create(uint32_t width, uint32_t height);
	static Ref<Texture2D> Create(const std::string& path);
};
```


2. **Shader**：提供抽象的着色器类，用于取消对具体渲染 API 的依赖，并封装实现了 OpenGL Shader 子类，支持从 GLSL 源码编译。
```cpp
class Shader
{
public:
	virtual ~Shader() = default;

	virtual void Bind() const = 0;
	virtual void Unbind() const = 0;

	virtual const std::string& GetName() const = 0;

	static Ref<Shader> Create(const std::string& filepath);
	static Ref<Shader> Create(const std::string& name, 
		const std::string& vertexSrc, const std::string& fragmentSrc);

	virtual void SetInt(const std::string& name, const int& value) = 0;
	virtual void SetIntArray(const std::string& name, int* values, uint32_t count) = 0;
	virtual void SetFloat(const std::string& name, const float& value) = 0;
	virtual void SetFloat3(const std::string& name, const glm::vec3& value) = 0;
	virtual void SetFloat4(const std::string& name, const glm::vec4& value) = 0;
	virtual void SetMat3(const std::string& name, const glm::mat3& value) = 0;
	virtual void SetMat4(const std::string& name, const glm::mat4& value) = 0;
};
```

3. **ShaderLibrary**：管理已加载的 Shader，防止重复加载。
4. **Asset Manager**：(开发中) 简单的路径管理，目前主要通过文件路径直接访问。


