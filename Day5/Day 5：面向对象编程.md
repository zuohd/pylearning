# Day 5：面向对象编程
> 学习目标：理解面向对象编程思想，掌握类的定义和使用，学会异常处理
>

---

## 1. 什么是面向对象
**面向过程** vs **面向对象**：
+ 面向过程：按步骤编写函数，数据和操作分离
+ 面向对象：将数据和操作封装到一起，形成"对象"

```python
# 面向过程
user = {"name": "张三", "balance": 100}

def deposit(user, amount):
    user["balance"] += amount

deposit(user, 50)

# 面向对象
class User:
    def __init__(self, name, balance):
        self.name = name
        self.balance = balance

    def deposit(self, amount):
        self.balance += amount

user = User("张三", 100)
user.deposit(50)
```

**为什么用 OOP**：
+ 代码组织更清晰
+ 数据和行为绑定在一起
+ 易于维护和扩展
+ 后续学习 Flask、SQLAlchemy 都基于 OOP

---

## 2. 定义类
```python
class Student:
    def __init__(self, name, age):
        self.name = name
        self.age = age
        self.scores = []

# 创建对象（实例化）
s1 = Student("张三", 20)
s2 = Student("李四", 22)

# 访问属性
print(s1.name)  # 张三
print(s2.age)   # 22
```

**关键概念**：
+ `class`：定义类
+ `__init__`：构造方法，创建对象时自动调用
+ `self`：指向当前对象本身
+ `self.xxx`：对象的属性

---

## 3. 方法
```python
class Student:
    def __init__(self, name, age):
        self.name = name
        self.age = age
        self.scores = []

    def add_score(self, score):
        self.scores.append(score)

    def average(self):
        if not self.scores:
            return 0
        return sum(self.scores) / len(self.scores)

    def info(self):
        avg = self.average()
        return f"{self.name}，{self.age}岁，平均分：{avg:.1f}"

# 使用
s = Student("张三", 20)
s.add_score(85)
s.add_score(92)
s.add_score(78)
print(s.info())      # 张三，20岁，平均分：85.0
print(s.average())   # 85.0
```

---

## 4. 实际应用
```python
class UserManager:
    def __init__(self):
        self.users = []
        self.next_id = 1

    def add_user(self, name, email):
        user = {
            "id": self.next_id,
            "name": name,
            "email": email
        }
        self.users.append(user)
        self.next_id += 1
        return user

    def get_user(self, user_id):
        for user in self.users:
            if user["id"] == user_id:
                return user
        return None

    def delete_user(self, user_id):
        user = self.get_user(user_id)
        if user:
            self.users.remove(user)
            return True
        return False

    def list_users(self):
        return self.users

# 使用
manager = UserManager()
manager.add_user("张三", "zhangsan@example.com")
manager.add_user("李四", "lisi@example.com")

print(manager.list_users())
print(manager.get_user(1))
manager.delete_user(2)
print(manager.list_users())
```

---

## 5. 类的进阶
### __str__ 方法
```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

    def __str__(self):
        return f"User({self.name}, {self.email})"

user = User("张三", "zhangsan@example.com")
print(user)  # User(张三, zhangsan@example.com)
```

### to_dict 方法
```python
class User:
    def __init__(self, name, email, age):
        self.name = name
        self.email = email
        self.age = age

    def to_dict(self):
        return {
            "name": self.name,
            "email": self.email,
            "age": self.age
        }

user = User("张三", "zhangsan@example.com", 20)
print(user.to_dict())
# {'name': '张三', 'email': 'zhangsan@example.com', 'age': 20}
```

to_dict() 在后续 Flask API 开发中非常常用，用于将对象转换为可序列化的字典。

### 私有约定
```python
class BankAccount:
    def __init__(self, owner, balance):
        self.owner = owner
        self._balance = balance  # 约定：下划线前缀表示"内部使用"

    def deposit(self, amount):
        if amount > 0:
            self._balance += amount

    def get_balance(self):
        return self._balance

account = BankAccount("张三", 1000)
account.deposit(500)
print(account.get_balance())  # 1500
```

---

## 6. 错误处理
### try-except
```python
# 基本用法
try:
    num = int(input("请输入数字："))
    print(f"你输入了：{num}")
except ValueError:
    print("输入无效，请输入数字")

# 捕获多种异常
try:
    numbers = [1, 2, 3]
    index = int(input("请输入索引："))
    print(numbers[index])
except ValueError:
    print("请输入有效数字")
except IndexError:
    print("索引超出范围")

# 通用异常
try:
    result = 10 / 0
except Exception as e:
    print(f"发生错误：{e}")

# try-except-finally
try:
    f = open("test.txt", "r")
    content = f.read()
except FileNotFoundError:
    print("文件不存在")
finally:
    print("操作完成")
```

### 常见异常
| 异常 | 说明 |
| --- | --- |
| ValueError | 值错误（如 int("abc")） |
| TypeError | 类型错误（如 "1" + 1） |
| IndexError | 索引越界 |
| KeyError | 字典键不存在 |
| FileNotFoundError | 文件不存在 |
| ZeroDivisionError | 除以零 |

### raise
```python
def set_age(age):
    if age < 0 or age > 150:
        raise ValueError("年龄必须在 0 到 150 之间")
    return age

try:
    set_age(200)
except ValueError as e:
    print(e)  # 年龄必须在 0 到 150 之间
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 类与对象 | class、实例化 |
| __init__ | 构造方法、self |
| 方法 | 实例方法、self 参数 |
| to_dict | 对象转字典 |
| __str__ | 自定义打印格式 |
| try-except | 异常捕获与处理 |
| raise | 主动抛出异常 |

---

## 课后练习
完成练习（见 [exercises.md](exercises.md)）

---
