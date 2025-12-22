JavaScript 原型（Prototype）是一个让对象能够从其他对象继承属性和方法的机制，理解原型对于掌握 JavaScript 至关重要，因为它是语言面向对象特性的基础。

原型链（Prototype Chain）使得对象可以共享方法，减少内存使用，并提供了灵活的继承机制，虽然 ES6 引入了类的概念，但底层仍然是基于原型的实现。

```
实例对象 → 构造函数.prototype → Object.prototype → null
    ↑               ↑                  ↑
    |__proto__      |__proto__         |__proto__
```

## 工作原理

1. **属性查找**：
```javascript
function Person(name) {
    this.name = name;
}

Person.prototype.sayHello = function() {
    console.log(`Hello, ${this.name}`);
};

const john = new Person('John');

// 属性查找过程
console.log(john.name);       // 1. 先在 john 对象上查找 → 找到 "John"
console.log(john.age);        // 1. john 上没有 → 2. Person.prototype 上查找 → 未找到
console.log(john.toString()); // 1. john 上无 → 2. Person.prototype 上无 → 3. Object.prototype 上找到
```

2. **原型链继承**：
```javascript
const animal = {
    eats: true,
    walk() {
        console.log('Animal walks');
    }
};

const rabbit = {
    jumps: true,
    __proto__: animal // 设置 rabbit 的原型为 animal
};

console.log(rabbit.eats); // true（从 animal 继承）
console.log(rabbit.jumps); // true（自身属性）
rabbit.walk(); // "Animal walks"（从 animal 继承）

// 继续原型链
const longEar = {
    earLength: 10,
    __proto__: rabbit
};

console.log(longEar.eats); // true（从 animal 通过 rabbit 继承）
longEar.walk(); // "Animal walks"
```

## 构造函数的原型

1. **`constructor` 属性**
```javascript
function Person(name) {
    this.name = name;
}

// Person.prototype 自动获得 constructor 属性
console.log(Person.prototype.constructor === Person); // true

const john = new Person('John');
console.log(john.constructor === Person); // true

// 可以修改 constructor，但通常不建议
Person.prototype = {
    sayHello() {
        console.log('Hello');
    }
};
console.log(Person.prototype.constructor === Person); // false
console.log(Person.prototype.constructor === Object); // true
```

2. **`new` 操作符的内部原理**
```javascript
function myNew(constructor, ...args) {
    // 1. 创建一个空对象，并设置其原型
    const obj = Object.create(constructor.prototype);
    
    // 2. 将构造函数的作用域赋给新对象（this 指向新对象）
    const result = constructor.apply(obj, args);
    
    // 3. 如果构造函数返回对象，则返回该对象；否则返回新对象
    return result instanceof Object ? result : obj;
}

// 测试
function Person(name) {
    this.name = name;
}
Person.prototype.greet = function() {
    console.log(`Hi, I'm ${this.name}`);
};

const p = myNew(Person, 'John');
p.greet(); // "Hi, I'm John"
```

## 原型操作的方法

1. **获取和设置原型**
```javascript
const animal = {
    eats: true
};

const rabbit = {
    jumps: true
};

// 设置原型（现代方式）
Object.setPrototypeOf(rabbit, animal);

// 获取原型（推荐方式）
console.log(Object.getPrototypeOf(rabbit) === animal); // true

// 检查对象是否在原型链上
console.log(animal.isPrototypeOf(rabbit)); // true
console.log(Object.prototype.isPrototypeOf(rabbit)); // true

// 创建指定原型的对象
const newRabbit = Object.create(animal, {
    jumps: {
        value: true,
        writable: true,
        enumerable: true,
        configurable: true
    }
});
```

2. **属性遍历和检测**
```javascript
function Person(name) {
    this.name = name;
}

Person.prototype.age = 30;

const john = new Person('John');

// 检测属性
console.log('name' in john); // true（自身或原型链）
console.log(john.hasOwnProperty('name')); // true（仅自身）
console.log(john.hasOwnProperty('age')); // false（来自原型）

// 遍历属性
for (let key in john) {
    console.log(key); // 'name', 'age'（原型链上的可枚举属性）
}

// 只获取自身属性
console.log(Object.keys(john)); // ['name']
console.log(Object.getOwnPropertyNames(john)); // ['name']
```

## 原型的实际应用

1. **方法共享和性能优化**：
```javascript
// 不好的做法：每个实例创建新函数
function BadCircle(radius) {
    this.radius = radius;
    this.area = function() {
        return Math.PI * this.radius ** 2;
    };
}

const c1 = new BadCircle(5);
const c2 = new BadCircle(10);
console.log(c1.area === c2.area); // false（不同的函数）

// 好的做法：方法放在原型上共享
function GoodCircle(radius) {
    this.radius = radius;
}

GoodCircle.prototype.area = function() {
    return Math.PI * this.radius ** 2;
};

GoodCircle.prototype.circumference = function() {
    return 2 * Math.PI * this.radius;
};

const g1 = new GoodCircle(5);
const g2 = new GoodCircle(10);
console.log(g1.area === g2.area); // true（相同的函数）
```


2. **内置对象的原型**
```javascript
// 数组原型
const arr = [1, 2, 3];
console.log(arr.__proto__ === Array.prototype); // true

// 扩展内置原型（谨慎使用）
if (!Array.prototype.last) {
    Array.prototype.last = function() {
        return this[this.length - 1];
    };
}

const numbers = [1, 2, 3, 4, 5];
console.log(numbers.last()); // 5

// 字符串原型
String.prototype.capitalize = function() {
    return this.charAt(0).toUpperCase() + this.slice(1);
};

console.log('hello'.capitalize()); // "Hello"
```


3. **原型继承的实现**
```javascript
// 父类
function Animal(name) {
    this.name = name;
    this.speed = 0;
}

Animal.prototype.run = function(speed) {
    this.speed = speed;
    console.log(`${this.name} runs with speed ${this.speed}`);
};

Animal.prototype.stop = function() {
    this.speed = 0;
    console.log(`${this.name} stopped`);
};

// 子类
function Rabbit(name, earLength) {
    Animal.call(this, name); // 调用父类构造函数
    this.earLength = earLength;
}

// 继承原型
Rabbit.prototype = Object.create(Animal.prototype);
Rabbit.prototype.constructor = Rabbit;

// 添加子类方法
Rabbit.prototype.jump = function() {
    console.log(`${this.name} jumps!`);
};

// 使用
const rabbit = new Rabbit('White Rabbit', 10);
rabbit.run(5); // "White Rabbit runs with speed 5"
rabbit.jump(); // "White Rabbit jumps!"
console.log(rabbit instanceof Rabbit); // true
console.log(rabbit instanceof Animal); // true
```

## ES6 类与原型的关系

```javascript
// ES6 类
class Person {
    constructor(name) {
        this.name = name;
    }
    
    greet() {
        console.log(`Hello, ${this.name}`);
    }
    
    static create(name) {
        return new Person(name);
    }
}

// 等价的原型写法
function PersonFunc(name) {
    this.name = name;
}

PersonFunc.prototype.greet = function() {
    console.log(`Hello, ${this.name}`);
};

PersonFunc.create = function(name) {
    return new PersonFunc(name);
};

// 验证
const p1 = new Person('John');
const p2 = new PersonFunc('John');

console.log(p1.__proto__ === Person.prototype); // true
console.log(p2.__proto__ === PersonFunc.prototype); // true
console.log(Person.prototype.greet === PersonFunc.prototype.greet); // false（内容相同，但不是同一个函数）
```


## 原型的内存模型

```javascript
// 内存图解示例
function Person(name) {
    this.name = name;
}

Person.prototype.sayHi = function() {
    console.log(`Hi, ${this.name}`);
};

const john = new Person('John');
const jane = new Person('Jane');

/*
内存结构：
john: {
    name: 'John',
    __proto__: → Person.prototype
}

jane: {
    name: 'Jane',
    __proto__: → Person.prototype
}

Person.prototype: {
    sayHi: function() {...},
    constructor: → Person,
    __proto__: → Object.prototype
}
*/
```


## 最佳实践

1. **不要直接修改 `__proto__`**
```javascript
// 不推荐
obj.__proto__ = newProto;

// 推荐
Object.setPrototypeOf(obj, newProto);
```

2. **避免过长的原型链**
```javascript
// 过长的原型链会影响性能
A → B → C → D → E → Object.prototype → null

// 考虑使用组合而不是深度继承
```

3. **使用 `Object.create()` 创建对象**
```javascript
// 清晰的原型继承
const animal = {
    eats: true,
    walk() {
        console.log('Animal walks');
    }
};

const rabbit = Object.create(animal, {
    jumps: {
        value: true,
        enumerable: true
    }
});
```

4. **原型污染防范**
```javascript
// 避免意外修改内置原型
Object.freeze(Object.prototype);
Object.freeze(Array.prototype);
```

