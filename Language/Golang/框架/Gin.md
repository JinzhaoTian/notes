Gin 是一个用 Go 语言（Golang）编写的高性能 Web 框架，以其简洁、快速和高效而闻名。

## 核心特点

1. **极高性能**：基于 httprouter，使用 radix 树实现的路由，速度非常快
2. **简洁API**：设计优雅，学习曲线平缓
3. **中间件支持**：灵活的中间件机制
4. **错误管理**：内置完善的错误处理
5. **JSON支持**：内置 [JSON](../../Data%20Format/JSON.md) 解析和验证


## 使用示例

```go
package main

import "github.com/gin-gonic/gin"

func main() {
    // 创建一个默认的Gin引擎
    r := gin.Default()
    
    // 定义路由和处理函数
    r.GET("/ping", func(c *gin.Context) {
        c.JSON(200, gin.H{
            "message": "pong",
        })
    })
    
    // 启动服务
    r.Run() // 默认监听 :8080
}
```

### 路由功能

```go
r.GET("/someGet", getting)
r.POST("/somePost", posting)
r.PUT("/somePut", putting)
r.DELETE("/someDelete", deleting)
r.PATCH("/somePatch", patching)
r.HEAD("/someHead", head)
r.OPTIONS("/someOptions", options)
```


### 路由参数

```go
// 路径参数
r.GET("/user/:name", func(c *gin.Context) {
    name := c.Param("name")
    c.String(http.StatusOK, "Hello %s", name)
})

// 查询参数
r.GET("/welcome", func(c *gin.Context) {
    firstname := c.DefaultQuery("firstname", "Guest")
    lastname := c.Query("lastname")
    c.String(http.StatusOK, "Hello %s %s", firstname, lastname)
})
```

### 中间件

```go
// 自定义中间件
func Logger() gin.HandlerFunc {
    return func(c *gin.Context) {
        // 中间件逻辑
        c.Next()
    }
}

// 使用中间件
r.Use(Logger())
```

