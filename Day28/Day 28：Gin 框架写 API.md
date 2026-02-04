# Day 28：Gin 框架写 API
> 学习目标：使用 Gin 框架搭建 RESTful API
>

---

## 1. Gin 简介
```
Gin 框架特点：
- 高性能 HTTP Web 框架
- API 友好
- 中间件支持
- 路由分组
- JSON 验证
- 错误处理

安装：
go get -u github.com/gin-gonic/gin
```

---

## 2. Hello Gin
```go
package main

import "github.com/gin-gonic/gin"

func main() {
    // 创建引擎
    r := gin.Default()

    // 定义路由
    r.GET("/", func(c *gin.Context) {
        c.JSON(200, gin.H{
            "message": "Hello, Gin!",
        })
    })

    // 启动服务
    r.Run(":8080")  // 监听 8080 端口
}
```

```bash
# 运行
go run main.go

# 测试
curl http://localhost:8080
```

---

## 3. 路由和参数
```go
package main

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default()

    // 路径参数
    r.GET("/users/:id", func(c *gin.Context) {
        id := c.Param("id")
        c.JSON(http.StatusOK, gin.H{
            "id": id,
        })
    })

    // 查询参数
    r.GET("/search", func(c *gin.Context) {
        keyword := c.Query("keyword")
        page := c.DefaultQuery("page", "1")
        c.JSON(http.StatusOK, gin.H{
            "keyword": keyword,
            "page":    page,
        })
    })

    // POST 表单
    r.POST("/login", func(c *gin.Context) {
        username := c.PostForm("username")
        password := c.PostForm("password")
        c.JSON(http.StatusOK, gin.H{
            "username": username,
            "password": password,
        })
    })

    r.Run(":8080")
}
```

---

## 4. JSON 处理
```go
package main

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

// 请求结构体
type CreateUserRequest struct {
    Name  string `json:"name" binding:"required"`
    Email string `json:"email" binding:"required,email"`
    Age   int    `json:"age" binding:"gte=0,lte=150"`
}

// 响应结构体
type User struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
    Age   int    `json:"age"`
}

func main() {
    r := gin.Default()

    // 接收 JSON
    r.POST("/users", func(c *gin.Context) {
        var req CreateUserRequest

        // 绑定并验证
        if err := c.ShouldBindJSON(&req); err != nil {
            c.JSON(http.StatusBadRequest, gin.H{
                "error": err.Error(),
            })
            return
        }

        // 创建用户（模拟）
        user := User{
            ID:    1,
            Name:  req.Name,
            Email: req.Email,
            Age:   req.Age,
        }

        c.JSON(http.StatusCreated, user)
    })

    r.Run(":8080")
}
```

---

## 5. RESTful API 示例
```go
package main

import (
    "net/http"
    "strconv"
    "github.com/gin-gonic/gin"
)

// 用户模型
type User struct {
    ID   int    `json:"id"`
    Name string `json:"name"`
    Age  int    `json:"age"`
}

// 模拟数据库
var users = []User{
    {ID: 1, Name: "张三", Age: 20},
    {ID: 2, Name: "李四", Age: 21},
}
var nextID = 3

func main() {
    r := gin.Default()

    // 获取所有用户
    r.GET("/users", func(c *gin.Context) {
        c.JSON(http.StatusOK, gin.H{
            "data": users,
        })
    })

    // 获取单个用户
    r.GET("/users/:id", func(c *gin.Context) {
        id, _ := strconv.Atoi(c.Param("id"))
        for _, u := range users {
            if u.ID == id {
                c.JSON(http.StatusOK, gin.H{"data": u})
                return
            }
        }
        c.JSON(http.StatusNotFound, gin.H{"error": "用户不存在"})
    })

    // 创建用户
    r.POST("/users", func(c *gin.Context) {
        var user User
        if err := c.ShouldBindJSON(&user); err != nil {
            c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
            return
        }
        user.ID = nextID
        nextID++
        users = append(users, user)
        c.JSON(http.StatusCreated, gin.H{"data": user})
    })

    // 更新用户
    r.PUT("/users/:id", func(c *gin.Context) {
        id, _ := strconv.Atoi(c.Param("id"))
        var input User
        if err := c.ShouldBindJSON(&input); err != nil {
            c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
            return
        }
        for i, u := range users {
            if u.ID == id {
                users[i].Name = input.Name
                users[i].Age = input.Age
                c.JSON(http.StatusOK, gin.H{"data": users[i]})
                return
            }
        }
        c.JSON(http.StatusNotFound, gin.H{"error": "用户不存在"})
    })

    // 删除用户
    r.DELETE("/users/:id", func(c *gin.Context) {
        id, _ := strconv.Atoi(c.Param("id"))
        for i, u := range users {
            if u.ID == id {
                users = append(users[:i], users[i+1:]...)
                c.JSON(http.StatusOK, gin.H{"message": "删除成功"})
                return
            }
        }
        c.JSON(http.StatusNotFound, gin.H{"error": "用户不存在"})
    })

    r.Run(":8080")
}
```

---

## 6. 路由分组
```go
package main

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default()

    // API v1 分组
    v1 := r.Group("/api/v1")
    {
        v1.GET("/users", getUsers)
        v1.POST("/users", createUser)

        // 嵌套分组
        admin := v1.Group("/admin")
        {
            admin.GET("/stats", getStats)
        }
    }

    // API v2 分组
    v2 := r.Group("/api/v2")
    {
        v2.GET("/users", getUsersV2)
    }

    r.Run(":8080")
}

func getUsers(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{"version": "v1", "data": []string{}})
}

func createUser(c *gin.Context) {
    c.JSON(http.StatusCreated, gin.H{"message": "created"})
}

func getStats(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{"stats": "admin stats"})
}

func getUsersV2(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{"version": "v2", "data": []string{}})
}
```

---

## 7. 中间件
```go
package main

import (
    "fmt"
    "net/http"
    "time"
    "github.com/gin-gonic/gin"
)

// 日志中间件
func Logger() gin.HandlerFunc {
    return func(c *gin.Context) {
        start := time.Now()
        path := c.Request.URL.Path

        // 处理请求
        c.Next()

        // 记录日志
        latency := time.Since(start)
        status := c.Writer.Status()
        fmt.Printf("[%s] %s %d %v\n",
                   c.Request.Method, path, status, latency)
    }
}

// 认证中间件
func Auth() gin.HandlerFunc {
    return func(c *gin.Context) {
        token := c.GetHeader("Authorization")
        if token == "" {
            c.JSON(http.StatusUnauthorized, gin.H{
                "error": "未授权",
            })
            c.Abort()  // 中止后续处理
            return
        }

        // 设置用户信息
        c.Set("userID", 1)
        c.Next()
    }
}

func main() {
    r := gin.New()  // 不使用默认中间件

    // 全局中间件
    r.Use(Logger())

    // 公开路由
    r.GET("/", func(c *gin.Context) {
        c.JSON(http.StatusOK, gin.H{"message": "public"})
    })

    // 需要认证的路由
    auth := r.Group("/api")
    auth.Use(Auth())
    {
        auth.GET("/profile", func(c *gin.Context) {
            userID := c.GetInt("userID")
            c.JSON(http.StatusOK, gin.H{
                "userID": userID,
            })
        })
    }

    r.Run(":8080")
}
```

---

## 8. 统一响应格式
```go
package main

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

// 统一响应结构
type Response struct {
    Code    int         `json:"code"`
    Message string      `json:"message"`
    Data    interface{} `json:"data,omitempty"`
}

// 成功响应
func Success(c *gin.Context, data interface{}) {
    c.JSON(http.StatusOK, Response{
        Code:    0,
        Message: "success",
        Data:    data,
    })
}

// 错误响应
func Error(c *gin.Context, code int, message string) {
    c.JSON(http.StatusOK, Response{
        Code:    code,
        Message: message,
    })
}

func main() {
    r := gin.Default()

    r.GET("/users", func(c *gin.Context) {
        users := []map[string]interface{}{
            {"id": 1, "name": "张三"},
            {"id": 2, "name": "李四"},
        }
        Success(c, users)
    })

    r.GET("/users/:id", func(c *gin.Context) {
        id := c.Param("id")
        if id == "0" {
            Error(c, 404, "用户不存在")
            return
        }
        Success(c, map[string]interface{}{
            "id":   id,
            "name": "张三",
        })
    })

    r.Run(":8080")
}
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 基础 | gin.Default()、r.Run() |
| 路由 | GET/POST/PUT/DELETE |
| 参数 | Param、Query、PostForm |
| JSON | ShouldBindJSON、c.JSON |
| 分组 | r.Group |
| 中间件 | Use、Next、Abort |

---

## 课后练习
完成练习题 32-34（见 [phase2_exercises.md](phase2_exercises.md#day-28-练习题gin-框架写-api)）

---
