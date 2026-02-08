# Day 9：用户认证
> 学习目标：实现用户注册登录功能，掌握密码加密和 JWT 认证
>

---

## 1. 认证概述
### 为什么需要认证
+ 确认用户身份
+ 保护用户数据
+ 控制访问权限

### Session vs Token
| 特性 | Session | Token (JWT) |
| --- | --- | --- |
| 存储位置 | 服务器 | 客户端 |
| 扩展性 | 需要共享 Session | 无状态，易扩展 |
| 适用场景 | 传统 Web | API / 前后端分离 |

### JWT 认证流程
```plain
1. 用户提交用户名和密码
2. 服务器验证成功，生成 JWT Token
3. 返回 Token 给客户端
4. 客户端后续请求携带 Token（放在 Header 中）
5. 服务器验证 Token，确认身份
```

---

## 2. 密码加密
密码不能明文存储，使用 werkzeug 提供的哈希函数加密。

```python
from werkzeug.security import generate_password_hash, check_password_hash

# 加密密码
password = "123456"
hashed = generate_password_hash(password)
print(hashed)
# pbkdf2:sha256:260000$xxx...（每次不同）

# 验证密码
print(check_password_hash(hashed, "123456"))  # True
print(check_password_hash(hashed, "654321"))  # False
```

在 User 模型中添加密码支持：
```python
class User(db.Model):
    __tablename__ = 'users'

    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password_hash = db.Column(db.String(256), nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)

    def set_password(self, password):
        self.password_hash = generate_password_hash(password)

    def check_password(self, password):
        return check_password_hash(self.password_hash, password)

    def to_dict(self):
        return {
            "id": self.id,
            "username": self.username,
            "email": self.email,
            "created_at": self.created_at.strftime("%Y-%m-%d %H:%M:%S") if self.created_at else None
        }
```

注意：to_dict() 中不返回 password_hash 字段。

---

## 3. 用户注册
```python
from flask import Blueprint, request
from app.utils.response import success, error
from app.models.user import User, db

auth_bp = Blueprint('auth', __name__)

@auth_bp.route('/register', methods=['POST'])
def register():
    data = request.get_json()
    if not data:
        return error("请提供 JSON 数据"), 400

    username = data.get('username', '').strip()
    email = data.get('email', '').strip()
    password = data.get('password', '')

    # 验证必填字段
    if not username or not email or not password:
        return error("username、email、password 必填"), 400

    # 验证用户名是否已存在
    if User.query.filter_by(username=username).first():
        return error("用户名已存在"), 400

    # 验证邮箱是否已存在
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

## 4. JWT 认证
### 安装
```plain
pip install pyjwt
```

### 生成和验证 Token
```python
import jwt
from datetime import datetime, timedelta

SECRET_KEY = "your-secret-key"

# 生成 Token
def generate_token(user_id):
    payload = {
        "user_id": user_id,
        "exp": datetime.utcnow() + timedelta(hours=24)  # 24 小时过期
    }
    token = jwt.encode(payload, SECRET_KEY, algorithm="HS256")
    return token

# 验证 Token
def verify_token(token):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        return payload["user_id"]
    except jwt.ExpiredSignatureError:
        return None  # Token 过期
    except jwt.InvalidTokenError:
        return None  # Token 无效
```

---

## 5. 用户登录
```python
@auth_bp.route('/login', methods=['POST'])
def login():
    data = request.get_json()
    if not data:
        return error("请提供 JSON 数据"), 400

    username = data.get('username', '').strip()
    password = data.get('password', '')

    if not username or not password:
        return error("username 和 password 必填"), 400

    # 查找用户
    user = User.query.filter_by(username=username).first()
    if not user or not user.check_password(password):
        return error("用户名或密码错误"), 401

    # 生成 Token
    token = generate_token(user.id)

    return success({
        "token": token,
        "user": user.to_dict()
    }, "登录成功")
```

---

## 6. 登录验证装饰器
```python
from functools import wraps
from flask import request, g
from app.utils.response import error

def login_required(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        # 从 Header 获取 Token
        auth_header = request.headers.get('Authorization', '')

        if not auth_header.startswith('Bearer '):
            return error("请提供 Token"), 401

        token = auth_header.split(' ')[1]

        # 验证 Token
        user_id = verify_token(token)
        if not user_id:
            return error("Token 无效或已过期"), 401

        # 将用户 ID 存入 g 对象
        g.user_id = user_id
        return f(*args, **kwargs)

    return decorated
```

使用装饰器保护接口：
```python
@user_bp.route('/users', methods=['GET'])
@login_required
def get_users():
    users = User.query.all()
    return success([u.to_dict() for u in users])

@user_bp.route('/profile', methods=['GET'])
@login_required
def get_profile():
    user = User.query.get(g.user_id)
    if not user:
        return error("用户不存在"), 404
    return success(user.to_dict())
```

`g` 是 Flask 提供的全局对象，在一次请求中共享数据。login_required 装饰器将 user_id 存入 g，后续视图函数可通过 `g.user_id` 获取当前登录用户的 ID。

---

## 7. 完整认证流程
```plain
# 1. 注册
curl -X POST http://localhost:5000/api/register \
  -H "Content-Type: application/json" \
  -d '{"username": "张三", "email": "zhangsan@example.com", "password": "123456"}'

# 响应：
# {"code": 0, "message": "注册成功", "data": {"id": 1, "username": "张三", ...}}

# 2. 登录
curl -X POST http://localhost:5000/api/login \
  -H "Content-Type: application/json" \
  -d '{"username": "张三", "password": "123456"}'

# 响应：
# {"code": 0, "message": "登录成功", "data": {"token": "eyJ...", "user": {...}}}

# 3. 使用 Token 访问受保护接口
curl http://localhost:5000/api/users \
  -H "Authorization: Bearer eyJ..."

# 4. 不带 Token 访问（被拒绝）
curl http://localhost:5000/api/users
# {"code": 1, "message": "请提供 Token"}
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 认证方式 | Session vs Token (JWT) |
| 密码加密 | generate/check_password_hash |
| 注册 | 验证、创建用户、加密密码 |
| JWT | encode/decode、过期时间 |
| 登录 | 验证密码、生成 Token |
| login_required | 装饰器、Authorization Header、g.user_id |

---

## 课后练习
完成练习（见 [exercises.md](exercises.md)）

---
