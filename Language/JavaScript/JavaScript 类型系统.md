JavaScript 是一个动态的、弱类型的类型系统，变量只需要用 `var` 来声明，允许隐式类型转换，变量的类型在运行时确定，并且可以随时间改变。

在 ES6 之前，JavaScript 中永远都是用 `var` 来定义变量。

## 基本类型

1. **原始类型**：（栈内存）
	- **`undefined`**：表示未定义，变量声明但未赋值时就是 `undefined`。
	- **`null`**：表示空值，注意 `typeof null` 返回 `object`（历史遗留问题）。
	- **`boolean`**：布尔值，`true` 或 `false`。
	- **`number`**：数字，包括整数和浮点数，以及 `NaN`、`Infinity`。
	- **`string`**：字符串。
	- **`symbol`**：符号，ES6 新增，表示唯一的标识符。
	- **`bigint`**：大整数，ES2020 新增，用于表示任意大的整数。

2. **引用类型**：（堆内存存储，栈中存引用）
	- **`object`**：普通的对象，如 `{ key: 'value' }`。
		- **`Date`**：日期
		- **`RegExp`**：正则表达式
	- **`array`**：特殊对象，如 `[1, 2, 3]`。
	- **`function`**：可调用对象。



## 类型判断

1. **`typeof` 运算符**：
```javascript
typeof undefined;   // "undefined"
typeof null;        // "object"（注意，这是历史遗留问题）
typeof true;        // "boolean"
typeof 42;          // "number"
typeof "hello";     // "string"
typeof Symbol();    // "symbol"
typeof 123n;        // "bigint"
typeof {};          // "object"
typeof [];          // "object"（数组也是对象）
typeof function(){};// "function"
```

2. **`instanceof` 运算符**：
```javascript
[] instanceof Array;        // true
[] instanceof Object;       // true
(function() {}) instanceof Function; // true
```

3. **`Object.prototype.toString.call()`**：更准确的类型判断方法
```javascript
Object.prototype.toString.call(undefined);   // "[object Undefined]"
Object.prototype.toString.call(null);        // "[object Null]"
Object.prototype.toString.call(true);        // "[object Boolean]"
Object.prototype.toString.call(42);          // "[object Number]"
Object.prototype.toString.call("hello");     // "[object String]"
Object.prototype.toString.call(Symbol());    // "[object Symbol]"
Object.prototype.toString.call(123n);        // "[object BigInt]"
Object.prototype.toString.call({});          // "[object Object]"
Object.prototype.toString.call([]);          // "[object Array]"
Object.prototype.toString.call(function(){});// "[object Function]"
```

## 类型转换

1. **显式类型转换**
```javascript
// 转换为数字
Number("42");           // 42
Number("42px");         // NaN
parseInt("42px");       // 42
parseFloat("3.14");     // 3.14
+"42";                  // 42 (一元加号运算符)

// 转换为字符串
String(42);             // "42"
(42).toString();        // "42"
(42).toString(2);       // "101010" (二进制)
JSON.stringify({a: 1}); // '{"a":1}'

// 转换为布尔值
Boolean(0);             // false
Boolean(42);            // true
Boolean("");            // false
Boolean("hello");       // true
Boolean(null);          // false
Boolean(undefined);     // false
Boolean([]);            // true (陷阱!)
Boolean({});            // true (陷阱!)
!!"hello";              // true (双非运算符)
```

2. **隐式类型转换规则表**

|操作|规则|示例|
|---|---|---|
|`+` (二元)|有一方是字符串则字符串连接|`1 + "2" = "12"`|
|`-`, `*`, `/`|转换为数字|`"3" * 2 = 6`|
|`==`, `!=`|类型转换后比较|`"5" == 5` → true|
|`===`, `!==`|严格比较，不转换类型|`"5" === 5` → false|
|`!`, `!!`|转换为布尔值|`!!"hello"` → true|
|`if`, `while` 条件|转换为布尔值|`if("")` → false|

3. **对象到原始值的转换**
```javascript
// 对象到原始值的转换过程
const obj = {
    value: 42,
    // Symbol.toPrimitive 方法 (ES6+)
    [Symbol.toPrimitive](hint) {
        if (hint === 'number') return this.value;
        if (hint === 'string') return `Value: ${this.value}`;
        return this.value; // default
    },
    // toString 方法
    toString() {
        return `[Object ${this.value}]`;
    },
    // valueOf 方法
    valueOf() {
        return this.value;
    }
};

console.log(Number(obj));      // 42 (调用valueOf)
console.log(String(obj));      // "Value: 42" (调用toString)
console.log(obj + 10);         // 52 (hint为default)
```


## 原始类型

1. **Number 类型**
```javascript
// 数字表示方式
const decimal = 42;
const binary = 0b101010;   // 二进制
const octal = 0o52;        // 八进制
const hex = 0x2A;          // 十六进制
const scientific = 4.2e1;  // 科学计数法

// 特殊数值
console.log(Number.MAX_VALUE);      // 1.7976931348623157e+308
console.log(Number.MIN_VALUE);      // 5e-324
console.log(Number.MAX_SAFE_INTEGER); // 9007199254740991
console.log(Number.MIN_SAFE_INTEGER); // -9007199254740991
console.log(Number.POSITIVE_INFINITY); // Infinity
console.log(Number.NEGATIVE_INFINITY); // -Infinity
console.log(Number.EPSILON);        // 2.220446049250313e-16

// NaN (Not a Number)
console.log(NaN === NaN);          // false (唯一不等于自身的值)
console.log(Number.isNaN(NaN));    // true
console.log(isNaN("hello"));       // true (全局函数会进行类型转换)
console.log(Number.isNaN("hello")); // false
```

2. **String 类型**
```javascript
// 字符串创建
const str1 = "hello";
const str2 = 'world';
const str3 = `template ${str1}`; // 模板字符串

// 字符串不可变性
let str = "hello";
str[0] = "H"; // 无效，字符串不可变
str = "H" + str.slice(1); // 正确方式

// UTF-16编码
console.log("😀".length);          // 2 (表情符号占2个码元)
console.log("𝄞".length);          // 2 (某些字符也占2个码元)
console.log(Array.from("😀").length); // 1 (正确方式)
```

3. **Boolean 类型**
```javascript
// 假值（falsy values）
const falsyValues = [
    false,      // false
    0, -0,      // 零
    0n,         // BigInt的0
    "", '', ``  // 空字符串
    null,       // null
    undefined,  // undefined
    NaN         // NaN
];

// 所有其他值都是真值（truthy）
const truthyValues = [
    true,
    "false",    // 非空字符串
    "0",        // 非空字符串
    [],         // 数组(真值)
    {},         // 对象(真值)
    function(){},
    Infinity,
    -Infinity
];
```

4. **null 与 undefined**
```javascript
// undefined
let x;
console.log(x);               // undefined
console.log(typeof x);        // "undefined"
console.log(void 0);          // undefined (void运算符)

// null
let y = null;
console.log(typeof y);        // "object" (历史bug)

// 区别
console.log(null == undefined);   // true (宽松相等)
console.log(null === undefined);  // false (严格相等)
console.log(!null);               // true
console.log(!undefined);          // true

// 使用场景
function foo(param) {
    // param可能为undefined
    return param ?? 'default'; // nullish coalescing
}
```

5. **Symbol 类型（ES6+）**
```javascript
// 创建Symbol
const sym1 = Symbol();
const sym2 = Symbol("description");
const sym3 = Symbol("description");

console.log(sym2 === sym3); // false (每个Symbol都是唯一的)
console.log(sym2.toString()); // "Symbol(description)"

// 全局Symbol注册表
const globalSym1 = Symbol.for("key"); // 创建或获取
const globalSym2 = Symbol.for("key");
console.log(globalSym1 === globalSym2); // true

// Symbol.keyFor获取描述
console.log(Symbol.keyFor(globalSym1)); // "key"

// 内置Symbol
const obj = {
    [Symbol.iterator]() { /* 迭代器 */ },
    [Symbol.toStringTag]: "MyObject",
    [Symbol.toPrimitive](hint) { /* 类型转换 */ }
};
```

6. **BigInt 类型（ES2020+）**
```javascript
// 创建BigInt
const big1 = 9007199254740991n;   // 字面量
const big2 = BigInt("9007199254740991");
const big3 = BigInt(Number.MAX_SAFE_INTEGER);

// 运算
console.log(big1 + 1n);           // 9007199254740992n
console.log(big1 * 2n);           // 18014398509481982n

// 注意事项
console.log(typeof 1n);           // "bigint"
console.log(1n == 1);             // true (宽松相等)
console.log(1n === 1);            // false (严格相等)
console.log(1n + 2n);             // 3n
// console.log(1n + 2);           // TypeError: 不能混合类型
```

## 引用类型

1. **Object 基础**
```javascript
// 对象创建
const obj1 = {};                          // 对象字面量
const obj2 = new Object();                // 构造函数
const obj3 = Object.create(null);         // 无原型对象
const obj4 = Object.create({ proto: 1 }); // 指定原型

// 属性描述符
const obj = {};
Object.defineProperty(obj, 'key', {
    value: 42,
    writable: false,      // 不可写
    enumerable: true,     // 可枚举
    configurable: false   // 不可配置
});

// 获取属性描述符
console.log(Object.getOwnPropertyDescriptor(obj, 'key'));

// 密封和冻结
const sealed = Object.seal({ x: 1 });    // 不能添加/删除属性，可修改
const frozen = Object.freeze({ x: 1 });  // 完全不可变
```

2. **原型链系统**
```javascript
// 构造函数和原型
function Animal(name) {
    this.name = name;
}

Animal.prototype.speak = function() {
    console.log(`${this.name} makes a noise`);
};

function Dog(name) {
    Animal.call(this, name); // 调用父类构造函数
}

// 设置原型链
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.speak = function() {
    console.log(`${this.name} barks`);
};

const dog = new Dog("Rex");
dog.speak(); // "Rex barks"

// 原型链查找
console.log(dog.__proto__ === Dog.prototype);           // true
console.log(Dog.prototype.__proto__ === Animal.prototype); // true
console.log(Animal.prototype.__proto__ === Object.prototype); // true
console.log(Object.prototype.__proto__ === null);       // true
```


## 内部机制

1. **执行上下文与类型**
```javascript
// 变量环境与词法环境
function test() {
    console.log(typeof x);    // "undefined" (暂时性死区前)
    // console.log(x);        // ReferenceError
    let x = 42;
    
    console.log(typeof y);    // "undefined" (变量提升)
    var y = "hello";
}

// 作用域链与类型查找
const globalValue = "global";
function outer() {
    const outerValue = "outer";
    
    function inner() {
        const innerValue = "inner";
        console.log(typeof innerValue); // "string"
        console.log(typeof outerValue); // "string"
        console.log(typeof globalValue); // "string"
    }
    inner();
}
outer();
```

2. **内存管理中的类型**
```javascript
// 原始类型：栈内存
let a = 42;      // 栈内存存储
let b = a;       // 值拷贝
b = 100;         // 不影响a
console.log(a);  // 42

// 引用类型：堆内存
let obj1 = { x: 1 };  // 堆内存存储，栈中存引用
let obj2 = obj1;      // 引用拷贝
obj2.x = 2;           // 影响obj1
console.log(obj1.x);  // 2

// 垃圾回收与类型
function createLargeObject() {
    const largeObj = new Array(1000000).fill("data");
    return function() {
        // largeObj可能被回收，因为闭包不再引用它
        console.log("Hello");
    };
}
```