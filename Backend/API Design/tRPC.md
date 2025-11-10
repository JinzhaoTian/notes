[tRPC](https://trpc.io/)（**TypeScript Remote Procedure Call**）是一个用于构建类型安全 API 的 [TypeScript](../../../Language/TypeScript/TypeScript.md) 框架。它允许开发者在前端和后端之间共享类型定义，从而实现端到端的类型安全，减少手动类型检查的工作量。

## 核心特点

1. **类型安全**
    - 前后端共享 TypeScript 类型定义，API 的输入输出自动推断类型，避免手动定义 DTO（数据传输对象）。
    - 开发时 IDE 会提供自动补全和类型错误提示。
2. **零样板代码**
    - 不需要手动生成客户端代码（如 OpenAPI / Swagger），直接通过类型推导生成客户端调用。
3. **轻量高效**
    - 基于 HTTP / JSON 通信，无需复杂的配置，兼容现代前端框架（React、Next.js、Vue 等）。
4. **框架无关性**
    - 后端可与 Express、Fastify、Next.js 等 Node.js 框架集成，前端适配多种技术栈。

## 使用示例

1. **定义后端 API**

```typescript
// server.ts
import { initTRPC } from '@trpc/server';

const t = initTRPC.create();
const appRouter = t.router({
  greet: t.procedure
    .input((val: unknown) => {
      if (typeof val === 'string') return val;
      throw new Error('Invalid input');
    })
    .query(({ input }) => {
      return { message: `Hello, ${input}!` };
    }),
});

export type AppRouter = typeof appRouter;
```

2. **前端调用 API**
```typescript
// client.ts
import { createTRPCProxyClient, httpBatchLink } from '@trpc/client';
import type { AppRouter } from './server';

const client = createTRPCProxyClient<AppRouter>({
  links: [httpBatchLink({ url: 'http://localhost:3000/trpc' })],
});

async function main() {
  const response = await client.greet.query('World');
  console.log(response.message); // 输出: "Hello, World!"
}
main();
```

## 适用场景

- 全栈 TypeScript 项目，追求类型安全和开发效率。
- 需要快速迭代的小型到中型应用。
- 避免维护单独的 API 文档（类型即文档）。


## 对比其他方案

| 特性       | tRPC      | REST      | GraphQL      | gRPC                 |
| -------- | --------- | --------- | ------------ | -------------------- |
| **类型安全** | ✅ 端到端     | ❌ 需手动维护   | ✅ 需代码生成      | ✅ 需 Protocol Buffers |
| **协议**   | HTTP/JSON | HTTP/JSON | HTTP/GraphQL | HTTP/2 + Protobuf    |
| **适用规模** | 中小型应用     | 任意规模      | 复杂数据查询       | 高性能微服务               |

tRPC 适合 TypeScript 全栈开发者，尤其看重类型安全和开发效率的场景。如果你厌倦了维护 REST/GraphQL 的繁琐类型定义，tRPC 是一个值得尝试的现代化替代方案。