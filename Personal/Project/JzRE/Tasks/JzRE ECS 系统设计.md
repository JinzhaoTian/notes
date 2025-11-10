---
project: JzRE
task: JzRE ECS 系统设计
status: To Do
priority: 5
progress: 0
---

设计 JzRE 的 [ECS](../../../../Computer%20Graphics/Game%20Engine/ECS.md)，可以参考下面的内容

## 简单设计

### 第一阶段：核心概念与设计目标

在开始写代码之前，必须明确ECS的核心思想和你的设计目标。

**1. 核心原则：**

- **实体 (Entity)：** 只是一个唯一的标识符（ID）。它本身没有任何数据或逻辑，仅仅是一个“袋子”或“索引”，用来将组件聚合在一起。**它不应该是一个包含组件列表的类。**
- **组件 (Component)：** 纯数据。没有行为（方法），只有状态（字段）。例如 `PositionComponent { float x, y; }`, `HealthComponent { int current, max; }`。
- **系统 (System)：** 纯逻辑。没有状态（除了可能对组件池的引用），只有行为。系统遍历拥有特定组件组合的实体，并对这些组件的**数据**进行操作。例如 `MovementSystem` 遍历所有拥有 `Position` 和 `Velocity` 组件的实体，并更新他们的位置。


**2. 设计目标：**

- **性能：** 数据局部性 (Data Locality) 是首要目标。将组件数据连续存储在内存中，以便系统迭代时最大限度利用CPU缓存。
- **解耦与灵活性：** 实体通过添加/删除组件来改变行为，而不是通过复杂的继承层次结构。
- **代码清晰度：** 逻辑被分离到不同的系统中，每个系统只关心一个特定的功能。
- **可测试性：** 系统是纯函数或接近纯函数，很容易单独测试。

### 第二阶段：架构设计与实现

我们将自底向上地构建这个ECS。

#### 1. 实体 (Entity) 的实现

实体就是一个ID。但为了高效地管理实体生命周期和组件查询，我们通常需要一些额外的机制。

```cpp
// 使用一个简单的整型作为ID。0可以代表无效实体。
using Entity = uint32_t;

// 一个版本号，与ID结合使用，可以检测“野指针”问题（即一个已被销毁的实体又被访问）。
// 将ID和版本号打包成一个更强大的句柄。
struct EntityHandle {
    uint32_t id;
    uint32_t generation; // 版本号
};


// 在实践中，你可能直接使用 `using Entity = uint32_t;` 开始，后期再升级为带版本号的句柄。
```

你需要一个 **实体管理器 (EntityManager)** 来负责创建和销毁实体，并回收ID。

```cpp

class EntityManager {
public:
    Entity Create() {
        if (!freeList.empty()) {
            Entity id = freeList.front();
            freeList.pop();
            // 可能还需要重置 generation? 取决于你的设计
            return id;
        }
        // ... 扩容逻辑
        return nextEntityId++;
    }

    void Destroy(Entity entity) {
        // 1. 通知所有组件池，此实体已被销毁，需要清理其组件。
        // 2. 增加该实体ID对应的 generation（如果使用Handle）。
        // 3. 将ID放入空闲列表 freeList。
        freeList.push(entity);
    }

private:
    std::queue<Entity> freeList;
    Entity nextEntityId{1}; // 从1开始，0作为无效ID
    // 如果使用Handle，需要一个数组来记录每个ID的当前generation
    std::vector<uint32_t> generations;
};
```

#### 2. 组件 (Component) 的实现

每个组件类型都是一个独立的`struct`或`class`。

```cpp

// 示例组件
struct TransformComponent {
    float x, y, z;
    float rotation;
    float scale;
};

struct VelocityComponent {
    float dx, dy, dz;
};

struct HealthComponent {
    int current;
    int max;
};
```

关键不在于组件本身，而在于如何存储它们。我们需要为**每种组件类型**创建一个 **组件池 (Component Pool)**。

**组件池的设计是ECS性能的核心：**
```cpp


// 这是一个类型擦除的基类，用于在World中管理所有不同类型的池。
class IComponentPool {
public:
    virtual ~IComponentPool() = default;
    virtual void EntityDestroyed(Entity entity) = 0; // 实体销毁时，清理对应组件
};

// 具体的、类型化的组件池
template<typename T>
class ComponentPool : public IComponentPool {
public:
    // 添加组件
    T& AddComponent(Entity entity, T&& component) {
        // 检查实体是否已有该组件...
        entityToIndexMap[entity] = components.size();
        indexToEntityMap[components.size()] = entity;
        components.push_back(std::move(component));
        return components.back();
    }

    // 删除组件
    void RemoveComponent(Entity entity) {
        // 1. 找到该实体组件的索引
        size_t indexOfRemoved = entityToIndexMap[entity];
        size_t indexOfLast = components.size() - 1;

        // 2. 用最后一个元素覆盖要删除的元素 (Swap-and-Pop)
        components[indexOfRemoved] = components[indexOfLast];

        // 3. 更新映射关系
        Entity entityOfLast = indexToEntityMap[indexOfLast];
        entityToIndexMap[entityOfLast] = indexOfRemoved;
        indexToEntityMap[indexOfRemoved] = entityOfLast;

        // 4. 移除旧的映射关系
        entityToIndexMap.erase(entity);
        indexToEntityMap.erase(indexOfLast);

        // 5. 弹出最后一个元素
        components.pop_back();
    }

    // 获取组件
    T& GetComponent(Entity entity) {
        return components[entityToIndexMap[entity]];
    }

    // 实体销毁时调用
    virtual void EntityDestroyed(Entity entity) override {
        if (entityToIndexMap.find(entity) != entityToIndexMap.end()) {
            RemoveComponent(entity);
        }
    }

    // 核心：组件数据数组。连续内存存储！
    std::vector<T> components;

private:
    // 映射表：实体ID -> 组件数组索引
    std::unordered_map<Entity, size_t> entityToIndexMap;
    // 映射表：组件数组索引 -> 实体ID
    std::unordered_map<size_t, Entity> indexToEntityMap;
};
```

_Swap-and-Pop_ 技巧确保了组件数组始终是紧凑的，没有空洞，这对于保持迭代性能至关重要。

#### 3. 系统 (System) 的实现

系统关心的是**拥有特定组件组合的实体集合**。这个集合称为 **Archetype** 或 **Group**。我们通过 **签名 (Signature)** 来定义系统需要的组件组合。

```cpp


// 使用位掩码（Bitset）来表示签名。
// 每个组件类型都有一个唯一的ID，用于在Bitset中标记自己。
using ComponentTypeID = std::size_t;

// 一个函数用于为每个组件类型生成唯一的ID
template<typename T>
ComponentTypeID GetComponentTypeID() {
    static ComponentTypeID typeID = nextComponentTypeID++;
    return typeID;
}
inline ComponentTypeID nextComponentTypeID{0};

// 签名类
constexpr size_t MAX_COMPONENTS = 64;
using Signature = std::bitset<MAX_COMPONENTS>;

// 示例：定义MovementSystem需要的签名
Signature movementSignature;
movementSignature.set(GetComponentTypeID<TransformComponent>());
movementSignature.set(GetComponentTypeID<VelocityComponent>());
```
系统基类可以非常简单：

```cpp


class System {
public:
    // 系统关心的实体签名
    Signature componentSignature;

    // 实体管理
    std::set<Entity> entities;

    // 当实体的签名改变时，由Coordinator调用
    void AddEntityToSystem(Entity entity) { entities.insert(entity); }
    void RemoveEntityFromSystem(Entity entity) { entities.erase(entity); }

    // 每帧更新逻辑（由子类实现）
    virtual void Update(float deltaTime) = 0;
};
```

具体的系统：

```cpp

class MovementSystem : public System {
public:
    virtual void Update(float deltaTime) override {
        for (auto entity : entities) {
            // 从World（或Coordinator）获取组件
            auto& transform = world->GetComponent<TransformComponent>(entity);
            auto& velocity = world->GetComponent<VelocityComponent>(entity);

            transform.x += velocity.dx * deltaTime;
            transform.y += velocity.dy * deltaTime;
            transform.z += velocity.dz * deltaTime;
        }
    }
    // 需要持有一个World的指针或引用
};
```
#### 4. 协调器/世界 (Coordinator / World) - 粘合剂

这是ECS的核心管理器，它将所有部分连接起来。
```cpp


class Coordinator {
public:
    void Init() {
        // 初始化组件池等
        componentPools.resize(MAX_COMPONENTS); // 预先分配空间
    }

    // 实体管理
    Entity CreateEntity() { return entityManager.Create(); }
    void DestroyEntity(Entity entity) {
        entityManager.Destroy(entity);
        // 通知所有系统该实体被销毁
        for (auto& system : systems) {
            system->RemoveEntityFromSystem(entity);
        }
        // 通知所有组件池该实体被销毁
        for (auto& pool : componentPools) {
            if (pool) {
                pool->EntityDestroyed(entity);
            }
        }
    }

    // 组件管理
    template<typename T>
    void RegisterComponent() {
        ComponentTypeID typeID = GetComponentTypeID<T>();
        // 确保池子数组足够大
        if (componentPools.size() <= typeID) {
            componentPools.resize(typeID + 1);
        }
        // 创建该类型组件的池
        componentPools[typeID] = std::make_unique<ComponentPool<T>>();
    }

    template<typename T>
    T& AddComponent(Entity entity, T&& component) {
        auto& pool = GetComponentPool<T>();
        // 添加组件
        T& comp = pool->AddComponent(entity, std::forward<T>(component));

        // 更新实体签名
        entitySignatures[entity].set(GetComponentTypeID<T>());

        // 通知所有系统，实体签名已更新
        for (auto& system : systems) {
            auto& systemSignature = system->componentSignature;
            if ((entitySignatures[entity] & systemSignature) == systemSignature) {
                system->AddEntityToSystem(entity);
            } else {
                system->RemoveEntityFromSystem(entity);
            }
        }
        return comp;
    }

    template<typename T>
    void RemoveComponent(Entity entity) {
        // ... 类似AddComponent，但清除签名位，并通知系统
    }

    template<typename T>
    T& GetComponent(Entity entity) {
        return GetComponentPool<T>()->GetComponent(entity);
    }

    // 系统管理
    template<typename T, typename... TArgs>
    std::shared_ptr<T> RegisterSystem(TArgs&&... args) {
        auto system = std::make_shared<T>(std::forward<TArgs>(args)...);
        systems.insert(system);
        return system;
    }

    template<typename T>
    void SetSystemSignature(Signature signature) {
        systemSignatures[GetSystemTypeID<T>()] = signature;
        // 为所有现有实体重新检查并加入该系统
    }

private:
    template<typename T>
    std::shared_ptr<ComponentPool<T>> GetComponentPool() {
        ComponentTypeID typeID = GetComponentTypeID<T>();
        return std::static_pointer_cast<ComponentPool<T>>(componentPools[typeID]);
    }

    EntityManager entityManager;
    // 索引是 ComponentTypeID
    std::vector<std::shared_ptr<IComponentPool>> componentPools;
    // 每个实体当前的组件签名
    std::unordered_map<Entity, Signature> entitySignatures;

    // 存储所有系统
    std::unordered_set<std::shared_ptr<System>> systems;
    // 每个系统的签名（可选，也可以用System自身的member）
    std::unordered_map<ComponentTypeID, Signature> systemSignatures;
};
```


### 第三阶段：使用示例

```cpp


// 初始化
Coordinator coordinator;
coordinator.Init();
coordinator.RegisterComponent<TransformComponent>();
coordinator.RegisterComponent<VelocityComponent>();
coordinator.RegisterComponent<HealthComponent>();

// 注册系统并设置其签名
auto movementSystem = coordinator.RegisterSystem<MovementSystem>();
Signature movementSig;
movementSig.set(GetComponentTypeID<TransformComponent>());
movementSig.set(GetComponentTypeID<VelocityComponent>());
coordinator.SetSystemSignature<MovementSystem>(movementSig);

// 创建实体
Entity player = coordinator.CreateEntity();
coordinator.AddComponent<TransformComponent>(player, {0, 0, 0, 0, 1.0f});
coordinator.AddComponent<VelocityComponent>(player, {1.0f, 0, 0});
coordinator.AddComponent<HealthComponent>(player, {100, 100});

// 游戏循环
while (gameIsRunning) {
    movementSystem->Update(deltaTime);
    // ... 其他系统更新
}
```


### 第四阶段：高级主题与优化

1. **Archetype-based ECS vs. Sparse-Set ECS:**
    - 上述实现是 **Sparse-Set** 风格（每个组件类型一个数组）。优点是实现简单，查询直接。
    - **Archetype-based**（如Unity DOTS，Bevy）：将具有完全相同组件组合的实体分组存储在同一个“Archetype”内存块中。性能更极致（迭代时缓存命中率接近100%），但实现复杂得多，实体添加/删除组件开销更大。

2. **多线程：**
    - 不同的系统如果没有数据依赖，可以并行执行。例如，`MovementSystem` 和 `AISystem` 可能可以并行。
    - 使用 Job System 将系统的工作分解成多个任务并行处理。
    - **注意：** 需要仔细管理数据竞争。通常使用读写锁或更高级的数据结构（如ECST的`ForEach`中的并行迭代）。

3. **缓存与性能分析：**
    - 始终使用分析工具（如VTune，Tracy）来定位热点。
    - 确保核心循环（系统迭代）是紧凑的，避免虚函数调用（如果可能，使用CRTP等静态多态）、分支预测失败。

4. **序列化：**
    - 因为组件是纯数据，序列化整个游戏状态变得非常容易。你可以遍历所有实体和它们的组件，将它们写入磁盘。

5. **与引擎其他部分集成：**
    - **渲染：** 一个 `RenderSystem` 会收集所有拥有 `TransformComponent` 和 `MeshComponent` 的实体，将它们的变换和网格数据提交给渲染API。
    - **物理：** 一个 `PhysicsSystem` 会处理拥有 `TransformComponent` 和 `PhysicsBodyComponent` 的实体，与物理引擎（如Bullet，Box2D）进行交互。


### 总结

设计一个ECS是一个权衡的过程。从简单的Sparse-Set实现开始是一个非常好的选择，它能解决大部分问题并且性能已经远超传统的OOP继承模型。当你遇到性能瓶颈并对底层有更深理解后，再考虑向更复杂的Archetype模式演进。