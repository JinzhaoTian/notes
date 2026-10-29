`Object` 是 ECMAScript 中最基础的标准内置对象，它既是 JavaScript 的一种数据类型，也是所有普通对象的**构造函数**，其核心作用是**存储键值对集合**以及**构建更复杂的实体**。

在 JavaScript 中，**几乎所有对象都是 `Object` 的实例**。一个典型对象会通过原型链继承 `Object.prototype` 上的属性和方法（这些属性可能被对象自身的同名属性覆盖）。唯一不继承自 `Object.prototype` 的对象，是那些以 `null` 为原型或派生自其他 `null` 原型对象的对象。

`Object` 构造函数是全局对象（Global Object）的 "Object" 属性的初始值。

> [!caution] ECMAScript 执行环境中有一个全局对象（Global Object）
> 全局对象是 ECMAScript 执行环境中的顶层对象（在浏览器中 window/globalThis，在 Node 中 global 等），它拥有许多内置属性，如 Object、Function、Array、String 等。
> 
> > [!quote] The Object constructor is the initial value of the "Object" property of the global object.
> 
> 全局对象有一个名为 "Object" 的属性，其初始值就是 `Object` 构造函数。在 ECMAScript 初始化时，全局对象的 `"Object"` 属性被绑定到内置的 `Object` 构造函数，即：
> ```javascript
> globalThis.Object === Object; // true
> ```
> 
> 右侧的 `Object` 标识符在全局作用域中解析时，会找到全局对象的 `"Object"` 属性，而该属性的值就是 `Object` 构造函数。

> [!questionj] 如何理解 "值" 是 "构造函数" ?
> 在 ECMAScript 里，构造函数首先是一个**函数对象**，而**函数对象是对象的一种**，因此它可以作为属性值保存。
> 
> ECMAScript 的语言类型包括 `Undefined`、`Null`、`Boolean`、`String`、`Symbol`、`BigInt`、`Number`和 `Object`，而函数属于 `Object` 类型。更精确地说，函数对象是**可调用对象（callable object）**：它首先是一个对象，然后额外具有 `[[Call]]` 内部方法，因而可以被调用；如果还具有 `[[Construct]]`，还可以被 `new` 调用。

## `[[Prototype]]`

`Object` 的 `[[Prototype]]` 内部槽（Internal Slot）指向 `Function.prototype`，并且自身具有一个 `length` 属性，值为 `1`。

> [!info] 内部槽（Internal Slot）
> 内部槽（Internal Slot）不对应任何可通过代码直接访问的字符串属性名，其名称用双方括号 `[[ ]]` 标记，表示这是规范内部使用的抽象机制，而非语言层面的真实属性。

> [!caution] 所有普通对象（Ordinary Object）都拥有一个名为 `[[Prototype]]` 的内部槽
> 普通对象（Ordinary Object）的 `[[Prototype]]` 内部槽的值只有两种可能：**`null`** 或 **另一个对象**。

`[[Prototype]]` 被用来实现 JavaScript 的继承机制：当你尝试读取对象 `obj` 上的一个属性时，引擎首先检查 `obj` 自身是否拥有该属性；如果没有，则沿着 `obj.[[Prototype]]` 指向的对象继续查找，再沿着那个对象的 `[[Prototype]]` 查找，依此类推。这条由 `[[Prototype]]` 链接构成的链条就是[原型链](JavaScript%20原型链.md)。当一个对象的 `[[Prototype]]` 为 `null` 时，意味着该对象不继承任何其他对象的属性，这通常是原型链的终点（`Object.prototype` 自身的 `[[Prototype]]` 就是 `null`）。

### 与 `__proto__`、`prototype` 的区别

这三个概念容易混淆，但含义截然不同：

1. **`[[Prototype]]`**：这是**规范层面的内部槽**，是所有对象实际拥有的原型引用。
2. **`__proto__`**：这是一个**历史遗留的访问器属性**（定义在 `Object.prototype` 上）。通过 `obj.__proto__` 读写，实际上会间接调用 `Object.getPrototypeOf(obj)` 和 `Object.setPrototypeOf(obj, ...)`。它并非标准早期就有的机制，但被广泛实现，如今在规范中作为 Annex B 的遗留特性存在。直接操作 `__proto__` 已被认为是不推荐的实践。
3. **`prototype`**：这是**函数对象才拥有的普通属性**。当你使用 `new Fn()` 创建实例时，新实例的 `[[Prototype]]` 会被设置为 `Fn.prototype` 所指向的对象。`prototype` 是函数的一个可写属性，而 `[[Prototype]]` 是每个对象都有的内部槽，二者不在同一个层面上。

## `Object.prototype`

`Object.prototype` 是原型链的顶端（其自身的原型始终为 `null`，且不可更改），这意味着对它的修改会影响所有继承自它的对象，因此存在“原型污染”的风险。现代 JavaScript 开发中，**操作对象应优先使用静态方法**，而非直接调用 `Object.prototype` 上的实例方法。

`Object.prototype` 上定义了一些基础实例方法，其中 `valueOf()`、`toString()` 和 `toLocaleString()` 被设计为可多态的，对象通常会提供自己的实现；而 `__defineGetter__` 等遗留方法已被弃用，应改用对应的静态方法。


## 静态方法

`Object` 构造函数本身提供了一系列静态方法，用于执行对象的通用操作。现代 JavaScript 中与对象相关的实用工具基本都是静态的。以下按功能分类列举其中的核心方法：

**对象创建与原型操作**

- **`Object.create(proto, [propertiesObject])`**：创建一个新对象，并使用指定的原型对象和属性描述符。
- **`Object.getPrototypeOf(obj)`**：返回指定对象的原型。
- **`Object.setPrototypeOf(obj, proto)`**：设置指定对象的原型（由于性能影响，通常建议使用 `Object.create()`）。


**属性描述符与元编程**

- **`Object.defineProperty(obj, prop, descriptor)`**：在对象上定义或修改一个属性，并精确控制其行为（如 `writable`、`enumerable`、`configurable`、`get`/`set`）。
- **`Object.defineProperties(obj, props)`**：批量定义或修改多个属性。
- **`Object.getOwnPropertyDescriptor(obj, prop)`**：获取指定对象自有属性的描述符。
- **`Object.getOwnPropertyDescriptors(obj)`**：获取对象所有自有属性的描述符。


**属性枚举与检查**

- **`Object.keys(obj)` / `Object.values(obj)` / `Object.entries(obj)`**：分别返回对象**自有可枚举属性**的键、值或键值对数组。
- **`Object.getOwnPropertyNames(obj)`**：返回对象所有自有属性（包括不可枚举属性）的名称数组。
- **`Object.hasOwn(obj, prop)`**：判断属性是否为对象的**自有属性**（替代 `Object.prototype.hasOwnProperty.call()`）。


**对象状态控制**

- **`Object.freeze(obj)` / `Object.isFrozen(obj)`**：冻结对象（禁止添加、删除、修改属性）及判断冻结状态。
- **`Object.seal(obj)` / `Object.isSealed(obj)`**：密封对象（禁止添加、删除属性，但可修改现有属性值）及判断密封状态。
- **`Object.preventExtensions(obj)` / `Object.isExtensible(obj)`**：禁止对象扩展（禁止添加新属性）及判断可扩展性。


**对象合并与复制**

- **`Object.assign(target, ...sources)`**：将所有**可枚举的自有属性**的值从一个或多个源对象复制到目标对象，返回目标对象。注意这是**浅复制**。