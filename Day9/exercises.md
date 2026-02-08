# Day 9 练习题：用户认证
> 要求：独立完成以下练习，巩固密码加密和 JWT 认证
>

---

## 练习 1：密码强度验证
```plain
在注册接口中添加密码强度验证：

1. 密码长度至少 6 位
2. 必须包含数字
3. 必须包含字母
4. 验证不通过返回具体的错误信息

实现一个 validate_password(password) 函数：
- 返回 (True, None) 表示通过
- 返回 (False, "错误信息") 表示不通过

测试用例：
validate_password("123")        # (False, "密码长度至少 6 位")
validate_password("123456")     # (False, "密码必须包含字母")
validate_password("abcdef")     # (False, "密码必须包含数字")
validate_password("abc123")     # (True, None)
```

---

## 练习 2：Token 刷新
```plain
实现 Token 刷新功能：

1. POST /api/refresh：
   - 需要携带有效的 Token
   - 返回一个新的 Token（重新设置过期时间）
   - 旧 Token 在过期前仍然可用

2. 实现思路：
   - 使用 login_required 验证当前 Token
   - 从 g.user_id 获取用户 ID
   - 生成新 Token 并返回

测试流程：
- 登录获取 Token A
- 使用 Token A 调用 /api/refresh
- 获得新 Token B
- 使用 Token B 访问受保护接口
```

---

## 练习 3：个人信息接口
```plain
实现个人信息管理接口：

1. GET /api/profile：
   - 获取当前登录用户的详细信息
   - 使用 g.user_id 查询用户

2. PUT /api/profile：
   - 修改当前用户信息（email）
   - 不允许修改用户名
   - 验证新 email 不与其他用户重复

3. PUT /api/profile/password：
   - 修改密码
   - 需要提供旧密码和新密码
   - 验证旧密码正确
   - 新密码需要通过强度验证
```

---

## 练习 4：curl 测试
```plain
使用 curl 完成以下完整测试流程：

1. 注册两个用户
2. 用第一个用户登录
3. 用 Token 获取用户列表
4. 获取个人信息
5. 修改个人邮箱
6. 修改密码
7. 用新密码重新登录
8. 不带 Token 访问受保护接口，确认被拒绝

记录每一步的 curl 命令和响应结果。
```

---

## 完成检查
- [ ] 密码强度验证功能正常
- [ ] Token 刷新接口工作正确
- [ ] 个人信息查看和修改正常
- [ ] 密码修改和重新登录流程正确
- [ ] 完成了完整的 curl 测试

---
