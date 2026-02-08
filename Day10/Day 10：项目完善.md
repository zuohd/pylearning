# Day 10：项目完善
> 学习目标：学会配置管理、分页查询、参数验证，完善项目功能
>

---

## 1. 配置管理
将配置从代码中提取到独立文件：
```python
# config.py
import os

class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY', 'dev-secret-key')
    SQLALCHEMY_DATABASE_URI = os.environ.get(
        'DATABASE_URI',
        'mysql+pymysql://root:password@localhost/myapp'
    )
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    JWT_EXPIRE_HOURS = 24
```

在工厂函数中加载配置：
```python
# app/__init__.py
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from config import Config

db = SQLAlchemy()

def create_app():
    app = Flask(__name__)
    app.config.from_object(Config)

    db.init_app(app)

    from app.routes.auth import auth_bp
    from app.routes.user import user_bp
    app.register_blueprint(auth_bp, url_prefix='/api')
    app.register_blueprint(user_bp, url_prefix='/api')

    return app
```

使用环境变量覆盖配置：
```plain
export SECRET_KEY="production-secret-key"
export DATABASE_URI="mysql+pymysql://user:pass@db-server/myapp"
python run.py
```

---

## 2. 分页查询
当数据量大时，不能一次返回所有数据，需要分页。

```python
@user_bp.route('/users', methods=['GET'])
@login_required
def get_users():
    page = request.args.get('page', 1, type=int)
    size = request.args.get('size', 10, type=int)

    # 限制每页数量
    if size > 100:
        size = 100

    pagination = User.query.order_by(
        User.created_at.desc()
    ).paginate(page=page, per_page=size, error_out=False)

    return success({
        "list": [u.to_dict() for u in pagination.items],
        "total": pagination.total,
        "page": pagination.page,
        "pages": pagination.pages
    })
```

paginate 返回的对象属性：
| 属性 | 说明 |
| --- | --- |
| items | 当前页的数据列表 |
| total | 总记录数 |
| page | 当前页码 |
| pages | 总页数 |
| has_prev | 是否有上一页 |
| has_next | 是否有下一页 |

测试分页：
```plain
# 第一页，每页 10 条
curl "http://localhost:5000/api/users?page=1&size=10" \
  -H "Authorization: Bearer <token>"

# 第二页
curl "http://localhost:5000/api/users?page=2&size=10" \
  -H "Authorization: Bearer <token>"
```

---

## 3. 参数验证
### 验证函数
```python
# app/utils/validators.py

def validate_required(data, fields):
    """验证必填字段"""
    errors = []
    for field in fields:
        value = data.get(field, '').strip() if isinstance(data.get(field), str) else data.get(field)
        if not value:
            errors.append(f"{field} 不能为空")
    return errors

def validate_length(value, field_name, min_len=None, max_len=None):
    """验证字符串长度"""
    if min_len and len(value) < min_len:
        return f"{field_name} 长度不能少于 {min_len} 个字符"
    if max_len and len(value) > max_len:
        return f"{field_name} 长度不能超过 {max_len} 个字符"
    return None

def validate_password(password):
    """验证密码强度"""
    if len(password) < 6:
        return "密码长度至少 6 位"
    if not any(c.isdigit() for c in password):
        return "密码必须包含数字"
    if not any(c.isalpha() for c in password):
        return "密码必须包含字母"
    return None
```

### 在接口中使用
```python
@auth_bp.route('/register', methods=['POST'])
def register():
    data = request.get_json()
    if not data:
        return error("请提供 JSON 数据"), 400

    # 验证必填字段
    errors = validate_required(data, ['username', 'email', 'password'])
    if errors:
        return error(errors[0]), 400

    username = data['username'].strip()
    email = data['email'].strip()
    password = data['password']

    # 验证长度
    err = validate_length(username, "用户名", min_len=2, max_len=50)
    if err:
        return error(err), 400

    # 验证密码强度
    err = validate_password(password)
    if err:
        return error(err), 400

    # 验证唯一性
    if User.query.filter_by(username=username).first():
        return error("用户名已存在"), 400

    if User.query.filter_by(email=email).first():
        return error("邮箱已被注册"), 400

    # 创建用户
    user = User(username=username, email=email)
    user.set_password(password)
    db.session.add(user)
    db.session.commit()

    return success(user.to_dict(), "注册成功"), 201
```

---

## 4. 日期时间处理
```python
from datetime import datetime

class User(db.Model):
    __tablename__ = 'users'

    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password_hash = db.Column(db.String(256), nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

    def to_dict(self):
        return {
            "id": self.id,
            "username": self.username,
            "email": self.email,
            "created_at": self.created_at.strftime("%Y-%m-%d %H:%M:%S") if self.created_at else None,
            "updated_at": self.updated_at.strftime("%Y-%m-%d %H:%M:%S") if self.updated_at else None
        }
```

`onupdate=datetime.utcnow` 会在每次更新记录时自动更新该字段。

### 时间查询
```python
from datetime import datetime, timedelta

# 查询今天创建的用户
today = datetime.utcnow().date()
users = User.query.filter(
    db.func.date(User.created_at) == today
).all()

# 查询最近 7 天创建的用户
week_ago = datetime.utcnow() - timedelta(days=7)
users = User.query.filter(
    User.created_at >= week_ago
).all()
```

---

## 5. 数据库迁移
当模型发生变化时（添加字段、修改类型），需要迁移数据库。

Flask-Migrate 提供数据库迁移功能：
```plain
pip install flask-migrate
```

```python
from flask_migrate import Migrate

migrate = Migrate(app, db)
```

基本命令：
```plain
flask db init      # 初始化迁移目录
flask db migrate   # 生成迁移脚本
flask db upgrade   # 执行迁移
```

在学习阶段，如果修改了模型，可以先删除旧表再重新创建：
```python
with app.app_context():
    db.drop_all()
    db.create_all()
```

注意：drop_all() 会删除所有数据，仅在开发阶段使用。

---

## 6. 项目结构回顾
```plain
my_app/
├── app/
│   ├── __init__.py          # create_app 工厂函数
│   ├── models/
│   │   ├── __init__.py
│   │   └── user.py          # User 模型
│   ├── routes/
│   │   ├── __init__.py
│   │   ├── auth.py          # 注册、登录接口
│   │   └── user.py          # 用户 CRUD 接口
│   └── utils/
│       ├── __init__.py
│       ├── response.py      # success/error 响应
│       ├── auth.py          # generate_token、login_required
│       └── validators.py    # 参数验证
├── config.py                # 配置文件
└── run.py                   # 启动文件
```

各模块协作关系：
```plain
run.py --> create_app() --> 注册 Blueprint --> 路由处理请求
                               |
                          routes/auth.py  -- 注册/登录
                          routes/user.py  -- 用户管理
                               |
                          models/user.py  -- 数据库操作
                               |
                          utils/response.py  -- 统一响应
                          utils/auth.py      -- Token 验证
                          utils/validators.py -- 参数验证
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 配置管理 | Config 类、环境变量 |
| 分页 | paginate、items/total/pages |
| 参数验证 | 必填、长度、密码强度 |
| 时间处理 | created_at、updated_at、onupdate |
| 数据库迁移 | Flask-Migrate 概念 |
| 项目结构 | 各模块分工协作 |

---

## 课后练习
完成练习（见 [exercises.md](exercises.md)）

---
