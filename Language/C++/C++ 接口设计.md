
1. **使用抽象类定义接口**
```cpp
// 接口类：纯虚函数 + 虚析构函数
class IShape {
public:
    virtual void Draw() const = 0;       // 纯虚函数
    virtual double Area() const = 0;     // 接口契约
    virtual ~IShape() = default;         // 必须声明虚析构函数
};
```

2. **遵循 SOLID 原则**
```cpp
// 单一职责原则：接口功能聚焦
class IDrawable {  // 只负责绘制
public:
    virtual void Draw() const = 0;
    virtual ~IDrawable() = default;
};


// 依赖倒置原则：依赖抽象而非实现
class Renderer {
public:
    void Render(const IDrawable& obj) {
        obj.Draw();  // 依赖抽象接口
    }
};
```


## 组合优于继承实践


1. **组合示例**
```cpp
// 引擎类（独立功能模块）
class Engine {
public:
    void Start() { /* 启动逻辑 */ }
};

// 通过组合复用功能，而非继承
class Car {
private:
    Engine engine;  // 组合关系

public:
    void StartCar() {
        engine.Start();  // 委托调用
    }
};
```

2. **灵活替换组件**
```cpp
// 抽象武器接口
class IWeapon {
public:
    virtual void Attack() const = 0;
    virtual ~IWeapon() = default;
};

// 具体武器实现
class Sword : public IWeapon {
public:
    void Attack() const override { std::cout << "Swing sword!\n"; }
};

class Bow : public IWeapon {
public:
    void Attack() const override { std::cout << "Shoot arrow!\n"; }
};

// 角色组合武器（运行时可更换）
class Character {
private:
    std::unique_ptr<IWeapon> weapon;  // 通过接口组合

public:
    void SetWeapon(std::unique_ptr<IWeapon> newWeapon) {
        weapon = std::move(newWeapon);
    }

    void Fight() {
        if(weapon) weapon->Attack();
    }
};

// 使用示例
Character hero;
hero.SetWeapon(std::make_unique<Sword>());
hero.Fight();  // 输出: Swing sword!

hero.SetWeapon(std::make_unique<Bow>());
hero.Fight();  // 输出: Shoot arrow!
```


3. **避免继承的深度耦合**
```cpp
// 错误示范：多重继承导致脆弱设计
class Bird {
public:
    virtual void Fly() { /* ... */ }
};

class Penguin : public Bird {  // 企鹅不能飞！
public:
    void Fly() override { throw std::runtime_error("Can't fly!"); }
};
```

```cpp
// 组合示范：分离飞行能力
class IFlyable {
public:
    virtual void Fly() = 0;
    virtual ~IFlyable() = default;
};

class DefaultFlyer : public IFlyable {
public:
    void Fly() override { /* 默认飞行实现 */ }
};

class Flightless : public IFlyable {
public:
    void Fly() override { /* 空实现或抛异常 */ }
};

// 鸟类组合飞行能力
class Bird {
private:
    std::unique_ptr<IFlyable> flyBehavior;

public:
    explicit Bird(std::unique_ptr<IFlyable> fb) 
        : flyBehavior(std::move(fb)) {}
    
    void PerformFly() {
        flyBehavior->Fly();
    }
};

// 使用示例
Bird eagle(std::make_unique<DefaultFlyer>());
Bird penguin(std::make_unique<Flightless>());
```

4. **优先定义小接口**：遵循接口隔离原则（ISP）
```cpp
class IReadable { /* read 方法 */ };
class IWritable { /* write 方法 */ };
```

5. **使用组合实现多态**：通过持有抽象指针/引用来委托操作
```cpp
class Computer {
	std::unique_ptr<IStorage> storage; // 可更换硬盘
};
```





