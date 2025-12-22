
## 常见规范

### `CommonJS`（CJS）

CommonJs 主要用于 Node.js 环境，是 Node.js 中的默认模块规范，在 ES6 版本正式加入了 ES Module，文件扩展名为 `.js` 或 `.cjs` 。

#### 核心设计

1. **同步加载**：适合服务器，文件系统 I/O 是同步的
	- 运行时加载
2. **值拷贝导出**：原始类型是拷贝，对象类型是引用（微妙差别）
3. **运行时确定**：依赖关系在代码执行时确定
4. **缓存机制**：提升性能，处理循环依赖
5. **简单直接**：`require()` 和 `module.exports` 语法简单
6. **每个文件都是模块**：天然的模块化单元


#### 使用示例

1. **导出模块**：
```javascript
// math.js

// 方式1：导出单个对象
module.exports = {
  add: (a, b) => a + b,
  multiply: (a, b) => a * b
};

// 方式2：逐个添加属性
exports.add = (a, b) => a + b;
exports.multiply = (a, b) => a * b;

// 方式3：混合使用（注意：不要同时使用）
module.exports.subtract = (a, b) => a - b;

// 方式4：导出函数
module.exports = function(config) {
  // ...
};

// 方式5：导出类
module.exports = class Calculator {
  // ...
};
```

2. **导入模块**：
```javascript
// 导入整个模块
const math = require('./math.js');
console.log(math.add(1, 2));

// 解构导入（Node.js支持）
const { add, multiply } = require('./math.js');

// 导入JSON文件
const config = require('./config.json');

// 导入Node.js核心模块
const fs = require('fs');
const path = require('path');

// 导入node_modules中的包
const lodash = require('lodash');
const axios = require('axios');
```

3. **内置变量**：
```javascript
console.log(__dirname);  // 当前文件所在目录的绝对路径
console.log(__filename); // 当前文件的绝对路径

// module对象自身
console.log(module.id);      // 模块标识符
console.log(module.paths);   // 模块搜索路径
console.log(module.parent);  // 父模块
console.log(module.children);// 子模块数组
console.log(module.loaded);  // 是否已加载完成
```

#### 实现原理

1. **CommonJS 模块的包装**：Node.js 在执行模块前会将其包裹在一个函数中
```javascript
// 用户写的模块代码：math.js
const PI = 3.1415926;
exports.area = (r) => PI * r * r;

// ========== Node.js 实际执行的代码 ==========
(function(exports, require, module, __filename, __dirname) {
  const PI = 3.1415926;
  exports.area = (r) => PI * r * r;
  // 最后：return module.exports;
});
```

2. **Module 类的实现**（简化版）：
```javascript
// 模拟 Node.js 的 Module 类核心逻辑
class Module {
  constructor(id) {
    this.id = id;           // 模块标识
    this.exports = {};      // 导出对象
    this.loaded = false;    // 加载状态
    this.children = [];     // 子模块
    this.parent = null;     // 父模块
    this.filename = null;   // 文件路径
    this.paths = [];        // 模块查找路径
  }
  
  // 核心加载方法
  load(filename) {
    // 1. 读取文件内容
    const content = fs.readFileSync(filename, 'utf-8');
    
    // 2. 包裹函数字符串
    const wrapper = [
      '(function(exports, require, module, __filename, __dirname) {',
      '\n});'
    ];
    const wrapped = wrapper[0] + content + wrapper[1];
    
    // 3. 编译执行
    const compiledWrapper = vm.runInThisContext(wrapped, {
      filename,
      lineOffset: 0,
      displayErrors: true
    });
    
    // 4. 准备参数
    const dirname = path.dirname(filename);
    const args = [this.exports, this.require, this, filename, dirname];
    
    // 5. 执行模块代码
    compiledWrapper.apply(this.exports, args);
    
    this.loaded = true;
    return this.exports;
  }
  
  // require方法实现
  require(id) {
    return Module._load(id, this);
  }
}
```

3. **`require` 的实现**
```javascript
// 模拟 require 函数的工作流程
Module._load = function(request, parent) {
  // 1. 解析模块路径（核心！）
  const filename = Module._resolveFilename(request, parent);
  
  // 2. 检查缓存（已加载模块直接返回）
  const cachedModule = Module._cache[filename];
  if (cachedModule) {
    return cachedModule.exports;
  }
  
  // 3. 检查是否是原生模块
  if (NativeModule.exists(filename)) {
    return NativeModule.require(filename);
  }
  
  // 4. 创建新模块实例
  const module = new Module(filename);
  
  // 5. 加入缓存（放在加载前，防止循环依赖无限递归）
  Module._cache[filename] = module;
  
  // 6. 加载模块
  try {
    module.load(filename);
  } catch (err) {
    // 加载失败，从缓存删除
    delete Module._cache[filename];
    throw err;
  }
  
  // 7. 返回模块的exports
  return module.exports;
};
```

4. **模块路径解析算法**：
```javascript
// 模拟 _resolveFilename 的核心逻辑
Module._resolveFilename = function(request, parent) {
  // 情况1：核心模块（如 fs、path）
  if (NativeModule.exists(request)) {
    return request;
  }
  
  // 情况2：相对路径或绝对路径
  if (request.startsWith('./') || request.startsWith('../') || path.isAbsolute(request)) {
    const resolved = path.resolve(parent ? path.dirname(parent.filename) : '.', request);
    
    // 尝试添加扩展名 .js, .json, .node
    const extensions = ['.js', '.json', '.node'];
    for (const ext of extensions) {
      const filename = resolved + ext;
      if (fs.existsSync(filename)) return filename;
    }
    
    // 检查是否是目录
    if (fs.existsSync(resolved) && fs.statSync(resolved).isDirectory()) {
      // 查找 package.json 的 main 字段
      const pkgPath = path.join(resolved, 'package.json');
      if (fs.existsSync(pkgPath)) {
        const pkg = JSON.parse(fs.readFileSync(pkgPath, 'utf-8'));
        if (pkg.main) {
          return Module._resolveFilename(pkg.main, { filename: resolved });
        }
      }
      
      // 查找 index.js, index.json, index.node
      for (const ext of extensions) {
        const filename = path.join(resolved, 'index' + ext);
        if (fs.existsSync(filename)) return filename;
      }
    }
    
    throw new Error(`Cannot find module '${request}'`);
  }
  
  // 情况3：node_modules 查找
  const paths = Module._resolveLookupPaths(request, parent);
  for (const dir of paths) {
    const filename = path.join(dir, request);
    try {
      return Module._resolveFilename(filename, parent);
    } catch (err) {
      // 继续尝试下一个路径
    }
  }
  
  throw new Error(`Cannot find module '${request}'`);
};

// 生成查找路径
Module._resolveLookupPaths = function(request, parent) {
  const paths = [];
  
  // 从当前目录向上查找 node_modules
  let currentDir = parent ? path.dirname(parent.filename) : process.cwd();
  while (currentDir !== path.dirname(currentDir)) {
    paths.push(path.join(currentDir, 'node_modules'));
    currentDir = path.dirname(currentDir);
  }
  
  // 添加全局 node_modules 路径
  paths.push(path.join(process.execPath, '..', '..', 'lib', 'node_modules'));
  
  // 添加 NODE_PATH 环境变量路径
  if (process.env.NODE_PATH) {
    paths.push(...process.env.NODE_PATH.split(path.delimiter));
  }
  
  return paths;
};
```

#### 特殊场景处理

1. **循环依赖处理**：
```javascript
// a.js
console.log('a 开始加载');
exports.done = false;
const b = require('./b.js');  // 同步加载 b
console.log('在 a 中，b.done = %j', b.done);
exports.done = true;
console.log('a 结束加载');

// b.js
console.log('b 开始加载');
exports.done = false;
const a = require('./a.js');  // 此时 a 还未完全加载完
console.log('在 b 中，a.done = %j', a.done);  // false，不是 true！
exports.done = true;
console.log('b 结束加载');

// main.js
const a = require('./a.js');
const b = require('./b.js');
console.log('在 main 中，a.done=%j, b.done=%j', a.done, b.done);
```
- **执行过程**：由于 `require` 是同步的，遇到循环依赖时：
	- a 开始加载，执行到 `require('./b')` 时暂停
	- b 开始加载，执行到 `require('./a')` 时，a 已在缓存中（但未完成）
	- b 得到 a 的部分导出（此时 `a.done` 还是 false）
	- b 继续执行完成，返回给 a
	- a 继续执行完成

2. **`exports` 和 `module.exports` 的关系**：
```javascript
// module.exports 和 exports 的初始关系
console.log(exports === module.exports); // true

// 错误的用法：改变 exports 的引用
exports = { a: 1 };          // ❌ 无效！require 得到的是 module.exports
module.exports = { b: 2 };   // ✅ 正确

// 正确的用法：修改 exports 的属性
exports.c = 3; // ✅ 等价于 module.exports.c = 3

// 证明：
const module = {
  exports: {}
};
const exports = module.exports;

console.log(exports === module.exports); // true

exports = { a: 1 }; // 只改变了局部变量 exports 的引用
console.log(module.exports); // 仍然是 {}，不是 {a:1}
```

3. **`require` 的缓存机制**：
```javascript
// module.js
let count = 0;
module.exports = { 
  increment: () => ++count,
  getCount: () => count
};

// main.js
const mod1 = require('./module.js');
const mod2 = require('./module.js');

console.log(mod1 === mod2); // true，同一个对象

mod1.increment();
console.log(mod2.getCount()); // 1，共享状态

// 清除缓存（开发时有用）
delete require.cache[require.resolve('./module.js')];
const mod3 = require('./module.js');
console.log(mod1 === mod3); // false，新实例
```

#### 高级特性

1. **`require.extensions`**：
```javascript
// 注册自定义文件类型处理器（已废弃，但了解原理）
require.extensions['.txt'] = function(module, filename) {
  const content = fs.readFileSync(filename, 'utf-8');
  module.exports = content;
};

// 使用
const data = require('./data.txt');
console.log(data); // 文件内容字符串
```

2. **`require.main`**：
```javascript
// 判断是否是入口文件
if (require.main === module) {
  // 直接执行 node this-file.js
  console.log('这是入口文件');
  // 启动应用
} else {
  // 被其他模块导入
  console.log('这是被导入的模块');
}
```

3. **模块预加载**：
```javascript
// 使用 --require 参数预加载模块
// node --require ./preload.js main.js

// preload.js
// 可以在所有模块加载前执行一些初始化
console.log('预加载模块执行');
global.someConfig = { /* ... */ };
```


#### 性能优化

1. **延迟加载**
```javascript
// 大型模块按需加载
function getHeavyModule() {
  return require('./heavy-module.js');
}

// 或使用工厂模式
module.exports = (config) => {
  const heavy = require('./heavy-module.js');
  return heavy.create(config);
};
```

2. **避免循环依赖**
```javascript
// 使用依赖注入代替直接require
class ServiceA {
  constructor(serviceB) {
    this.serviceB = serviceB;
  }
}

class ServiceB {
  constructor(serviceA) {
    this.serviceA = serviceA;
  }
}

// 在顶层统一解决依赖
const a = new ServiceA(null);
const b = new ServiceB(a);
a.serviceB = b;
```

3. **模块热替换**（HMR 基础）
```javascript
// 开发时热重载模块
function requireUncached(module) {
  delete require.cache[require.resolve(module)];
  return require(module);
}

// 监听文件变化
const fs = require('fs');
fs.watch('./module.js', () => {
  const newModule = requireUncached('./module.js');
  // 更新应用状态...
});
```

#### 与现代 ESM 的交互

```javascript
// CommonJS 中使用 ESM（需要异步）
// package.json 中设置 "type": "module"

// .cjs 文件中（CommonJS）
const { createRequire } = require('module');
const require = createRequire(import.meta.url);

const esModule = await import('./es-module.mjs');

// .mjs 文件中（ESM）使用 CommonJS
import { createRequire } from 'module';
const require = createRequire(import.meta.url);

const cjsModule = require('./cjs-module.cjs');
```




### ES Module（ESM）

ES Module 是 JavaScript 官方标准模块系统，文件扩展名为 `.mjs` 或 `.js`（需设置 `type="module"`）。

#### 核心设计

1. **静态结构**：编译时确定依赖，支持优化
	- 不可以动态加载语句，只能声明在该文件的最顶部，代码发生在编译时
2. **动态绑定**：导出值的实时引用（live bindings）
3. **异步加载**：天然支持按需加载和代码分割
	- `import` 是异步加载模块，静态编译时加载，有独立的模块依赖解析。
4. **语言级标准**：不再是库或工具，而是语言特性
5. **严格模式**：更好的错误检查和代码质量
6. **跨平台**：浏览器和 Node.js 统一标准


#### 使用示例

1. **导出模块**
```javascript
// math.mjs (或 math.js 在 type="module" 中)

// 1. 命名导出（推荐）
export const PI = 3.1415926;
export function add(a, b) {
  return a + b;
}
export class Calculator {
  // ...
}

// 2. 默认导出（每个模块一个）
export default function multiply(a, b) {
  return a * b;
}

// 3. 统一导出
const secret = 'private';
const log = (msg) => console.log(msg);

export { 
  secret as publicKey, // 重命名
  log
};

// 4. 重新导出（聚合导出）
export { add, multiply } from './math-utils.mjs';
export { default as React } from 'react'; // 重新导出默认
```

2. **导入模块**
```javascript
// 1. 命名导入
import { PI, add } from './math.mjs';

// 2. 默认导入
import multiply from './math.mjs';

// 3. 混合导入
import multiply, { PI, add as sum } from './math.mjs';

// 4. 全部导入（命名空间）
import * as math from './math.mjs';
math.add(1, 2);

// 5. 只执行模块，不导入任何内容
import './init.mjs';

// 6. 动态导入（返回 Promise）
const module = await import('./math.mjs');
const { add } = await import('./math.mjs');
```

3. **浏览器中使用**
```html
<!-- 方式1：内联模块 -->
<script type="module">
  import { add } from './math.mjs';
  console.log(add(1, 2));
</script>

<!-- 方式2：外部模块 -->
<script type="module" src="./app.mjs"></script>

<!-- 方式3：使用 importmap（现代浏览器） -->
<script type="importmap">
{
  "imports": {
    "lodash": "https://cdn.skypack.dev/lodash",
    "react": "./node_modules/react/index.js"
  }
}
</script>
```


#### 核心特性

1. **静态结构**（Static Module Structure）
```javascript
// ✅ 有效的导入导出（编译时可确定）
import { foo } from './module.mjs';
export { bar };

// ❌ 无效的（运行时确定）
const moduleName = './module.mjs';
import(moduleName); // 只能用于动态导入

if (condition) {
  import './module.mjs'; // 错误！必须是顶层
}
```
- **优势**：
	- Tree Shaking：打包时可删除未使用的代码 
	- 优化：提前解析依赖关系
	- 类型检查：支持静态类型分析
	- 循环依赖：更好的处理

2. **Live Bindings**（动态绑定）
```javascript
// counter.mjs
export let count = 0;
export function increment() {
  count++;
}

// main.mjs
import { count, increment } from './counter.mjs';

console.log(count); // 0
increment();
console.log(count); // 1 ✅ 实时更新！

// 对比 CommonJS（值拷贝）
// counter.js
let count = 0;
module.exports = { 
  getCount: () => count,
  increment: () => count++ 
};

// main.js
const { getCount, increment } = require('./counter.js');
console.log(getCount()); // 0
increment();
console.log(getCount()); // 1
```


3. **严格模式**（Always Strict）
```javascript
// ES Module 自动启用严格模式
// 以下代码在 ESM 中会报错，在 CommonJS 中可能不会

delete Object.prototype; // ❌ TypeError

undeclaredVar = 10; // ❌ ReferenceError

const obj = {};
Object.defineProperty(obj, 'readonly', { 
  value: 42, 
  writable: false 
});
obj.readonly = 100; // ❌ TypeError

function duplicateParam(a, a) { // ❌ SyntaxError
  // ...
}
```

#### 实现原理

1. **加载过程**：
```javascript
// 完整的 ESM 加载流程
class ESModuleLoader {
  async loadModule(specifier, referrer) {
    // 阶段1: 构造（Construction）
    const url = this.resolve(specifier, referrer);
    const module = new ModuleRecord(url);
    
    // 检查缓存
    if (this.cache.has(url)) {
      return this.cache.get(url);
    }
    
    // 阶段2: 实例化（Instantiation）
    const code = await this.fetchModule(url);
    const parsed = this.parseModule(code);
    
    // 创建模块环境记录（Module Environment Record）
    module.environment = this.createEnvironment(parsed);
    
    // 阶段3: 求值（Evaluation）
    await this.evaluateModule(module);
    
    this.cache.set(url, module);
    return module;
  }
  
  resolve(specifier, referrer) {
    // 解析算法：
    // 1. 如果是相对路径 ./ 或 ../
    // 2. 如果是绝对路径 /
    // 3. 如果是URL协议 https://
    // 4. 如果是裸模块名，查找 node_modules
    // 5. 使用 importmaps 映射
  }
}
```

2. **模块记录**（Module Record）结构
```javascript
// 模块的内部表示
class ModuleRecord {
  constructor(url) {
    this.url = url;                // 模块标识符
    this.status = 'unlinked';      // 状态：unlinked/linking/linked/evaluating/evaluated
    this.environment = null;       // 模块环境记录
    this.exports = new Map();      // 导出绑定
    this.imports = new Map();      // 导入绑定
    this.dependencies = new Set(); // 依赖模块
    this.hasTLA = false;           // 是否有顶层await
  }
  
  // 链接过程（建立导入导出连接）
  link(importBindings, exportBindings) {
    // 建立 import -> export 的链接
    for (const [localName, { module, exportName }] of importBindings) {
      const exportedValue = module.getExport(exportName);
      this.environment.createImportBinding(localName, exportedValue);
    }
    
    // 建立 export -> local 的链接
    for (const [exportName, localName] of exportBindings) {
      const localValue = this.environment.getBinding(localName);
      this.exports.set(exportName, localValue);
    }
  }
}
```

3. **浏览器实现**示例
```javascript
// 简化的浏览器 ESM 加载器
class BrowserModuleLoader {
  constructor() {
    this.moduleMap = new Map(); // URL -> 模块实例
    this.fetching = new Map();  // URL -> Promise
  }
  
  async import(specifier, referrer) {
    // 1. 解析URL
    const url = new URL(specifier, referrer).href;
    
    // 2. 检查缓存
    if (this.moduleMap.has(url)) {
      return this.moduleMap.get(url).namespace;
    }
    
    // 3. 防止重复请求
    if (this.fetching.has(url)) {
      return this.fetching.get(url);
    }
    
    // 4. 创建加载Promise
    const loadPromise = this.loadModule(url);
    this.fetching.set(url, loadPromise);
    
    try {
      const module = await loadPromise;
      return module.namespace;
    } finally {
      this.fetching.delete(url);
    }
  }
  
  async loadModule(url) {
    // 1. 获取源代码
    const response = await fetch(url);
    const source = await response.text();
    
    // 2. 解析模块依赖
    const deps = this.parseDependencies(source);
    
    // 3. 递归加载依赖
    await Promise.all(deps.map(dep => 
      this.import(dep, url)
    ));
    
    // 4. 创建模块实例
    const module = {
      url,
      status: 'loaded',
      dependencies: deps,
      namespace: null
    };
    
    // 5. 执行模块代码（使用Worker或eval隔离）
    const namespace = await this.evaluateModule(source, url);
    module.namespace = namespace;
    module.status = 'evaluated';
    
    this.moduleMap.set(url, module);
    return module;
  }
  
  parseDependencies(source) {
    // 使用正则匹配 import 语句
    const importRegex = /import\s+(?:["']([^"']+)["']|[\s\S]*?from\s+["']([^"']+)["'])/g;
    const deps = [];
    let match;
    
    while ((match = importRegex.exec(source)) !== null) {
      const dep = match[1] || match[2];
      if (dep) deps.push(dep);
    }
    
    return deps;
  }
}
```


#### 关键机制

1. **循环依赖处理**
```javascript
// a.mjs
import { b } from './b.mjs';

export const a = 'a';
console.log('a执行，b=', b);

// b.mjs
import { a } from './a.mjs';

export const b = 'b';
console.log('b执行，a=', a); // 能正常访问到 'a'

// main.mjs
import './a.mjs';
```
- **处理流程**：
	1. 深度优先遍历依赖图
	2. 先解析所有模块（Construction）
	3. 建立所有导入导出绑定（Instantiation）
	4. 从入口开始执行（Evaluation）

2. **动态导入**（`import()`）
```javascript
// import() 返回 Promise
const loadModule = async (moduleName, condition) => {
  if (condition) {
    // 按需加载
    const module = await import(`./modules/${moduleName}.mjs`);
    return module;
  }
  return null;
};

// 错误处理
try {
  const module = await import('./non-existent.mjs');
} catch (error) {
  console.error('模块加载失败:', error);
}

// 并行加载多个模块
const [moduleA, moduleB] = await Promise.all([
  import('./a.mjs'),
  import('./b.mjs')
]);

// 与普通导入的不同
console.log(typeof import); // "function"
console.log(import.meta);   // { url: "file:///..." }
```

3. `import.meta`
```javascript
// 获取当前模块信息
console.log(import.meta.url);     // 模块的完整URL
console.log(import.meta.resolve); // 解析相对路径

// 使用示例
const modulePath = import.meta.url;
const dataPath = import.meta.resolve('./data.json');

// 自定义元数据（提案阶段）
import.meta.env = process.env.NODE_ENV;
import.meta.hot?.accept(); // Vite HMR

// 获取查询参数
const url = new URL(import.meta.url);
const searchParams = url.searchParams;
```

4. **顶层 `await`**（Top-Level Await）
```javascript
// 在ESM中可以直接使用await
const response = await fetch('https://api.example.com/data');
const data = await response.json();

// 模块导出的值可以依赖于异步操作
export const config = await loadConfig();

// 这使模块可以延迟执行
await new Promise(resolve => setTimeout(resolve, 1000));
console.log('延迟1秒后执行');

// 注意事项：
// 1. 有TLA的模块会影响依赖它的模块执行时机
// 2. 可能会阻塞依赖链
```


#### Node.js 中的 ESM 实现

1. **文件扩展名规则**
```javascript
// package.json
{
  "type": "module",     // .js 文件视为ESM
  "type": "commonjs"    // 默认，.js 文件视为CommonJS
}

// 文件扩展名优先规则：
// 1. .mjs → 总是ESM
// 2. .cjs → 总是CommonJS
// 3. .js → 根据package.json的type字段
```
2. **Node.js 加载器原理**

```javascript
// Node.js 的 ESM 加载器（简化）
class NodeESMLoader {
  async load(url, context, defaultLoad) {
    // 1. 解析URL
    const parsed = new URL(url);
    
    // 2. 文件类型判断
    if (parsed.pathname.endsWith('.mjs') || 
        (parsed.pathname.endsWith('.js') && this.isESMPackage(parsed))) {
      
      // 3. 读取源代码
      const source = await fs.promises.readFile(parsed, 'utf8');
      
      // 4. 转译（如果需要）
      const transformed = await this.transform(source, parsed);
      
      // 5. 返回模块格式
      return {
        format: 'module',
        source: transformed,
        shortCircuit: true
      };
    }
    
    // 否则使用默认加载器
    return defaultLoad(url, context);
  }
  
  isESMPackage(url) {
    // 向上查找 package.json
    let dir = path.dirname(url.pathname);
    while (dir !== '/') {
      const pkgPath = path.join(dir, 'package.json');
      if (fs.existsSync(pkgPath)) {
        const pkg = JSON.parse(fs.readFileSync(pkgPath, 'utf8'));
        return pkg.type === 'module';
      }
      dir = path.dirname(dir);
    }
    return false;
  }
}
```

3. **Node.js 中的特殊处理**

```javascript
// __filename 和 __dirname 的替代方案
import { fileURLToPath } from 'url';
import { dirname } from 'path';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

// JSON 模块导入
import config from './config.json' assert { type: 'json' };
import data from './data.json' with { type: 'json' }; // 新语法

// Wasm 模块导入
import wasmModule from './module.wasm' assert { type: 'webassembly' };

// 模块条件导出
// package.json
{
  "exports": {
    ".": {
      "import": "./dist/esm/index.js",
      "require": "./dist/cjs/index.js",
      "default": "./dist/legacy/index.js"
    },
    "./feature": {
      "browser": "./feature-browser.js",
      "node": "./feature-node.js"
    }
  }
}
```


#### 打包工具中的处理

1. **Webpack 的 ESM 处理**
```javascript
// webpack 将 ESM 转换为自己的模块系统
// 输入：
import { add } from './math.mjs';
export const result = add(1, 2);

// 输出：
__webpack_require__.r(__webpack_exports__);
/* harmony import */ var _math_mjs__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./math.mjs */ "./src/math.mjs");

const result = (0,_math_mjs__WEBPACK_IMPORTED_MODULE_0__.add)(1, 2);
/* harmony export */ __webpack_exports__["result"] = (() => (result));
```

2. **Tree Shaking 原理**
```javascript
// 原始代码
export function usedFunction() {
  return 'used';
}

export function unusedFunction() {
  return 'unused';
}

// 构建工具分析依赖关系
// 发现 unusedFunction 未被导入
// 最终输出：
export function usedFunction() {
  return 'used';
}
// unusedFunction 被移除
```

3. 代码分割（Code Splitting）
```javascript
// 使用动态导入实现代码分割
const loadComponent = async () => {
  // 单独打包成 chunk
  const { default: HeavyComponent } = await import(
    /* webpackChunkName: "heavy" */ './HeavyComponent.mjs'
  );
  return HeavyComponent;
};

// 预加载/预获取
const prefetchModule = () => import(
  /* webpackPrefetch: true */ './future-module.mjs'
);
```


#### 性能优化

1. **预加载模块**
```html
<!-- 预加载关键模块 -->
<link rel="modulepreload" href="./critical.mjs">

<!-- 预加载依赖 -->
<script type="module">
  // 主模块
  import './app.mjs';
</script>
```

2. **使用 HTTP/2 推送**
```javascript
// 服务器端推送依赖模块
// Node.js + Express 示例
app.use((req, res, next) => {
  if (req.url.endsWith('.mjs')) {
    const dependencies = analyzeDependencies(req.url);
    dependencies.forEach(dep => {
      res.push(dep, {
        request: { accept: 'application/javascript' }
      });
    });
  }
  next();
});
```

3. **模块缓存策略**
```javascript
// 使用版本号或哈希
import(`./module.v${version}.mjs`);
import(`./module-${hash}.mjs`);

// Service Worker 缓存
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js', {
    scope: '/',
    type: 'module' // Service Worker 也可以是模块
  });
}
```

#### 常见问题

1. **混合模块系统**
```javascript
// ESM 中引入 CJS
import { createRequire } from 'module';
const require = createRequire(import.meta.url);

const cjsModule = require('./cjs-module.cjs');
const packageJson = require('../package.json');

// CJS 中引入 ESM（必须异步）
async function loadESM() {
  const esmModule = await import('./esm-module.mjs');
}

// 双模式包的最佳实践
// package.json
{
  "name": "my-package",
  "exports": {
    "import": "./dist/esm/index.js",
    "require": "./dist/cjs/index.js"
  },
  "main": "./dist/cjs/index.js"
}
```

2. **路径解析差异**
```javascript
// ESM 必须使用完整扩展名
import './module.mjs';  // ✅
import './module';      // ❌ 在浏览器中会失败

// 解决方案：使用构建工具或 import.meta.resolve
const modulePath = import.meta.resolve('./module');
const module = await import(modulePath);

// Node.js 中的处理
import { readFile } from 'fs/promises';  // ✅ Node.js内置模块
import fs from 'fs/promises';            // ✅ 也支持
```

3. **作用域隔离**
```javascript
// 模块有自己的作用域
// module.mjs
const privateVar = 'secret'; // 不会污染全局
window.myGlobal = 'test';    // ❌ 避免这样用

// 全局变量访问
console.log(globalThis); // 跨平台的全局对象
console.log(self);       // Web Workers 中的全局对象
```



#### 未来发展趋势

1. **导入断言**（Import Assertions）
```javascript
// 类型安全的导入
import jsonData from './data.json' assert { type: 'json' };
import wasmModule from './program.wasm' assert { type: 'webassembly' };

// 新的 with 语法（提案）
import jsonData from './data.json' with { type: 'json' };
```

2. **模块片段**（Module Fragments）
```javascript
// 提案：允许导入模块的一部分
import { feature } from './module.mjs#feature';

// 内联模块
<script type="module">
  // 模块代码
</script>
```

3. **更细粒度的导出**
```javascript
// 导出表达式（提案）
export const value = someExpression;
export { someExpression as value };
```


## 演进历史

|时间|推动因素|代表性技术|
|---|---|---|
|2005|Ajax兴起，前端复杂度增加|命名空间模式|
|2009|Node.js诞生，需要服务器模块化|CommonJS|
|2010|单页应用（SPA）流行|AMD/RequireJS|
|2012|前端工程化需求|Browserify（打包CommonJS）|
|2014|大型应用需要更好工具|Webpack（支持各种模块）|
|2015|ES6标准发布|ES Modules|
|2017|现代浏览器原生支持|`<script type="module">`|
|2020|Node.js正式支持ESM|Node.js v13.2+|

### 无模块化时代（1995~2009）

早期 JavaScript 开发很容易存在全局污染和依赖管理混乱问题，问题表现如下：
```html
<!-- 页面中直接写脚本 -->
<script>
  var globalVar = '我是全局变量';

  function foo() {
    // 可能修改其他脚本的变量
    globalVar = '被意外修改了';
  }
</script>

<!-- 多个脚本文件，互相依赖 -->
<script src="jquery.js"></script>
<script src="plugin1.js"></script> <!-- 依赖jquery -->
<script src="plugin2.js"></script> <!-- 依赖jquery和plugin1 -->
<script src="main.js"></script> <!-- 依赖上面所有 -->
```

1. **全局污染**：所有变量都在全局作用域
2. **命名冲突**：不同库的相同变量名会互相覆盖
3. **依赖管理困难**：`script` 标签顺序必须正确
4. **维护困难**：代码耦合度高


### 模块化萌芽期（2006~2009）

1. **命名空间模式**：
```javascript
// 使用对象作为命名空间
var MYAPP = MYAPP || {};

MYAPP.utils = {
  formatDate: function(date) {
    // ...
  }
};

MYAPP.models = {
  User: function(name) {
    this.name = name;
  }
};
```

2. IIFE 模式（立即执行函数表达式）：
```javascript
// 模块1
var module1 = (function() {
  var privateVar = '私有变量';
  
  return {
    publicMethod: function() {
      return privateVar;
    }
  };
})();

// 模块2，依赖模块1
var module2 = (function(mod1) {
  mod1.publicMethod(); // 使用模块1
  return {
    // ...
  };
})(module1);
```

3. **依赖注入模式**（RequireJS 的前身）：
```javascript
// 简单的依赖管理
define('moduleA', [], function() {
  return { /* 模块内容 */ };
});

define('moduleB', ['moduleA'], function(moduleA) {
  // 使用moduleA
  return { /* 模块内容 */ };
});
```


### 模块化标准形成期（2009~2014）

1. **`CommonJS`**：Node.js 的诞生推动了服务器端 JavaScript 的模块化需求
```javascript
// 解决的核心问题：让每个文件都有自己的作用域
var localVar = '我是局部变量'; // 不会污染全局

// 明确的导入导出机制
module.exports = {
  // 只暴露需要公开的部分
};
```
- **特点**：
	- 同步加载（适合服务器） 
	- 简单直观的语法
	- 每个文件是一个模块


2. **AMD（Asynchronous Module Definition）**：浏览器需要异步加载模块，RequireJS 推广
```javascript
// 异步模块定义
define(['jquery', 'lodash'], function($, _) {
  // 依赖加载完成后执行
  return {
    init: function() {
      $('#app').html('Hello');
    }
  };
});
```

3. **UMD（Universal Module Definition）**：兼容 AMD、CommonJS、全局变量的通用模式
```javascript
// 复杂的兼容代码
(function(root, factory) {
  if (typeof define === 'function' && define.amd) {
    // AMD
    define(['exports'], factory);
  } else if (typeof exports === 'object') {
    // CommonJS
    factory(exports);
  } else {
    // 浏览器全局变量
    factory((root.myModule = {}));
  }
}(this, function(exports) {
  // 模块代码
}));
```

### 标准化时代（2015~）

1. **ES6 Modules**：随着 JavaScript 成为多平台语言（浏览器、服务器、移动端），并且主流打包工具（Webpack、Rollup）的普及，JavaScript 需要官方的、统一的模块标准。
```javascript
// 官方的静态模块语法
import { util1, util2 } from './utils.js';
export const myFunction = () => { /* ... */ };
```
- **意义**：
	- **语言级别支持**：不再是第三方方案
	- **静态分析**：支持 Tree Shaking、优化等
	- **跨平台一致**：浏览器、Node.js 统一语法


## 未来趋势

1. **Bundleless 开发**：Vite、Snowpack 利用 ESM 原生支持
2. **Import Maps**：浏览器原生模块映射
```html
<script type="importmap">
{
  "imports": {
	"vue": "https://unpkg.com/vue@3/dist/vue.esm-browser.js"
  }
}
</script>
```
3. **WebAssembly 模块化**：与 JavaScript 模块互操作
4. **更好的工具链支持**：TypeScript、Deno 等