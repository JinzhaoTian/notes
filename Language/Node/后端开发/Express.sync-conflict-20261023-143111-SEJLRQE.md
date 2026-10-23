
Express 是一个用于 Node.js 的快速、不拘一格、极简主义的 Web 框架，是一个保持最小规模的灵活的 Node.js Web 应用程序开发框架，为 Web 和移动应用程序提供一组强大的功能。

Express 是最受欢迎的、基于 MVC 的 Node.js 框架，它有许多与 Nodejs 同步的库和组件，以创建漂亮而强大的动态 Web 应用程序。 Express 提供了所有 HTTP 实用方法、函数和中间件，可帮助开发人员编写健壮的 API。

Express 通过自己的路由机制完成模块化开发，根据功能或者角色或者其他依据，将模块进行拆分，最后在 app.js 入口模块中进行统一的注册引入。其自身只具有最低程度的功能：Express 应用程序基本上是一系列中间件函数调用。


Express 框架核心特性：
- 可以设置中间件来响应 HTTP 请求。
- 定义了路由表用于执行不同的 HTTP 请求动作。
- 可以通过向模板传递参数来动态渲染 HTML 页面。

![](_imgs/Pasted%20image%2020240115094131.png)


**Express框架建立在node.js内置的http模块上**。


## Middleware

**Express 应用是由一系列中间件构成的**。简单说，中间件（middleware）就是处理HTTP请求的函数。它最大的特点就是，一个中间件处理完，再传递给下一个中间件。App实例在运行过程中，会调用一系列的中间件。

**每个中间件可以从App实例，接收三个参数，依次为request对象、response对象，next回调函数**。每个中间件都可以对HTTP请求（request对象）进行加工，并且决定是否调用next方法，将request对象再传给下一个中间件。

```js
var express = require("express");
var app = express();

app.use(function(request, response, next) {
  console.log("In comes a " + request.method + " to " + request.url);
  next();
});

app.use(function(request, response) {
  response.writeHead(200, { "Content-Type": "text/plain" });
  response.end("Hello world!\n");
});

app.listen(3000);
```


## 用途

1. 利用中间件可以设置 [Redis 缓存](../../../Database/Redis/使用.md#中间件)防止入侵业务代码。





## 内置方法

- `.use()` - use 是 express 注册中间件的方法，它返回一个函数，可以对访问路径进行判断，据此就能实现简单的路由，根据不同的请求网址，返回不同的网页内容。
- `.all()` - all 方法表示，路径中的所有请求都必须通过该中间件。

```js
var express = require("express");
var http = require("http");
var app = express();

app.all("*", function(request, response, next) {
  response.writeHead(200, { "Content-Type": "text/plain" });
  next();
});

app.get("/", function(request, response) {
  response.end("Welcome to the homepage!");
});

app.get("/about", function(request, response) {
  response.end("Welcome to the about page!");
});

app.get("*", function(request, response) {
  response.end("404!");
});

http.createServer(app).listen(1337);
```

HTTP动词：
- `.get()` - 这些方法的第一个参数，都是请求的路径，除了绝对匹配以外，Express允许模式匹配。
- `.post()` -
- `.put()` - 
- `.delete()` - 
- ...

## Requst

- `request.ip` - 获得HTTP请求的IP地址
- `request.files` - 获取上传的文件

## Response

- `response.redirect()` - 网址的重定向
- `response.sendFile()` - 发送文件
- `response.render()` - 渲染网页模板


必要时可以对Response的方法进行重写，以完成一些额外工作。

```js
const oldJSON = res.json;
res.json = async (data) => {
	res.json = oldJSON;
	
	// do some work
	
	return res.json(data);
};
next();
```

## Router

从Express 4.0开始，路由器功能成了一个单独的组件Express.Router。
```javascript
var router = express.Router();

router.route('/api')
	.post(function(req, res) {
		// ...
	})
	.get(function(req, res) {
		// ...
	});

app.use('/', router);
```



## 代码结构

```
- controller/          定义路由和其他的逻辑
- middlewares/         定义中间件
- models/              定义模型层
- services/            定义视图层
- public/              静态资源目录
- app.js               入口文件 
- package.json         依赖管理文件
```

[不容错过的 Node.js 项目架构 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/97120305)

