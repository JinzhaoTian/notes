Monorepo（Monolithic Repository）是一种软件开发策略，指将多个相关项目的代码（无论使用什么语言或技术栈）存储在同一个版本控制仓库中（如 Git），而不是为每个项目或模块单独建立仓库。

![](../_imgs/Pasted%20image%2020241114101852.png)


**阶段一：（Monolithic）单仓库巨石应用**，一个 Git 仓库维护着项目代码，随着迭代业务复杂度的提升，项目代码会变得越来越多，越来越复杂，大量代码构建效率也会降低，最终导致了单体巨石应用，这种代码管理方式称之为 Monolith。

```
project
├── node_modules/
│   ├── lib@1.0.0
├── src/
│   ├── compA
│   ├── compB
│   └── compC
└── package.json
```

**阶段二：（Multirepo）多仓库多模块应用**，于是将项目拆解成多个业务模块，并在多个 Git 仓库管理，模块解耦，降低了巨石应用的复杂度，每个模块都可以独立编码、测试、发版，代码管理变得简化，构建效率也得以提升，这种代码管理方式称之为 MultiRepo。

```
project
├── node_modules/
│   ├── lib@1.0.0
│   ├── lib@2.0.0
│   ├── pkgA
│   ├── pkgB
│   └── ..
├── src/
└── package.json


packageA
├── node_modules/
│   └── lib@1.0.0
├── src/
└── package.json


packageB
├── node_modules/
│   └── lib@2.0.0
├── src/
└── package.json
```

**阶段三：单仓库多模块应用**，随着业务复杂度的提升，模块仓库越来越多，MultiRepo 这种方式虽然从业务上解耦了，但增加了项目工程管理的难度，随着模块仓库达到一定数量级，会有几个问题：跨仓库代码难共享；分散在单仓库的模块依赖管理复杂（底层模块升级后，其他上层依赖需要及时更新，否则有问题）；增加了构建耗时。于是将多个项目集成到一个仓库下，共享工程配置，同时又快捷地共享模块代码，成为趋势，这种代码管理方式称之为 MonoRepo。




### 适用场景

- 多项目共享大量代码（如微服务、前后端一体化项目）。
- 需要频繁跨项目修改（如组件库和业务项目联动）。
- 团队规模较大，希望统一开发流程。


### 示例项目结构

```
monorepo/
├── packages/
│   ├── shared-utils/  # 共享工具库
│   ├── frontend/      # 前端项目
│   └── backend/       # 后端项目
├── node_modules/      # 共享依赖
├── package.json       # 根目录配置
└── turbo.json         # Turborepo 构建配置
```


### 使用示例

1. [pnpm](../../Language/Node/构建/pnpm.md)

