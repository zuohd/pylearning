# Day 13：项目实战（三）-- Git 与文档
> 学习目标：学会使用 Git 管理代码，编写项目文档
>

---

## 1. Git 基础
### 什么是版本控制
Git 是一个版本控制系统，帮助你：
+ 记录代码的每次修改
+ 在不同版本之间切换
+ 多人协作开发
+ 备份代码到远程仓库

### 基本命令
```plain
# 初始化仓库
git init

# 查看状态
git status

# 添加文件到暂存区
git add filename        # 添加单个文件
git add .               # 添加所有文件

# 提交
git commit -m "提交说明"

# 查看提交历史
git log
git log --oneline       # 简洁模式
```

### 工作流程
```plain
工作区 --> 暂存区 --> 仓库
  |    git add   |  git commit  |
  |              |              |
  编辑文件     选择要提交的文件   保存一个版本
```

---

## 2. Git 工作流
### 有意义的 commit
每次提交应该是一个完整的、有意义的修改：
```plain
# 好的提交信息
git commit -m "添加用户注册接口"
git commit -m "实现 JWT 认证功能"
git commit -m "修复登录密码验证错误"

# 不好的提交信息
git commit -m "update"
git commit -m "fix"
git commit -m "改了一些东西"
```

### .gitignore
```plain
# .gitignore
__pycache__/
*.pyc
.env
venv/
*.db
.DS_Store
.idea/
.vscode/
```

### git diff
```plain
# 查看未暂存的修改
git diff

# 查看已暂存的修改
git diff --staged

# 查看某次提交的修改
git diff HEAD~1
```

---

## 3. GitHub
### 创建远程仓库
1. 登录 GitHub（https://github.com）
2. 点击 "New repository"
3. 输入仓库名（如 user-management）
4. 选择 Public 或 Private
5. 不勾选初始化选项（已有本地仓库）
6. 点击 "Create repository"

### 推送代码
```plain
# 添加远程仓库
git remote add origin https://github.com/your-username/user-management.git

# 推送代码
git branch -M main
git push -u origin main
```

### 后续推送
```plain
# 修改代码后
git add .
git commit -m "添加新功能"
git push
```

---

## 4. README 文档
每个项目都应该有 README.md，说明项目的用途和使用方法。

```markdown
# 用户管理系统

基于 Flask 的用户管理 RESTful API。

## 功能

+ 用户注册和登录
+ JWT 身份认证
+ 用户信息 CRUD
+ 分页查询

## 技术栈

+ Python 3.x
+ Flask
+ MySQL + SQLAlchemy
+ JWT

## 安装

1. 克隆项目

git clone https://github.com/your-username/user-management.git
cd user-management

2. 安装依赖

pip install -r requirements.txt

3. 配置数据库

创建 MySQL 数据库：
CREATE DATABASE user_management CHARACTER SET utf8mb4;

4. 启动

python run.py

## 接口列表

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| POST | /api/register | 用户注册 |
| POST | /api/login | 用户登录 |
| GET | /api/users | 用户列表 |
| GET | /api/users/<id> | 用户详情 |
| PUT | /api/users/<id> | 修改用户 |
| DELETE | /api/users/<id> | 删除用户 |
| GET | /api/profile | 个人信息 |
```

---

## 5. API 文档
详细的接口文档帮助其他开发者使用你的 API。

### 文档格式
```markdown
### 用户注册

POST /api/register

请求体：
{
    "username": "张三",
    "email": "zhangsan@example.com",
    "password": "abc123"
}

成功响应（201）：
{
    "code": 0,
    "message": "注册成功",
    "data": {
        "id": 1,
        "username": "张三",
        "email": "zhangsan@example.com",
        "created_at": "2025-01-15 14:30:00"
    }
}

错误响应（400）：
{
    "code": 1,
    "message": "用户名已存在",
    "data": null
}
```

### curl 示例
为每个接口提供 curl 测试命令：
```plain
# 注册
curl -X POST http://localhost:5000/api/register \
  -H "Content-Type: application/json" \
  -d '{"username": "test", "email": "test@example.com", "password": "test123"}'

# 登录
curl -X POST http://localhost:5000/api/login \
  -H "Content-Type: application/json" \
  -d '{"username": "test", "password": "test123"}'

# 获取用户列表（需要 Token）
curl http://localhost:5000/api/users \
  -H "Authorization: Bearer <token>"
```

---

## 6. 代码审查
在提交代码前，进行自我审查：

### 检查清单
+ 代码能正常运行
+ 没有硬编码的密码或密钥
+ 错误处理完善
+ 接口返回格式统一
+ 没有调试代码（如 print 语句）
+ 命名清晰规范

### 常见问题
```python
# 问题 1：硬编码密钥
SECRET_KEY = "my-secret"  # 应该用环境变量

# 问题 2：缺少错误处理
user = User.query.get(user_id)
print(user.name)  # user 可能为 None

# 问题 3：返回敏感信息
def to_dict(self):
    return {
        "password_hash": self.password_hash  # 不应该返回
    }

# 问题 4：未验证输入
data = request.get_json()
user.email = data['email']  # 应该先验证
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| Git 基础 | init/add/commit/status/log |
| Git 工作流 | 有意义的 commit、.gitignore |
| GitHub | 远程仓库、push |
| README | 项目说明、安装步骤、接口列表 |
| API 文档 | 请求/响应格式、curl 示例 |
| 代码审查 | 自查清单、常见问题 |

---

## 课后练习
完成练习（见 [exercises.md](exercises.md)）

---
