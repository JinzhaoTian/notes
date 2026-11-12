JavaScript 是基于 [Prototype（原型）](JavaScript%20Object.md#Prototype（原型）)的面向对象语言，不同于基于类的语言（如 Java、C++）。在 JavaScript 中，对象是属性的集合，每个对象都有一个指向原型对象的内部链接。

> [!caution] 与 C++、Java 的面向对象核心区别
> C++ / Java 的继承发生在类与类之间，类定义了实例的内存布局和方法，C++ / Java 的对象布局和类继承关系在编译/类加载后基本固定；JavaScript 的继承发生在对象与对象之间，对象通过内部 [Prototype Chain（原型链）](JavaScript%20Prototype%20Chain.md)委托查找属性和方法，让继承关系不再是编译期固定的类层级，而是运行时可建立、可改变、可绕过的对象委托关系。
> 
> **关键具体差异**：
> 1. **继承不是复制父类成员**：
> 	- JavaScript 中 `b.m()` 的查找路径是：`b` 自身 → `b.__proto__` → `B.prototype` → `A.prototype` → `Object.prototype` → `null`。
> 	- C++ / Java 的继承会体现在对象布局/类层级中，方法在类元数据或虚表中分派。
> 2. **ES6 `class` 不是传统类，只是一个语法糖**：JavaScript 底层仍是 `B.prototype.__proto__ === A.prototype`。`class` 只是让写法更像 C++ / Java，并附带严格模式、必须使用 `new`、方法不可枚举等语义。
> ```javascript
> class B extends A { m(){ super.m() } }
> ```
> 3. **多继承能力不同**：
>    - C++ 支持类多继承和虚继承；
>    - Java 是单类继承+多接口继承；
>    - JavaScript 原型链只能单链继承，多继承要靠 `mixin` 复制属性或组合，不是语言级多继承。
> 4. **封装强度不同**：
> 	- C++ / Java 的 `private/protected` 是编译期访问控制；
> 	- JavaScript 普通属性运行时都能访问，`#field` 才是硬私有。JavaScript 还可通过[属性描述符](JavaScript%20属性描述符.md)控制可枚举、可写、可配置。
> 5. **多态机制不同**：
> 	- C++ 有虚函数、模板；
> 	- Java 有重写、接口、重载；
> 	- JavaScript 主要靠“对象有没有这个属性/方法”的**鸭子类型**和原型链动态查找。JavaScript 没有 Java/C++ 那种按参数签名的方法重载，后定义同名方法会覆盖。
> 6. **`this` 和函数模型不同**：
> 	- C++ / Java 的方法调用和 `this` 基本绑定明确；
> 	- JavaScript 函数可独立传递，`this` 取决于调用方式，所以常需 `bind` 或箭头函数。
> 
> 对比如下：
> 
> |维度|C++|Java|JavaScript|
> |---|---|---|---|
> |对象模型|类/结构体模板生成对象，有值语义、栈/堆对象|类模板生成堆对象，变量是引用|对象是动态属性包，每个对象有原型链接；`class` 底层仍是函数+原型|
> |继承载体|类之间继承，支持多继承、虚继承|类单继承 + 接口多实现|对象到对象的原型链委托；单原型链，`extends`单继承，可用 mixin/组合模拟多继承|
> |成员复制/布局|子类对象包含基类子对象，字段布局编译期确定|对象包含父类字段布局，类元数据存方法|实例不“复制”原型成员；方法通常在 `prototype` 上共享，查找沿原型链直到 `null`|
> |方法分派|非虚函数静态绑定；虚函数经虚表运行期分派；模板编译期多态|重载编译期解析；可重写方法运行期动态分派；接口多态|属性动态查找；函数是一等值，可随时替换；鸭子类型，无真正方法重载|
> |多态/类型|静态强类型，名义类型为主，模板支持结构化|静态强类型，名义类型，接口|动态弱类型，结构化/鸭子类型；`instanceof`查原型链，不是编译期类型|
> |封装|`public/protected/private/friend`，编译期访问控制|`public/protected/private/包私有`，编译期控制|传统属性默认公开，靠约定 `_`；ES2022 `#private` 才有语言级硬私有；可用闭包模拟|
> |对象创建|栈/堆、构造/拷贝/移动、RAII、确定性析构|`new` 堆对象、GC、构造器|字面量、`new`、`Object.create`、`class`；GC；可随时增删属性，无析构|
> |`this`|成员函数隐式绑定当前对象|方法中绑定当前实例|由调用方式决定：默认/隐式/显式/`new`/箭头；可 `call/apply/bind`，最动态|
> |运行时可变性|类结构编译后基本固定|类加载后结构基本固定，反射有限|可改实例、改原型、猴子补丁；原型上属性修改会影响未遮蔽的实例|
> |构造/初始化|初始化列表、析构、拷贝/移动语义|构造器、GC|`new` 做：建对象、设原型、执行构造、返回对象；`class` 必须 `new`，`super` 前不能用 `this`|



## 核心机制

1. **构造函数模式**
```javascript
function Animal(name) {
    // 实例属性
    this.name = name;
    this.age = 0;
    
    // 实例方法（每个实例都会创建新函数，不推荐）
    this.eat = function() {
        console.log(`${this.name} is eating`);
    };
}

// 原型方法（推荐）
Animal.prototype.sleep = function() {
    console.log(`${this.name} is sleeping`);
};

const cat = new Animal('Kitty');
```
- **要点**：
	- 属性（Properties）在构造函数体中定义
	- 方法（Method）需要手动添加到原型中


2. **原型继承**
```javascript
function Cat(name, color) {
    // 调用父类构造函数
    Animal.call(this, name);
    this.color = color;
}

// 设置原型链
Cat.prototype = Object.create(Animal.prototype);
Cat.prototype.constructor = Cat;

// 添加子类特有方法
Cat.prototype.meow = function() {
    console.log(`${this.name} says meow!`);
};
```


3. **ES6 类语法**（语法糖）（**推荐**）
```javascript
class Animal {
    constructor(name) {
        this.name = name;
        this.age = 0;
    }
    
    // 实例方法
    eat() {
        console.log(`${this.name} is eating`);
    }
    
    // 静态方法
    static isAnimal(obj) {
        return obj instanceof Animal;
    }
    
    // Getter/Setter
    get info() {
        return `${this.name}, ${this.age} years old`;
    }
    
    set info(value) {
        const [name, age] = value.split(', ');
        this.name = name;
        this.age = parseInt(age);
    }
}

class Cat extends Animal {
    constructor(name, color) {
        super(name); // 调用父类构造函数
        this.color = color;
    }
    
    meow() {
        console.log(`${this.name} says meow!`);
    }
    
    // 方法重写
    eat() {
        super.eat(); // 调用父类方法
        console.log('...and purring');
    }
}
```

> [!caution] ES6 的 `class` 只是语法糖，底层仍然是基于原型的。
> 




## 深入原理

1. **`new` 操作符的工作流程**
```javascript
// new 操作符实际执行的操作
function myNew(constructor, ...args) {
    // 1. 创建空对象，设置原型
    const obj = Object.create(constructor.prototype);
    
    // 2. 执行构造函数，绑定 this
    const result = constructor.apply(obj, args);
    
    // 3. 返回对象（如果构造函数返回对象，则使用该对象）
    return result instanceof Object ? result : obj;
}
```

2. **原型链图示**
```
实例对象 → 构造函数.prototype → Object.prototype → null
    ↑
    |__proto__
```

3. **属性描述符**
```javascript
const obj = { x: 10 };

// 获取属性描述符
const descriptor = Object.getOwnPropertyDescriptor(obj, 'x');
console.log(descriptor);
// { value: 10, writable: true, enumerable: true, configurable: true }

// 设置属性描述符
Object.defineProperty(obj, 'y', {
    value: 20,
    writable: false,    // 不可写
    enumerable: false,  // 不可枚举
    configurable: true  // 可配置
});
```



## 使用示例

### 创建对象

JavaScript 的对象有三种创建方式:

1. **利用 `new Object()`**：
```javascript
 var obj1 = new Object();
```

2. **利用字面量**：
```javascript
var obj2 = { name: 'John' };
```

3. **利用构造函数**：
```javascript
function Person(name) {
    this.name = name;
}
const obj3 = new Person('John');
```

4. **利用 `Object.create()`**：
```javascript
const obj4 = Object.create(Object.prototype, {
    name: { value: 'John', writable: true }
});
```

5. **工厂模式**：
```javascript
function createPerson(name) {
    return {
        name,
        greet() {
            console.log(`Hi, I'm ${this.name}`);
        }
    };
}
```

6. **类语法**（ES6 推荐）：
```javascript
class Person {
    constructor(name) {
        this.name = name;
    }
}
```

### 继承

1. **原型链继承**
```javascript
function Parent(name) {
    this.name = name;
    this.colors = ['red', 'blue'];
}

Parent.prototype.sayName = function() {
    console.log(this.name);
};

function Child(name, age) {
    Parent.call(this, name); // 继承实例属性
    this.age = age;
}

Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child;
```

2. **组合继承**（最常用）
```javascript
function Parent(name) {
    this.name = name;
}

function Child(name, age) {
    Parent.call(this, name); // 第二次调用 Parent
    this.age = age;
}

Child.prototype = new Parent(); // 第一次调用 Parent
Child.prototype.constructor = Child;
```

3. **寄生组合式继承**（最佳实践）
```javascript
function inheritPrototype(child, parent) {
    const prototype = Object.create(parent.prototype);
    prototype.constructor = child;
    child.prototype = prototype;
}

function Parent(name) {
    this.name = name;
}

function Child(name, age) {
    Parent.call(this, name);
    this.age = age;
}

inheritPrototype(Child, Parent);
```


### 多态

```javascript
class Shape {
    area() {
        throw new Error('子类必须实现 area 方法');
    }
}

class Circle extends Shape {
    constructor(radius) {
        super();
        this.radius = radius;
    }
    
    area() {
        return Math.PI * this.radius ** 2;
    }
}

class Rectangle extends Shape {
    constructor(width, height) {
        super();
        this.width = width;
        this.height = height;
    }
    
    area() {
        return this.width * this.height;
    }
}

// 多态调用
const shapes = [new Circle(5), new Rectangle(4, 6)];
shapes.forEach(shape => console.log(shape.area()));
```


### 封装与访问控制

```javascript
// 使用闭包实现私有属性
function Counter() {
    let count = 0; // 私有变量
    
    this.increment = function() {
        count++;
        return this.getValue();
    };
    
    this.decrement = function() {
        count--;
        return this.getValue();
    };
    
    this.getValue = function() {
        return count;
    };
}

// 使用 Symbol 实现私有属性
const _count = Symbol('count');
class BetterCounter {
    constructor() {
        this[_count] = 0;
    }
    
    increment() {
        this[_count]++;
    }
    
    get value() {
        return this[_count];
    }
}

// ES2022 私有字段
class ModernCounter {
    #count = 0; // 真正的私有字段
    
    increment() {
        this.#count++;
    }
    
    get value() {
        return this.#count;
    }
}
```

## 高级特性

1. **Mixin 模式**
```javascript
// Mixin 函数
const canEat = Base => class extends Base {
    eat(food) {
        console.log(`${this.name} eats ${food}`);
        return this;
    }
};

const canSleep = Base => class extends Base {
    sleep() {
        console.log(`${this.name} sleeps`);
        return this;
    }
};

class Animal {
    constructor(name) {
        this.name = name;
    }
}

// 应用 Mixin
class Dog extends canSleep(canEat(Animal)) {
    bark() {
        console.log('Woof!');
    }
}
```

2. **代理与反射**
```javascript
const person = {
    name: 'John',
    age: 30
};

const handler = {
    get(target, property) {
        console.log(`Getting ${property}`);
        return property in target ? target[property] : 'Not found';
    },
    
    set(target, property, value) {
        if (property === 'age' && typeof value !== 'number') {
            throw new Error('Age must be a number');
        }
        console.log(`Setting ${property} to ${value}`);
        target[property] = value;
        return true;
    }
};

const proxy = new Proxy(person, handler);
```

3. **对象不变性**
```javascript
const obj = {
    name: 'John',
    details: {
        age: 30,
        city: 'New York'
    }
};

// 浅冻结
Object.freeze(obj);
obj.name = 'Jane'; // 不会生效（严格模式会报错）

// 深冻结函数
function deepFreeze(object) {
    Object.freeze(object);
    
    Object.getOwnPropertyNames(object).forEach(prop => {
        const value = object[prop];
        if (value && typeof value === 'object' && !Object.isFrozen(value)) {
            deepFreeze(value);
        }
    });
    
    return object;
}
```

