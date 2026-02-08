# Day 8 练习题：数据库基础
> 要求：独立完成以下练习，巩固 SQLAlchemy ORM 操作
>

---

## 练习 1：扩展 User 字段
```plain
扩展 User 模型，添加以下字段：

1. phone（手机号，字符串，可选）
2. age（年龄，整数，可选）
3. status（状态，整数，默认 1，1 表示正常，0 表示禁用）

完成以下任务：
- 更新模型定义
- 更新 to_dict() 方法
- 更新创建接口，支持新字段
- 更新修改接口，支持新字段
- 测试所有接口
```

---

## 练习 2：用户名搜索
```plain
实现用户搜索功能：

1. GET /api/users?keyword=张
   - 按用户名模糊搜索
   - 使用 User.username.like()

2. GET /api/users?keyword=张&status=1
   - 支持按状态筛选

3. 无搜索条件时返回所有用户

提示：
- query = User.query
- if keyword:
-     query = query.filter(User.username.like(f'%{keyword}%'))
- users = query.all()
```

---

## 练习 3：时间戳查询
```plain
实现基于时间的查询：

1. 添加 updated_at 字段（更新时自动修改）

2. GET /api/users?start=2025-01-01&end=2025-01-31
   - 查询指定时间范围内创建的用户

3. GET /api/users?sort=created_at&order=desc
   - 支持按创建时间排序

提示：
- 使用 datetime.strptime() 解析日期字符串
- 使用 User.query.filter(User.created_at >= start)
- 使用 User.query.order_by(User.created_at.desc())
```

---

## 完成检查
- [ ] User 模型包含所有新字段
- [ ] 搜索功能正常工作
- [ ] 时间查询和排序正常
- [ ] 所有接口 curl 测试通过

---
