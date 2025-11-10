
与 Swagger 或者 OpenAPI 相关的包有：
- `swagger-ui-express` 
	- 是一个更轻量级的库，它专注于提供 Swagger UI，以便用户可以通过浏览器访问交互式的 API 文档。
	- 需要手动编写 Express 路由来处理 API 请求，但提供了一个简单的方法来将 Swagger UI 集成到 Express 应用程序中。
	- 适用于那些希望快速添加 Swagger UI 到现有 Express 应用程序中的开发人员。
- `express-openapi` ：
	- 更全面的工具，它不仅仅提供了 Swagger UI，还提供了其他功能，例如自动验证和处理请求参数、生成交互式 API 文档等。
	- 它基于 OpenAPI 规范，并且可以直接从规范文件中生成路由和中间件来处理请求。
	- 允许您根据 OpenAPI 规范文件自动生成 Express 路由和中间件，从而简化了 API 的开发和维护过程。

#### 使用

以 OpenAPI 为例，

1. 安装依赖：
```
npm install express-openapi
```

2. 创建并编写 OpenAPI 规范文件，如：
```yaml
openapi: 3.0.0
info:
  title: Your API Title
  description: Description of your API
  version: 1.0.0

servers:
  - url: http://api.example.com/v1

paths:
  /users:
    get:
      summary: Retrieve a list of users
      description: Retrieve a list of all users from the system
      responses:
        '200':
          description: A list of users
          content:
            application/json:
              schema:
                type: array
                items:
                  type: object
                  properties:
                    id:
                      type: integer
                    username:
                      type: string
                    email:
                      type: string
    post:
      summary: Create a new user
      description: Create a new user in the system
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                username:
                  type: string
                email:
                  type: string
      responses:
        '201':
          description: User created successfully

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        username:
          type: string
        email:
          type: string

security:
  - apiKeyAuth: []

```


3. 设置 express-openapi ：
```js
const express = require('express');
const app = express();
const openapi = require('express-openapi');

const apiSpec = require('./path/to/openapi/spec.yaml'); // 导入您的 OpenAPI 规范文件

openapi.initialize({
  app,
  apiDoc: apiSpec,
  operations: {
    // 这里定义您的 API 操作
  },
});

// 启动应用程序
const port = process.env.PORT || 3000;
app.listen(port, () => {
  console.log(`Server is running on port ${port}`);
});

```


4. 实现 API 操作：
```js
const getUsers = (req, res) => {
  // 处理 GET /users 请求的逻辑
  res.status(200).json({ message: 'GET /users handler' });
};

module.exports = {
  getUsers,
  // 其他操作...
};
```
