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

