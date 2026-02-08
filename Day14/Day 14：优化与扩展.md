# Day 14：优化与扩展
> 学习目标：优化项目代码，扩展功能，为 Day 15 做好衔接准备
>

---

## 1. 代码优化
### 提取公共模式
将重复的验证逻辑提取为通用函数：
```python
# app/utils/validators.py

def validate_required(data, fields):
    """验证必填字段，返回第一个缺失字段的错误信息"""
    for field in fields:
        value = data.get(field)
        if isinstance(value, str):
            value = value.strip()
        if not value:
            return f"{field} 不能为空"
    return None

def validate_length(value, name, min_len=None, max_len=None):
    """验证字符串长度"""
    if min_len and len(value) < min_len:
        return f"{name} 长度不能少于 {min_len} 个字符"
    if max_len and len(value) > max_len:
        return f"{name} 长度不能超过 {max_len} 个字符"
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

### 日志基础
```python
import logging

# 配置日志
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s [%(levelname)s] %(message)s'
)
logger = logging.getLogger(__name__)

# 在代码中使用
logger.info(f"用户注册：{username}")
logger.error(f"登录失败：{username}")
logger.warning(f"无效请求：缺少必填字段")
```

---

## 2. 输入验证增强
### 邮箱格式验证
```python
import re

def validate_email(email):
    """验证邮箱格式"""
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    if not re.match(pattern, email):
        return "邮箱格式不正确"
    return None
```

### 密码强度增强
```python
def validate_password_strong(password):
    """增强版密码验证"""
    if len(password) < 6:
        return "密码长度至少 6 位"
    if len(password) > 20:
        return "密码长度不能超过 20 位"
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
        return error("请提供 JSON 数据")

    # 验证必填字段
    err = validate_required(data, ['username', 'email', 'password'])
    if err:
        return error(err)

    username = data['username'].strip()
    email = data['email'].strip()
    password = data['password']

    # 验证用户名长度
    err = validate_length(username, "用户名", min_len=2, max_len=50)
    if err:
        return error(err)

    # 验证邮箱格式
    err = validate_email(email)
    if err:
        return error(err)

    # 验证密码强度
    err = validate_password(password)
    if err:
        return error(err)

    # 验证唯一性
    if User.query.filter_by(username=username).first():
        return error("用户名已存在")
    if User.query.filter_by(email=email).first():
        return error("邮箱已被注册")

    user = User(username=username, email=email)
    user.set_password(password)
    db.session.add(user)
    db.session.commit()

    logger.info(f"新用户注册：{username}")
    return success(user.to_dict(), "注册成功"), 201
```

---

## 3. 查询优化
### 排序
```python
@user_bp.route('/users', methods=['GET'])
@login_required
def get_users():
    page = request.args.get('page', 1, type=int)
    size = request.args.get('size', 10, type=int)
    sort = request.args.get('sort', 'created_at')
    order = request.args.get('order', 'desc')

    if size > 100:
        size = 100

    # 动态排序
    sort_column = getattr(User, sort, User.created_at)
    if order == 'asc':
        query = User.query.order_by(sort_column.asc())
    else:
        query = User.query.order_by(sort_column.desc())

    pagination = query.paginate(page=page, per_page=size, error_out=False)

    return success({
        "list": [u.to_dict() for u in pagination.items],
        "total": pagination.total,
        "page": pagination.page,
        "pages": pagination.pages
    })
```

### 多条件筛选
```python
@user_bp.route('/users', methods=['GET'])
@login_required
def get_users():
    page = request.args.get('page', 1, type=int)
    size = request.args.get('size', 10, type=int)
    keyword = request.args.get('keyword', '').strip()
    status = request.args.get('status', type=int)

    if size > 100:
        size = 100

    query = User.query

    # 关键词搜索
    if keyword:
        query = query.filter(
            (User.username.like(f'%{keyword}%')) |
            (User.email.like(f'%{keyword}%'))
        )

    # 状态筛选
    if status is not None:
        query = query.filter(User.status == status)

    pagination = query.order_by(
        User.created_at.desc()
    ).paginate(page=page, per_page=size, error_out=False)

    return success({
        "list": [u.to_dict() for u in pagination.items],
        "total": pagination.total,
        "page": pagination.page,
        "pages": pagination.pages
    })
```

### 模糊查询
```python
# like 模糊查询
users = User.query.filter(User.username.like(f'%{keyword}%')).all()

# 多字段搜索
from sqlalchemy import or_
users = User.query.filter(
    or_(
        User.username.like(f'%{keyword}%'),
        User.email.like(f'%{keyword}%')
    )
).all()
```

---

## 4. 接口扩展
### 搜索参数
为 Day 15 的搜索功能做铺垫：
```plain
# 搜索用户
curl "http://localhost:5000/api/users?keyword=张" \
  -H "Authorization: Bearer <token>"

# 搜索 + 分页
curl "http://localhost:5000/api/users?keyword=张&page=1&size=5" \
  -H "Authorization: Bearer <token>"

# 搜索 + 状态筛选
curl "http://localhost:5000/api/users?keyword=张&status=1" \
  -H "Authorization: Bearer <token>"
```

### 统计端点
```python
from sqlalchemy import func
from datetime import datetime, timedelta

@user_bp.route('/stats/users', methods=['GET'])
@login_required
def user_stats():
    total = User.query.count()

    today = datetime.utcnow().date()
    today_count = User.query.filter(
        func.date(User.created_at) == today
    ).count()

    week_ago = datetime.utcnow() - timedelta(days=7)
    week_count = User.query.filter(
        User.created_at >= week_ago
    ).count()

    return success({
        "total": total,
        "today": today_count,
        "week": week_count
    })
```

---

## 5. 项目回顾
### 14 天所学技能总结
| 阶段 | 内容 | 关键技能 |
| --- | --- | --- |
| Day 1-4 | Python 基础 | 变量、数据类型、条件循环、函数、文件操作 |
| Day 5 | 面向对象 | class、__init__、方法、异常处理 |
| Day 6-7 | Flask 入门 | 路由、Blueprint、工厂函数、CRUD |
| Day 8 | 数据库 | SQLAlchemy ORM、模型定义、CRUD |
| Day 9 | 认证 | 密码加密、JWT、login_required |
| Day 10 | 项目完善 | 配置、分页、验证 |
| Day 11-13 | 项目实战 | 完整项目、Git、文档 |
| Day 14 | 优化扩展 | 验证增强、查询优化、搜索统计 |

### 已掌握的能力
+ 使用 Python 编写脚本和处理数据
+ 使用 Flask 搭建 RESTful API
+ 使用 SQLAlchemy 操作 MySQL 数据库
+ 实现 JWT 用户认证
+ 使用 Git 管理代码和协作

---

## 6. 下一步预告
Day 15 将继续扩展项目功能：
+ 用户搜索功能（基于 Day 14 的 keyword 参数）
+ 数据统计接口（基于 Day 14 的统计端点）
+ 操作日志记录
+ 用户状态管理

Day 14 实现的搜索和统计功能是 Day 15 的基础，确保这些功能已经正常工作。

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 代码优化 | 提取公共函数、日志 |
| 验证增强 | 邮箱正则、密码强度 |
| 查询优化 | 排序、多条件筛选、like |
| 搜索 | 关键词搜索、分页 |
| 统计 | count、日期查询 |
| 回顾 | 14 天技能总结 |

---

## 课后练习
完成练习（见 [exercises.md](exercises.md)）

---
