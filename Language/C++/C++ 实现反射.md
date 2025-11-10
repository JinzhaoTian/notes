C++ 本身不提供原生反射支持

## 核心思路

1. **构建类型注册系统**：为每个类创建元数据描述
2. **属性访问接口**：通过字符串名称访问成员
3. **对象工厂**：支持通过类名创建实例
4. **序列化支持**：可选，用于保存/加载状态


## 实现方案

| 方案          | 核心原理                                                  | 优点                                                  | 缺点                                                      | 适用场景            |
| ----------- | ----------------------------------------------------- | --------------------------------------------------- | ------------------------------------------------------- | --------------- |
| **运行时手动注册** | 在程序启动时，手动向一个全局工厂注册类的元信息（名称、属性、方法）。                    | - 实现简单，完全可控  <br>- 跨平台性最好  <br>- 无需额外工具链            | - 工作量大，易出错  <br>- 维护成本高（需同步代码与注册）                       | 小型项目或作为临时方案     |
| **代码生成工具**  | 使用宏、模板等C++特性，**在编译前**通过外部工具解析源代码生成反射元数据。              | - 自动化程度高  <br>- 功能强大（可捕获几乎所有信息）  <br>- 成熟方案可选（如 Qt） | - 引入外部工具链，构建系统变复杂  <br>- 可能需要修改代码风格（如引入特定宏）             | 中型到大型项目，追求自动化   |
| **编译时静态反射** | 利用C++的**模板元编程**、**constexpr**、**静态初始化**等技术，在编译期生成元数据。 | - 无外部依赖，纯C++方案  <br>- 编译期完成，无运行时开销  <br>- 类型安全      | - 实现极其复杂，对C++技能要求极高  <br>- C++标准支持有限（依赖新特性）  <br>- 调试困难 | 追求极致性能、零开销的顶级项目 |
| **使用第三方库**  | 集成现有的开源反射库，如 `rttr`, `cpp-reflection` 等。              | - 省去造轮子的时间  <br>- 功能相对完善                            | - 可能无法完全满足引擎特定需求  <br>- 有依赖和许可问题  <br>- 可能与引擎架构有冲突      | 想快速原型验证或非核心功能   |


### 方案一：手动注册

在程序启动时，手动向一个全局工厂注册类的元信息（名称、属性、方法）。需要创建一个中心化的 `ReflectionRegistry` 类，并提供注册和查询接口。

#### 实现步骤

1. 定义元数据类（如 `Class`，`Property`，`Function`，`Enum`）。
2. 在每一需要反射的类的初始化代码中（如静态块或全局注册函数），手动创建其元数据对象并注册。
3. 通过 `ReflectionRegistry` 按名称查找类型，并访问其成员。


#### 示例代码

##### 实现

```cpp
#include <iostream>
#include <string>
#include <unordered_map>
#include <functional>
#include <vector>
#include <memory>
#include <type_traits>

//---------------------- 反射系统核心 ----------------------
class TypeDescriptor;
class BaseProperty;

// 全局类型注册表
class ReflectionRegistry {
public:
    static std::unordered_map<std::string, TypeDescriptor*>& GetTypeMap() {
        static std::unordered_map<std::string, TypeDescriptor*> typeMap;
        return typeMap;
    }
};

// 属性基类
class BaseProperty {
public:
    virtual ~BaseProperty() = default;
    virtual void* GetPtr(void* object) const = 0;
    virtual const std::type_info& GetType() const = 0;
    virtual void SetFromString(void* object, const std::string& value) = 0;
    virtual std::string GetAsString(void* object) const = 0;
};

// 类型描述符
class TypeDescriptor {
public:
    std::string name;
    std::unordered_map<std::string, std::unique_ptr<BaseProperty>> properties;
    
    // 创建对象实例的工厂函数
    std::function<void*()> factory;
    
    void AddProperty(const std::string& name, BaseProperty* prop) {
        properties.emplace(name, prop);
    }
    
    BaseProperty* GetProperty(const std::string& name) const {
        auto it = properties.find(name);
        return it != properties.end() ? it->second.get() : nullptr;
    }
};

// 属性实现 (支持基本类型和std::string)
template <typename Class, typename T>
class Property : public BaseProperty {
public:
    using MemberPtr = T(Class::*);
    using Setter = void (Class::*)(T);
    using Getter = T (Class::*)() const;

    Property(MemberPtr member) : memberPtr(member) {}
    
    Property(Setter setter, Getter getter) 
        : setterFunc(setter), getterFunc(getter) {}

    void* GetPtr(void* object) const override {
        Class* obj = static_cast<Class*>(object);
        if (memberPtr) return &(obj->*memberPtr);
        return nullptr;
    }

    const std::type_info& GetType() const override {
        return typeid(T);
    }

    void SetFromString(void* object, const std::string& value) override {
        Class* obj = static_cast<Class*>(object);
        if constexpr (std::is_same_v<T, int>) {
            if (memberPtr) obj->*memberPtr = std::stoi(value);
            else if (setterFunc) (obj->*setterFunc)(std::stoi(value));
        }
        else if constexpr (std::is_same_v<T, float>) {
            if (memberPtr) obj->*memberPtr = std::stof(value);
            else if (setterFunc) (obj->*setterFunc)(std::stof(value));
        }
        else if constexpr (std::is_same_v<T, std::string>) {
            if (memberPtr) obj->*memberPtr = value;
            else if (setterFunc) (obj->*setterFunc)(value);
        }
        else if constexpr (std::is_same_v<T, bool>) {
            bool bValue = (value == "true" || value == "1");
            if (memberPtr) obj->*memberPtr = bValue;
            else if (setterFunc) (obj->*setterFunc)(bValue);
        }
    }

    std::string GetAsString(void* object) const override {
        Class* obj = static_cast<Class*>(object);
        T value;
        
        if (memberPtr) value = obj->*memberPtr;
        else if (getterFunc) value = (obj->*getterFunc)();
        
        if constexpr (std::is_same_v<T, int> || 
                      std::is_same_v<T, float>) {
            return std::to_string(value);
        }
        else if constexpr (std::is_same_v<T, bool>) {
            return value ? "true" : "false";
        }
        else if constexpr (std::is_same_v<T, std::string>) {
            return value;
        }
        return "";
    }

private:
    MemberPtr memberPtr = nullptr;
    Setter setterFunc = nullptr;
    Getter getterFunc = nullptr;
};

// 反射宏简化注册
#define REFLECTABLE() \
    static void __RegisterReflection(); \
    static TypeDescriptor* __GetTypeDescriptor(); \
    virtual TypeDescriptor* GetType() const { return __GetTypeDescriptor(); }

#define BEGIN_REGISTER_TYPE(ClassName) \
    TypeDescriptor* ClassName::__GetTypeDescriptor() { \
        static TypeDescriptor desc; \
        static bool initialized = false; \
        if (!initialized) { \
            desc.name = #ClassName; \
            desc.factory = []() -> void* { return new ClassName; }; \
            ClassName::__RegisterReflection(); \
            ReflectionRegistry::GetTypeMap()[#ClassName] = &desc; \
            initialized = true; \
        } \
        return &desc; \
    } \
    void ClassName::__RegisterReflection()

#define REGISTER_PROPERTY(PropertyName) \
    __GetTypeDescriptor()->AddProperty(#PropertyName, \
        new Property<ClassName, decltype(PropertyName)>(&ClassName::PropertyName));

#define REGISTER_PROPERTY_ACCESSORS(PropertyName, SetterName, GetterName) \
    __GetTypeDescriptor()->AddProperty(#PropertyName, \
        new Property<ClassName, decltype(std::declval<ClassName>().GetterName())>( \
            &ClassName::SetterName, &ClassName::GetterName));
```

##### 使用

```cpp
//---------------------- 应用示例 ----------------------
// 渲染引擎中的物体基类
class GameObject {
public:
    REFLECTABLE()
    virtual ~GameObject() = default;
    
    std::string name;
    bool visible = true;
    
    // 示例：通过访问器注册的属性
    void SetScale(float s) { scale = s; }
    float GetScale() const { return scale; }
    
private:
    float scale = 1.0f;
};

// 具体游戏物体实现
BEGIN_REGISTER_TYPE(GameObject)
    REGISTER_PROPERTY(name)
    REGISTER_PROPERTY(visible)
    REGISTER_PROPERTY_ACCESSORS(scale, SetScale, GetScale)
END_REGISTER_TYPE()

class MeshObject : public GameObject {
public:
    REFLECTABLE()
    
    int meshID = 0;
    float opacity = 1.0f;
};

BEGIN_REGISTER_TYPE(MeshObject)
    REGISTER_PROPERTY(meshID)
    REGISTER_PROPERTY(opacity)
END_REGISTER_TYPE()

//---------------------- 使用示例 ----------------------
int main() {
    // 初始化类型系统
    GameObject::__GetTypeDescriptor();
    MeshObject::__GetTypeDescriptor();
    
    // 模拟点击到的物体
    std::vector<std::unique_ptr<GameObject>> sceneObjects;
    sceneObjects.push_back(std::make_unique<MeshObject>());
    GameObject* clickedObject = sceneObjects[0].get();
    
    // 获取对象类型信息
    TypeDescriptor* typeDesc = clickedObject->GetType();
    std::cout << "Selected object type: " << typeDesc->name << "\n";
    
    // 显示所有属性
    std::cout << "\nProperties:\n";
    for (auto& [name, prop] : typeDesc->properties) {
        std::cout << "- " << name << ": " 
                  << prop->GetAsString(clickedObject) << "\n";
    }
    
    // 通过名称修改属性
    if (auto prop = typeDesc->GetProperty("opacity")) {
        std::cout << "\nChanging opacity to 0.5...\n";
        prop->SetFromString(clickedObject, "0.5");
        std::cout << "New opacity: " 
                  << dynamic_cast<MeshObject*>(clickedObject)->opacity << "\n";
    }
    
    // 通过名称修改scale
    if (auto prop = typeDesc->GetProperty("scale")) {
        prop->SetFromString(clickedObject, "2.5");
        std::cout << "New scale: " 
                  << clickedObject->GetScale() << "\n";
    }
    
    return 0;
}
```



#### 优缺点

- **优点**：
	- 绝对控制，实现简单，与任何平台、任何图形 API 都无关。
	- 不依赖外部工具，仅靠 C++ 预处理器即可工作。
	- 运行时性能高，类信息一旦注册，访问速度很快。
- **缺点**：
	- **代码侵入性强**：每个需要反射的类都必须添加宏。
	- **极其繁琐**，每当类成员改变时，必须同步更新注册代码，否则会导致运行时错误。
	- 不适合大型项目。

### 方案二：代码生成工具（推荐）

使用宏、模板等 C++ 特性，在编译前通过外部工具解析源代码生成反射元数据。这是最成熟、最可靠的方案，被许多大型引擎（如 Unreal Engine、Qt）采用。

核心思想是使用一个**独立的解析工具**在编译之前扫描你的源代码，根据特定标记（如自定义宏 `REFLECT()` 或注解 `[[reflect]]` ）生成包含元数据注册代码的 `*.generated.cpp` 文件。

#### 实现步骤

1. **设计标记宏**：在你的引擎头文件中定义宏，用于标记需要反射的类、属性或方法。
```cpp
#define REFLECT_CLASS(class_name) \
	/* 一些静态声明，用于后续工具识别 */

#define REFLECT_PROPERTY(property_name) \
	/* 标记需要反射的成员变量 */
```

2. **开发或选择解析工具**：
    - **自研工具**：可以用 Python、C++ 写一个工具，利用 Clang LibTooling 来解析 AST ，这是最精准的方式。
    - **使用现有工具**：例如 [refl-cpp](https://github.com/veselink1/refl-cpp) 

3. **集成到构建系统**：在 CMake、Premake 等配置中，添加一个自定义构建步骤（Custom Build Step）。
	- 规则：对于每个包含反射宏的 `.h` 文件，先运行你的解析工具，生成一个`XXX.generated.cpp` 文件，然后将这些生成的文件一起编译。

4. **在生成的文件中实现注册**：解析工具生成的代码会包含类似方案1中的手动注册代码，自动计算 `offset` 、类型名称等。


#### 示例

1. **自定义 Component**：`MyComponent.h`
```cpp
#include "ReflectMacros.h"

class MyComponent {
    REFLECT_CLASS(MyComponent)
public:
    REFLECT_PROPERTY(value)
    int value = 100;

    REFLECT_PROPERTY(name)
    std::string name;
};
```

2. **运行工具后，生成 `MyComponent.generated.cpp`**：
```cpp
// 警告：这是自动生成的文件，不要手动修改！
#include "MyComponent.h"
#include "TypeRegistry.h"

void Register_MyComponent() {
    Class* cls = new Class{"MyComponent"};
    cls->properties.push_back({"value", "int", offsetof(MyComponent, value)});
    cls->properties.push_back({"name", "std::string", offsetof(MyComponent, name)});
    TypeRegistry::RegisterClass(cls);
}
```


#### 优缺点

- **优点**：
	- **功能极其强大**：可以自动获取任何元信息，包括变量名、类型、函数参数、注释等。
	- **代码非侵入性**：标记宏比实现宏简洁得多，业务代码更干净。
	- **高性能**：生成的代码是纯C++，编译后运行效率高。
	- **一劳永逸**：自动化程度高，能捕获最全面的信息，与手动注册一样灵活。
	- **易于维护**：反射逻辑集中在代码生成器中，而非散落在无数宏里。
- **缺点**：
	- **构建流程变复杂**，，需要将代码生成工具深度集成到构建流程中（如 CMake）
	- **开发成本高**：需要开发维护一个解析工具（如果自研），或使用 `libclang` 等库，这是一个不小的工程。


### 方案三：编译时静态反射

利用 C++ 的模板元编程、`constexpr`、静态初始化等技术，在编译期获取类型信息生成元数据。这是 C++ 元编程的“圣杯”，目前还没有完美的标准解决方案，但社区有很多基于模板黑魔法的探索。

它旨在不生成额外文件的情况下，在编译期内完成所有反射信息的收集。

#### 常见技术

1. **特化 Traits**：为每个可反射类特化一个模板类，在其中静态定义其成员信息。
2. **宏 + 静态初始化**：结合宏和 `constexpr` 函数，在编译期构造出类型的元数据链表。
3. **C++2x 的反射提案**：等待未来的 C++ 标准（如 `^` 操作符），但目前还不能用。


#### 实现思路

1. **手动注册**：为每个需要反射的类和属性，编写一个外部的注册函数或结构体。
2. **模板技巧**：使用模板来抽象注册过程，通过成员函数指针来访问属性。
3. **类型擦除**：使用`std::any`或自定义的`Variant`类型来存储和操作不同类型的属性。


#### 示例

```cpp
// 为一个类特化反射Traits
template<typename T>
struct ReflectionTraits;

template<>
struct ReflectionTraits<MyComponent> {
    static constexpr auto class_name = "MyComponent";
    static constexpr auto properties = std::make_tuple(
        PropertyTraits<&MyComponent::value>{},
        PropertyTraits<&MyComponent::name>{}
    );
};
```


#### 优缺点

- **优点**：
	- 无需外部工具，无宏污染。
	- 类型安全，利用编译器进行类型检查。
	- 代码更现代，符合现代 C++ 编程范式，无比优雅。
- **缺点**：
	- **需要手动注册**：每个类、每个属性都需要写一行注册代码，非常繁琐且容易遗漏。
	- **无法自动获取成员变量名**：必须以字符串形式传入 `"x"` ，无法保证与实际变量名一致。
	- **运行时开销**：`std::function` 和 `std::any` 会带来一定的性能开销（堆分配、虚函数调用）。
	- 对编译器支持要求高。



### 方案四：使用第三方库

直接使用像 `RTTR`（Run Time Type Reflection）这样的库。

#### 示例

```cpp
#include <rttr/registration>
using namespace rttr;

RTTR_REGISTRATION
{
    registration::class_<MyComponent>("MyComponent")
        .property("value", &MyComponent::value)
        .property("name", &MyComponent::name);
}
```

#### 优缺点

- **优点**：
	- **快速上手**，功能丰富（包括动态创建、方法调用等）。
- **缺点**：
	- **外部依赖**，库可能很重，许可证可能与你的引擎不兼容（RTTR 是 MIT，很友好），你可能不需要它的所有功能。


