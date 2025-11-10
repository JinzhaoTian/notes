Postman 是一款功能强大的 API 开发和测试工具，主要用于发送 HTTP 请求、调试接口、自动化测试以及管理 API 文档。它支持 REST、GraphQL、SOAP 等多种 API 类型，并提供图形化界面，使开发者能够高效地进行接口调试和协作开发。

**核心功能**

1. **发送 HTTP 请求**
    - 支持 GET、POST、PUT、DELETE 等 HTTP 方法。
    - 可发送 JSON、表单、文件上传（`multipart/form-data`）、XML 等不同格式的请求。
    - 自动解析并格式化响应数据（JSON、XML、HTML 等）。

2. **管理请求集合（Collections）**
    - 可以将多个 API 请求组织成集合，方便管理和批量执行。
    - 支持文件夹分类，适用于模块化测试。

3. **环境变量（Environments）**
    - 允许定义不同环境（如开发、测试、生产），动态切换变量（如 `{{base_url}}`）。
    - 变量分为：
        - **Initial Value**（共享值，同步到团队）。
        - **Current Value**（本地值，仅当前会话使用）。

4. **自动化测试（Tests）**
    - 支持 JavaScript 编写断言，验证响应状态码、响应体、响应时间等。
    - 示例：
```javascript
pm.test("Status code is 200", () => pm.response.to.have.status(200));
pm.test("Response contains user data", () => {
	const jsonData = pm.response.json();
	pm.expect(jsonData).to.have.property("name");
});
```

5. **Pre-request Script（请求前置脚本）**
    - 在发送请求前执行脚本，如动态生成参数或调用其他 API。
    - 示例：
```javascript
pm.environment.set("timestamp", new Date().getTime());
```

6. **Mock 服务 & 监控**
    - 可创建 Mock API 模拟后端响应，便于前端开发。
    - 支持定时监控 API 可用性。

7. **命令行运行（Newman）**
    - 通过 `newman` 工具在 CI/CD 中运行 Postman 测试集合。


#### 安装使用

1. **安装** Postman
2. **发送请求**
	- 新建请求（`+ New Request`）。
	- 输入 URL（如 `https://api.example.com/users`）。
	- 选择 HTTP 方法（如 GET）。
	- 点击 **Send** 查看响应。

3. **设置请求参数**
	- **Headers**：添加 `Content-Type: application/json` 等请求头。
	- **Body**：
	    - `form-data`：文件上传或表单提交。
	    - `x-www-form-urlencoded`：标准表单数据。
	    - `raw`：JSON/XML 数据。

4. **使用环境变量**
	- 创建环境（如 `Dev` 和 `Prod`）。
	- 定义变量（如 `base_url = https://dev.api.com`）。
	- 在请求 URL 中使用 `{{base_url}}/users`。


5. **编写测试脚本**

在 **Tests** 标签页编写断言，如：
```javascript
pm.test("Check status code", () => pm.response.to.have.status(200));
pm.test("Check response time", () => pm.expect(pm.response.responseTime).to.be.below(500));
```


6. **导出 & 运行集合**
	- 导出集合为 JSON 文件，使用 `newman run collection.json` 运行。