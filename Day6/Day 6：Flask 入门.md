# Day 6：Flask 入门
> 学习目标：理解 Web 开发基础概念，学会用 Flask 创建简单的 API
>

---

## 1. Web 开发简介
### 客户端-服务器模型
```plain
客户端（浏览器/App）  <--HTTP-->  服务器（Flask 应用）
        |                              |
    发送请求                       处理请求
    显示响应                       返回响应
```

### HTTP 基础
+ **URL**：资源地址，如 `http://localhost:5000/api/users`
+ **HTTP 方法**：
  + GET：获取数据
  + POST：创建数据
  + PUT：更新数据
  + DELETE：删除数据
+ **状态码**：
  + 200：成功
  + 201：创建成功
  + 400：请求错误
  + 404：未找到
  + 500：服务器错误

### API
API（Application Programming Interface）是服务器提供的接口，客户端通过 API 与服务器交互，数据格式通常为 JSON。

---

## 2. Flask 安装与第一个应用
### 安装
```plain
pip install flask
```

### 最小 Flask 应用
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return 'Hello, Flask!'

if __name__ == '__main__':
    app.run(debug=True)
```

运行程序后，在浏览器访问 `http://localhost:5000`。

**代码解析**：
+ `Flask(__name__)`：创建 Flask 应用实例
+ `@app.route('/')`：路由装饰器，将 URL 映射到函数
+ `app.run(debug=True)`：启动开发服务器，debug 模式下代码修改自动重启

---

## 3. 路由基础
```python
from flask import Flask

app = Flask(__name__)

# 基本路由
@app.route('/')
def index():
    return 'Hello, Flask!'

# 多个路由
@app.route('/about')
def about():
    return '关于我们'

# 动态路由
@app.route('/user/<username>')
def show_user(username):
    return f'用户：{username}'

# 带类型的动态路由
@app.route('/user/<int:user_id>')
def get_user(user_id):
    return f'用户 ID：{user_id}'

if __name__ == '__main__':
    app.run(debug=True)
```

访问测试：
+ `http://localhost:5000/` -- Hello, Flask!
+ `http://localhost:5000/about` -- 关于我们
+ `http://localhost:5000/user/zhangsan` -- 用户：zhangsan
+ `http://localhost:5000/user/1` -- 用户 ID：1

---

## 4. 返回 JSON
```python
from flask import Flask, jsonify

app = Flask(__name__)

# 返回 JSON 数据
@app.route('/api/info')
def info():
    return jsonify({
        "name": "Flask API",
        "version": "1.0"
    })

# 统一响应格式
def success(data=None, message="success"):
    return jsonify({
        "code": 0,
        "message": message,
        "data": data
    })

def error(message="error", code=1):
    return jsonify({
        "code": code,
        "message": message,
        "data": None
    })

@app.route('/api/users')
def get_users():
    users = [
        {"id": 1, "name": "张三"},
        {"id": 2, "name": "李四"}
    ]
    return success(users)

@app.route('/api/user/<int:user_id>')
def get_user(user_id):
    if user_id == 1:
        return success({"id": 1, "name": "张三"})
    return error("用户不存在"), 404

if __name__ == '__main__':
    app.run(debug=True)
```

---

## 5. HTTP 方法
```python
from flask import Flask, jsonify, request

app = Flask(__name__)

# 指定 HTTP 方法
@app.route('/api/users', methods=['GET'])
def get_users():
    return jsonify({"message": "获取用户列表"})

@app.route('/api/users', methods=['POST'])
def create_user():
    return jsonify({"message": "创建用户"}), 201

@app.route('/api/users/<int:user_id>', methods=['PUT'])
def update_user(user_id):
    return jsonify({"message": f"更新用户 {user_id}"})

@app.route('/api/users/<int:user_id>', methods=['DELETE'])
def delete_user(user_id):
    return jsonify({"message": f"删除用户 {user_id}"})

if __name__ == '__main__':
    app.run(debug=True)
```

---

## 6. 请求参数
```python
from flask import Flask, jsonify, request

app = Flask(__name__)

# GET 查询参数
@app.route('/api/search')
def search():
    keyword = request.args.get('keyword', '')
    page = request.args.get('page', 1, type=int)
    return jsonify({
        "keyword": keyword,
        "page": page
    })
# 访问：/api/search?keyword=python&page=2

# POST JSON 数据
@app.route('/api/users', methods=['POST'])
def create_user():
    data = request.get_json()
    if not data:
        return jsonify({"error": "请提供 JSON 数据"}), 400

    name = data.get('name')
    email = data.get('email')

    if not name or not email:
        return jsonify({"error": "name 和 email 必填"}), 400

    return jsonify({
        "message": "创建成功",
        "user": {"name": name, "email": email}
    }), 201

if __name__ == '__main__':
    app.run(debug=True)
```

使用 curl 测试 POST 请求：
```plain
curl -X POST http://localhost:5000/api/users \
  -H "Content-Type: application/json" \
  -d '{"name": "张三", "email": "zhangsan@example.com"}'
```

---

## 7. 完整示例
```python
from flask import Flask, jsonify, request

app = Flask(__name__)

# 内存存储
users = []
next_id = 1

def success(data=None, message="success"):
    return jsonify({"code": 0, "message": message, "data": data})

def error(message="error"):
    return jsonify({"code": 1, "message": message, "data": None})

# 获取用户列表
@app.route('/api/users', methods=['GET'])
def get_users():
    return success(users)

# 创建用户
@app.route('/api/users', methods=['POST'])
def create_user():
    global next_id
    data = request.get_json()

    if not data or not data.get('name'):
        return error("name 必填"), 400

    user = {
        "id": next_id,
        "name": data['name'],
        "email": data.get('email', '')
    }
    users.append(user)
    next_id += 1
    return success(user, "创建成功"), 201

# 获取单个用户
@app.route('/api/users/<int:user_id>', methods=['GET'])
def get_user(user_id):
    for user in users:
        if user['id'] == user_id:
            return success(user)
    return error("用户不存在"), 404

# 删除用户
@app.route('/api/users/<int:user_id>', methods=['DELETE'])
def delete_user(user_id):
    for user in users:
        if user['id'] == user_id:
            users.remove(user)
            return success(message="删除成功")
    return error("用户不存在"), 404

if __name__ == '__main__':
    app.run(debug=True)
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| Web 概念 | 客户端-服务器、HTTP、API |
| Flask 基础 | Flask()、app.run() |
| 路由 | @app.route、动态路由 |
| JSON 响应 | jsonify、统一响应格式 |
| HTTP 方法 | GET/POST/PUT/DELETE |
| 请求参数 | request.args.get、request.get_json |

---

## 课后练习
完成练习（见 [exercises.md](exercises.md)）

---
