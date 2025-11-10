TypeScript 是 JavaScript 的一个超集，包含了 JavaScript 的所有元素，可以载入 JavaScript 代码运行，并扩展了 JavaScript 的语法。

## 核心特性

1. [类型系统](TypeScript%20类型系统.md) ：TypeScript 增加了静态类型、类、模块、接口和类型注解
2. 类和接口 ：TypeScript 支持面向对象编程，包括类、接口、继承等特性。
3. 泛型 ：泛型允许编写可重用的组件，可以处理多种数据类型，而无需为每种类型单独编写代码。
4. 模块化 ：TypeScript 支持模块化，可以将代码分割成独立的模块，便于组织和管理。


## 与 JavaScript 主要差异

- TypeScript 从核心语言方面和类概念的模塑方面对 JavaScript 对象模型进行扩展。
- JavaScript 代码可以在无需任何修改的情况下与 TypeScript 一同工作，同时可以使用编译器将 TypeScript 代码转换为 JavaScript。
- TypeScript 通过类型注解提供编译时的静态类型检查。
- TypeScript 中的数据要求带有明确的类型，JavaScript不要求。
- TypeScript 为函数提供了缺省参数值。
- TypeScript 引入了 JavaScript 中没有的“类”概念。
- TypeScript 中引入了模块的概念，可以把声明、数据、函数和类封装在模块中。


## 编译

TypeScript 代码（`.ts` 文件）需要通过 TypeScript 编译器（`tsc`）或工具（如 Babel、webpack）编译成 JavaScript（`.js` 文件）才能运行。

```bash
tsc hello.ts  # 生成 hello.js
```

