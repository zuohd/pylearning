# Day 28 练习题：Gin 框架写 API
> 要求：使用 Gin 创建 RESTful API
>

---

## 练习 1：基础 API
```plain
创建一个 Gin 项目，实现以下接口：

1. GET /
   返回 {"message": "Hello, Gin!"}

2. GET /hello/:name
   返回 {"message": "Hello, {name}!"}

3. GET /search?keyword=xxx
   返回 {"keyword": "xxx"}

4. POST /echo
   接收 JSON，原样返回
```

---

## 练习 2：用户 CRUD
```plain
创建用户管理 API：

数据结构：
type User struct {
    ID   int    `json:"id"`
    Name string `json:"name"`
    Age  int    `json:"age"`
}

接口：
1. GET /users - 获取所有用户
2. GET /users/:id - 获取单个用户
3. POST /users - 创建用户
4. PUT /users/:id - 更新用户
5. DELETE /users/:id - 删除用户

要求：
- 使用内存存储（切片）
- 统一响应格式
- 处理用户不存在的情况
```

---

## 练习 3：中间件
```plain
为练习 2 添加中间件：

1. 日志中间件
   - 记录请求方法、路径、耗时

2. 认证中间件
   - 检查 Authorization 头
   - 没有认证返回 401

3. 路由分组
   - 公开路由: GET /
   - 需要认证的路由: /api/users/*
```

---

## 完成检查
- [ ] 能创建 Gin 项目
- [ ] 能处理不同 HTTP 方法
- [ ] 能获取路径参数和查询参数
- [ ] 能处理 JSON 请求和响应
- [ ] 能使用中间件

---
