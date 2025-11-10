Gemini CLI is an open-source AI agent that brings the power of Gemini directly into your terminal. It provides lightweight access to Gemini, giving you the most direct path from your prompt to our model.

- **🎯 Free tier**: 60 requests/min and 1,000 requests/day with personal Google account
- **🧠 Powerful Gemini 2.5 Pro**: Access to 1M token context window
- **🔧 Built-in tools**: Google Search grounding, file operations, shell commands, web fetching
- **🔌 Extensible**: MCP (Model Context Protocol) support for custom integrations
- **💻 Terminal-first**: Designed for developers who live in the command line
- **🛡️ Open source**: Apache 2.0 licensed


## 安装

1. Windows
```bash
npm install -g @google/gemini-cli
```
2. macOS/Linux
```bash
brew install gemini-cli
```


## 登陆

1. 打开梯子
2. 设置代理端口
```bash
set https_proxy=http://127.0.0.1:7890       # Windows

export https_proxy=http://127.0.0.1:7897    # macOs\Linux
```


## GEMINI.md

```md
# {{ 你的项目名称 }} - AI 协作配置

##   项目概览
- **项目类型**: 例如：企业级 RESTful API 服务
- **核心目标**: 描述项目要解决的核心问题。

##   技术栈
- **后端**: Node.js, TypeScript, Express.js, PostgreSQL
- **前端**: React, Vite, Tailwind CSS
- **关键依赖**: Prisma, Zustand, Vitest

##   开发规范
### 代码风格
- 使用 ESLint + Prettier 进行格式化。
- 函数名使用 camelCase，常量使用 UPPER_SNAKE_CASE。

### 测试要求
- 关键业务逻辑必须有单元测试覆盖。
- 测试文件命名为 `*.test.ts`。

##   AI 助手配置
### 角色定义
你是一个资深的全栈开发工程师，精通本项目使用的技术栈。

### 沟通语气
- **教学导向**: 解释为什么这样做，不只是怎样做。
- **实用主义**: 提供可直接使用的解决方案。
- **简洁明了**: 避免冗长的解释。

##   常见任务示例
### 示例1：添加一个新的 API 端点
**用户问题**: "我需要为 'products' 创建一个 GET /api/v1/products 的端点，用来获取所有产品列表。"
**期望回答**: (在这里描述你期望 AI 如何回答，包括代码结构、文件位置等)
```

### 分层配置 

Gemini CLI 允许你在不同级别放置 `GEMINI.md` 文件，实现配置的继承和覆盖。

1. **全局级别（`~/.gemini/GEMINI.md`）**：定义适用于你所有项目的通用规范，如 Git 提交格式、通用代码风格等。
2. **项目级别（`./GEMINI.md`）**：定义项目专属的配置，如技术栈、架构、项目级规范等。它可以继承全局配置并进行覆盖。
3. **组件级别（`./src/components/GEMINI.md`）**：为项目的特定模块或组件定义更细粒度的规范，如 “React 组件必须使用 forwardRef 处理 ref 传递”。

这种分层系统让你可以在保持全局一致性的同时，为不同项目和模块提供灵活的、定制化的指导。
