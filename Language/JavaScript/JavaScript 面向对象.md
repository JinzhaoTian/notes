
## 核心原理

1. **原型链机制**（Prototype Chain）：JavaScript 是基于原型（Prototype）的面向对象语言，不同于基于类的语言（如 Java、C++）。
```javascript
// 原型链示例
function Person(name) {
    this.name = name;
}

Person.prototype.greet = function() {
    console.log(`Hello, I'm ${this.name}`);
};

const john = new Person('John');
john.greet(); // Hello, I'm John

// 查找顺序：john -> Person.prototype -> Object.prototype -> null
```

> [!tip] 原型（Prototype）
> [原型](JavaScript%20原型链.md)（Prototype）是 JavaScript 面向对象编程的核心机制，每个 JavaScript 对象都有一个内置的 `[[Prototype]]` 属性（可通过 `__proto__` 访问），指向它的原型对象。

2. **对象创建的本质**：在 JavaScript 中，对象是属性的集合，每个对象都有一个指向原型对象的内部链接。


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

