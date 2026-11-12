JavaScript 主要被作为客户端脚本语言在用户的浏览器上运行，不需要服务器的支持。可以嵌入HTML中，放在 `<body>` 标签内的 `<script>` 代码在页面载入时就会执行，且是按代码顺序（包括html、css代码）加载执行的。除非这段代码被定义在一个函数function里面，这个时候函数function被调用的时候才执行。

前端界面的组成有三个重要组成部分：
- HTML：从语义的角度，描述页面结构
- CSS：从审美的角度，描述样式（美化页面）
- JavaScript：从交互的角度，描述行为（提升用户体验）
JavaScript 是世界上用的最多的脚本语言，ECMAScript 在 2015 年 6 月，发布了 ECMAScript 6 版本语法标准。

一般来说，完整的 JavaScript 包括以下几个部分：
- ECMAScript，描述了该语言的语法和基本对象；
- 文档对象模型（DOM），描述处理网页内容的方法和接口；
- 浏览器对象模型（BOM），描述与浏览器进行交互的方法和接口。

JavaScript 常用来完成以下任务：
- 嵌入动态文本于 HTML 页面；
- 对浏览器事件作出响应；
- 读写 HTML 元素；
- 在数据被提交到服务器之前验证数据；
- 检测访客的浏览器信息；
- 控制 Cookie，包括创建和修改等；


## 动态语言

JavaScript 的**很多语义被推迟到运行时决定，并且程序运行中可以改变结构/行为**，因此被称为“动态语言”。

JavaScript 被称为“动态语言”，核心不是“语法随便”，而是**很多语义被推迟到运行时决定，并且程序运行中可以改变结构/行为**。主要体现在这些方面：

1. **动态类型**：变量没有固定类型，值才有类型
```js
let x = 1;      // number
x = "hello";    // 运行时变成 string
x = { a: 1 };   // 又变成 object
```
JS 变量本身不携带固定类型标注，类型在运行时由当前值决定；还伴随隐式类型转换：
```js
"1" + 1   // "11"
"1" - 1   // 0
```

注意：**动态类型 ≠ 弱类型**，但 JS 通常也被认为有较弱/隐式转换倾向。

 2. **对象结构是动态的**：属性可随时增删改
```js
const obj = {};
obj.name = "Tom";          // 运行时加属性
delete obj.name;           // 运行时删属性
obj[`key${1}`] = 123;      // 计算属性名
Object.defineProperty(obj, "x", { value: 1 });
```
很多静态语言对象形状在编译期固定；JS 对象可在运行时改变形状，这也是 JS 引擎要做隐藏类、内联缓存、去优化等优化。

3. **原型链可在运行时被修改**
```js
const obj = {};
Object.getPrototypeOf(obj); // 运行时查原型
obj.__proto__ = { hello() { return "hi"; } };
obj.hello(); // "hi"
```
属性查找沿原型链在运行时进行；甚至可以修改内置原型，比如给 `Array.prototype` 加方法，影响所有数组。

4. **`this` 绑定是运行时决定的**
```js
function f() { return this; }
const a = { f };
a.f();        // this 是 a
const g = a.f;
g();          // 非严格模式下 this 是全局对象/undefined（严格模式）
```
同一个函数，调用方式不同，`this` 不同。箭头函数例外，它捕获词法 `this`。

5. **函数是一等公民**，可运行时创建、传递、调用
```js
const add = (a, b) => a + b;
function makeAdder(x) {
  return y => x + y; // 运行时形成闭包
}
```
函数可以赋值给变量、作为参数、作为返回值，闭包捕获运行时环境。

6. **可以在运行时执行代码**
```js
eval("1 + 2");
new Function("a", "b", "return a + b");
```
`eval` / `new Function` 能把字符串当代码执行。虽然不推荐、有限制和安全风险，但它体现了“代码也可在运行时生成/求值”。

7. **反射与拦截：`Proxy` / `Reflect`**
```js
const p = new Proxy({}, {
  get(target, key) {
    return key in target ? target[key] : "default";
  }
});
p.foo; // "default"
```
对象的读取、赋值、删除、`in` 等操作可在运行时被拦截和改写。

8. **动态模块导入**
```js
const mod = await import(`./locale/${lang}.js`);
```
模块路径和加载时机可以运行时决定。

9. **错误和行为多在运行时暴露**
```js
function f(o) { return o.x.y; }
f({}); // 运行时才报错
```
没有编译期类型检查时，很多错误要到执行到那一行才出现。TypeScript 是在编译期加静态检查，但编译产物仍是动态的 JS。

10. **全局环境和内置对象可被“猴子补丁”**
```js
Array.prototype.last = function () {
  return this[this.length - 1];
};
```
运行中可修改内置对象行为，这在静态语言里通常很难或不允许。

> [!tip] 猴子补丁（monkey patch）
> **猴子补丁（monkey patch）**是指在**程序运行时**动态修改、替换或扩展已有的对象、函数、类、模块、原型等行为，而**不修改原始源码**。它依赖语言的动态性，JavaScript 因为对象和原型可在运行时改，所以很容易做。
> 
> 术语来源没有完全统一的定论，常见说法是“monkey patch”带有**运行时乱改、像猴子一样东补西补**的意味；也有说法与早期 Python/Zope 社区有关。重点不是词源，而是“运行中打补丁”。

## 灵活性

JavaScript 的灵活性是“原型链 + 动态属性 + 动态类型 + 一等函数 + 属性描述符/Proxy”等共同赋予的。

JavaScript 的灵活性可以从以下几个主要方面体现：

### 1. 动态类型与运行时行为
变量类型在运行时才确定，且可以随时改变：

```js
let x = 1;
x = "hello";
x = { a: 1 };
```

同一变量可保存不同类型的值，函数参数也不限制类型。这带来开发便利，但也容易产生隐式类型转换和运行时错误。

### 2. 函数是一等公民
函数可以赋值给变量、作为参数传递、作为返回值、动态创建：

```js
const add = (a, b) => a + b;
const apply = (fn, x, y) => fn(x, y);
apply(add, 1, 2); // 3
```

同时支持闭包，函数可以捕获并保持外部作用域变量：

```js
function counter() {
  let count = 0;
  return () => ++count;
}
```

### 3. 基于原型的动态对象模型
对象可以在运行时增删属性，也可以通过原型链继承和扩展：

```js
const obj = {};
obj.name = "JS";
obj.say = function () { return this.name; };
```

还可以修改原型、动态扩展对象，甚至给内置对象原型添加方法，虽然实际项目中应谨慎。

### 4. 支持多种编程范式
JavaScript 同时支持：

- 面向过程
- 面向对象
- 函数式编程
- 事件驱动
- 异步编程

例如数组方法 `map/filter/reduce` 体现函数式风格，`class` 和原型体现面向对象风格。

### 5. 元编程能力强
可以在运行时检查和修改程序结构：

- `eval`
- `Function` 构造器
- `Proxy` / `Reflect`
- 动态属性访问 `obj[key]`
- 修改对象属性描述符
- 动态创建和调用函数

```js
const obj = new Proxy({}, {
  get(target, key) {
    return key in target ? target[key] : "default";
  }
});
```

### 6. 异步与事件驱动模型灵活
JavaScript 支持回调、Promise、async/await、事件监听、定时器等多种异步组织方式：

```js
setTimeout(() => console.log("later"), 1000);
fetch("/api").then(r => r.json()).then(console.log);
```

同一个异步任务可以用不同风格表达，适合浏览器事件、网络请求、Node.js I/O 等场景。

### 7. 语法和 API 较宽松
JavaScript 有不少“宽松”设计：

- 自动类型转换
- 可选参数、默认参数、剩余参数
- 解构赋值
- 模板字符串
- 可选链 `?.`
- 空值合并 `??`
- 对象和数组字面量
- 动态属性名

例如：

```js
const { name = "anonymous" } = user ?? {};
console.log(user?.profile?.age);
```

这些特性让表达更简洁，但也要求开发者注意边界。

### 8. `this`、作用域和模块机制灵活
JavaScript 中 `this` 的绑定方式多样：默认绑定、隐式绑定、显式绑定、`new` 绑定、箭头函数词法绑定。模块方面也有 CommonJS、ES Module、动态 `import()` 等形式。

### 9. 跨平台与生态扩展性强
同一门语言可运行在：

- 浏览器
- Node.js
- 移动端 / 桌面端
- 嵌入式 / 边缘运行时
- WebAssembly 周边生态

并通过 npm 生态、框架、转译器、打包工具扩展能力。

### 总结
JavaScript 的灵活性主要体现在：**动态类型、函数一等公民、原型对象模型、多范式、元编程、异步事件驱动、宽松语法、跨平台生态**。这种灵活性提高了表达力和开发效率，但也带来类型安全、可维护性和隐式行为方面的挑战，所以大型项目常配合 TypeScript、ESLint、测试和规范来约束。