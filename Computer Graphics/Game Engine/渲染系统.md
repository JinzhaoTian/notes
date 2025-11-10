传统游戏引擎的子系统和 ECS（Entity-Component-System）架构代表了两种不同的设计哲学。

## 核心思想

1. **传统游戏引擎子系统（Monolithic/Object-Oriented）**：
    - **核心思想**：基于**继承**和**强耦合**的对象模型。游戏对象（`GameObject`）是一个庞大的类，或者通过继承衍生出各种子类（如 `RenderableObject` ，`PhysicsObject` ）。渲染子系统是一个**中心化**、**全局性**的管理器，它直接管理和操作这些游戏对象。
    - **渲染流程**：“告诉我画什么，我就画什么”，渲染子系统持有所有可渲染对象（如 Mesh、材质、变换矩阵）的列表或引用，并在每一帧遍历这些列表，调用它们的渲染方法或直接提交其数据到图形 API 。

2. **ECS（Data-Oriented Design）**
    - **核心思想**： **组合优于继承**，**数据与行为彻底分离**。
        - **Entity**：只是一个ID，代表一个存在。
        - **Component**：纯数据，没有行为（例如 `TransformComponent`，`MeshComponent`，`MaterialComponent` ）。
        - **System**：纯行为，没有状态，它负责处理拥有特定 Component 组合的 Entity 。
    - **渲染流程**：“我有这些数据，我来处理它们”。渲染系统不关心“对象”，只关心数据。它查询所有拥有 `TransformComponent`，`MeshComponent`，`MaterialComponent` 的 Entity，获取它们的数据，然后高效地、批处理地将这些数据提交到 GPU。


## 实现

### 传统游戏引擎的渲染子系统

1. **核心部分**：
	- **渲染器（Renderer）**：单例或全局可访问的模块。负责管理GPU资源（Shader、Texture、Buffer）、设置渲染状态、执行绘制命令。
	- **场景管理器（SceneManager）**：管理场景图（Scene Graph）或所有游戏对象的列表。负责可见性剔除、排序等。
	- **可渲染对象（Renderable Object）**：如 `Mesh`，`Model`，`Sprite` 等类。它们通常包含顶点数据、材质属性和一个 `Draw()` 方法。

2. **伪代码**：
```cpp
// --- 传统的游戏对象 ---
class GameObject {
public:
    virtual void Update(float deltaTime) = 0;
    virtual void Draw(RHICommandList& CommandList) = 0; // 每个对象自己负责如何绘制自己
    Transform transform;
};

class StaticMeshObject : public GameObject {
public:
    void Update(float deltaTime) override { ... }
    
    void Draw(RHICommandList& CommandList) override {
        // 1. 通过RHI接口设置状态
	    CommandList.SetShader(m_Material->GetRHIShader());
	    CommandList.SetTexture(0, m_Material->GetRHITexture());
	    CommandList.SetVertexBuffer(m_Mesh->GetRHIVertexBuffer());
	    CommandList.SetIndexBuffer(m_Mesh->GetRHIIndexBuffer());
	
	    // 2. 设置模型矩阵Uniform (假设通过Uniform Buffer)
	    CommandList.SetShaderUniform("ModelMatrix", transform.GetMatrix());
	
	    // 3. 发起绘制调用
	    CommandList.DrawIndexed(m_Mesh->GetIndexCount(), 1, 0, 0, 0);
    }

    Mesh* m_Mesh;
    Material* m_Material;
};

// --- 渲染子系统 (通常是单例) ---
class Renderer {
public:
    void RegisterObject(GameObject* obj) {
        m_RenderQueue.push_back(obj);
    }

    void RenderFrame() {
        RHICommandList* CmdList = RHI::GetDevice()->CreateCommandList();
	    CmdList->BeginFrame();
	
	    for (auto& obj : m_RenderQueue) {
	        if (IsVisible(obj)) {
	            obj->Draw(*CmdList); // 每个对象独立提交命令，难以批处理
	        }
	    }
	
	    CmdList->EndFrame();
	    RHI::GetDevice()->SubmitCommandList(CmdList);
    }

private:
    std::vector<GameObject*> m_RenderQueue;
};

// --- 游戏循环中 ---
GameObject* player = new StaticMeshObject(...);
Renderer::Instance()->RegisterObject(player);

while (gameIsRunning) {
    ...
    Renderer::Instance()->RenderFrame();
}
```

- **优点**：
	- 直观，符合面向对象思维，易于理解和实现小型项目。
- **缺点**：
    - **耦合性高**：`GameObject` 和 `Renderer` 紧密耦合，`GameObject` 要知道如何绘制自己（ `Draw()` 方法）。
    - **性能不友好**：大量的虚拟函数调用、数据在内存中分散（Cache 不友好）、难以进行高效的批处理（因为每个 `Draw()` 调用可能切换不同的 Shader、Texture 状态）。
    - 每个对象独立提交 Draw Call 的方式依然无法充分发挥新 API 的批处理优势，因为 Draw Call 本身是分散的。
    - 状态管理也集中在渲染管理器，复杂度高。
- **数据流**：
	- `GameObject -> (Draw Call) -> RHI`

### ECS 的渲染系统

1. **核心部分**：
	- **组件（Components）**：纯数据 struct
	    - `TransformComponent`: `vec3 position`, `quat rotation`, `vec3 scale`
	    - `MeshComponent`: `VertexBufferHandle vbo`, `IndexBufferHandle ibo`
	    - `MaterialComponent`: `ShaderHandle shader`, `TextureHandle albedoTexture`
	- **系统（System）**：无状态的函数集合或类，它定义了一组它关心的组件（即查询条件）。
	- **渲染系统（RenderSystem）**：一个查询特定组件组合并执行渲染逻辑的系统。

2. **实现伪代码**：
```cpp
// --- ECS 的组件是纯数据 ---
struct TransformComponent {
    glm::vec3 position;
    glm::quat rotation;
    glm::vec3 scale;
};

struct MeshComponent {
    VertexBufferHandle vbo;
    IndexBufferHandle ibo;
    int indexCount;
};

struct MaterialComponent {
    ShaderHandle shader;
    TextureHandle albedoMap;
};

// --- ECS 的渲染系统 ---
class RenderSystem : public System {
public:
    // 定义该系统需要处理哪些组件
    RenderSystem() {
        RequireComponent<TransformComponent>();
        RequireComponent<MeshComponent>();
        RequireComponent<MaterialComponent>();
    }

    // 每帧执行
    void Update(double deltaTime) {
        auto& RHI = RHISingleton::Get();
	    RHICommandList* CmdList = RHI.GetActiveCommandList();
	
	    // 1. 获取所有具有渲染组件的实体（数据是连续存储的，迭代速度快）
	    auto view = m_Registry.view<TransformComponent, MeshComponent, MaterialComponent>();
	    
	    // 2. 【关键】按材质ID对实体进行排序，实现状态切换最小化
	    view.sort([](const MaterialComponent& a, const MaterialComponent& b) {
	        return a.materialId < b.materialId;
	    });
	
	    // 3. 准备批处理
	    ShaderHandle currentShader = nullptr;
	    TextureHandle currentTexture = nullptr;
	
	    // 4. 遍历所有已排序的实体（数据在内存中连续，Cache友好）
	    for (auto entity : view) {
	        auto [transform, mesh, material] = view.get<TransformComponent, MeshComponent, MaterialComponent>(entity);
	
	        // 5. 检查状态是否变化
	        if (material.rhiShader != currentShader) {
	            FlushBatch(CmdList); // 提交当前批次
	            CmdList->SetGraphicsPipelineState(material.rhiPipelineState);
	            currentShader = material.rhiShader;
	        }
	        if (material.rhiTexture != currentTexture) {
	            FlushBatch(CmdList);
	            CmdList->SetTexture(0, material.rhiTexture);
	            currentTexture = material.rhiTexture;
	        }
	
	        // 6. 不是立即绘制，而是将实例数据（如模型矩阵）加入一个动态缓冲区
	        AddInstanceData(transform.CalculateWorldMatrix());
	    }
	
	    // 7. 绘制最后一批数据
	    FlushBatch(CmdList);
    }

private:
    void FlushBatch() {
        if (m_InstanceCount > 0) {
	        // 8. 【高效提交】绑定共享的网格VB/IB
	        CmdList->SetVertexBuffer(m_CommonMeshVB);
	        CmdList->SetIndexBuffer(m_CommonMeshIB);
	
	        // 9. 绑定包含所有实例矩阵的Uniform Buffer或顶点缓冲区
	        CmdList->SetShaderUniform("InstanceData", m_InstanceDataBuffer);
	
	        // 10. 一个RHI调用绘制整个批次！
	        CmdList->DrawIndexedInstanced(m_IndexCountPerMesh, m_InstanceCount, 0, 0, 0);
	        m_InstanceCount = 0;
	    }
    }
};

// --- 游戏循环中 ---
// 创建实体和组件
Entity player = registry.CreateEntity();
registry.AddComponent<TransformComponent>(player, {...});
registry.AddComponent<MeshComponent>(player, {...});
registry.AddComponent<MaterialComponent>(player, {...});

// 系统在每帧更新
while (gameIsRunning) {
    ...
    renderSystem->Update(&renderer);
}
```
- **优点**：
    - **性能极高**：数据与行为分离，使得 `Transform`、`Mesh` 等数据可以以SOA（结构体数组）方式连续存储在内存中，迭代时 Cache 命中率极高。
    - **灵活性强**：轻松添加新功能。要给一个物体添加渲染效果，只需挂上对应的组件即可，无需修改继承体系。
    - **易于优化**：系统可以高效地对实体进行排序和批处理，极大减少 GPU 状态切换（如Shader、Texture 切换），提升渲染效率。
- **缺点**：
    - **概念复杂**：理解 ECS 和 DOD 需要转变思维模式。
    - **架构复杂**： 需要实现 ECS 框架本身（注册组件、查询实体、系统执行顺序等）。
- **数据流**：
	- `Component Data -> RenderSystem (排序/批处理) -> (Batched Draw Call) -> RHI`



### 对比

| 特性        | 传统游戏引擎子系统                       | ECS                                     |
| --------- | ------------------------------- | --------------------------------------- |
| **设计哲学**  | 面向对象，继承与封装                      | 数据导向，组合与分离                              |
| **核心单元**  | 游戏对象（GameObject）                | 实体（Entity） + 组件（Component） + 系统（System） |
| **数据与行为** | 耦合在一起（对象有自己的方法）                 | 彻底分离（Component是数据，System是行为）            |
| **渲染实现**  | 渲染管理器调用每个对象的`Draw`方法            | 渲染系统查询并处理所有渲染相关组件的数据                    |
| **内存访问**  | 数据分散，Cache不友好                   | 数据连续存储（SOA），Cache友好                     |
| **性能优化**  | 批处理困难，状态切换多                     | 天然易于排序和批处理，状态切换最少化                      |
| **灵活性**   | 依赖深层次的继承树，难以修改                  | 通过组合添加功能，高度灵活和可扩展                       |
| **典型代表**  | 早期版本的 Unity, Unreal Engine (部分) | Unity DOTS, 《守望先锋》引擎, Bevy引擎            |


## 融合架构

将高度优化的传统渲染引擎作为底层基础设施，与上层的 ECS 游戏玩法层相结合，是一种实用且强大的架构模式。这种融合允许团队利用现有成熟渲染引擎的优势（如高级特效、稳定性和工具链），同时享受 ECS 架构在游戏逻辑开发中的灵活性和性能优势。

这种融合架构通常分为三个主要层次：
1. **底层基础设施层**：传统渲染引擎（如 Unity 的旧有渲染器、Unreal 的渲染模块或自研引擎）
2. **核心游戏层**：ECS 架构驱动的游戏逻辑和实体管理
3. **桥梁层**：专门的渲染同步系统，负责在两者之间高效地同步数据

### 组件设计

在 ECS 层，我们需要设计专门的组件来与传统渲染引擎交互：
```cpp
// ECS 层的渲染相关组件
struct RenderProxyComponent {
    RenderableHandle renderHandle; // 指向底层渲染引擎中的可渲染对象
    bool needsUpdate;              // 脏标记，指示是否需要同步
};

// 传统的变换、网格、材质组件
struct TransformComponent {
    glm::vec3 position;
    glm::quat rotation;
    glm::vec3 scale;
    // ... 其他变换数据
};

struct MeshComponent {
    MeshHandle meshHandle; // 指向底层渲染引擎中的网格资源
    // ... 其他网格数据
};

struct MaterialComponent {
    MaterialHandle materialHandle; // 指向底层渲染引擎中的材质资源
    // ... 其他材质数据
};
```

### 渲染同步系统实现

渲染同步系统是连接 ECS 和传统渲染引擎的核心桥梁：
```cpp
class RenderSyncSystem : public System {
public:
    RenderSyncSystem() {
        RequireComponent<TransformComponent>();
        RequireComponent<MeshComponent>();
        RequireComponent<MaterialComponent>();
        RequireComponent<RenderProxyComponent>();
    }
    
    void Update(EngineRenderer& renderer) {
        // 获取所有需要处理的实体
        auto entities = GetSystemEntities();
        
        // 处理新创建的实体 - 在渲染引擎中创建代理对象
        for (auto entity : entities) {
            if (!entity.HasComponent<RenderProxyComponent>()) {
                CreateRenderProxy(entity, renderer);
            }
        }
        
        // 同步变换数据
        SyncTransforms(renderer);
        
        // 同步网格和材质数据
        SyncRenderData(renderer);
        
        // 处理被销毁的实体 - 清理渲染引擎中的代理对象
        CleanupDestroyedEntities(renderer);
    }
    
private:
    void CreateRenderProxy(Entity entity, EngineRenderer& renderer) {
        auto& transform = entity.GetComponent<TransformComponent>();
        auto& mesh = entity.GetComponent<MeshComponent>();
        auto& material = entity.GetComponent<MaterialComponent>();
        
        // 在底层渲染引擎中创建可渲染对象
        RenderableHandle handle = renderer.CreateRenderable(
            mesh.meshHandle,
            material.materialHandle,
            TransformToMatrix(transform)
        );
        
        // 添加渲染代理组件
        entity.AddComponent<RenderProxyComponent>(handle, true);
    }
    
    void SyncTransforms(EngineRenderer& renderer) {
        // 获取所有需要更新变换的实体
        auto view = m_registry.view<TransformComponent, RenderProxyComponent>();
        
        for (auto entity : view) {
            auto [transform, proxy] = view.get<TransformComponent, RenderProxyComponent>(entity);
            
            // 检查脏标记或直接比较变换是否变化
            if (proxy.needsUpdate || TransformChanged(transform)) {
                // 更新底层渲染引擎中的变换
                renderer.SetTransform(
                    proxy.renderHandle,
                    TransformToMatrix(transform)
                );
                
                // 重置脏标记
                proxy.needsUpdate = false;
            }
        }
    }
    
    void SyncRenderData(EngineRenderer& renderer) {
        // 处理网格和材质变化的同步
        // 类似变换同步的逻辑，但频率可能较低
    }
    
    void CleanupDestroyedEntities(EngineRenderer& renderer) {
        // 检查所有有RenderProxyComponent但已被标记为销毁的实体
        auto view = m_registry.view<RenderProxyComponent>(entt::exclude<ActiveComponent>);
        
        for (auto entity : view) {
            auto& proxy = view.get<RenderProxyComponent>(entity);
            
            // 从渲染引擎中移除对象
            renderer.DestroyRenderable(proxy.renderHandle);
            
            // 从ECS中移除组件
            m_registry.remove<RenderProxyComponent>(entity);
        }
    }
    
    // 辅助函数：将ECS变换转换为渲染引擎使用的矩阵
    glm::mat4 TransformToMatrix(const TransformComponent& transform) {
        glm::mat4 matrix = glm::translate(glm::mat4(1.0f), transform.position);
        matrix = matrix * glm::mat4_cast(transform.rotation);
        matrix = glm::scale(matrix, transform.scale);
        return matrix;
    }
};
```


### 传统渲染引擎适配器

为了使传统渲染引擎能够与 ECS 系统协同工作，我们需要创建一个适配器接口：
```cpp
// 传统渲染引擎的抽象接口
class EngineRenderer {
public:
    // 资源管理
    virtual MeshHandle LoadMesh(const std::string& path) = 0;
    virtual MaterialHandle CreateMaterial(const MaterialDesc& desc) = 0;
    virtual TextureHandle LoadTexture(const std::string& path) = 0;
    
    // 可渲染对象管理
    virtual RenderableHandle CreateRenderable(
        MeshHandle mesh, 
        MaterialHandle material, 
        const glm::mat4& transform
    ) = 0;
    
    virtual void DestroyRenderable(RenderableHandle handle) = 0;
    virtual void SetTransform(RenderableHandle handle, const glm::mat4& transform) = 0;
    virtual void SetMaterial(RenderableHandle handle, MaterialHandle material) = 0;
    
    // 渲染执行
    virtual void BeginFrame() = 0;
    virtual void RenderFrame() = 0;
    virtual void EndFrame() = 0;
    
    // 其他渲染相关功能...
};
```


### 游戏循环集成

在游戏主循环中，我们需要协调 ECS 系统和传统渲染引擎的执行：
```cpp
// 游戏主循环
while (gameRunning) {
    // 1. 处理输入
    InputSystem::Update();
    
    // 2. 更新游戏逻辑（ECS 系统）
    float deltaTime = GetDeltaTime();
    movementSystem.Update(deltaTime);
    animationSystem.Update(deltaTime);
    physicsSystem.Update(deltaTime);
    
    // 3. 同步渲染数据
    renderSyncSystem.Update(engineRenderer);
    
    // 4. 执行渲染（传统渲染引擎）
    engineRenderer.BeginFrame();
    engineRenderer.RenderFrame();
    engineRenderer.EndFrame();
    
    // 5. 其他后期处理
    // ...
}
```


### 高级优化策略

#### 脏标记与增量更新

为了最小化同步开销，实现精细化的脏标记系统：
```cpp
struct TransformComponent {
    glm::vec3 position;
    glm::quat rotation;
    glm::vec3 scale;
    
    // 脏标记
    uint32_t dirtyFlags;
    
    static constexpr uint32_t POSITION_DIRTY = 1 << 0;
    static constexpr uint32_t ROTATION_DIRTY = 1 << 1;
    static constexpr uint32_t SCALE_DIRTY = 1 << 2;
    
    bool IsDirty() const { return dirtyFlags != 0; }
    void ClearDirty() { dirtyFlags = 0; }
};

// 在移动系统中设置脏标记
class MovementSystem : public System {
    void Update(float deltaTime) {
        auto view = m_registry.view<TransformComponent, VelocityComponent>();
        
        for (auto entity : view) {
            auto [transform, velocity] = view.get<TransformComponent, VelocityComponent>(entity);
            
            // 更新位置
            transform.position += velocity.linear * deltaTime;
            transform.dirtyFlags |= TransformComponent::POSITION_DIRTY;
            
            // 更新旋转
            // ...
        }
    }
};
```

#### 批处理与实例化

```cpp
void RenderSyncSystem::SyncTransforms(EngineRenderer& renderer) {
    // 按材质分组，实现实例化渲染
    std::unordered_map<MaterialHandle, std::vector<RenderableHandle>> materialGroups;
    
    auto view = m_registry.view<TransformComponent, RenderProxyComponent, MaterialComponent>();
    
    for (auto entity : view) {
        auto [transform, proxy, material] = view.get<TransformComponent, RenderProxyComponent, MaterialComponent>(entity);
        
        if (transform.IsDirty()) {
            materialGroups[material.materialHandle].push_back(proxy.renderHandle);
            
            // 批量更新变换
            // 注意：实际实现需要收集变换数据
        }
    }
    
    // 批量提交变换更新
    for (const auto& [material, renderables] : materialGroups) {
        if (renderables.size() > BATCH_THRESHOLD) {
            renderer.SetTransformsBatch(renderables.data(), renderables.size());
        } else {
            for (auto handle : renderables) {
                renderer.SetTransform(handle, /* 对应的变换矩阵 */);
            }
        }
    }
}
```


#### 多线程同步

利用 ECS 的数据导向特性实现多线程同步：
```cpp
// 使用工作线程处理渲染数据准备
void PrepareRenderDataJob(void* data) {
    RenderDataJob* job = static_cast<RenderDataJob*>(data);
    
    for (int i = job->start; i < job->end; ++i) {
        Entity entity = job->entities[i];
        // 准备渲染数据...
    }
}

void RenderSyncSystem::Update(EngineRenderer& renderer) {
    // 获取需要处理的实体
    auto entities = GetSystemEntities();
    
    // 多线程准备渲染数据
    const int threadCount = 4;
    const int batchSize = entities.size() / threadCount;
    
    std::vector<RenderDataJob> jobs(threadCount);
    std::vector<std::thread> threads;
    
    for (int i = 0; i < threadCount; ++i) {
        jobs[i].start = i * batchSize;
        jobs[i].end = (i == threadCount - 1) ? entities.size() : (i + 1) * batchSize;
        jobs[i].entities = entities.data();
        
        threads.emplace_back(PrepareRenderDataJob, &jobs[i]);
    }
    
    // 等待所有线程完成
    for (auto& thread : threads) {
        thread.join();
    }
    
    // 主线程执行实际的渲染引擎调用
    // ...
}
```