# Day 7 练习题：Flask 项目组织
> 要求：独立完成以下练习，巩固 Blueprint 和项目组织
>

---

## 练习 1：输入验证
```plain
为 CRUD 接口添加完整的输入验证：

1. POST /api/users（创建）：
   - name 必填，长度 2-50
   - email 必填，不能重复

2. PUT /api/users/<id>（更新）：
   - name 长度 2-50（如果提供）
   - email 不能与其他用户重复（如果提供）

3. 所有验证失败返回 400 和具体错误信息

测试用例：
- 创建时不传 name -> 返回错误
- name 只有 1 个字符 -> 返回错误
- email 重复 -> 返回错误
```

---

## 练习 2：健康检查端点
```plain
添加以下端点：

1. GET /api/health：
   返回应用健康状态
   {
       "status": "healthy",
       "timestamp": "2025-01-15T14:30:00",
       "version": "1.0.0"
   }

2. GET /api/stats：
   返回统计信息
   {
       "total_users": 5,
       "api_version": "1.0.0"
   }

3. 将这些端点放在新的 Blueprint（system_bp）中
4. 注册到 /api 前缀下
```

---

## 练习 3：重构待办 API
```plain
将 Day 6 的待办事项 API 重构为项目结构：

1. 目录结构：
   todo_app/
   ├── app/
   │   ├── __init__.py
   │   ├── routes/
   │   │   ├── __init__.py
   │   │   └── todo.py
   │   └── utils/
   │       ├── __init__.py
   │       └── response.py
   └── run.py

2. 使用 Blueprint 和工厂函数
3. 添加输入验证
4. 添加全局错误处理
5. 用 curl 测试所有接口
```

---

## 完成检查
- [ ] 输入验证覆盖了所有情况
- [ ] 健康检查端点正常工作
- [ ] 待办 API 重构为项目结构
- [ ] 所有接口 curl 测试通过

---
