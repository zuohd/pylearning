# Day 7：Flask 项目组织
> 学习目标：学会使用 Blueprint 和工厂函数组织 Flask 项目
>

---

## 1. 项目结构
当应用变大时，所有代码放在一个文件中会难以维护。合理的项目结构：
```plain
my_app/
├── app/
│   ├── __init__.py      # 工厂函数
│   ├── models/          # 数据模型
│   ├── routes/          # 路由（Blueprint）
│   │   ├── __init__.py
│   │   └── user.py
│   └── utils/           # 工具函数
│       ├── __init__.py
│       └── response.py
├── config.py            # 配置文件
└── run.py               # 启动文件
```

---

## 2. Blueprint
Blueprint 是 Flask 提供的模块化组织方式，可以将路由分组管理。

```python
# app/routes/user.py
from flask import Blueprint, jsonify, request

user_bp = Blueprint('user', __name__)

users = []
next_id = 1

@user_bp.route('/users', methods=['GET'])
def get_users():
    return jsonify({"code": 0, "data": users})

@user_bp.route('/users', methods=['POST'])
def create_user():
    global next_id
    data = request.get_json()
    user = {"id": next_id, "name": data.get("name")}
    users.append(user)
    next_id += 1
    return jsonify({"code": 0, "data": user}), 201
```

### 注册 Blueprint
```python
# app/__init__.py
from flask import Flask

def create_app():
    app = Flask(__name__)

    from app.routes.user import user_bp
    app.register_blueprint(user_bp, url_prefix='/api')

    return app
```

注册时使用 `url_prefix='/api'`，Blueprint 中定义的 `/users` 实际路径变为 `/api/users`。

---

## 3. 工厂函数
工厂函数 `create_app()` 是 Flask 推荐的应用创建模式：
```python
# app/__init__.py
from flask import Flask

def create_app():
    app = Flask(__name__)

    # 加载配置
    app.config['DEBUG'] = True

    # 注册 Blueprint
    from app.routes.user import user_bp
    app.register_blueprint(user_bp, url_prefix='/api')

    return app
```

```python
# run.py
from app import create_app

app = create_app()

if __name__ == '__main__':
    app.run(debug=True)
```

**为什么用工厂函数**：
+ 避免循环导入
+ 方便创建不同配置的应用（开发、测试、生产）
+ Flask 推荐做法

---

## 4. 响应工具
将统一的响应格式提取为工具函数：
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

在路由中使用：
```python
# app/routes/user.py
from flask import Blueprint, request
from app.utils.response import success, error

user_bp = Blueprint('user', __name__)

users = []
next_id = 1

@user_bp.route('/users', methods=['GET'])
def get_users():
    return success(users)

@user_bp.route('/users', methods=['POST'])
def create_user():
    global next_id
    data = request.get_json()
    if not data or not data.get('name'):
        return error("name 必填")

    user = {"id": next_id, "name": data['name'], "email": data.get('email', '')}
    users.append(user)
    next_id += 1
    return success(user, "创建成功"), 201
```

---

## 5. 完整 CRUD
```python
# app/routes/user.py
from flask import Blueprint, request
from app.utils.response import success, error

user_bp = Blueprint('user', __name__)

users = []
next_id = 1

def find_user(user_id):
    for user in users:
        if user['id'] == user_id:
            return user
    return None

# 创建用户
@user_bp.route('/users', methods=['POST'])
def create_user():
    global next_id
    data = request.get_json()
    if not data or not data.get('name'):
        return error("name 必填")

    user = {
        "id": next_id,
        "name": data['name'],
        "email": data.get('email', '')
    }
    users.append(user)
    next_id += 1
    return success(user, "创建成功"), 201

# 获取用户列表
@user_bp.route('/users', methods=['GET'])
def get_users():
    return success(users)

# 获取单个用户
@user_bp.route('/users/<int:user_id>', methods=['GET'])
def get_user(user_id):
    user = find_user(user_id)
    if not user:
        return error("用户不存在", http_code=404)
    return success(user)

# 更新用户
@user_bp.route('/users/<int:user_id>', methods=['PUT'])
def update_user(user_id):
    user = find_user(user_id)
    if not user:
        return error("用户不存在", http_code=404)

    data = request.get_json()
    if data.get('name'):
        user['name'] = data['name']
    if data.get('email'):
        user['email'] = data['email']

    return success(user, "更新成功")

# 删除用户
@user_bp.route('/users/<int:user_id>', methods=['DELETE'])
def delete_user(user_id):
    user = find_user(user_id)
    if not user:
        return error("用户不存在", http_code=404)

    users.remove(user)
    return success(message="删除成功")
```

---

## 6. 错误处理
```python
# app/__init__.py
from flask import Flask, jsonify

def create_app():
    app = Flask(__name__)

    from app.routes.user import user_bp
    app.register_blueprint(user_bp, url_prefix='/api')

    # 全局错误处理
    @app.errorhandler(404)
    def not_found(e):
        return jsonify({
            "code": 1,
            "message": "接口不存在",
            "data": None
        }), 404

    @app.errorhandler(500)
    def server_error(e):
        return jsonify({
            "code": 1,
            "message": "服务器内部错误",
            "data": None
        }), 500

    @app.errorhandler(405)
    def method_not_allowed(e):
        return jsonify({
            "code": 1,
            "message": "请求方法不允许",
            "data": None
        }), 405

    return app
```

### 输入验证
```python
@user_bp.route('/users', methods=['POST'])
def create_user():
    data = request.get_json()

    # 验证请求体
    if not data:
        return error("请提供 JSON 数据")

    # 验证必填字段
    name = data.get('name', '').strip()
    if not name:
        return error("name 不能为空")

    # 验证长度
    if len(name) > 50:
        return error("name 长度不能超过 50")

    email = data.get('email', '').strip()

    # 创建用户...
```

---

## 7. 测试 API
### 使用 curl
```plain
# 创建用户
curl -X POST http://localhost:5000/api/users \
  -H "Content-Type: application/json" \
  -d '{"name": "张三", "email": "zhangsan@example.com"}'

# 获取用户列表
curl http://localhost:5000/api/users

# 获取单个用户
curl http://localhost:5000/api/users/1

# 更新用户
curl -X PUT http://localhost:5000/api/users/1 \
  -H "Content-Type: application/json" \
  -d '{"name": "张三三"}'

# 删除用户
curl -X DELETE http://localhost:5000/api/users/1
```

### 使用 Postman
Postman 是一个图形化 API 测试工具：
+ 下载安装 Postman
+ 选择 HTTP 方法（GET/POST/PUT/DELETE）
+ 输入 URL
+ 设置 Body（选择 raw、JSON 格式）
+ 点击 Send

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 项目结构 | app/routes/utils/config |
| Blueprint | 路由分组、url_prefix |
| 工厂函数 | create_app 模式 |
| 响应工具 | success/error 统一格式 |
| CRUD | Create/Read/Update/Delete |
| 错误处理 | errorhandler、输入验证 |
| 测试 | curl、Postman |

---

## 课后练习
完成练习（见 [exercises.md](exercises.md)）

---
