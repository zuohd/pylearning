# Day 6 练习题：Flask 入门
> 要求：独立完成以下练习，巩固 Flask 基础
>

---

## 练习 1：待办事项 API
```plain
使用 Flask 创建一个待办事项 API：

1. 数据结构：
   - id：自增编号
   - title：标题
   - done：是否完成（默认 False）

2. 实现接口：
   - GET /api/todos：获取所有待办
   - POST /api/todos：创建待办（传入 title）
   - PUT /api/todos/<id>：标记完成/未完成
   - DELETE /api/todos/<id>：删除待办

3. 使用统一响应格式（success/error）

4. 使用 curl 测试所有接口

测试命令：
curl http://localhost:5000/api/todos
curl -X POST -H "Content-Type: application/json" \
  -d '{"title": "学习 Flask"}' \
  http://localhost:5000/api/todos
```

---

## 练习 2：about 和 status 路由
```plain
为 Flask 应用添加以下路由：

1. GET /api/about：
   返回应用信息（名称、版本、作者）

2. GET /api/status：
   返回服务状态（运行状态、用户数量、当前时间）

3. GET /api/user/<username>：
   返回指定用户信息，不存在返回 404

示例响应：
GET /api/status
{
    "code": 0,
    "message": "success",
    "data": {
        "status": "running",
        "user_count": 5,
        "time": "2025-01-15 14:30:00"
    }
}
```

---

## 练习 3：响应格式实验
```plain
创建以下路由，观察不同响应的效果：

1. GET /api/text：返回纯文本
2. GET /api/json：返回 JSON
3. GET /api/html：返回 HTML
4. GET /api/error：返回 400 错误
5. GET /api/not-found：返回 404 错误

使用浏览器或 curl 访问每个路由，观察：
- 响应内容
- Content-Type
- 状态码

记录观察结果。
```

---

## 完成检查
- [ ] 待办 API 四个接口都能正常工作
- [ ] about 和 status 路由返回正确信息
- [ ] 理解不同响应类型的区别
- [ ] 能使用 curl 测试 API

---
