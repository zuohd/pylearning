# Day 11：项目实战（一）-- 项目搭建
> 学习目标：从零搭建一个完整的用户管理系统项目
>

---

## 1. 项目规划
### 需求分析
构建一个用户管理系统 API，包含以下功能：
+ 用户注册和登录
+ 用户信息的增删改查
+ JWT 身份认证
+ 分页查询
+ 参数验证

### API 列表
| 方法 | 路径 | 说明 | 认证 |
| --- | --- | --- | --- |
| POST | /api/register | 用户注册 | 否 |
| POST | /api/login | 用户登录 | 否 |
| GET | /api/users | 用户列表（分页） | 是 |
| GET | /api/users/\<id\> | 用户详情 | 是 |
| PUT | /api/users/\<id\> | 修改用户 | 是 |
| DELETE | /api/users/\<id\> | 删除用户 | 是 |
| GET | /api/profile | 个人信息 | 是 |

### 技术栈
+ Python 3.x
+ Flask
+ Flask-SQLAlchemy + MySQL
+ PyJWT
+ werkzeug（密码加密）

---

## 2. 项目初始化
### 创建目录
```plain
mkdir user_management
cd user_management
```

### 安装依赖
```plain
pip install flask flask-sqlalchemy pymysql pyjwt
```

### 初始化 Git
```plain
git init
```

创建 `.gitignore`：
```plain
__pycache__/
*.pyc
.env
venv/
```

---

## 3. 完整项目结构
```plain
user_management/
├── app/
│   ├── __init__.py          # 工厂函数
│   ├── models/
│   │   ├── __init__.py
│   │   └── user.py          # User 模型
│   ├── routes/
│   │   ├── __init__.py
│   │   ├── auth.py          # 认证接口
│   │   └── user.py          # 用户接口
│   └── utils/
│       ├── __init__.py
│       ├── response.py      # 响应工具
│       └── auth.py          # 认证工具
├── config.py                # 配置文件
├── run.py                   # 启动文件
├── .gitignore
└── requirements.txt
```

逐步创建目录和文件：
```plain
mkdir -p app/models app/routes app/utils
touch app/__init__.py
touch app/models/__init__.py app/models/user.py
touch app/routes/__init__.py app/routes/auth.py app/routes/user.py
touch app/utils/__init__.py app/utils/response.py app/utils/auth.py
touch config.py run.py
```

---

## 4. 配置文件
```python
# config.py
import os

class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY', 'dev-secret-key-change-in-production')
    SQLALCHEMY_DATABASE_URI = os.environ.get(
        'DATABASE_URI',
        'mysql+pymysql://root:password@localhost/user_management'
    )
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    JWT_EXPIRE_HOURS = 24
```

```python
# run.py
from app import create_app

app = create_app()

if __name__ == '__main__':
    app.run(debug=True)
```

---

## 5. User 模型
```python
# app/models/user.py
from datetime import datetime
from werkzeug.security import generate_password_hash, check_password_hash
from app import db

class User(db.Model):
    __tablename__ = 'users'

    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password_hash = db.Column(db.String(256), nullable=False)
    phone = db.Column(db.String(20))
    status = db.Column(db.Integer, default=1)  # 1:正常 0:禁用
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

    def set_password(self, password):
        self.password_hash = generate_password_hash(password)

    def check_password(self, password):
        return check_password_hash(self.password_hash, password)

    def to_dict(self):
        return {
            "id": self.id,
            "username": self.username,
            "email": self.email,
            "phone": self.phone,
            "status": self.status,
            "created_at": self.created_at.strftime("%Y-%m-%d %H:%M:%S") if self.created_at else None,
            "updated_at": self.updated_at.strftime("%Y-%m-%d %H:%M:%S") if self.updated_at else None
        }

    def __repr__(self):
        return f'<User {self.username}>'
```

---

## 6. 应用工厂
```python
# app/__init__.py
from flask import Flask, jsonify
from flask_sqlalchemy import SQLAlchemy
from config import Config

db = SQLAlchemy()

def create_app():
    app = Flask(__name__)
    app.config.from_object(Config)

    # 初始化数据库
    db.init_app(app)

    # 注册 Blueprint
    from app.routes.auth import auth_bp
    from app.routes.user import user_bp
    app.register_blueprint(auth_bp, url_prefix='/api')
    app.register_blueprint(user_bp, url_prefix='/api')

    # 全局错误处理
    @app.errorhandler(404)
    def not_found(e):
        return jsonify({"code": 1, "message": "接口不存在", "data": None}), 404

    @app.errorhandler(500)
    def server_error(e):
        return jsonify({"code": 1, "message": "服务器内部错误", "data": None}), 500

    @app.errorhandler(405)
    def method_not_allowed(e):
        return jsonify({"code": 1, "message": "请求方法不允许", "data": None}), 405

    # 创建数据库表
    with app.app_context():
        db.create_all()

    return app
```

### 验证项目能启动
```plain
python run.py
```

访问 `http://localhost:5000/`，应该返回 404（因为还没有定义根路由），说明项目启动成功。

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 项目规划 | 需求分析、API 设计、技术栈 |
| 项目初始化 | 目录结构、pip install、git init |
| 项目结构 | app/models/routes/utils 分层 |
| 配置文件 | Config 类、环境变量 |
| User 模型 | 完整字段、密码方法、to_dict |
| 工厂函数 | create_app、db.init_app、Blueprint |

---

## 课后练习
完成练习（见 [exercises.md](exercises.md)）

---
