JavaScript 和 TypeScript 中的字符串本质上是相同的，因为 TypeScript 是 JavaScript 的超集，它们在运行时都是 JavaScript 的字符串表示。

## 原理

1. **UTF-16 编码**
```typescript
// JavaScript使用UTF-16编码，每个字符占用1-2个16位码元
const str = "Hello 世界 😀";

// 获取字符串长度（码元数量）
console.log(str.length); // 11（注意：表情符号占2个码元）

// 遍历字符（按码元）
for (let i = 0; i < str.length; i++) {
  console.log(str[i], str.charCodeAt(i).toString(16));
}

// 正确遍历所有字符（考虑代理对）
for (let char of str) {
  console.log(char);
}

// 获取码点（考虑代理对）
console.log(str.codePointAt(7)); // 19990 (世)
```

2. **内存结构**
```typescript
// 字符串在内存中的表示
const text = "AB🎯";

// 内存布局（近似）：
// 索引: 0   1   2   3
// 字符: A   B   🎯
// 码元: 0x0041 0x0042 0xD83C 0xDFAF
// 实际：A(1码元) B(1码元) 🎯(2码元)

console.log(text.length); // 4 (不是3!)
```

3. **字符串不可变性**
```typescript
// JavaScript字符串是不可变的
let immutable = "hello";
immutable[0] = "H";  // 静默失败或严格模式下报错

// 任何修改都返回新字符串
let newStr = immutable.replace("h", "H");
console.log(immutable);  // "hello" (未改变)
console.log(newStr);     // "Hello" (新字符串)
```

## 使用

### 创建

1. **三种引号表示法**
```typescript
// 1. 双引号
let str1: string = "Hello World";

// 2. 单引号  
let str2: string = 'Hello World';

// 3. 反引号（模板字符串）
let str3: string = `Hello World`;

// 这三种在运行时表现相同
console.log(str1 === str2); // true
console.log(str2 === str3); // false（不同实例，但值相同）
```

2. **转义字符**
```typescript
// 常见转义序列
const examples = {
  newline: "Line1\nLine2",        // 换行
  tab: "Col1\tCol2",              // 制表符
  backslash: "Path: C:\\Windows", // 反斜杠
  quote: 'He said, "Hello"',      // 引号
  unicode: "\u0041",              // Unicode (A)
  hex: "\x41",                    // 十六进制 (A)
  es6unicode: "\u{1F600}"         // ES6 Unicode (😀)
};

// 原始字符串（ES2021+）
const raw = String.raw`C:\Users\Name\nNew Line`;
console.log(raw); // C:\Users\Name\nNew Line （\n不会被转义）
```

### 操作

1. **拼接**
```typescript
// 字符串连接
const concat1 = "Hello" + " " + "World";
const concat2 = "Hello".concat(" ", "World");

// 模板字符串（ES6+）
const name = "Alice";
const greeting = `Hello, ${name}!`;
const multiline = `
  This is
  a multi-line
  string
`;

// 标签模板
function highlight(strings: TemplateStringsArray, ...values: any[]) {
  return strings.reduce((result, str, i) => 
    result + str + (values[i] ? `<mark>${values[i]}</mark>` : ''), '');
}
const result = highlight`Hello ${name}, welcome to ${"TypeScript"}`;
```

2. **查询**：
```typescript
const str = "JavaScript is awesome!";
str.indexOf("Script");      // 4
str.includes("Java");       // true
str.startsWith("Java");     // true
str.endsWith("!");          // true
```

3. **提取**：
```typescript
str.slice(0, 10);           // "JavaScript"
str.substring(0, 10);       // "JavaScript"
str.substr(0, 4);           // "Java" (已废弃)
```

4. **大小写转换**：
```typescript
str.toUpperCase();          // "JAVASCRIPT IS AWESOME!"
str.toLowerCase();          // "javascript is awesome!"
"ß".toLocaleUpperCase("de"); // "SS" (区域敏感)
```

5. **修改**：
```typescript
// 修剪空白
"  Hello  ".trim();         // "Hello"
"  Hello  ".trimStart();    // "Hello  "
"  Hello  ".trimEnd();      //  "Hello"

// 填充
"5".padStart(3, "0");       // "005"
"5".padEnd(3, "0");         // "500"
```



## TypeScript 特有的字符串特性

1. **字符串字面量类型**
```typescript
// 字符串字面量类型
type Direction = "north" | "south" | "east" | "west";
let dir: Direction = "north";  // 只能是指定值之一

// 模板字面量类型（TypeScript 4.1+）
type EventName<T extends string> = `${T}Changed`;
type Concat<A extends string, B extends string> = `${A}${B}`;

// 实用示例
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type ApiEndpoint = `/${string}`;
type FullEndpoint = `${HttpMethod} ${ApiEndpoint}`;

const endpoint: FullEndpoint = "GET /api/users";
```

2. **类型安全操作**
```typescript
// 常量断言
const colors = ["red", "green", "blue"] as const;
type Color = typeof colors[number]; // "red" | "green" | "blue"

// 键名约束
type User = {
  id: number;
  name: string;
};

// 自动提示支持的值
type UserKey = keyof User; // "id" | "name"
```
