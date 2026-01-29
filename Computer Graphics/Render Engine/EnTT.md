EnTT 是一个用现代 C++ 编写的高性能实体组件系统（ECS）开源框架，以其轻量级、灵活和高性能的设计而闻名。

## 核心特点

1. **集成便捷**：纯头文件（header-only）库，只需包含 `<entt/entt.hpp>` 即可使用。
2. **核心数据结构**：采用**稀疏集**来存储和管理实体与组件数据。
	- **稀疏集组件池**：同类型组件数据完全连续存储，直接线性遍历数组，完美利用 CPU 缓存。添加/删除组件是 `O(1)` 操作，但跨类型查询多个组件可能有一定开销。
3. **设计理念**：在易用性、内存和性能之间提供灵活选择，不做过度设计。
4. **现代 C++**：需要 C++17 或以上标准编译器，充分利用现代 C++ 特性。


## 使用示例

以下代码展示了 EnTT 最基本的使用流程：
```cpp
#include <entt/entt.hpp>

// 1. 定义组件（纯数据结构）
struct Position { float x, y; };
struct Velocity { float dx, dy; };

// 2. 创建注册表
entt::registry registry;

// 3. 创建实体并添加组件
for(int i = 0; i < 10; ++i) {
    entt::entity e = registry.create(); // 创建实体
    registry.emplace<Position>(e, i*1.0f, i*1.0f); // 添加位置组件
    if(i % 2 == 0) {
        registry.emplace<Velocity>(e, i*0.1f, i*0.1f); // 半数实体添加速度组件
    }
}

// 4. 系统：移动所有具有位置和速度的实体
auto view = registry.view<Position, Velocity>(); // 创建视图，高效筛选实体
view.each([](auto& pos, auto& vel) { // 遍历所有符合条件的实体
    pos.x += vel.dx;
    pos.y += vel.dy;
});
```


## 核心模块

| 模块名称    | 核心接口/类                      | 主要功能与特点                      |
| ------- | --------------------------- | ---------------------------- |
| **注册表** | `entt::registry`            | **世界容器**，管理所有实体和组件，是库的核心。    |
| **实体**  | `entt::entity`              | **唯一标识符**，本质上是一个整数，没有数据。     |
| **组件**  | 用户定义的结构体                    | **纯数据**，不包含逻辑，可通过注册表挂载到实体。   |
| **视图**  | `registry.view<Comp...>()`  | **只读或读写遍历**，高效查询拥有特定组件组合的实体。 |
| **观察器** | `registry.observer()`       | **主动响应变化**，监听组件的新增、移除或更新。    |
| **组**   | `registry.group<Comp...>()` | **最高性能遍历**，对固定组件组合进行缓存和极速迭代。 |

### 详细功能

#### `entt::registry`

`entt::registry` 是世界容器，用来管理所有实体和组件，是 EnTT 的核心。它不仅是创建、销毁实体的工厂，也是存储和索引所有组件的数据库。

```cpp
entt::registry registry; // 创建一个世界
auto entity = registry.create(); // 创建实体
registry.destroy(entity); // 销毁实体
```

##### 相关方法

1. **实体生命周期**：
	- `.create()`：创建实体
	- `.destroy()`：销毁实体
2. 组件操作：
	- `.emplace()`
	- `.get()`
	- `.remove()`
	- `.patch()` 添加、获取、删除和修改组件
3. 关系与层次
	- `.emplace_as<Parent>()`
	- `.get_as()` 建立父子关系（从v3.12开始）
4. 存储与视图：
	- `.storage<Comp>()`
	- `.view<Comp...>()`
	- `.group<Comp...>()` 访问底层存储，创建视图/组进行遍历
5. 观察与监听：
	- `.observer()`
	- `.on_construct<Comp>()` 监听组件变化，实现响应式逻辑
6. **上下文数据**
	- `.ctx()`：提供一个固定的地方来存放**全局单例数据**，例如游戏状态、物理世界指针、渲染上下文、资源管理器等。
		- `.emplace<T>()`：构造并放置
		- `.insert_or_assign()`：构造后赋值（有则替换）
		- `.get<T>()`：获取数据
		- `.erase<T>()`：删除数据
		- `.contains<T>()`：检查是否存在
```cpp
#include <entt/entt.hpp>

// 定义一些全局数据类
struct GameState {
    bool isPaused = false;
    int currentLevel = 1;
};

struct PhysicsWorld {
    // ... 物理引擎相关数据
};

int main() {
    entt::registry registry;

    // 1. 设置/替换数据
    // 使用 emplace 存放数据，如果该类型已存在则会被替换
    registry.ctx().emplace<GameState>();
    registry.ctx().emplace<PhysicsWorld>();

    // 也可以直接构造或替换
    registry.ctx().insert_or_assign(GameState{false, 2});

    // 2. 获取数据引用
    GameState& state = registry.ctx().get<GameState>();
    state.currentLevel = 3;

    // 3. 检查数据是否存在
    if (registry.ctx().contains<GameState>()) {
        // 进行相关操作...
    }

    // 4. 删除数据
    registry.ctx().erase<GameState>();

    return 0;
}
```




#### 组件

组件是用户定义的普通 C++ 结构体，纯数据，不包含逻辑，可通过注册表挂载到实体。

```cpp
// 定义组件
struct Position { float x, y; };
struct Velocity { float dx, dy; };

// 添加/获取/删除组件
registry.emplace<Position>(entity, 1.0f, 2.0f);
auto& pos = registry.get<Position>(entity);
registry.remove<Position>(entity);
```


#### `registry.view<Comp...>()`

`registry.view<Comp...>()` 是视图，用于只读或读写地高效遍历拥有指定组件的实体。

```cpp
// 遍历所有同时拥有Position和Velocity的实体
auto view = registry.view<Position, Velocity>();
for (auto entity : view) {
    auto& pos = view.get<Position>(entity);
    auto& vel = view.get<Velocity>(entity);
    pos.x += vel.dx;
}
// 更简洁的lambda写法
view.each([](Position& pos, const Velocity& vel) {
    pos.x += vel.dx;
});
```

#### `registry.observer()`

`registry.observer()` 观察器，主动响应变化，监听组件的新增、移除或更新。用于在组件发生变化时执行代码，非常适合触发事件。

```cpp
// 监听Velocity组件被添加到任何实体
auto observer = registry.observer<Velocity>()
    .connect<&MySystem::onVelocityAdded>(); // 连接回调函数
```


#### `registry.group<Comp...>()`

当需要频繁迭代一个固定的组件组合时，组 `registry.group<Comp...>()` 是最高效的选择。它会在内部进行数据布局优化，但会限制使用灵活性（如不能在组存在时动态增删涉及的组件类型）。




### 扩展模块

除了上述核心 ECS 模块，EnTT 还提供了一些强大的官方插件，它们虽然不是核心，但在构建复杂应用时非常有用：

| 扩展模块       | 核心类                            | 作用                          |
| ---------- | ------------------------------ | --------------------------- |
| **委托与信号**  | `entt::delegate`, `entt::sigh` | 实现灵活的事件和回调系统，用于系统间解耦。       |
| **资源管理**   | `entt::resource_cache`         | 用于缓存和管理（如图片、音频等）资源的加载与生命周期。 |
| **运行时反射**  | `entt::meta`                   | 在运行时动态获取和操作类型信息，无需宏或代码生成。   |
| **协作式调度器** | `entt::organizer`              | 帮助定义和管理各处理系统的执行顺序与依赖关系。     |
