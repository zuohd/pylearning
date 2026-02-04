# Day 29：项目整理
> 学习目标：整理项目代码，完善文档，提交到 GitHub
>

---

## 1. 项目检查清单
```
代码检查：
[ ] 代码能正常运行
[ ] 没有语法错误
[ ] 没有未使用的导入
[ ] 变量命名规范
[ ] 函数有适当注释

文件检查：
[ ] 项目结构清晰
[ ] 不包含敏感信息（密码、密钥）
[ ] 不包含临时文件
[ ] .gitignore 配置正确

文档检查：
[ ] README.md 完整
[ ] 有安装说明
[ ] 有使用说明
[ ] 有 API 文档（如果是后端项目）
```

---

## 2. README 模板
```markdown
# 项目名称

简短描述项目功能

## 功能特点

- 功能1
- 功能2
- 功能3

## 技术栈

- Python 3.x
- Flask / pandas / ...

## 安装

```bash
# 克隆项目
git clone https://github.com/username/project.git
cd project

# 安装依赖
pip install -r requirements.txt
```

## 使用

```bash
# 运行
python main.py
```

## 项目结构

```
project/
├── app/
│   ├── __init__.py
│   └── main.py
├── data/
├── requirements.txt
└── README.md
```

## API 文档

### 获取用户列表
```
GET /api/users
```

响应：
```json
{
  "code": 0,
  "data": [...]
}
```

## 作者

- 姓名 - email@example.com

## 许可证

MIT
```

---

## 3. .gitignore 配置
```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
.venv/

# IDE
.idea/
.vscode/
*.swp
*.swo

# 环境变量
.env
.env.local

# 数据库
*.db
*.sqlite

# 日志
*.log

# 系统文件
.DS_Store
Thumbs.db

# 临时文件
*.tmp
*.temp
```

---

## 4. 代码规范检查
```bash
# Python 代码检查
pip install flake8
flake8 app/

# 代码格式化
pip install black
black app/

# Go 代码格式化
go fmt ./...

# Go 代码检查
go vet ./...
```

---

## 5. Git 提交规范
```bash
# 提交信息格式
# <type>: <description>

# type 类型：
# feat: 新功能
# fix: 修复bug
# docs: 文档更新
# style: 代码格式
# refactor: 重构
# test: 测试
# chore: 构建/工具

# 示例
git commit -m "feat: 添加用户登录功能"
git commit -m "fix: 修复分页参数错误"
git commit -m "docs: 更新 README"
```

---

## 6. 提交到 GitHub
```bash
# 1. 检查状态
git status

# 2. 添加文件
git add .

# 3. 提交
git commit -m "完成项目开发"

# 4. 推送
git push origin main

# 如果是新仓库
git remote add origin https://github.com/username/project.git
git branch -M main
git push -u origin main
```

---

## 7. 项目展示准备
```
准备内容：
1. 项目截图/演示 GIF
2. 核心功能说明
3. 技术难点和解决方案
4. 学到了什么

展示要点：
- 项目背景和目标
- 主要功能演示
- 代码亮点
- 遇到的问题和解决方法
- 后续改进计划
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 代码检查 | flake8、black、go fmt |
| 文档 | README.md 规范 |
| Git | .gitignore、提交规范 |
| 展示 | 截图、演示、总结 |

---

## 课后练习
完成练习题 35-36（见 [exercises.md](exercises.md)）

---
