`this` 是 JavaScript 中最令人困惑但也最强大的特性之一，它的值**取决于调用上下文，而不是定义位置**。

## 绑定规则

1. **默认绑定**：独立函数调用时，`this` 指向全局对象（非严格模式）或 `undefined`（严格模式）。
```javascript
// 非严格模式
function showThis() {
  console.log(this); // 浏览器中：window，Node.js 中：global
}
showThis();

// 严格模式
function strictShowThis() {
  'use strict';
  console.log(this); // undefined
}
strictShowThis();
```

2. **隐式绑定**：方法被对象调用时，`this` 指向调用它的对象。
```javascript
const person = {
  name: 'Alice',
  sayHello() {
    console.log(`Hello, ${this.name}`); // this 指向 person
  }
};

person.sayHello(); // Hello, Alice

// 注意：赋值会丢失 this 绑定
const greet = person.sayHello;
greet(); // Hello, undefined（this 指向全局）
```

3. **显式绑定**：使用 `call()`、`apply()`、`bind()` 明确指定 `this`。
```javascript
function introduce(age, city) {
  console.log(`${this.name}, ${age}岁，来自${city}`);
}

const person = { name: 'Bob' };

// call - 立即执行，参数逐个传递
introduce.call(person, 25, '北京');

// apply - 立即执行，参数以数组传递
introduce.apply(person, [25, '北京']);

// bind - 返回新函数，延迟执行
const boundIntroduce = introduce.bind(person, 25);
boundIntroduce('北京');
```

4. **new 绑定**：构造函数调用时，`this` 指向新创建的实例。
```javascript
function Person(name) {
  this.name = name;
  this.sayName = function() {
    console.log(this.name);
  };
}

const alice = new Person('Alice');
alice.sayName(); // Alice
```

5. **箭头函数**：箭头函数没有自己的 `this`，继承外层作用域的 `this`。
```javascript
const obj = {
  name: 'Object',
  
  regularMethod() {
    setTimeout(function() {
      console.log(this.name); // undefined（this 指向 Timeout 对象）
    }, 100);
    
    setTimeout(() => {
      console.log(this.name); // 'Object'（继承外层 this）
    }, 100);
  }
};

obj.regularMethod();
```


## 优先级规则

绑定优先级：**new 绑定** > **显式绑定** > **隐式绑定** > **默认绑定**

```javascript
function test() {
  console.log(this.name);
}

const obj1 = { name: 'obj1' };
const obj2 = { name: 'obj2' };

// 隐式绑定
obj1.test = test;
obj1.test(); // 'obj1'

// 显式绑定优先级更高
obj1.test.call(obj2); // 'obj2'

// new 绑定优先级最高
const boundTest = test.bind(obj1);
const instance = new boundTest(); // undefined（this 指向新实例）
```

## 特殊场景

1. **事件处理函数**
```javascript
button.addEventListener('click', function() {
  console.log(this); // 指向触发事件的元素
});

button.addEventListener('click', () => {
  console.log(this); // 继承外层 this（通常是 window 或 undefined）
});
```

2. **定时器中的 this**
```javascript
const obj = {
  value: 42,
  
  delayedLog() {
    setTimeout(function() {
      console.log(this.value); // undefined
    }, 100);
    
    setTimeout(() => {
      console.log(this.value); // 42
    }, 100);
    
    // 解决方案1：保存 this
    const self = this;
    setTimeout(function() {
      console.log(self.value); // 42
    }, 100);
    
    // 解决方案2：使用 bind
    setTimeout(function() {
      console.log(this.value); // 42
    }.bind(this), 100);
  }
};
```

3. **类中的 this**
```javascript
class MyClass {
  constructor(value) {
    this.value = value;
  }
  
  // 普通方法 - this 指向实例
  getValue() {
    return this.value;
  }
  
  // 箭头函数作为实例方法
  arrowMethod = () => {
    return this.value; // 始终指向实例
  }
  
  // 静态方法 - this 指向类本身
  static staticMethod() {
    console.log(this === MyClass); // true
  }
}
```


## 常见问题

### 回调函数丢失 `this`

```javascript
class Counter {
  constructor() {
    this.count = 0;
  }
  
  increment() {
    this.count++;
    console.log(this.count);
  }
  
  start() {
    // 错误：this 丢失
    setInterval(this.increment, 1000);
    
    // 解决方案1：箭头函数
    setInterval(() => this.increment(), 1000);
    
    // 解决方案2：bind
    setInterval(this.increment.bind(this), 1000);
    
    // 解决方案3：在构造函数中绑定
    // constructor() {
    //   this.increment = this.increment.bind(this);
    // }
  }
}
```

### 嵌套函数中的 `this`

```javascript
const obj = {
  data: 'important',
  
  process() {
    function helper() {
      console.log(this.data); // undefined（独立的函数调用）
    }
    helper();
    
    // 解决方案1：箭头函数
    const arrowHelper = () => {
      console.log(this.data); // 'important'
    };
    
    // 解决方案2：保存 this 引用
    const self = this;
    function fixedHelper() {
      console.log(self.data); // 'important'
    }
  }
};
```


## `this` 在 Node.js 与浏览器中的核心区别

1. **全局对象不同**
```javascript
// 浏览器中
console.log(this === window); // true（顶级作用域）
console.log(window); // 全局对象

// Node.js 中
console.log(this === global); // false（模块中）
console.log(global); // 全局对象
```

2. **模块作用域的 `this` 指向不同**
```javascript
// 浏览器脚本文件（直接引入）
<script>
  console.log(this); // window
</script>

// Node.js 模块文件
// module.js
console.log(this === module.exports); // true
console.log(this); // {}（空对象）
```

3. **事件循环环境的差异**
```javascript
// 浏览器中 - 定时器的 this
setTimeout(function() {
  console.log(this); // window（非严格模式）
}, 0);

// Node.js 中 - 定时器的 this
setTimeout(function() {
  console.log(this); // Timeout 对象
  console.log(this === global); // false
}, 0);
```

4. **CommonJS vs ES Modules**
```javascript
// CommonJS（Node.js 默认）
exports.name = 'Node';
console.log(this.name); // 'Node'

// ES Module（浏览器/Node.js 都支持）
export const name = 'Module';
// this.name 未定义
```

## [NodeJS 中的 this](../Node/运行原理/NodeJS%20this.md)


## [TypeScript 中的 this](../TypeScript/TypeScript%20this.md)