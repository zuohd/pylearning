# Day 13 练习题：项目实战（三）
> 要求：独立完成以下练习，掌握 Git 和文档编写
>

---

## 练习 1：3 次有意义的 commit
```plain
将项目代码分 3 次有意义地提交：

1. 第一次提交：项目基础结构
   - 包含：config.py、run.py、app/__init__.py、models/
   - 提交信息示例："初始化项目结构和用户模型"

2. 第二次提交：认证功能
   - 包含：routes/auth.py、utils/auth.py、utils/response.py
   - 提交信息示例："实现用户注册和登录认证"

3. 第三次提交：用户管理
   - 包含：routes/user.py
   - 提交信息示例："实现用户 CRUD 接口"

要求：
- 每次提交都能独立运行（不会报错）
- 提交信息清楚描述了修改内容
```

---

## 练习 2：完整 README
```plain
为项目编写完整的 README.md，包含：

1. 项目标题和简介
2. 功能列表
3. 技术栈
4. 安装步骤（从 clone 到运行）
5. 接口列表（表格形式）
6. 至少 3 个接口的 curl 使用示例
7. 项目结构说明

确保按照 README 的步骤操作，能成功运行项目。
```

---

## 练习 3：推送到 GitHub
```plain
完成以下操作：

1. 在 GitHub 上创建新仓库
2. 添加远程仓库地址
3. 推送代码到 GitHub
4. 在 GitHub 上确认：
   - 代码文件完整
   - README 正确显示
   - .gitignore 生效（没有 __pycache__ 等）
5. 复制仓库链接

记录操作命令：
git remote add origin <url>
git branch -M main
git push -u origin main
```

---

## 完成检查
- [ ] 3 次 commit 都有意义，信息清晰
- [ ] README 内容完整，步骤可操作
- [ ] 代码已推送到 GitHub
- [ ] GitHub 上显示正确
- [ ] .gitignore 生效

---
