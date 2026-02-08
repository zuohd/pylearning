# Day 8：数据库基础
> 学习目标：理解数据库概念，学会用 SQLAlchemy ORM 操作 MySQL 数据库
>

---

## 1. 数据库简介
之前的数据存储在内存列表中，程序重启后数据丢失。数据库可以持久化存储数据。

### 为什么用数据库
+ 数据持久化存储
+ 支持大量数据的高效查询
+ 数据安全性和完整性
+ 多用户并发访问

### MySQL
MySQL 是最流行的关系型数据库之一。安装方式：
```plain
# Ubuntu/Debian
sudo apt install mysql-server

# macOS
brew install mysql

# Windows
下载安装包：https://dev.mysql.com/downloads/
```

安装后创建数据库：
```plain
mysql -u root -p
CREATE DATABASE myapp CHARACTER SET utf8mb4;
```

---

## 2. SQLAlchemy 简介
SQLAlchemy 是 Python 的 ORM（Object Relational Mapping）框架。ORM 将数据库表映射为 Python 类，将行映射为对象。

```plain
数据库表 User        <-->  Python 类 User
表中的一行           <-->  User 对象
字段（name, email）  <-->  对象属性（self.name, self.email）
SQL 查询             <-->  Python 方法调用
```

### 安装
```plain
pip install flask-sqlalchemy pymysql
```

### 配置
```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://root:password@localhost/myapp'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)
```

配置说明：
+ `mysql+pymysql`：使用 pymysql 驱动连接 MySQL
+ `root:password`：数据库用户名和密码
+ `localhost`：数据库地址
+ `myapp`：数据库名

---

## 3. 定义模型
```python
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

db = SQLAlchemy()

class User(db.Model):
    __tablename__ = 'users'

    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)

    def __repr__(self):
        return f'<User {self.username}>'
```

**常用字段类型**：
| SQLAlchemy 类型 | 说明 |
| --- | --- |
| db.Integer | 整数 |
| db.String(n) | 字符串，n 为最大长度 |
| db.Text | 长文本 |
| db.Float | 浮点数 |
| db.Boolean | 布尔值 |
| db.DateTime | 日期时间 |

**常用约束**：
| 约束 | 说明 |
| --- | --- |
| primary_key=True | 主键 |
| unique=True | 唯一 |
| nullable=False | 不可为空 |
| default=value | 默认值 |

---

## 4. 创建数据库表
```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://root:password@localhost/myapp'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)

class User(db.Model):
    __tablename__ = 'users'

    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)

# 创建表
with app.app_context():
    db.create_all()
    print("数据库表创建成功")
```

`app_context()` 是 Flask 应用上下文，数据库操作必须在应用上下文中执行。

---

## 5. 基本 CRUD 操作
### 创建（Create）
```python
with app.app_context():
    user = User(username="张三", email="zhangsan@example.com")
    db.session.add(user)
    db.session.commit()
    print(f"创建用户：{user.id}")
```

### 查询（Read）
```python
with app.app_context():
    # 查询所有
    users = User.query.all()
    for u in users:
        print(u.username, u.email)

    # 按 ID 查询
    user = User.query.get(1)
    print(user.username)

    # 条件查询
    user = User.query.filter_by(username="张三").first()
    print(user.email)

    # 多条件查询
    users = User.query.filter(
        User.username.like("%张%")
    ).all()

    # 排序
    users = User.query.order_by(User.created_at.desc()).all()

    # 限制数量
    users = User.query.limit(10).all()
```

### 更新（Update）
```python
with app.app_context():
    user = User.query.get(1)
    if user:
        user.email = "new_email@example.com"
        db.session.commit()
        print("更新成功")
```

### 删除（Delete）
```python
with app.app_context():
    user = User.query.get(1)
    if user:
        db.session.delete(user)
        db.session.commit()
        print("删除成功")
```

---

## 6. 模型方法
```python
class User(db.Model):
    __tablename__ = 'users'

    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)

    def __repr__(self):
        return f'<User {self.username}>'

    def to_dict(self):
        return {
            "id": self.id,
            "username": self.username,
            "email": self.email,
            "created_at": self.created_at.strftime("%Y-%m-%d %H:%M:%S") if self.created_at else None
        }
```

to_dict() 将模型对象转换为字典，方便通过 jsonify 返回 JSON 响应。

---

## 7. 集成到 Flask
将内存列表替换为数据库操作：
```python
from flask import Flask, request
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://root:password@localhost/myapp'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

db = SQLAlchemy(app)

class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)

    def to_dict(self):
        return {
            "id": self.id,
            "username": self.username,
            "email": self.email,
            "created_at": self.created_at.strftime("%Y-%m-%d %H:%M:%S") if self.created_at else None
        }

def success(data=None, message="success"):
    from flask import jsonify
    return jsonify({"code": 0, "message": message, "data": data})

def error(message="error"):
    from flask import jsonify
    return jsonify({"code": 1, "message": message, "data": None})

@app.route('/api/users', methods=['GET'])
def get_users():
    users = User.query.all()
    return success([u.to_dict() for u in users])

@app.route('/api/users', methods=['POST'])
def create_user():
    data = request.get_json()
    if not data or not data.get('username') or not data.get('email'):
        return error("username 和 email 必填"), 400

    user = User(username=data['username'], email=data['email'])
    db.session.add(user)
    db.session.commit()
    return success(user.to_dict(), "创建成功"), 201

@app.route('/api/users/<int:user_id>', methods=['GET'])
def get_user(user_id):
    user = User.query.get(user_id)
    if not user:
        return error("用户不存在"), 404
    return success(user.to_dict())

@app.route('/api/users/<int:user_id>', methods=['DELETE'])
def delete_user(user_id):
    user = User.query.get(user_id)
    if not user:
        return error("用户不存在"), 404
    db.session.delete(user)
    db.session.commit()
    return success(message="删除成功")

if __name__ == '__main__':
    with app.app_context():
        db.create_all()
    app.run(debug=True)
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 数据库 | 持久化存储、MySQL |
| ORM | 表映射为类、行映射为对象 |
| 模型定义 | db.Model、Column 类型和约束 |
| 创建表 | db.create_all()、app_context |
| CRUD | add/commit/query/filter/delete |
| to_dict | 模型转字典，用于 JSON 响应 |

---

## 课后练习
完成练习（见 [exercises.md](exercises.md)）

---
