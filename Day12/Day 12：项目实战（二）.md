# Day 12：项目实战（二）-- 接口开发
> 学习目标：实现项目的所有 API 接口，完成完整的用户管理功能
>

---

## 1. 响应工具
```python
# app/utils/response.py
from flask import jsonify

def success(data=None, message="success"):
    return jsonify({
        "code": 0,
        "message": message,
        "data": data
    })

def error(message="error", code=1, http_code=400):
    return jsonify({
        "code": code,
        "message": message,
        "data": None
    }), http_code
```

---

## 2. 认证工具
```python
# app/utils/auth.py
import jwt
from datetime import datetime, timedelta
from functools import wraps
from flask import request, g, current_app
from app.utils.response import error

def generate_token(user_id):
    expire_hours = current_app.config.get('JWT_EXPIRE_HOURS', 24)
    payload = {
        "user_id": user_id,
        "exp": datetime.utcnow() + timedelta(hours=expire_hours)
    }
    token = jwt.encode(payload, current_app.config['SECRET_KEY'], algorithm="HS256")
    return token

def verify_token(token):
    try:
        payload = jwt.decode(token, current_app.config['SECRET_KEY'], algorithms=["HS256"])
        return payload["user_id"]
    except jwt.ExpiredSignatureError:
        return None
    except jwt.InvalidTokenError:
        return None

def login_required(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        auth_header = request.headers.get('Authorization', '')

        if not auth_header.startswith('Bearer '):
            return error("请提供 Token", http_code=401)

        token = auth_header.split(' ')[1]
        user_id = verify_token(token)

        if not user_id:
            return error("Token 无效或已过期", http_code=401)

        g.user_id = user_id
        return f(*args, **kwargs)

    return decorated
```

---

## 3. 认证接口
```python
# app/routes/auth.py
from flask import Blueprint, request
from app.models.user import User
from app import db
from app.utils.response import success, error
from app.utils.auth import generate_token

auth_bp = Blueprint('auth', __name__)

@auth_bp.route('/register', methods=['POST'])
def register():
    data = request.get_json()
    if not data:
        return error("请提供 JSON 数据")

    username = data.get('username', '').strip()
    email = data.get('email', '').strip()
    password = data.get('password', '')

    # 验证必填字段
    if not username or not email or not password:
        return error("username、email、password 必填")

    # 验证长度
    if len(username) < 2 or len(username) > 50:
        return error("用户名长度应为 2-50 个字符")

    # 验证密码
    if len(password) < 6:
        return error("密码长度至少 6 位")

    # 验证唯一性
    if User.query.filter_by(username=username).first():
        return error("用户名已存在")

    if User.query.filter_by(email=email).first():
        return error("邮箱已被注册")

    # 创建用户
    user = User(username=username, email=email)
    user.set_password(password)
    db.session.add(user)
    db.session.commit()

    return success(user.to_dict(), "注册成功"), 201

@auth_bp.route('/login', methods=['POST'])
def login():
    data = request.get_json()
    if not data:
        return error("请提供 JSON 数据")

    username = data.get('username', '').strip()
    password = data.get('password', '')

    if not username or not password:
        return error("username 和 password 必填")

    user = User.query.filter_by(username=username).first()
    if not user or not user.check_password(password):
        return error("用户名或密码错误", http_code=401)

    # 检查用户状态
    if user.status != 1:
        return error("账户已被禁用", http_code=403)

    token = generate_token(user.id)

    return success({
        "token": token,
        "user": user.to_dict()
    }, "登录成功")
```

---

## 4. 用户列表接口
```python
# app/routes/user.py
from flask import Blueprint, request, g
from app.models.user import User
from app import db
from app.utils.response import success, error
from app.utils.auth import login_required

user_bp = Blueprint('user', __name__)

@user_bp.route('/users', methods=['GET'])
@login_required
def get_users():
    page = request.args.get('page', 1, type=int)
    size = request.args.get('size', 10, type=int)

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

---

## 5. 用户详情与修改
```python
# app/routes/user.py（续）

@user_bp.route('/users/<int:user_id>', methods=['GET'])
@login_required
def get_user(user_id):
    user = User.query.get(user_id)
    if not user:
        return error("用户不存在", http_code=404)
    return success(user.to_dict())

@user_bp.route('/users/<int:user_id>', methods=['PUT'])
@login_required
def update_user(user_id):
    user = User.query.get(user_id)
    if not user:
        return error("用户不存在", http_code=404)

    data = request.get_json()
    if not data:
        return error("请提供 JSON 数据")

    # 更新邮箱
    if 'email' in data:
        email = data['email'].strip()
        if email and email != user.email:
            if User.query.filter_by(email=email).first():
                return error("邮箱已被使用")
            user.email = email

    # 更新手机号
    if 'phone' in data:
        user.phone = data['phone']

    db.session.commit()
    return success(user.to_dict(), "更新成功")

@user_bp.route('/profile', methods=['GET'])
@login_required
def get_profile():
    user = User.query.get(g.user_id)
    if not user:
        return error("用户不存在", http_code=404)
    return success(user.to_dict())
```

---

## 6. 用户删除
```python
# app/routes/user.py（续）

@user_bp.route('/users/<int:user_id>', methods=['DELETE'])
@login_required
def delete_user(user_id):
    user = User.query.get(user_id)
    if not user:
        return error("用户不存在", http_code=404)

    db.session.delete(user)
    db.session.commit()
    return success(message="删除成功")
```

---

## 7. 接口测试
使用 curl 完整测试所有接口：
```plain
# 1. 注册用户
curl -X POST http://localhost:5000/api/register \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "email": "admin@example.com", "password": "admin123"}'

# 2. 登录
curl -X POST http://localhost:5000/api/login \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "admin123"}'
# 记录返回的 Token

# 3. 获取用户列表
curl http://localhost:5000/api/users \
  -H "Authorization: Bearer <your-token>"

# 4. 获取用户详情
curl http://localhost:5000/api/users/1 \
  -H "Authorization: Bearer <your-token>"

# 5. 修改用户
curl -X PUT http://localhost:5000/api/users/1 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <your-token>" \
  -d '{"email": "new@example.com", "phone": "13800138000"}'

# 6. 获取个人信息
curl http://localhost:5000/api/profile \
  -H "Authorization: Bearer <your-token>"

# 7. 删除用户（先创建一个新用户再删除）
curl -X DELETE http://localhost:5000/api/users/2 \
  -H "Authorization: Bearer <your-token>"

# 8. 分页查询
curl "http://localhost:5000/api/users?page=1&size=5" \
  -H "Authorization: Bearer <your-token>"
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 响应工具 | success/error 统一格式 |
| 认证工具 | generate_token、login_required |
| 注册接口 | 验证 + 创建 + 加密密码 |
| 登录接口 | 验证密码 + 生成 Token |
| 用户 CRUD | 列表、详情、修改、删除 |
| 接口测试 | curl 全流程测试 |

---

## 课后练习
完成练习（见 [exercises.md](exercises.md)）

---
