# Day 15：项目扩展
> 学习目标：为项目添加新功能，巩固所学知识
>

---

## 1. 可选功能
选择 1-2 个功能实现：

+ 用户搜索
+ 数据统计
+ 操作日志
+ 用户状态管理

---

## 2. 用户搜索
```python
@user_bp.route('/users', methods=['GET'])
@login_required
def list_users():
    page = request.args.get('page', 1, type=int)
    size = request.args.get('size', 10, type=int)
    keyword = request.args.get('keyword', '').strip()

    query = User.query
    if keyword:
        query = query.filter(
            (User.username.like(f'%{keyword}%')) |
            (User.email.like(f'%{keyword}%'))
        )

    pagination = query.order_by(User.created_at.desc()).paginate(
        page=page, per_page=size, error_out=False
    )

    return success({
        "list": [u.to_dict() for u in pagination.items],
        "total": pagination.total
    })
```

---

## 3. 数据统计
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

## 4. 操作日志
```python
# app/models/log.py
class OperationLog(db.Model):
    __tablename__ = 'operation_logs'

    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, nullable=False)
    action = db.Column(db.String(50), nullable=False)
    target = db.Column(db.String(100))
    ip = db.Column(db.String(50))
    created_at = db.Column(db.DateTime, default=datetime.utcnow)

# 记录日志
def log_action(action, target=None):
    log = OperationLog(
        user_id=getattr(g, 'user_id', 0),
        action=action,
        target=target,
        ip=request.remote_addr
    )
    db.session.add(log)
    db.session.commit()
```

---

## 5. 用户状态
```python
# 添加状态字段
status = db.Column(db.Integer, default=1)  # 1:正常 0:禁用

# 禁用/启用接口
@user_bp.route('/users/<int:user_id>/status', methods=['PUT'])
@login_required
def update_status(user_id):
    user = User.query.get(user_id)
    data = request.get_json()
    user.status = data.get('status')
    db.session.commit()
    return success(user.to_dict())
```

---

## 6. 学习总结
### 15 天回顾
| 阶段 | 内容 |
| --- | --- |
| Day 1-4 | Python 基础 |
| Day 5-7 | Flask 入门 |
| Day 8-10 | 数据库认证 |
| Day 11-13 | 项目实战 |
| Day 14-15 | 优化扩展 |


### 掌握技能
+ Python 基础语法和 OOP
+ Flask Web 框架
+ RESTful API 设计
+ MySQL + SQLAlchemy
+ JWT 认证
+ Git 版本控制

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 搜索 | like 模糊查询 |
| 统计 | count/func |
| 日志 | 记录操作 |
| 状态 | 用户管理 |


---

## 课后练习
完成练习题 26-28（见 [exercises.md](exercises.md)）

---

**15 天 Python 后端学习完成**

