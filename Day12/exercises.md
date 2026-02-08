# Day 12 练习题：项目实战（二）
> 要求：独立完成以下练习，完善接口功能
>

---

## 练习 1：个人信息接口
```plain
实现以下接口：

1. GET /api/profile：
   - 获取当前登录用户信息
   - 使用 g.user_id 查询

2. PUT /api/profile：
   - 修改当前用户信息
   - 可修改：email、phone
   - 不允许修改 username
   - 验证 email 唯一性

3. PUT /api/profile/password：
   - 修改密码
   - 请求体：{"old_password": "xxx", "new_password": "xxx"}
   - 验证旧密码正确
   - 新密码长度至少 6 位
```

---

## 练习 2：重复检查
```plain
优化注册和修改接口的重复检查：

1. 创建辅助函数 check_duplicate(field, value, exclude_id=None)
   - field: "username" 或 "email"
   - value: 要检查的值
   - exclude_id: 排除的用户 ID（更新时排除自己）
   - 返回 True/False

2. 在注册接口中使用
3. 在修改接口中使用（排除当前用户）

测试用例：
- 注册时用户名已存在 -> 报错
- 修改时使用自己的邮箱 -> 不报错
- 修改时使用别人的邮箱 -> 报错
```

---

## 练习 3：全接口测试
```plain
编写完整的测试脚本（使用 curl）：

1. 注册 3 个用户（admin、user1、user2）
2. 用 admin 登录
3. 获取用户列表（确认 3 个用户）
4. 获取 user1 详情
5. 修改 user1 的邮箱
6. 删除 user2
7. 获取用户列表（确认 2 个用户）
8. 获取个人信息
9. 修改个人密码
10. 用新密码重新登录

将所有命令和结果保存到 test.sh 文件中。
```

---

## 完成检查
- [ ] profile 接口能获取和修改个人信息
- [ ] 密码修改接口工作正常
- [ ] 重复检查逻辑正确
- [ ] 全接口测试通过
- [ ] 代码已提交到 Git

---
