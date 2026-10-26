`Proxy` 是 JavaScript 语言标准**内置的**全局构造函数，属于 ECMAScript 标准的一部分，由 JavaScript 引擎直接提供：
- 浏览器：V8 / SpiderMonkey / JavaScriptCore 等提供
- Node.js：V8 提供
- Deno / Bun：各自运行时提供

## 用法

```javascript
const proxy = new Proxy(target, handler)
```

其中：
- `target` 是被代理的对象
- `handler` 是拦截器对象，比如 `ReflectService.handler`





## 实例

在 Cordis 框架中，其核心的依赖注入系统就是用 `Proxy` 实现的：
```javascript
// cordis/src/context.ts

// ...
import { ReflectService } from './reflect.ts'
import { RegistryService, type InjectKey } from './registry.ts'

/**
 * Public shape of a Cordis context.
 */
export interface Context {
  // ...
}

/**
 * Root and child dependency containers for Cordis plugins.
 */
export class Context {
  // ...
  
  /** Create the root context and install the built-in services. */
  constructor() {
    // ... 初始化 isolate/intercept 映射 ...
    const self = new Proxy<this>(this, ReflectService.handler)
    this.root = self
    // ...
    return self  // 注意这里返回的是代理对象，而不是原始 this
  }
  
  // ...
}

```

其中，`ReflectService.handler` 是一个符合 ES6 `ProxyHandler` 规范的拦截器对象，它定义了当访问 `Context` 代理对象上的属性时，应该执行哪些操作。

> [!caution] 构造函数最终返回的是代理对象 `self`，而不是原始的 `this`
> 这意味着，通过 `new Context()` 创建出来的每一个 `ctx` 实例，从诞生起就是一个被代理的对象。此后，你对 `ctx` 的任何属性操作，都会先经过 `ReflectService.handler` 的处理。

其工作流程如下：

1. **拦截属性读取**：当你尝试读取 `ctx.llm` 时，`get` 陷阱首先会检查 `Context` 实例本身是否已有这个属性。
2. **服务解析**：如果没有，它会**转交给 `ReflectService` 进行服务解析**。`ReflectService` 会沿着 Fiber 链（从当前上下文向上查找到根上下文），在服务注册表中查找名为 `llm` 的服务实现。
3. **返回结果**：一旦找到，就返回该服务实例。如果找不到，根据依赖注入的规则，可能会返回 `undefined` 或抛出错误。

