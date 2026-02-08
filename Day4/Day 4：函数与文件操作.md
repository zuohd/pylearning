# Day 4：函数与文件操作
> 学习目标：掌握函数定义与调用，学会文件读写和 JSON 处理
>

---

## 1. 函数定义
```python
# 基本函数
def greet():
    print("你好，欢迎学习 Python")

greet()

# 带参数的函数
def greet_user(name):
    print(f"你好，{name}")

greet_user("张三")

# 带返回值的函数
def add(a, b):
    return a + b

result = add(3, 5)
print(result)  # 8

# 多个返回值
def get_info(name, age):
    greeting = f"你好，{name}"
    birth_year = 2025 - age
    return greeting, birth_year

msg, year = get_info("张三", 20)
print(msg)   # 你好，张三
print(year)  # 2005
```

---

## 2. 函数参数
```python
# 位置参数
def power(base, exp):
    return base ** exp

print(power(2, 10))  # 1024

# 默认值参数
def greet(name, greeting="你好"):
    print(f"{greeting}，{name}")

greet("张三")              # 你好，张三
greet("张三", "早上好")    # 早上好，张三

# 关键字参数
def create_user(name, age, city="未知"):
    return {"name": name, "age": age, "city": city}

user = create_user(name="张三", age=20, city="北京")
print(user)

# 可变参数
def calc_sum(*numbers):
    return sum(numbers)

print(calc_sum(1, 2, 3))        # 6
print(calc_sum(1, 2, 3, 4, 5))  # 15
```

---

## 3. 函数实践
```python
# 判断素数
def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n ** 0.5) + 1):
        if n % i == 0:
            return False
    return True

for i in range(1, 20):
    if is_prime(i):
        print(i, end=" ")  # 2 3 5 7 11 13 17 19
print()

# 列表处理
def filter_list(items, min_value):
    return [x for x in items if x >= min_value]

scores = [45, 78, 92, 33, 67, 88, 55]
passed = filter_list(scores, 60)
print(passed)  # [78, 92, 67, 88]

# 字典处理
def format_student(student):
    return f"{student['name']}（{student['age']}岁）- {student['score']}分"

s = {"name": "张三", "age": 20, "score": 85}
print(format_student(s))
```

---

## 4. 文件写入
```python
# 写入文件
with open("output.txt", "w", encoding="utf-8") as f:
    f.write("第一行内容\n")
    f.write("第二行内容\n")
    f.write("第三行内容\n")

# 追加写入
with open("output.txt", "a", encoding="utf-8") as f:
    f.write("追加的内容\n")

# 写入多行
lines = ["Python\n", "Java\n", "Go\n"]
with open("languages.txt", "w", encoding="utf-8") as f:
    f.writelines(lines)
```

**with 语句**：自动管理文件的打开和关闭，推荐始终使用 with。

文件模式：
+ `"w"`：写入（覆盖已有内容）
+ `"a"`：追加（在末尾添加）
+ `"r"`：读取（默认模式）

---

## 5. 文件读取
```python
# 读取全部内容
with open("output.txt", "r", encoding="utf-8") as f:
    content = f.read()
    print(content)

# 逐行读取
with open("output.txt", "r", encoding="utf-8") as f:
    for line in f:
        print(line.strip())  # strip() 去除换行符

# 读取所有行到列表
with open("output.txt", "r", encoding="utf-8") as f:
    lines = f.readlines()
    print(lines)

# 读取指定行
with open("output.txt", "r", encoding="utf-8") as f:
    lines = f.readlines()
    if len(lines) >= 2:
        print(lines[1].strip())  # 第二行
```

**encoding 参数**：处理中文内容时，使用 `encoding="utf-8"` 避免乱码。

---

## 6. JSON 文件
```python
import json

# Python 对象转 JSON 字符串
data = {"name": "张三", "age": 20, "scores": [85, 92, 78]}
json_str = json.dumps(data, ensure_ascii=False, indent=2)
print(json_str)

# JSON 字符串转 Python 对象
json_str = '{"name": "张三", "age": 20}'
data = json.loads(json_str)
print(data["name"])  # 张三

# 写入 JSON 文件
users = [
    {"name": "张三", "age": 20},
    {"name": "李四", "age": 22}
]
with open("users.json", "w", encoding="utf-8") as f:
    json.dump(users, f, ensure_ascii=False, indent=2)

# 读取 JSON 文件
with open("users.json", "r", encoding="utf-8") as f:
    users = json.load(f)
    for user in users:
        print(f"{user['name']}，{user['age']}岁")
```

**JSON 与 Python 类型对照**：
| JSON | Python |
| --- | --- |
| object | dict |
| array | list |
| string | str |
| number | int / float |
| true/false | True / False |
| null | None |

---

## 7. 综合练习
```python
import json

# 联系人通讯录
FILENAME = "contacts.json"

def load_contacts():
    try:
        with open(FILENAME, "r", encoding="utf-8") as f:
            return json.load(f)
    except FileNotFoundError:
        return []

def save_contacts(contacts):
    with open(FILENAME, "w", encoding="utf-8") as f:
        json.dump(contacts, f, ensure_ascii=False, indent=2)

def add_contact(contacts, name, phone):
    contacts.append({"name": name, "phone": phone})
    save_contacts(contacts)
    print(f"已添加：{name}")

def find_contact(contacts, name):
    for c in contacts:
        if c["name"] == name:
            return c
    return None

def list_contacts(contacts):
    if not contacts:
        print("通讯录为空")
        return
    for i, c in enumerate(contacts, 1):
        print(f"{i}. {c['name']} - {c['phone']}")

# 使用
contacts = load_contacts()
add_contact(contacts, "张三", "13800138000")
add_contact(contacts, "李四", "13900139000")
list_contacts(contacts)

result = find_contact(contacts, "张三")
if result:
    print(f"找到：{result['name']} - {result['phone']}")
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 函数定义 | def、return、调用 |
| 参数 | 位置、默认值、关键字、*args |
| 文件写入 | open、with、write、append |
| 文件读取 | read、readline、readlines |
| JSON | dumps/loads、dump/load |
| 编码 | encoding="utf-8" |

---

## 课后练习
完成练习（见 [exercises.md](exercises.md)）

---
