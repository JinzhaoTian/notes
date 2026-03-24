Uvicorn 是一个基于 ASGI（Asynchronous Server Gateway Interface）的 Python Web 服务器，用于运行异步 Web 框架（如 FastAPI、Starlette、Quart 等）。它是目前 Python 生态中最快、最流行的 ASGI 服务器之一。

> [!tip] ASGI vs WSGI
> - **WSGI**（Web Server Gateway Interface）：传统的 Python Web 服务器规范，同步工作模式（如 Gunicorn、uWSGI 用于 Flask、Django）
> - **ASGI**（Asynchronous Server Gateway Interface）：现代的异步规范，支持 WebSocket、HTTP/2 和长连接（Uvicorn、Hypercorn、Daphne）
> 
> ```python
> # WSGI 同步模式
> def app(environ, start_response):
>     # 同步处理请求
>     pass
> 
> # ASGI 异步模式
> async def app(scope, receive, send):
>     # 异步处理请求
>     pass
> ```

## 主要特点

1. **极致性能**
	- 基于 **uvloop**（libuv 的 Python 封装）和 **httptools**
	- 性能接近 Node.js 和 Go 的服务器
	- 比传统的 WSGI 服务器快数倍
2. **异步支持**
	- 原生支持 `async/await`
	- 高效处理并发请求和长连接（WebSocket、SSE）
3. **轻量级**
	- 安装包小，启动快速
	- 适合容器化部署（Docker、Kubernetes）
4. **多协议支持**
	- HTTP/1.1
	- WebSocket
	- HTTP/2（有限支持）
	- HTTPS（通过 SSL）


## 运行

```python
# main.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def read_root():
    return {"Hello": "World"}
```

```bash
# 基本启动
uvicorn main:app

# 常用参数
uvicorn main:app --host 0.0.0.0 --port 8000 --reload

# 解释：
# --host: 绑定地址（0.0.0.0 允许外部访问）
# --port: 端口号
# --reload: 开发模式，代码变动自动重启
```

在生产环境运行：
```bash
# 多 worker 进程
uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4

# 结合 Gunicorn 作为进程管理器
gunicorn -k uvicorn.workers.UvicornWorker main:app
```

### 常用参数

|参数|说明|示例|
|---|---|---|
|`--host`|监听地址|`0.0.0.0`（所有接口）|
|`--port`|端口号|`8000`|
|`--reload`|自动重启（开发）|`--reload`|
|`--workers`|worker 进程数|`--workers 4`|
|`--loop`|事件循环类型|`uvloop`（默认）、`asyncio`|
|`--http`|HTTP 协议实现|`httptools`（默认）、`h11`|
|`--ssl-keyfile`|SSL 私钥文件|`--ssl-keyfile ./key.pem`|
|`--ssl-certfile`|SSL 证书文件|`--ssl-certfile ./cert.pem`|
|`--log-level`|日志级别|`info`、`debug`、`warning`|

### 生产部署建议