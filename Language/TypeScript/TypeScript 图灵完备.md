TypeScript 是[图灵完备](../../Operation%20System/图灵完备.md)的，不是指它的运行时（那本来就是 JavaScript，图灵完备毫无悬念），而是指 TypeScript 的“类型系统”本身具备了图灵完备的计算能力，只靠 TypeScript 的类型推断和类型计算，就能完成任意可计算函数，例如实现数学运算、字符串处理，甚至是模拟一个完整的图灵机。

## 关键特性

1. **条件类型（Conditional Types）**：`T extends U ? X : Y`  
	- 相当于类型世界里的 `if/else`，是分支逻辑的基础。
2. **递归类型（Recursive Types）**：  
    - TypeScript 4.1+ 允许类型在条件类型中引用自身，这让循环和迭代成为可能。
3. **映射类型与索引访问**：  
    - 用于构造和变换对象、元组等复杂结构，就像操作数据结构。
4. **模板字面量类型（Template Literal Types）**：  
	- TypeScript 4.1 引入，允许对字符串做模式匹配、拼接、分割等操作，本质上是字符串计算能力。
5. **`infer` 关键字**：  
    - 在条件类型中提取局部类型，相当于“类型解构”或“模式匹配”。

这些组合在一起，就形成了一套“纯类型层面的函数式编程语言”——没有变量、没有 IO，但有递归、分支和数据结构操作，刚好迈过图灵完备的门槛。

## 示例

可以用元组的长度来模拟自然数，然后用递归和条件类型实现加法：
```typescript
// 用一个递归构造的元组表示自然数：数字 N = 长度为 N 的元组
type BuildTuple<N extends number, T extends any[] = []> =
  T['length'] extends N ? T : BuildTuple<N, [...T, any]>;

// 加法 = 两个元组合并后的长度
type Add<A extends number, B extends number> =
  [...BuildTuple<A>, ...BuildTuple<B>]['length'];

type Result = Add<3, 5>; // 类型 Result = 8
```
这里 `Add` 完全在类型层面工作，没有任何运行时代码。类似的，乘除、取模、甚至 FizzBuzz、JSON 解析器、SQL 查询都可以用类型实现。

## 证据：社区在类型系统里都塞了什么

多年来，开发者们不断试探 TypeScript 类型系统的极限，创造了许多“恐怖”的项目：

1. **TS 类型体操仓库**（`type-challenges`）：大量纯粹用类型解决的算法题目。
2. **TS 类型层面的四则运算/逻辑电路/图灵机模拟器**：直接证明图灵完备。
3. **`ts-toolbelt` / `type-fest` 等工具库**：把类型操作变成了函数式工具集。
4. **用类型解析 JSON、字符串正则匹配、甚至写 Brainfuck 解释器**：这些都是图灵完备的强证据。

最著名的一个证明是有人在 TypeScript 类型里实现了一个**通用的图灵机模拟器**，输入状态转移规则和初始纸带，类型系统会输出执行结果。只要 TypeScript 编译器能处理，就等价于承认它是图灵完备的。

## 实践中的限制

虽然**理论上**图灵完备，但 TypeScript 编译器有明确的保护措施：
1. **递归深度限制**：递归类型默认最多展开 50 层左右（可配置），超过会报错 `Type instantiation is excessively deep and possibly infinite`。
2. **计算时间和内存**：复杂类型计算会让编译器严重变慢，甚至卡死。

所以严格来说，TypeScript 类型系统是一个“受限制的”图灵完备系统和真实计算机内存无限但物理上有限的道理一样，它“如果编译器不做限制，就是图灵完备的”。


