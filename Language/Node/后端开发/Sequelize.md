
地址：[Sequelize 简介 | Sequelize中文文档 | Sequelize中文网](https://www.sequelize.cn/)

Sequelize 是一个基于 promise 的 Node.js ORM，目前支持 Postgres，MySQL，MariaDB，SQLite 以及 Microsoft SQL Server。它具有强大的事务支持，关联关系，预读和延迟加载，读取复制等功能。

Sequelize is a modern TypeScript and Node.js ORM for Oracle, Postgres, MySQL, MariaDB, SQLite and SQL Server, and more. Featuring solid transaction support, relations, eager and lazy loading, read replication and more.

# 连接数据库

## 代码示例

```js
const { Sequelize } = require('sequelize');

// 方法 1: 传递一个连接 URI
const sequelize = new Sequelize('sqlite::memory:') // Sqlite 示例
const sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname') // Postgres 示例


// 方法 2: 分别传递参数 (sqlite)
const sequelize = new Sequelize({
  dialect: 'sqlite',
  storage: 'path/to/database.sqlite'
});

const sequelize = new Sequelize({
  dialect: 'mssql',
  host: 'your_sql_server_host',
  port: 'your_sql_server_port',
  username: 'your_sql_server_username',
  password: 'your_sql_server_password',
  database: 'your_database_name',
  dialectOptions: {
    options: {
      encrypt: true, // for Azure
    },
  },
}); // mssql 示例


// 方法 3: 分别传递参数 (其它数据库)
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: /* one of 'mysql' | 'postgres' | 'sqlite' | 'mariadb' | 'mssql' | 'db2' | 'snowflake' | 'oracle' */
});
```

## 数据库默认端口

关系型数据库：
1. MSSQL：1433
2. MySQL：3306
3. PostgreSQL：5432
4. Oracle：1521
5. DB2：5000

NoSQL数据库：
1. Redis：6379
2. Memcached：11211
3. MongoDB：27017


# 基本概念

## 模型 Model

模型是 Sequelize 的本质，**模型是代表数据库中表的抽象**，是一个 Model 的继承类，该模型告诉 Sequelize 有关它所代表的实体，例如数据库中表的名称以及它具有的列（及其数据类型）。

### 定义 Model 结构

**方法一**：直接调用 `sequelize.define(modelName, attributes, options)`
```js
const { Sequelize, DataTypes } = require('sequelize');
const sequelize = new Sequelize('sqlite::memory:');

const User = sequelize.define('User', {
  // 在这里定义模型属性
  firstName: {
    type: DataTypes.STRING,
    allowNull: false
  },
  lastName: {
    type: DataTypes.STRING
    // allowNull 默认为 true
  }
}, {
  // 这是其他模型参数
});

// `sequelize.define` 会返回模型
console.log(User === sequelize.models.User); // true
```
在内部，`sequelize.define` 调用 `Model.init`，因此两种方法本质上是等效的。


**方法二**：继承 `Model` 类，然后调用 `init()` 方法。
```js
const { Sequelize, DataTypes, Model } = require('sequelize');
const sequelize = new Sequelize('sqlite::memory:');

class User extends Model {}

User.init({
  // 在这里定义模型属性
  firstName: {
    type: DataTypes.STRING,
    allowNull: false
  },
  lastName: {
    type: DataTypes.STRING,
    defaultValue: "John Doe"       // Sequelize 假定列的默认值为 `NULL`
    // allowNull 默认为 true
  }
}, {
  // 这是其他模型参数
  sequelize,                // 传递连接实例
  modelName: 'User',        // 模型名称
  tableName: 'Employees',   // 表名
  timestamps: true,         // 启用时间戳
  createdAt: false,         // 去掉 createdAt
  updatedAt: 'updateTimestamp' // 想要 updatedAt 但是希望名称叫做 updateTimestamp  
});

// 定义的模型是类本身
console.log(User === sequelize.models.User); // true
```


利用模型作为类：
```js
class User extends Model {
  static classLevelMethod() {
    return 'foo';
  }
  instanceLevelMethod() {
    return 'bar';
  }
  getFullname() {
    return [this.firstname, this.lastname].join(' ');
  }
}
User.init({
  firstname: Sequelize.TEXT,
  lastname: Sequelize.TEXT
}, { sequelize });

console.log(User.classLevelMethod()); // 'foo'
const user = User.build({ firstname: 'Jane', lastname: 'Doe' });
console.log(user.instanceLevelMethod()); // 'bar'
console.log(user.getFullname()); // 'Jane Doe'
```

### 创建 Model 实例

**方法一**：首先使用 `.build()` 方法，然后调用 `.save()` 进行持久化。创建实例时，不能用 `new` 。
```js
const jane = User.build({ firstname: "Jane" });
console.log(jane instanceof User);                // true
console.log(jane.name);                           // "Jane"

await jane.save();
```
**几乎每个 Sequelize 方法都是异步的**，`.build` 是极少数例外之一。


**方法二**：直接使用 `.create()` 方法,该方法将上述的 `.build()` 方法和 `.save()` 方法合并为一个方法：
```js
const jane = await User.create({ name: "Jane" });
// console.log(jane);          // 不要这样!
console.log(jane.toJSON());    // 这样最好!
console.log(JSON.stringify(jane, null, 4));   // 这样也不错!
```


### 同步 Model

定义模型时，你要告诉 Sequelize 有关数据库中表的一些信息。但是，**如果该表实际上不存在于数据库中怎么办？如果存在，但具有不同的列，较少的列或任何其他差异，该怎么办？**

> 这是后端开发时，数据库常见的问题。

Sequelize 有一个异步函数 `model.sync()` ，通过此调用，Sequelize 将自动对数据库执行 SQL 查询。 请注意，这仅更改数据库中的表，而不更改 JavaScript 端的模型。

- `.sync()` ：如果表不存在，则创建该表（如果已经存在，则**不执行任何操作**）。
- `.sync({ force: true })` ：将创建表，如果表已经存在，则**首先将其删除**。
- `.sync({ alter: true })` ：这将检查数据库中表的当前状态（它具有哪些列，它们的数据类型等），然后在表中进行必要的更改以使其与模型匹配。


**方法一**：每个模型都调用一次 `.sync()` 
```js
await User.sync({ force: true });
```

**方法二**：使用连接实例 sequenlize 一次同步所有的模型。
```js
await sequelize.sync({ force: true });
```


#### 相关问题

`.sync({ force: true })` 和 `.sync({ alter: true })` 可能是**破坏性操作**。因此，**不建议将它们用于生产级软件中。**

> 应该可以用 `.sync()`。

推荐在 Sequelize CLI 的帮助下使用高级概念 Migrations（迁移） 进行[生产环境同步](#Migrations%20迁移)。



## 外键

```js
const { Model, DataTypes, Deferrable } = require("sequelize");

class Foo extends Model {}

Foo.init({

  // 创建外键：
  bar_id: {
    type: DataTypes.INTEGER,

    references: {
      // 这是对另一个模型的参考
      model: Bar,

      // 这是引用模型的列名
      key: 'id',

      // 使用 PostgreSQL,可以通过 Deferrable 类型声明何时检查外键约束.
      deferrable: Deferrable.INITIALLY_IMMEDIATE
      // 参数:
      // - `Deferrable.INITIALLY_IMMEDIATE` - 立即检查外键约束
      // - `Deferrable.INITIALLY_DEFERRED` - 将所有外键约束检查推迟到事务结束
      // - `Deferrable.NOT` - 完全不推迟检查(默认) - 这将不允许你动态更改事务中的规则
    }
  },
}, 
{
  sequelize,
  modelName: 'foo',
});
```




## 数据类型


# 关联

Sequelize 支持标准关联关系：一对一，一对多和多对多，为此，Sequelize 提供了四种关联类型，并将它们组合起来以创建关联：
- `A.hasOne(B)` ：关联意味着 A 和 B 之间存在一对一的关系，外键在目标模型 B 中定义。
- `A.belongsTo(B)` ：关联意味着 A 和 B 之间存在一对一的关系，外键在源模型 A 中定义。
- `A.hasMany(B)` ：关联意味着 A 和 B 之间存在一对多关系，外键在目标模型 B 中定义。
- `A.belongsToMany(B, { through: 'C' })` ：关联意味着将表 C 用作关系表，在 A 和 B 之间存在多对多关系。Sequelize 将**自动**创建模型 C （除非已经存在），并在其上定义适当的外键（如，aId 和 bId）。

Sequelize 会**自动**将外键添加到适当的模型中，除非它们已经存在。上述方法调用时，Sequelize 将自动添加额外的方法到模型中，如：
- 对于 `Foo.hasOne(Bar)` ，会添加：
	- `fooInstance.getBar()`
	- `fooInstance.setBar()`
	- `fooInstance.createBar()`

其中，`get`、`set`、`create` 后面默认是模型名称，也可以指定别名，如：
```js
Ship.belongsTo(Captain); // 这将在 Ship 中创建 `captainId` 外键.
const hisShip = await awesomeCaptain.getShip();


Ship.belongsTo(Captain, { as: 'leader', foreignKey: 'bossId' });
const hisShip = await ship.getLeader();
```

**必要时，可以使用复数形式**，例如 `fooInstance.setBars()`。同样，**不规则复数也由 Sequelize 自动处理**。例如，Person 变成 People 或者 Hypothesis 变成 Hypotheses。



## 一对一关系

创建一对一关系时，`.hasOne()` 和 `.belongsTo()` 一起成对使用。

```js
// Sequelize 推断出要创建的外键在Bar中，应称为 fooId
Foo.hasOne(Bar);
Bar.belongsTo(Foo);

// 指定外键名为 myFooId
Foo.hasOne(Bar, {  
	foreignKey: 'myFooId'  
});  
Bar.belongsTo(Foo);


```

## 一对多关系

创建一对多关系时，`.hasMany()` 和 `.belongsTo()` 一起使用。在数据库中，一对多的关系是在多的一方表中添加外键，外键是少的一方的主键或者唯一键。

```js
Team.hasMany(Player);
Player.belongsTo(Team);
```

例如，如果一个 Foo 有很多 Bar （因此每个 Bar 都属于一个 Foo），那么唯一明智的方式就是在 Bar 表中有一个 fooId 列。

## 多对多关系

创建多对多关系时，两个 `.belongsToMany()` 一起使用。在数据库表中，多对多关系的实现只有添加关系表。

```js
const Movie = sequelize.define('Movie', { name: DataTypes.STRING });
const Actor = sequelize.define('Actor', { name: DataTypes.STRING });
Movie.belongsToMany(Actor, { through: 'ActorMovies' });
Actor.belongsToMany(Movie, { through: 'ActorMovies' });
```


## 关联查询


如，设置模型：
```js
const Ship = sequelize.define('ship', {
  name: DataTypes.STRING,
  crewCapacity: DataTypes.INTEGER,
  amountOfSails: DataTypes.INTEGER
}, { timestamps: false });

const Captain = sequelize.define('captain', {
  name: DataTypes.STRING,
  skillLevel: {
    type: DataTypes.INTEGER,
    validate: { min: 1, max: 10 }
  }
}, { timestamps: false });

Captain.hasOne(Ship);
Ship.belongsTo(Captain);
```

### 预先加载

```js
const awesomeCaptain = await Captain.findOne({
  where: {
    name: "Jack Sparrow"
  },
  include: Ship
});

// 现在 ship 跟着一起来了
console.log('Name:', awesomeCaptain.name);
console.log('Skill Level:', awesomeCaptain.skillLevel);
console.log('Ship Name:', awesomeCaptain.ship.name);
console.log('Amount of Sails:', awesomeCaptain.ship.amountOfSails)
```

### 延迟加载

```js
const awesomeCaptain = await Captain.findOne({
  where: {
    name: "Jack Sparrow"
  }
});

// 只获取到的 captain
console.log('Name:', awesomeCaptain.name);
console.log('Skill Level:', awesomeCaptain.skillLevel);

// 再获取有关他的 ship 的信息
const hisShip = await awesomeCaptain.getShip();
console.log('Ship Name:', hisShip.name);
console.log('Amount of Sails:', hisShip.amountOfSails);
```



### 相同模型关联查询

```js
Team.hasOne(Game, { as: 'HomeTeam', foreignKey: 'homeTeamId' });
Team.hasOne(Game, { as: 'AwayTeam', foreignKey: 'awayTeamId' });
Game.belongsTo(Team);
```


# CRUD

SELECT 查询：
```sql
SELECT * FROM ...
```

```js
const users = await User.findAll();
```

UPDATE 查询：
```js
// 将所有没有姓氏的人更改为 "Doe"
await User.update({ lastName: "Doe" }, {
  where: {
    lastName: null
  }
});
```


DELETE 查询：
```js
// 删除所有名为 "Jane" 的人 
await User.destroy({
  where: {
    firstName: "Jane"
  }
});
```


## 分页

使用 `limit` 和 `offset` 参数可以进行限制和分页：
```js
// 跳过8个实例,然后获取5个实例
Project.findAll({ offset: 8, limit: 5 });
```



# 事务


Sequelize 支持两种使用事务的方式：
1. **非托管事务**：提交和回滚事务应由用户手动完成（通过调用适当的 Sequelize 方法）。
2. **托管事务**：如果引发任何错误，Sequelize 将自动回滚事务，否则将提交事务。


## 非托管事务

```js
// 首先, 我们从你的连接开始一个事务并将其保存到一个变量中
const t = await sequelize.transaction();

try {

  // 然后,我们进行一些调用以将此事务作为参数传递:

  const user = await User.create({
    firstName: 'Bart',
    lastName: 'Simpson'
  }, { transaction: t });

  await user.addSibling({
    firstName: 'Lisa',
    lastName: 'Simpson'
  }, { transaction: t });

  // 如果执行到此行,且没有引发任何错误.
  // 我们提交事务.
  await t.commit();

} catch (error) {

  // 如果执行到达此行,则抛出错误.
  // 我们回滚事务.
  await t.rollback();

}
```


## 托管事务

```js
try {
  const result = await sequelize.transaction(async (t) => {
    const user = await User.create({
      firstName: 'Abraham',
      lastName: 'Lincoln'
    }, { transaction: t });
    await user.setShooter({
      firstName: 'John',
      lastName: 'Boothe'
    }, { transaction: t });
    return user;
  });

  // 如果执行到此行,则表示事务已成功提交,`result`是事务返回的结果
  // `result` 就是从事务回调中返回的结果(在这种情况下为 `user`)

} catch (error) {

  // 如果执行到此,则发生错误.
  // 该事务已由 Sequelize 自动回滚！

}
```


# Migrations 迁移

[迁移 | Sequelize中文文档 | Sequelize中文网](https://www.sequelize.cn/other-topics/migrations)

## 项目配置

1. 创建项目：数据库迁移是属于一个生产动作，因此要创建一个新项目，使用 `npm init` 初始化。
```
npm init
npm install --save sequelize tedious
```

2. 安装 sequelize-cli
```
npm install --save sequelize-cli
```

3. 初始化
```
npx sequelize-cli init
```

此时项目文件夹下会有如下结构：
```
. 
	├── config\
	│ ├── config.js
	│ └── config.json
	├── migrations\ 
	├── models\
	│ └── index.js 
	├── node_modules\
	├── seeders\
	├── .gitignore
	├── .sequenlizerc
	├── package-lock.json
	└── package.json
```

**目录文件说明**：

- **config**：主要提供链接数据所需要的一些配置。其中，默认是使用 `config.json` 进行配置，其内容如下：
```json
{
    "development": {
        "username": "root",
        "password": null,
        "database": "database_development",
        "host": "127.0.0.1",
        "dialect": "mssql"
    },
    "test": {
        "username": "root",
        "password": null,
        "database": "database_test",
        "host": "127.0.0.1",
        "dialect": "mssql"
    },
    "production": {
        "username": "root",
        "password": null,
        "database": "database_production",
        "host": "127.0.0.1",
        "dialect": "mssql"
    }
}
```
配置文件中默认会有三个不同环境的配置：development、test、production，分别对应：开发环境、测试环境、生产环境（可以根据具体情况增减）。

当然也可以配合 `.sequenlizerc` 文件，指定使用 `config.js`，如在 `.sequenlizerc` 文件中配置：
```js
// .sequenlizerc

const path = require("path");

module.exports = {
    config: path.resolve("config", "config.js"),
};
```

这样就可以使用 `config.js` 进行配置，使用 js 文件的形式更加灵活：
```js
// config.js

const path = require("path");
const dotenv = require("dotenv");

dotenv.config({ path: path.join(__dirname, "../../.env.local") });

module.exports = {
    development: {
        username: process.env.DB_USERNAME,
        password: process.env.DB_PASSWORD,
        database: process.env.DB_NAME,
        host: process.env.DB_HOST,
        port: process.env.DB_PORT,
        dialect: "mssql",
        dialectOptions: {
            bigNumberStrings: true,
        },
    },
};

```

之后可以要更改 `model/index.js` 文件，使用 `require` 来加载新的配置文件 `config.js`：
```js
const config = require('../config/config');
```

同时配合当前系统环境变量 NODE_ENV 自动载入不同的配置。

- **migrations**：迁移脚本，数据库中的每一张表、字段的建立，以及后续的更新都是通过执行迁移脚本来完成的。一般脚本的格式是：
```js
'use strict';

/** @type {import('sequelize-cli').Migration} */
module.exports = {
  async up (queryInterface, Sequelize) {
    /**
     * Add altering commands here.
     *
     * Example:
     * await queryInterface.createTable('users', { id: Sequelize.INTEGER });
     */
  },

  async down (queryInterface, Sequelize) {
    /**
     * Add reverting commands here.
     *
     * Example:
     * await queryInterface.dropTable('users');
     */
  }
};

```


- **models**：模型文件，描述数据库表/字段的文件。这个主要用于项目实际的业务代码中使用，如果只是迁移或者后续的种子，可不需要模型。

- **seeders**：种子脚本，初始化的一些测试数据。


## 相关命令

1. 帮助命令：`npx sequelize-cli --help`
![](imgs/Pasted%20image%2020240321114528.png)

2. 创建数据库：`npx sequelize-cli db:create`
3. 销毁数据库：`npx sequelize-cli db:drop`
4. 创建模型：`npx sequelize-cli model:generate --name [model name] --attributes firstName:string,lastName:string,email:string`
5. 创建迁移脚本：`npx sequelize-cli migration:create --name [script name]`
6. 开始迁移：`npx sequelize-cli db:migrate --env [env name]` 
7. 撤销迁移：`npx sequelize-cli db:migrate:undo --env [env name]` 


## 迁移脚本格式

一般通过命令 `npx sequelize-cli migration:create --name [script name]` 直接创建的迁移脚本的格式是：
```js
'use strict';

/** @type {import('sequelize-cli').Migration} */
module.exports = {
  async up (queryInterface, Sequelize) {
    /**
     * Add altering commands here.
     *
     * Example:
     * await queryInterface.createTable('users', { id: Sequelize.INTEGER });
     */
  },

  async down (queryInterface, Sequelize) {
    /**
     * Add reverting commands here.
     *
     * Example:
     * await queryInterface.dropTable('users');
     */
  }
};

```

一个迁移脚本有两个方法 `up` 或 `down` ，一般这两个方式是对称操作：

- 一个创建表另外一个就是销毁：
```js
"use strict";
/** @type {import('sequelize-cli').Migration} */
module.exports = {
    async up(queryInterface, Sequelize) {
        await queryInterface.createTable("Users", {
            id: {
                allowNull: false,
                autoIncrement: true,
                primaryKey: true,
                type: Sequelize.INTEGER,
            },
            firstName: {
                type: Sequelize.STRING,
            },
            lastName: {
                type: Sequelize.STRING,
            },
            email: {
                type: Sequelize.STRING,
            },
            createdAt: {
                allowNull: false,
                type: Sequelize.DATE,
            },
            updatedAt: {
                allowNull: false,
                type: Sequelize.DATE,
            },
        });
    },
    async down(queryInterface, Sequelize) {
        await queryInterface.dropTable("Users");
    },
};

```

- 如果一个增加列，另外一个就是删除列：
```js
'use strict';

module.exports = {
  up: async (queryInterface, Sequelize) => {
    /**
     * Add altering commands here.
     *
     * Example:
     * await queryInterface.createTable('users', { id: Sequelize.INTEGER });
     */
    await queryInterface.addColumn(
      'users',
      'age', {
        type: Sequelize.TINYINT
      }
    )
  },

  down: async (queryInterface, Sequelize) => {
    /**
     * Add reverting commands here.
     *
     * Example:
     * await queryInterface.dropTable('users');
     */
    await queryInterface.removeColumn(
      'users',
      'age'
    )
  }
};

```


`queryInterface` 是 Sequelize 提供的一个用于执行数据库查询和操作的对象，通常在 Sequelize 的迁移文件中使用，用于创建、修改和删除数据库表以及执行其他数据库操作。
常用命令有：
- **创建表**：`queryInterface.createTable('tableName', { column1: Sequelize.STRING, column2: Sequelize.INTEGER });`
- **删除表**：`queryInterface.dropTable('tableName');`
- **添加列**：`queryInterface.addColumn('tableName', 'columnName', Sequelize.STRING);`
- **删除列**：`queryInterface.removeColumn('tableName', 'columnName');
- **修改列**：`queryInterface.changeColumn('tableName', 'columnName', { type: Sequelize.STRING, allowNull: false });`
- **添加索引**：`queryInterface.addIndex('tableName', ['column1', 'column2']);`
- **删除索引**：`queryInterface.removeIndex('tableName', ['column1', 'column2']);`
- **执行原始 SQL 查询**：`queryInterface.sequelize.query('SELECT * FROM tableName');`


`Sequelize` 是 Sequelize 库的一个对象，它包含了 Sequelize 提供的各种数据类型和操作方法。通过使用 Sequelize 对象，你可以定义模型的属性、创建数据表、定义索引等等。


## 推荐工作流

1. 首先初始化，运行命令
```
npx sequelize-cli init
```
然后配置 `.sequenlizerc` 文件，更改为 `config.js` 文件，配置 `.env` 环境，这样一些私密配置就可以不用暴露。

2. 根据情况，创建数据库
```
npx sequelize-cli db:create
```

3. 对于新建一个表，需要创建模型，使用命令：
```
npx sequelize-cli model:generate --name User --attributes name:string,gender:bool
```
这个命令会在 `models` 文件夹中创建了一个 `user` 模型文件，然后在 `migrations` 文件夹中创建了一个名字像 `[date time]-create-user.js` 的迁移文件。

但是，这种使用 CLI 的写法过于繁琐，一般先用 CLI 建立个 base 版，再在执行迁移命令之前，手动同步修改 `models` 和 `migrations`。

4. 执行迁移命令，**推荐同时指定环境**：
```
npx sequelize-cli db:migrate --env development
```
此命令将完成如下动作：
- 在数据库中建立一个名为 `SequelizeMeta` 的表，此表用于记录在当前数据库上运行的迁移。
- 开始寻找尚未运行的任何迁移文件，这可以通过检查 `SequelizeMeta` 表。如，运行新创建的 `[date time]-create-user.js` 迁移脚本。
- 如果是创建模型，创建一个名为 `Users` 的表，其中包含其迁移文件中指定的所有列。


5. 后续如果对表的格式有变动，可以继续创建迁移脚本：
```
npx sequelize-cli migration:create --name [script name]
```
此命令将在 `migrations` 文件夹中创建了一个名字像 `[date time]-[script name].js` 的迁移文件，然后在该文件下补充相应的命令。




# Sequenlize 安装

1. 首先安装 Sequenlize
```
npm install --save sequelize
```

2. 必须手动为所选数据库安装驱动程序
```
# 选择以下之一:
$ npm install --save pg pg-hstore # Postgres
$ npm install --save mysql2
$ npm install --save mariadb
$ npm install --save sqlite3
$ npm install --save tedious # Microsoft SQL Server
$ npm install --save oracledb # Oracle Database
```
