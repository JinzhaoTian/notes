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

> [!tip] Uvicorn
> [Uvicorn](Uvicorn.md) 是 FastAPI 的**官方推荐服务器**，它以高性能、轻量级和完整的异步支持成为现代 Python Web 应用的最佳运行环境。开发时使用 `--reload` 提升效率，生产环境结合反向代理和多 worker 实现高可用。



## 使用示例

### 项目结构

```
my_fastapi_project/
├── app/
│   ├── __init__.py
│   ├── main.py          # 应用入口
│   ├── models.py        # 数据模型
│   ├── schemas.py       # Pydantic 模型
│   ├── crud.py          # 数据库操作
│   ├── database.py      # 数据库连接
│   └── routers/         # 路由模块
│       ├── __init__.py
│       ├── items.py
│       └── users.py
├── requirements.txt
└── .env                 # 环境变量
```


### 简单应用

```python
# main.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"Hello": "World"}

@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}
```


### 启动服务

```python
# 开发模式（自动重启）
uvicorn main:app --reload

# 指定主机和端口
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```



## 请求处理

1. **路径参数**
	- **路径**：指的是 URL 中从第一个 `/` 起的后半部分。如：，在一个这样的 URL 中：`https://example.com/items/foo`，路径会是：`/items/foo`
	- **操作**：指的是一种 HTTP 方法。通常有：
		- `POST`：创建数据，`@app.get()`
		- `GET`：读取数据，`@app.post()`
		- `PUT`：更新数据，`@app.put()`
		- `DELETE`：删除数据，`@app.delete()`
		- 以及更少见的几种：`OPTIONS`、`HEAD`、`PATCH`、`TRACE`，`@app.options()`、`@app.head()`、`@app.patch()`、`@app.trace()`。

```python
@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"user_id": user_id}

# 路径参数验证
from fastapi import Path

@app.get("/items/{item_id}")
def read_item(
    item_id: int = Path(..., title="物品ID", ge=1, le=1000)
):
    return {"item_id": item_id}
```

> [!tip] 装饰器
> `@something` 语法在 Python 中被称为[装饰器（Decorator）](../原理/Python%20Decorator.md)。

2. **查询参数**
```python
from typing import Optional

@app.get("/search/")
def search_items(
    q: str,                      # 必需查询参数
    skip: int = 0,               # 默认值
    limit: int = 10,             # 默认值
    sort: Optional[str] = None   # 可选参数
):
    return {"q": q, "skip": skip, "limit": limit, "sort": sort}

# 访问: /search/?q=python&skip=5&limit=20&sort=desc
```

3. **请求体**
```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None

@app.post("/items/")
def create_item(item: Item):
    # 自动验证和转换
    item_dict = item.dict()
    if item.tax:
        price_with_tax = item.price + item.tax
        item_dict.update({"price_with_tax": price_with_tax})
    return item_dict
```

4. **表单数据**
```python
from fastapi import Form

@app.post("/login/")
def login(
    username: str = Form(...),
    password: str = Form(...)
):
    return {"username": username}
```

5. **文件上传**
```python
from fastapi import File, UploadFile

# 简单文件上传
@app.post("/files/")
async def upload_file(file: bytes = File(...)):
    return {"file_size": len(file)}

# 高级文件上传
@app.post("/upload/")
async def upload_file(
    file: UploadFile = File(...)
):
    contents = await file.read()
    return {
        "filename": file.filename,
        "content_type": file.content_type,
        "size": len(contents)
    }
```

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

声明不属于路径参数的其他函数参数时，它们将被自动解释为"查询字符串"参数。查询字符串是键值对的集合，这些键值对位于 URL 的 `?` 之后，并以 `&` 符号分隔。

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


### 请求体

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



## 响应处理

1. **响应模型**
```python
from pydantic import BaseModel
from typing import List

class UserOut(BaseModel):
    id: int
    name: str
    email: str
    
    class Config:
        orm_mode = True  # 支持 ORM 对象

@app.get("/users/{user_id}", response_model=UserOut)
def get_user(user_id: int):
    # 自动过滤、验证和序列化
    return {"id": 1, "name": "John", "email": "john@example.com", "password": "hidden"}
    # password 不会出现在响应中
```

2. **状态码**
```python
from fastapi import status

@app.post("/items/", status_code=status.HTTP_201_CREATED)
def create_item(item: Item):
    return item

@app.delete("/items/{item_id}", status_code=204)
def delete_item(item_id: int):
    # 返回空响应
    return
```

3. **自定义响应**
```python
from fastapi.responses import JSONResponse, HTMLResponse, RedirectResponse

@app.get("/custom/")
def custom_response():
    return JSONResponse(
        content={"message": "自定义响应"},
        status_code=200,
        headers={"X-Custom": "value"}
    )

@app.get("/html/", response_class=HTMLResponse)
def get_html():
    return "<h1>Hello World</h1>"

@app.get("/redirect/")
def redirect():
    return RedirectResponse(url="/docs")
```


## 依赖注入

```python
from fastapi import Depends

# 定义依赖
def common_parameters(q: str = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}

# 使用依赖
@app.get("/items/")
def read_items(commons: dict = Depends(common_parameters)):
    return commons

# 类依赖
class CommonQueryParams:
    def __init__(self, q: str = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit

@app.get("/items2/")
def read_items(commons: CommonQueryParams = Depends()):
    return commons
```


## 数据库集成

1. **SQLAlchemy**
```python
# database.py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

SQLALCHEMY_DATABASE_URL = "sqlite:///./sql_app.db"
engine = create_engine(SQLALCHEMY_DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

# 依赖：获取数据库会话
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# models.py
from sqlalchemy import Column, Integer, String, Float

class Item(Base):
    __tablename__ = "items"
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, index=True)
    price = Column(Float)
    description = Column(String)

# schemas.py
from pydantic import BaseModel

class ItemCreate(BaseModel):
    name: str
    price: float
    description: str = None

class ItemResponse(BaseModel):
    id: int
    name: str
    price: float
    description: str = None
    
    class Config:
        orm_mode = True

# main.py
from fastapi import Depends
from sqlalchemy.orm import Session

@app.post("/items/", response_model=ItemResponse)
def create_item(item: ItemCreate, db: Session = Depends(get_db)):
    db_item = Item(**item.dict())
    db.add(db_item)
    db.commit()
    db.refresh(db_item)
    return db_item
```


## 路由组织

```python
# routers/items.py
from fastapi import APIRouter, Depends

router = APIRouter(prefix="/items", tags=["items"])

@router.get("/")
def read_items():
    return [{"name": "Item1"}, {"name": "Item2"}]

@router.get("/{item_id}")
def read_item(item_id: int):
    return {"item_id": item_id}

# routers/users.py
router = APIRouter(prefix="/users", tags=["users"])

@router.get("/")
def read_users():
    return [{"username": "user1"}, {"username": "user2"}]

# main.py
from app.routers import items, users

app.include_router(items.router)
app.include_router(users.router)
```


## 中间件

```python
from fastapi.middleware.cors import CORSMiddleware
import time

# CORS 中间件
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # 生产环境应指定具体域名
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 自定义中间件
@app.middleware("http")
async def add_process_time_header(request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

## 异常处理

```python
from fastapi import HTTPException, status
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse

# 自定义异常
@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    return JSONResponse(
        status_code=422,
        content={"detail": exc.errors(), "body": exc.body},
    )

# 抛出 HTTP 异常
@app.get("/items/{item_id}")
def read_item(item_id: int):
    if item_id not in items:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Item not found",
            headers={"X-Error": "Not Found"},
        )
    return items[item_id]
```

## 异步支持

```python
import asyncio

@app.get("/async-example/")
async def async_endpoint():
    # 异步操作
    await asyncio.sleep(1)
    return {"message": "This is async"}

@app.get("/mixed/")
def mixed_endpoint():
    # 同步操作
    return {"message": "This is sync"}

# 混合使用
@app.get("/parallel/")
async def parallel_operations():
    # 并行执行多个异步任务
    results = await asyncio.gather(
        async_operation1(),
        async_operation2(),
    )
    return results
```

## 配置管理

```python
# 使用 Pydantic Settings
from pydantic import BaseSettings

class Settings(BaseSettings):
    app_name: str = "My API"
    database_url: str
    secret_key: str
    
    class Config:
        env_file = ".env"

settings = Settings()

# 在应用中使用
app = FastAPI(title=settings.app_name)
```

## 测试

```python
# test_main.py
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"Hello": "World"}

def test_create_item():
    response = client.post(
        "/items/",
        json={"name": "Test Item", "price": 10.5}
    )
    assert response.status_code == 200
    data = response.json()
    assert data["name"] == "Test Item"
    assert data["price"] == 10.5
```

## 部署

1. **使用 Gunicorn + Uvicorn**（生产推荐）
```python
# 安装
pip install gunicorn

# 启动
gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app
```

2. **Docker 部署**
```dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

