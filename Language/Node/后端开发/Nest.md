[NestJS](https://docs.nestjs.com/) 是一个用于构建高效、可扩展的 Node.js 服务端应用程序的框架，它基于 TypeScript 和 Express（或 Fastify）构建，结合了面向对象编程（OOP）、函数式编程（FP）和响应式编程（RP）的最佳实践。NestJS 提供了一种结构化、模块化的开发方式，使得开发人员能够以类似 Angular 的方式构建和组织后端应用程序。


**主要特点**

1. **模块化**：
    - NestJS 强调将应用拆分成独立的模块，模块内包含控制器、服务、提供者等，方便进行功能划分和代码组织。
2. **基于装饰器的编程模型**：
    - NestJS 使用装饰器（decorators）来简化代码，提供对路由、请求处理、服务注入等功能的声明式支持。比如 `@Controller()`、`@Get()`、`@Post()` 等。
3. **依赖注入（DI）**：
    - NestJS 使用依赖注入模式来管理应用中的服务和组件。开发者可以方便地注入需要的依赖，从而提高了代码的可测试性和可维护性。
4. **支持多种类型的应用架构**：
    - NestJS 可以用来开发多种类型的应用，如 RESTful API、GraphQL、微服务架构、WebSockets、gRPC 等。
5. **支持 TypeScript**：
    - NestJS 完全支持 TypeScript，提供静态类型检查和 IDE 自动补全功能，提升了开发效率和代码质量。
6. **与流行工具集成**：
    - NestJS 可以与 TypeORM、Mongoose、Sequelize 等 ORM/ODM 工具集成，支持数据库操作。
    - 还可以与 Passport、JWT、GraphQL、Redis、CQRS、Swagger 等流行库和工具结合使用。
7. **灵活的路由和中间件**：
    - NestJS 提供强大的路由系统，支持定义路由、请求过滤器、管道（Pipes）、守卫（Guards）、拦截器（Interceptors）等功能。


**基本架构**

1. **模块（Module）**：
    - 每个 NestJS 应用由多个模块组成，模块是应用的基本构建块。每个模块可以包含服务、控制器、提供者等内容。每个 NestJS 应用必须有一个根模块（通常是 `AppModule`）。
2. **控制器（Controller）**：
    - 控制器处理客户端请求，通常使用装饰器来定义路由和处理方法。控制器的职责是接受请求并返回响应。
3. **服务（Service）**：
    - 服务是业务逻辑的核心，负责处理请求的具体操作。它通常由控制器调用，服务类可以通过构造函数注入所需要的依赖（如数据库操作）。
4. **提供者 （Provider）** ：
	- Provider 是一个更广泛的概念，任何可以用 `@Injectable()` 装饰的类都是 Provider。
	- 服务类是最常见的 Provider 类型，还包括仓库（Repository）、工厂（Factory）、助手（Helper）等。
5. **中间件（Middleware）**：
    - NestJS 支持中间件，可以在请求生命周期的不同阶段执行代码，比如请求验证、日志记录、身份认证等。
6. **管道（Pipes）**：
    - 管道主要用于数据验证、数据转换等功能，在处理请求之前对数据进行预处理。
7. **守卫（Guards）**：
    - 守卫用于控制请求是否能够进入特定的路由处理函数，通常用于身份验证和权限控制。
8. **拦截器（Interceptors）**：
    - 拦截器用于修改请求和响应，通常用于缓存、日志记录、性能计时等。

## TypeScript 语法依赖


### 装饰器（Decorators）

Nest.js 重度依赖 TypeScript 装饰器来实现声明式编程，关键装饰器：
- `@Controller()`：定义控制器类。
- `@Injectable()`：标记可注入的提供者（如服务）。
- `@Module()`：定义模块。
- `@Get()`, `@Post()` 等：定义路由处理器。
- `@Inject()`：显式依赖注入。

**依赖的 TS 配置**：
```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```


### 泛型（Generics）

用于通用组件设计，如：
- `Repository<T>`（TypeORM 集成）。
- `CrudService<T>`。

```ts
@Injectable()
export class BaseService<T> {
  constructor(private repository: Repository<T>) {}
}
```

### 模块（Namespaces/Modules）

通过 ES6 模块化（`import/export`）组织代码，但较少使用 TS 的 `namespace`。



## 核心概念

两个重头的概念：**控制反转**（IoC）与**切面编程**（AOP），其中：
- `Controller`、`Provider`、`Module` 是依赖注入的能力实现；
- `Exception Filters`、`Pipes`、`Guards`、`Interceptors` 是 AOP 的能力实现；


### 依赖注入能力

在传统面向对象编程中，我们自己主动获取依赖对象，而 IoC 中不需要关心依赖者实例，其依靠 IoC 容器自动注入依赖对象，因此称之为控制反转。 依赖注入（DI）是实现 IoC 最常用的一种方式。

Nest 中实现依赖注入主要靠三个概念：
- Module 是依赖图的节点，其入节点是 Module，出节点是 Provider；
- Provider 是被注入的最小逻辑，真正被共享的逻辑均放在 Provider 中；
- Controller 是贴近路由的逻辑，用于承接依赖注入以及各种 AOP 中间件；

#### Controller

MVC 中的 C，有众多装饰器，实质上都是对 Express 各种参数的解析。例如：参数装饰器（`@Req`、`@Res` 等），状态码、Header 装饰器 （`@HttpCode`、`@Header` 等）。路由解析逻辑基本被封闭在 Controller 中，Nest 使用了装饰器路由取代了 Express 中的配置式路由，书写便捷也更为直观。

#### Provider

是 MVC 中的 M，数据或者服务提供者（数据库 ORM 封装，三方服务 RPC 封装等等）。Provider 是 IoC 中被注入的基本逻辑，因此各方依赖的**公共逻辑应该封装在 Provider**。

Nest 在这里处理了依赖的传递性，即『我依赖的依赖还是我的依赖』，例如 A → B → C，那么 C 也可以被 Inject 到 A 里。

#### Module

Module 是**依赖关系图的节点**，Module 内部又包含了自己的 Controller 与 Provider，如果想给其他的 Module 提供 Provider 需要 exports；如果注入了其他模块的 Provider，则可以在 Controller 和自己的 Service 里通过构造函数随意使用。

Nest 通过 forwardRef，前向引用技术来避免循环依赖。

有意思的一点是模块注册时虽然可以用 register 方法，但 Nest 还是提供了一些 alias，例如：`forRoot/forChild/forFeature(Async)` 等等，用于表示应该在根模块导入、子模块导入、部分导入，以及是不是动态模块。

由于大部分模块可能依赖其他公共模块的注入（例如配置模块、日志模块），所以大概率都是动态的。



### 切面编程能力

AOP 范式很多，例如 Webpack、Vue 中的 Plugin，Redux、Express 中的 Middleware。AOP 的一般玩儿法是抽象出框架内部的生命周期，对外暴露可注入的钩子（钩子可以细分为阻断非阻断、异步同步、串行并行等，因此 Webpack 才使用了 tapable 作为管理）。

#### 三个 SRP

下面三个功能的设计都符合单一职责原则（SRP），可以让代码更为 DRY：
- **Exception Filters**：异常过滤器可以横向捕获生命周期中产生的异常，并对其进行统一的处理。例如统一的异常日志处理等等。
- **Pipes**：管道用于处理参数转换和验证。常见情况是项目中全局使用 ValidationPipe，只要 body 上有对应的 Dto 即可自动完成验证。
- **Guards**：通常用来处理授权和身份验证，每个 guards 必须实现 canActivate，返回一个布尔值。

#### 两个中间件

Interceptor vs Middleware。其实 Interceptor 能干的事情，Middleware 基本都能干，那这俩有啥不同？

1. Interceptor 的逻辑是通过装饰器应用到 Controller 里的（可以 Route 层面共用）；Middleware 逻辑放在 Module 中，要单独配置路由逻辑，跟 Nest 的装饰器路由对比割裂；
2. Middleware 可以兼容现有的 Express 中间件，如 `body-parser` 等；Interceptor 不能；
3. Interceptor 可以返回 `Observable`；Middleware 只能 next；


## 基本用法

NestJS 是一个用于构建高效、可扩展 Node.js 服务器端应用程序的框架，它采用了模块化的架构设计。


### Nest CLI

Nest CLI 是 NestJS 框架的官方命令行工具，用于快速创建、开发和管理 NestJS 应用程序，提供多种包括搭建项目框架、在开发模式下提供服务以及构建和打包应用程序以供生产分发等功能，以鼓励构建结构良好的应用程序。

#### Installation

```bash
npm install -g @nestjs/cli
```

#### Basic Workflow

1. **查看命令详情**
```bash
nest --help
```

2. **创建一个新的 Nest 项目**
```bash
nest new <name> [options]
nest n <name> [options]
```

3. **根据 Schema 生成或者修改文件**
```bash
nest generate <schematic> <name> [options]
nest g <schematic> <name> [options]
```
- `<schematic>` 
	- `app`
	- `controller`
	- `class`
	- `decorator`
	- `interface`
	- `module`
	- `provider`
	- `resource`
	- `service`


4. **构建应用**
```bash
nest build <name> [options]
```
- `[options]`
	- `--path [path]` ： 指定 `tsconfig` 的路径
	- `--config [path]` ：指定 `nest-cli` 配置的路径
	- `--watch` ：监视模式运行，热重载
	- `--builder [name]` ：指定构建工具进行编译（如 `tsc`, `swc`,  `webpack`）
	- `--webpack` ：（**deprecated**）用  `--builder webpack` 替换
	- `--webpackPath` ：指定 webpack 配置的路径









5. **构建并启动应用**
```bash
nest start <name> [options]
```
- `[options]`
	- `--path [path]` ： 指定 `tsconfig` 的路径
	- `--config [path]` ：指定 `nest-cli` 配置的路径
	- `--watch` ：监视模式运行，热重载
	- `--builder [name]` ：指定构建工具进行编译（如 `tsc`, `swc`,  `webpack`）
	- `--debug [hostport]` ：在 Debug 模式下运行
	- `--env-file` ：加载环境变量文件








### Mongoose

```bash
npm install --save @nestjs/mongoose mongoose
```

#### Schema

在 Mongoose 中，每个 `Schema` 都会映射到 MongoDB 的一个集合，并定义集合内文档的结构。`Schema` 被用来定义模型，而模型负责从底层创建和读取 MongoDB 的文档。

可以用 NestJS 内置的装饰器 `@Schema` 来创建（也可以使用 Mongoose 的常规方式），使用装饰器来创建 `Schema` 会极大减少引用并且提高代码的可读性。

```typescript
// schemas/cat.schema.ts
import { Prop, Schema, SchemaFactory } from '@nestjs/mongoose';
import { HydratedDocument } from 'mongoose';

export type CatDocument = HydratedDocument<Cat>;

@Schema()
export class Cat {
  @Prop()
  name: string;

  @Prop()
  age: number;

  @Prop()
  breed: string;
}

export const CatSchema = SchemaFactory.createForClass(Cat);
```

- **`@Schema`** ：标记一个类作为 `Schema` 定义，它将 `Cat` 类映射到 MongoDB 的一个同名复数集合 Cats，这个装饰器接受一个可选的 `Schema` 对象。
- **`@Prop`** ：定义文档中的一个属性。


#### 注册 Register

```typescript
// cats.module.ts
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';
import { Cat, CatSchema } from './schemas/cat.schema';

@Module({
  imports: [MongooseModule.forFeature([{ name: Cat.name, schema: CatSchema }])],
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```

- `MongooseModule.forFeature`
	- 配置模块，包括定义哪些模型应该注册在当前范围中。
	- 如果想在另外的模块中使用这个模型，将 `MongooseModule` 添加到 `CatsModule` 的 `exports` 部分并在其他模块中导入 `CatsModule` 。

#### 注入 Inject

```typescript
// cats.service.ts
import { Model } from 'mongoose';
import { Injectable } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Cat } from './schemas/cat.schema';
import { CreateCatDto } from './dto/create-cat.dto';

@Injectable()
export class CatsService {
  constructor(@InjectModel(Cat.name) private catModel: Model<Cat>) {}

  async create(createCatDto: CreateCatDto): Promise<Cat> {
    const createdCat = new this.catModel(createCatDto);
    return createdCat.save();
  }

  async findAll(): Promise<Cat[]> {
    return this.catModel.find().exec();
  }
}

```

- **`@InjectModel()`** ：将 `Cat` 模型注入到 `CatsService` 中


#### 连接 Connection

1. `MongooseModule.forRoot` 
	- **同步配置**：用于直接提供配置对象
	- **适用于静态配置**：当你的数据库连接参数是固定的或直接从环境变量读取时使用
	- **简单直接**：配置立即生效

```typescript
@Module({
  imports: [
    MongooseModule.forRoot('mongodb://localhost/nest'),
    // 或使用配置对象
    MongooseModule.forRoot({
      uri: 'mongodb://localhost/nest',
      useNewUrlParser: true,
      useUnifiedTopology: true
    }),
  ],
})
export class AppModule {}
```

2. `MongooseModule.forRootAsync` 
	- **异步配置**：用于动态获取配置
	- **适用于需要从服务或工厂函数获取配置的场景**：如从配置服务、环境变量服务或 secrets 管理服务获取
	- **更灵活**：支持三种注入方式：
	    1. `useFactory` - 通过工厂函数返回配置
	    2. `useClass` - 使用提供者类
	    3. `useExisting` - 使用已存在的提供者

```typescript
@Module({
  imports: [
    MongooseModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: async (configService: ConfigService) => ({
        uri: configService.get('MONGO_URI'),
        useNewUrlParser: true,
        useUnifiedTopology: true
      }),
      inject: [ConfigService],
    }),
  ],
})
export class AppModule {}
```


### Bull

Bull 是一个流行的、支持良好的、高性能的基于 Node.js 的消息队列系统应用，使用 Redis 持久化工作数据。

由于基于 Redis 的，你的队列结构可以是完全分布式的并且和平台无关。例如，你可以有一些队列生产者、消费者和监听者，他们运行在 Nest 的一个或多个节点上，同时，其他生产者、消费者和监听者在其他 Node.js 平台或者其他网络节点上。

```bash
npm install --save @nestjs/bull bull
npm install --save-dev @types/bull
```



### GraphQL

```bash
npm i @nestjs/graphql @nestjs/apollo graphql apollo-server-express
```






### 模板项目

1. [nestjs-boilerplate](https://github.com/brocoders/nestjs-boilerplate)

