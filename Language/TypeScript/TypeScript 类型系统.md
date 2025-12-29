
## 类型系统


### 基础类型

TypeScript 支持 JavaScript 的所有基础类型，并扩展了额外类型：

1. **原始类型**：
	- `number`
	- `string`
	- `boolean`
	- `null`
	- `undefined`
	- `symbol`
	- `bigint`
2. **复杂类型**：
	- `object`
	- `array`
	- `function`
3. **TypeScript 特有类型**：
    - `any`：禁用类型检查（慎用）。
    - `unknown`：类型安全的 `any`，使用时需显式断言。
    - `void`：表示函数没有返回值。
    - `never`：表示永远不会返回（如抛出错误的函数）。

#### 静态类型检查

1. **编译时类型检查**：TypeScript 在编译阶段（而非运行时）检查类型错误。

#### 类型注解与推断

1. **显式注解**：手动声明变量类型。
```TypeScript
let name: string = "Alice";
```

2. **类型推断**：TypeScript 自动推断类型（如根据初始值）。
```TypeScript
let age = 30; // 自动推断为 number
```


### 联合类型

**联合类型**（`|`）表示值可以是多种类型之一。
```TypeScript
let id: number | string;
```


### 交叉类型

**交叉类型**（`&`）用于合并多个类型的属性。
```TypeScript
type Person = { name: string };
type Employee = { id: number };
type Staff = Person & Employee; // { name: string, id: number }
```

### 接口

**接口**（Interface）用于定义对象的结构，支持继承和扩展。
```TypeScript
interface User {
    name: string;
    age?: number; // 可选属性
}
```

### 类型别名

**类型别名**（Type Alias）用于为类型命名，支持更复杂的类型组合。
```TypeScript
type Point = { x: number; y: number };
```

### 泛型

**泛型**（Generics）提供代码复用性，允许在定义函数、类或接口时使用类型参数。
```TypeScript
function identity<T>(arg: T): T {
    return arg;
}
let output = identity<string>("hello");
```

### 高级类型

1. **条件类型**：
```typescript
T extends U ? X : Y
```

2. **映射类型**：基于旧类型创建新类型
```typescript
Readonly<T>
```

3. **模板字面量类型**：结合字符串字面量类型
```typescript
type Event = "click" | "hover"
```


### 类型守卫

**类型守卫**用于运行时检查类型（如 `typeof`，`instanceof`，或自定义函数）。
```TypeScript
if (typeof val === "string") {
    console.log(val.toUpperCase());
}
```

### 类型断言

**类型断言**用于手动指定类型（避免滥用）。
```TypeScript
let input = document.getElementById("input") as HTMLInputElement;
```

### 类型兼容性

TypeScript 使用**结构化类型系统**（鸭子类型），只要结构匹配即视为兼容：
```TypeScript
interface Cat { name: string; }
let pet: Cat = { name: "Whiskers", age: 2 }; // 兼容，因为包含 name
```

> [!鸭子系统]
> **鸭子系统**（**Duck Typing**）源自**鸭子测试**（**Duck Test**）的哲学思想：
> 
> "If it walks like a duck and it quacks like a duck, then it must be a duck. 如果它走起路来像鸭子，叫起来像鸭子，那么它就是鸭子。"
> 
> 鸭子类型意味着类型兼容性不是由明确的继承关系决定的，而是由对象实际具有的属性和方法决定的。


## `@types`


TypeScript 作为一个有类型的语言，在 `.ts` 文件引用包时，默认时必须要有类型声明的，不能是 `any`，而 TypeScript 对于包的类型声明要求为提供 `.d.ts`，否则就要在 `tsconfig.json` 中声明。

TypeScript 对于包/模块的**声明寻找规则**如下：
1. TypeScript 编译器先在当前编译上下文找模块的定义
2. 如果找不到，则会去 `node_modules` 中的 `@types`（默认情况，目录可以修改，后面会提到）目录下去寻找对应包名的模块声明文件

使用 TypeScript 时，要注意添加的是哪种类型的依赖。因为 TypeScript 是一个开发工具，而且TypeScript 类型不存在于运行时，**与 TypeScript 相关的包一般属于 `devDependencies`** 。


### `extends`

1. `extends` 关键字可用于 `class` 的继承
2. `extends` 关键字可以实现 `interface` 类型的扩展
```typescript
interface Animal {
    name: string
}
​
interface Person extends Animal {
    level: number
}
```

3. `extends` 实现类型约束
```typescript
type MyPick<T, Keys extends keyof T> = {
    [key in Keys]: T[key]
}
```

4. `extends` 实现条件类型判断
```typescript
type MyExclude<T, Key> =  T extends Key ? never : T
```



## 实用类型

TypeScript 提供了一系列内置的**实用类型**（Utility Types），它们是预定义的类型操作工具，用于对现有类型进行转换、组合或操作，从而简化复杂类型的定义，可以帮助开发者更灵活地操作和转换现有类型，而无需重复编写复杂的类型定义。

### 核心实用类型

#### 对象操作类型

1. **`Partial<T>`** ：使 `T` 的所有属性变为可选
2. **`Required<T>`** ：使 `T` 的所有属性变为必需
3. **`Readonly<T>`** ：使 `T` 的所有属性变为只读
4. **`Pick<T, K>`** ：从 `T` 中选取指定属性 `K`
5. **`Omit<T, K>`** ：从 `T` 中排除指定属性 `K`
6. **`Record<K, T>`** ：创建键为 `K`，值为 `T` 的类型

#### 联合类型操作

1. **`Exclude<T, U>`** ：从 `T` 中排除可赋值给 `U` 的类型
2. **`Extract<T, U>`** ：从 `T` 中提取可赋值给 `U` 的类型
3. **`NonNullable<T>`** ：从 `T` 中排除 `null` 和 `undefined`

#### 函数操作类型

1. **`Parameters<T>`** ：获取函数参数类型元组
2. **`ReturnType<T>`** ：获取函数返回值类型
3. **`ConstructorParameters<T>`** ：获取构造函数参数类型
4. **`InstanceType<T>`** ：获取构造函数实例类型

### 高级实用类型

1. **条件类型**
```TypeScript
type NonFunctionKeys<T> = {
    [K in keyof T]: T[K] extends Function ? never : K
} [keyof T];
```

2. **递归类型**
```TypeScript
type DeepPartial<T> = {
    [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};
```

3. **模板字面量类型**
```TypeScript
type EventName<T extends string> = `${T}Changed`;
type Concat<A extends string, B extends string> = `${A}${B}`;
```


### 使用

#### `Exclude<T, U>`

`Exclude` 是 TypeScript 中的一个内置实用类型（Utility Type），用于从**联合类型**中**排除**某些指定的类型。

`Exclude<T, U>` 会从类型 `T` 中排除所有可以赋值给 `U` 的类型，返回剩余的类型组成的新类型：
```TypeScript
type T0 = Exclude<"a" | "b" | "c", "a">;  // "b" | "c"
type T1 = Exclude<"a" | "b" | "c", "a" | "b">;  // "c"
type T2 = Exclude<string | number | (() => void), Function>;  // string | number
type T3 = Exclude<"a" | "b" | "c", "a" | "d">;  // "b" | "c" ("d"不存在于原类型中，被忽略)
```

##### 实现原理

```TypeScript
type Exclude<T, U> = T extends U ? never : T;
```


##### 使用场景

1. 过滤掉不需要的类型
2. 创建更严格的类型约束
3. 与其他实用类型结合使用（如 `Extract`、`Omit` 等）


#### `Extract<T, U>`

`Extract` 是 TypeScript 中的一个内置实用类型（Utility Type），用于从**联合类型**中**提取**与指定类型匹配的类型成员。

`Extract<T, U>` 会从类型 `T` 中提取所有可以赋值给 `U` 的类型，返回这些类型的联合：
```TypeScript
type T0 = Extract<"a" | "b" | "c", "a">;  // "a"
type T1 = Extract<"a" | "b" | "c", "a" | "b">;  // "a" | "b"
type T2 = Extract<string | number | (() => void), Function>;  // () => void
type T3 = Extract<"a" | "b" | "c", "a" | "d">;  // "a"（"d"不存在于原类型中，被忽略）
```

##### 实现原理

```TypeScript
type Extract<T, U> = T extends U ? T : never;
```

##### 使用场景

1. 从联合类型中筛选出特定类型的成员
2. 获取两个类型的交集
3. 与其他实用类型结合使用


#### `Omit<T, K>`

`Omit` 是 TypeScript 中的一个内置实用类型（Utility Type），用于从**对象类型**中**排除**指定的属性，并返回一个新的对象类型。

`Omit<T, K>` 会创建一个新类型，这个新类型包含 `T` 类型中除了 `K` 指定的属性之外的所有属性：
```Typescript
interface User {
    id: number;
    name: string;
    age: number;
    email: string;
}

// 排除 'age' 属性
type UserWithoutAge = Omit<User, 'age'>;
/* 等同于：
{
    id: number;
    name: string;
    email: string;
}
*/

// 排除多个属性
type SimpleUser = Omit<User, 'age' | 'email'>;
/* 等同于：
{
    id: number;
    name: string;
}
*/

// 排除不存在的属性（会被忽略）
type UserTest = Omit<User, 'address'>; // 原样保留所有属性
```
##### 实现原理

```TypeScript
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

##### 使用场景

1. 创建部分对象类型（排除敏感或不必要的字段）
2. 在继承或组合类型时精简接口
3. 处理 API 响应时选择需要的字段


#### `Pick<T, K>`

`Pick` 是 TypeScript 中的一个内置实用类型（Utility Type），用于从**对象类型**中**选择**指定的属性，创建一个新的对象类型。

`Pick<T, K>` 会从类型 `T` 中挑选出 `K` 指定的属性，返回这些属性组成的新类型：
```TypeScript
interface User {
    id: number;
    name: string;
    age: number;
    email: string;
}

// 选择 'id' 和 'name' 属性
type UserBasicInfo = Pick<User, 'id' | 'name'>;
/* 等同于：
{
    id: number;
    name: string;
}
*/

// 选择单个属性
type UserId = Pick<User, 'id'>;  // { id: number }

// 尝试选择不存在的属性（会报错）
type UserTest = Pick<User, 'address'>;  // 错误：'address' 不在类型 'User' 中
```

##### 实现原理

```TypeScript
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};
```

##### 使用场景

1. 创建对象类型的子集
2. 定义需要部分属性的函数参数
3. 与其它实用类型结合使用