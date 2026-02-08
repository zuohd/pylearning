# Day 11 练习题：项目实战（一）
> 要求：独立完成项目搭建，确保项目能正常启动
>

---

## 练习 1：扩展模型
```plain
为 User 模型添加以下字段：

1. nickname（昵称，字符串，可选，最长 50）
2. bio（个人简介，Text 类型，可选）

完成以下任务：
- 修改 User 模型
- 更新 to_dict() 方法
- 重新创建数据库表
- 验证新字段可用
```

---

## 练习 2：生成 requirements.txt
```plain
1. 使用以下命令生成依赖文件：
   pip freeze > requirements.txt

2. 检查文件内容，确认包含以下依赖：
   - Flask
   - Flask-SQLAlchemy
   - PyMySQL
   - PyJWT
   - Werkzeug

3. 验证依赖文件可用：
   pip install -r requirements.txt
```

---

## 练习 3：首次 Git 提交
```plain
完成以下 Git 操作：

1. 确认 .gitignore 已创建
2. 添加所有文件到暂存区
3. 创建首次提交：
   git add .
   git commit -m "初始化项目结构"

4. 查看提交记录：
   git log --oneline
```

---

## 练习 4：验证数据库
```plain
启动项目后，手动验证数据库：

1. 使用 MySQL 客户端连接数据库
2. 确认 users 表已创建
3. 查看表结构：DESCRIBE users;
4. 确认所有字段和约束正确

如果使用命令行：
mysql -u root -p
USE user_management;
SHOW TABLES;
DESCRIBE users;
```

---

## 完成检查
- [ ] 项目目录结构完整
- [ ] User 模型包含所有字段
- [ ] 项目能正常启动（python run.py）
- [ ] 数据库表创建成功
- [ ] requirements.txt 已生成
- [ ] 首次 Git 提交完成

---
