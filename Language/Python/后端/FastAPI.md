FastAPI 是一个用于构建 API 的现代、快速（高性能）的 Web 框架，使用 Python 3.6+ 并基于标准的 Python 类型提示。

## 核心特性

1. **极高性能**
	- 可与 NodeJS 和 Go 比肩的极高性能
	- 基于 Starlette（ASGI 框架）和 Pydantic（数据验证）
2. **自动交互式 API 文档**
	- 自动生成 Swagger UI（访问 `/docs`）
	- 自动生成 ReDoc（访问 `/redoc`）
	- 无需额外编写文档，代码即文档
3. **基于 Python 类型提示**
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float
    is_offer: bool = False

@app.post("/items/")
def create_item(item: Item):
    # FastAPI 自动验证请求数据
    return {"item_name": item.name, "price": item.price}
```
4. **自动数据验证和序列化**
	- 使用 Pydantic 自动验证请求/响应数据
	- 自动转换数据类型（如字符串转整数）
5. **异步支持**
	- 原生支持 `async` / `await`
	- 可与异步数据库、第三方 API 高效协作


## 主要优势

1. **开发效率**：代码简洁，自动文档减少沟通成本
2. **类型安全**：编辑器支持自动补全、错误检测
3. **依赖注入系统**：易于管理数据库会话、认证等依赖
4. **安全性**：内置对 OAuth2、JWT 等认证方案的支持
5. **生产就绪**：支持 CORS、GZip、HTTPS 重定向等中间件



## 安装

```
pip install fastapi uvicorn
```

> Uvicorn 是基于 uvloop 和 httptools 构建的非常快速的 ASGI 服务器。 uvloop 用于替换标准库 asyncio 中的事件循环，使用 Cython 实现，它非常快，可以使 asyncio 的速度提高 2-4 倍。写异步代码离不开asyncio。httptools 是 nodejs HTTP 解析器的 Python 实现。ASGI 是异步网关协议接口，一个介于网络协议服务和 Python 应用之间的标准接口，能够处理多种通用的协议类型，包括 HTTP，HTTP2 和 WebSocket。



## Hello World

最简单的FastAPI文件<main.py>：

```python
from fastapi import FastAPI           # 导入框架

app = FastAPI()                       # 创建一个实例

@app.get("/")                         # 创建一个路径操作装饰器
async def root():                     # 定义路径操作函数
    return {"message": "Hello World"}
```

运行开发服务器：

```
uvicorn main:app --reload
```

* `main`：是`main.py`文件的名字
* `app`：是FastAPI实例的名字，在 `main.py` 文件中通过 `app = FastAPI()` 创建的对象。
* `--reload`：是让服务器在更新代码后重新启动。仅在开发时使用该选项。



我觉得这个**运行模式**就是，前端通过URL，调用这个路径装饰器，然后这个路径装饰器调用路径操作函数，函数运行完，在传递给前端一个JSON，前端最后在处理这个JSON。



### 创建实例

app是FastAPI实例的名字，你可以自己取，如myapp。



### 路径操作装饰器

* 「路径」：指的是 URL 中从第一个 `/` 起的后半部分。如：，在一个这样的 URL 中：`https://example.com/items/foo`，路径会是：`/items/foo`
* 「操作」：指的是一种 HTTP「方法」。通常有：`POST`：创建数据，`GET`：读取数据，`PUT`：更新数据，`DELETE`：删除数据，以及更少见的几种：`OPTIONS`，`HEAD`，`PATCH`，`TRACE`。
  对应的调用为：`@app.get()`，`@app.post()`，`@app.put()`，`@app.delete()`。以及更少见的：`@app.options()`，`@app.head()`，`@app.patch()`，`@app.trace()`。

* 「装饰器」：`@something` 语法在 Python 中被称为「装饰器」。像一顶漂亮的装饰帽一样，将它放在一个函数的上方。



### 返回内容

FastAPI的返回值是JSON。文档说可以返回一个 `dict`、`list`，或者像 `str`、`int` 一样的单个值，等等，还可以返回 Pydantic 模型，以及许多其他将会自动转换为 JSON 的对象和模型（包括 ORM 对象等）。

我觉得主要是因为这些类型能转化为JSON类型，所以才可以使用这些值返回。所以要求这个返回值必须是JSON。



## 路径操作函数的参数

路径操作函数的参数分为两种，路径参数和查询参数。

### 路径参数



> 路径参数是啥，我认为就是`@app.get("/items/{item_id}")`中的`item_id`。

路径参数就是使用与 **Python 格式化字符串**相同的语法来声明包含在路径中的"参数"或"变量"，所以这个参数是通过路径来传递的，它本质上还是一个参数。这个参数要传递给谁呢，是**传递给路径操作函数**。如：

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

`item_id`是路径中获取的，然后传参给路径操作函数。

#### 顺序

路径操作是按顺序依次运行的，如果路径 `/users/me` 声明在路径 `/users/{user_id}` 之前，那么找路径的时候会先找是否符合`/users/me` ，不符合再去执行`/users/{user_id}`。

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/me")
async def read_user_me():
    return {"user_id": "the current user"}

@app.get("/users/{user_id}")
async def read_user(user_id: str):
    return {"user_id": user_id}
```

#### 预设值

如果你有一个接收路径参数的路径操作，但你希望预先设定可能的有效参数值，则可以使用标准的 Python `Enum` 类型。

```python
from enum import Enum
from fastapi import FastAPI

class ModelName(str, Enum):                    # 定义的枚举类，继承自 str 和 Enum
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"

app = FastAPI()

@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):    # 声明路径参数
    if model_name == ModelName.alexnet:        # 比较枚举成员
        return {"model_name": model_name, "message": "Deep Learning FTW!"}

    if model_name.value == "lenet":            # 获取枚举值
        return {"model_name": model_name, "message": "LeCNN all the images"}

    return {"model_name": model_name, "message": "Have some residuals"}
```

#### 包含路径

假设你有一个路径操作，它的路径为 `/files/{file_path}`，但是你需要 `file_path` 自身也包含路径，比如 `home/johndoe/myfile.txt`，因此，该文件的URL将类似于这样：`/files/home/johndoe/myfile.txt`。

OpenAPI 不支持任何方式去声明路径参数以在其内部包含路径，但是可以使用直接来自 Starlette 的选项来声明一个包含路径的路径参数。如：`/files/{file_path:path}`，参数的名称为 `file_path`，结尾部分的 `:path` 说明该参数应匹配任意的路径。

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/files/{file_path:path}")
async def read_file(file_path: str):
    return {"file_path": file_path}
```

### 查询参数

声明不属于路径参数的其他函数参数时，它们将被自动解释为"查询字符串"参数。查询字符串是键值对的集合，这些键值对位于 URL 的 `？` 之后，并以 `&` 符号分隔。

```python
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]

@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]
```

对应的 URL 为：`http://127.0.0.1:8000/items/?skip=0&limit=10`，其中的查询参数为：

* `skip`：对应的值为 `0`
* `limit`：对应的值为 `10`

**查询参数也是通过路径给出的，只不过具有不同的格式**。

#### 默认值

由于查询参数不是路径的固定部分，因此它们可以是可选的，并且可以有默认值。在上面的示例中，它们具有 skip=0 和 limit=10 的默认值。因此，访问 URL：`http://127.0.0.1:8000/items/`，将与访问以下地址相同：`http://127.0.0.1:8000/items/?skip=0&limit=10`。路径中的查询参数部分也可以给出不同的值。

#### 可选查询参数

可以将查询参数的默认值设置为 `None` 来声明可选查询参数，如：

```python
from typing import Optional
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: str, q: Optional[str] = None):
    if q:
        return {"item_id": item_id, "q": q}
    return {"item_id": item_id}
```

函数参数 `q` 将是可选的，并且默认值为 `None`

#### 必需查询参数

当你为非路径参数声明了默认值时（目前而言，我们所知道的仅有查询参数），则该参数不是必需的。如果你不想添加一个特定的值，而只是想使该参数成为可选的，则将默认值设置为 None。但当你想让一个查询参数成为必需的，**不声明任何默认值就可以**。如：

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_user_item(item_id: str, needy: str):
    item = {"item_id": item_id, "needy": needy}
    return item
```

如果在URL中不给出查询参数的值，那么就会报错。



## 请求体

HTTP 的工作方式是客户端与服务器之间的请求-应答协议。当你需要将数据从客户端（例如浏览器）发送给 API 时，你将其作为「请求体」发送。请求体是客户发送给 API 的数据，响应体是 API 发送给客户端的数据。不能使用 GET 操作（HTTP 方法）向服务器端发送请求体，要向服务器上发送数据，必须使用下列方法之一：POST（较常见）、PUT、DELETE 或 PATCH。

> HTTP 协议中的 GET 和 POST 方法。二者主要区别如下：
>  1、GET 是用来**从服务器上获得数据**，而 POST 是用来**向服务器上传递数据**。
>  2、GET 将表单中数据的按照 variable=value 的形式，添加到 action 所指向的 URL 后面，并且两者使用“?”连接，而各个变量之间使用“&”连接；POST 是将表单中的数据放在 form 的数据体中，按照变量和值相对应的方式，传递到 action 所指向 URL。

```python
from typing import Optional
from fastapi import FastAPI
from pydantic import BaseModel          # 导入 Pydantic 的 BaseModel

class Item(BaseModel):                  # 创建数据模型
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None

app = FastAPI()

@app.post("/items/")
async def create_item(item: Item):      # 声明为参数
    return item
```

