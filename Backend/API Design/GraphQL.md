GraphQL 是一种用于 API 的查询语言和运行时环境，由 Facebook 最先创建并标准化，在构建进程间通信方面非常流行。

GraphQL 为传统的客户端-服务器通信提供了一种完全不一样的实现，是 API 的一种查询语言，允许客户端来决定他们想要什么数据、他们想要怎么获取数据以及他们想要什么格式的数据。


**核心特点**

1. **声明式数据获取**
    - 客户端可以精确指定需要的数据字段，避免过度获取（Over-fetching）或获取不足（Under-fetching）。
    - 示例查询：
```graphql
{
  user(id: "123") {
    name
    email
    posts(last: 3) {
      title
    }
  }
}
```

2. **单一端点**
    - 所有请求通过一个端点（如 `/graphql`）处理，通过不同的查询实现不同操作，而非 REST 的多端点。

3. **强类型系统**
	- 基于 Schema 定义数据类型和关系，支持验证查询的合法性。例如：
```graphql
type User {
  id: ID!
  name: String!
  email: String
  posts: [Post!]
}
```


4. **操作类型**
    - **Query**：读取数据（类似 REST 的 `GET`）。
    - **Mutation**：修改数据（类似 `POST/PUT/DELETE`）。
    - **Subscription**：实时数据推送（基于 WebSocket）。


**工具生态**

- **服务端**：Apollo Server、GraphQL-JS（Node）、Django-Graphene（Python）。
- **客户端**：Apollo Client、Relay（React）。
- **开发工具**：GraphiQL（交互式查询界面）、Playground。



## Apollo

Apollo 是一套围绕 GraphQL 的全栈工具链，由 Apollo GraphQL 团队开发和维护，覆盖了从前端客户端到后端服务再到运维监控的完整生命周期。

#### 服务端工具（Backend）

**核心组件和功能**

- **Apollo Server** ：构建生产级 GraphQL 服务端，支持 Express、Koa、Serverless 等环境。
- **Apollo Federation** ：将多个 GraphQL 服务合并为统一 API（微服务架构）。
- **Apollo Datasources** ：封装数据库/REST/gRPC 数据源，提供缓存、批处理等优化。
- **Subscriptions** ：基于 WebSocket 的实时数据推送（如聊天室、实时通知）。


**Apollo Server** 是一个开源的、与 GraphQL 规范兼容的服务器库，用于构建 GraphQL API。它是 **Apollo 生态系统**的核心部分（由 Apollo GraphQL 团队维护），支持 Node.js 和各种现代后端框架（如 Express、Koa、Lambda 等）。开发者可以用它快速搭建高性能的 GraphQL 服务，并轻松集成数据源（数据库、REST API、微服务等）。

**核心特性**

1. **即插即用**
    - 提供开箱即用的功能，包括查询验证、错误处理、缓存、实时订阅（Subscriptions）等。
    - 自动生成 API 文档（通过 Introspection 和 GraphQL Playground）。

2. **多数据源整合**
    - 统一聚合数据库、REST API、gRPC 或其他 GraphQL 服务的数据，通过 **Resolver** 函数灵活定义数据获取逻辑。

3. **生产级工具**
    - 内置性能监控（Apollo Studio）、查询缓存、批处理（Batching）和请求追踪。
    - 支持文件上传、自定义指令（Directives）等扩展功能。

4. **跨平台兼容**
    - 可与 Express、Fastify、Serverless（AWS Lambda）、Cloudflare Workers 等集成。


#### 客户端工具（Frontend）

**核心组件和功能**

- **Apollo Client** ：主力的 GraphQL 客户端，支持 React、Vue、Angular 等框架。
- **Apollo Cache** ：客户端缓存管理（内存、Normalized Cache、持久化）。
- **React Apollo** ：React 集成包（现合并到 Apollo Client 3+ 中）。
- **Apollo iOS/Android** ：移动端原生客户端库。


#### 开发与运维（DevOps & Monitoring）

**核心组件和功能**

- **Apollo Studio** ：云端平台，提供 Schema 注册、性能监控、查询分析、团队协作等功能。
- **Graph Manager** ：（旧版）已整合到 Apollo Studio。
- **Rover CLI** ：命令行工具，用于管理 Apollo Studio 中的 Graph 和 Schema。



#### 工作流

1. **开发阶段**
    - 用 Apollo Server 快速搭建 GraphQL API。
    - 通过 Apollo Sandbox 测试查询。
    - 使用 Apollo Client 连接前端应用。

2. **生产阶段**
    - 将 Schema 发布到 Apollo Studio，监控 API 性能。
    - 利用 Federation 整合多个服务。
    - 通过 Client 的缓存优化用户体验。

3. **运维阶段**
    - 分析 Studio 中的查询性能，识别慢查询。
    - 用 Rover CLI 管理 Schema 变更。


