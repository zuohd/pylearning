# Day 3：列表与字典
> 学习目标：掌握列表和字典的基本操作，能用数据结构组织数据
>

---

## 1. 列表基础
```python
# 创建列表
fruits = ["苹果", "香蕉", "橘子"]
numbers = [1, 2, 3, 4, 5]
mixed = ["张三", 20, True, 3.14]
empty = []

# 访问元素（索引从 0 开始）
print(fruits[0])   # 苹果
print(fruits[1])   # 香蕉
print(fruits[-1])  # 橘子（最后一个）

# 列表长度
print(len(fruits))  # 3

# 检查元素是否存在
print("苹果" in fruits)  # True
print("西瓜" in fruits)  # False
```

---

## 2. 列表操作
### 增删改
```python
fruits = ["苹果", "香蕉", "橘子"]

# 添加元素
fruits.append("西瓜")         # 末尾添加
fruits.insert(1, "葡萄")      # 指定位置插入
print(fruits)  # ['苹果', '葡萄', '香蕉', '橘子', '西瓜']

# 删除元素
fruits.remove("香蕉")         # 按值删除
last = fruits.pop()           # 删除并返回最后一个
second = fruits.pop(1)        # 删除并返回指定位置
print(fruits)

# 修改元素
fruits[0] = "红苹果"
print(fruits)
```

### 切片
```python
numbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

print(numbers[2:5])    # [2, 3, 4]
print(numbers[:3])     # [0, 1, 2]
print(numbers[7:])     # [7, 8, 9]
print(numbers[::2])    # [0, 2, 4, 6, 8]
print(numbers[::-1])   # [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
```

### 排序
```python
numbers = [3, 1, 4, 1, 5, 9, 2, 6]

# 排序（修改原列表）
numbers.sort()
print(numbers)  # [1, 1, 2, 3, 4, 5, 6, 9]

# 降序排序
numbers.sort(reverse=True)
print(numbers)  # [9, 6, 5, 4, 3, 2, 1, 1]

# 不修改原列表
original = [3, 1, 4, 1, 5]
sorted_list = sorted(original)
print(original)     # [3, 1, 4, 1, 5]
print(sorted_list)  # [1, 1, 3, 4, 5]
```

---

## 3. 列表遍历
```python
fruits = ["苹果", "香蕉", "橘子"]

# 基本遍历
for fruit in fruits:
    print(fruit)

# 带索引遍历
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")

# 列表推导式
numbers = [1, 2, 3, 4, 5]
squares = [n ** 2 for n in numbers]
print(squares)  # [1, 4, 9, 16, 25]

# 带条件的列表推导式
evens = [n for n in numbers if n % 2 == 0]
print(evens)  # [2, 4]

# 常用操作
print(sum(numbers))  # 15
print(max(numbers))  # 5
print(min(numbers))  # 1
```

---

## 4. 字典基础
```python
# 创建字典
student = {
    "name": "张三",
    "age": 20,
    "grade": "大二"
}

# 访问值
print(student["name"])     # 张三
print(student.get("age"))  # 20
print(student.get("phone", "未填写"))  # 未填写（默认值）

# 字典长度
print(len(student))  # 3

# 检查键是否存在
print("name" in student)   # True
print("phone" in student)  # False
```

### 增删改
```python
student = {"name": "张三", "age": 20}

# 添加 / 修改
student["grade"] = "大二"     # 添加新键
student["age"] = 21           # 修改已有键
print(student)

# 删除
del student["grade"]          # 删除指定键
age = student.pop("age")     # 删除并返回值
print(student)
print(age)  # 21
```

---

## 5. 字典操作
```python
student = {"name": "张三", "age": 20, "grade": "大二"}

# 获取所有键
print(student.keys())    # dict_keys(['name', 'age', 'grade'])

# 获取所有值
print(student.values())  # dict_values(['张三', 20, '大二'])

# 获取所有键值对
print(student.items())   # dict_items([('name', '张三'), ...])

# 遍历字典
for key in student:
    print(f"{key}: {student[key]}")

# 遍历键值对
for key, value in student.items():
    print(f"{key}: {value}")
```

### 嵌套
```python
# 列表中嵌套字典
students = [
    {"name": "张三", "score": 85},
    {"name": "李四", "score": 92},
    {"name": "王五", "score": 78}
]

for s in students:
    print(f"{s['name']}: {s['score']}分")

# 字典中嵌套列表
classroom = {
    "class_name": "一班",
    "students": ["张三", "李四", "王五"],
    "scores": [85, 92, 78]
}
print(classroom["students"][0])  # 张三
```

---

## 6. 元组与集合
### 元组
```python
# 元组：不可变的列表
point = (10, 20)
print(point[0])  # 10

# 元组不能修改
# point[0] = 30  # TypeError

# 元组拆包
x, y = point
print(x, y)  # 10 20

# 函数返回多个值
def get_range(numbers):
    return min(numbers), max(numbers)

low, high = get_range([3, 1, 4, 1, 5])
print(low, high)  # 1 5
```

### 集合
```python
# 集合：无序、不重复
numbers = {1, 2, 3, 3, 2, 1}
print(numbers)  # {1, 2, 3}

# 列表去重
names = ["张三", "李四", "张三", "王五", "李四"]
unique = list(set(names))
print(unique)

# 集合操作
a = {1, 2, 3}
b = {2, 3, 4}
print(a & b)  # {2, 3} 交集
print(a | b)  # {1, 2, 3, 4} 并集
print(a - b)  # {1} 差集
```

---

## 7. 综合实例
```python
# 学生成绩管理器
students = []

# 添加学生
def add_student(name, score):
    students.append({"name": name, "score": score})

# 查找学生
def find_student(name):
    for s in students:
        if s["name"] == name:
            return s
    return None

# 计算平均分
def average_score():
    if not students:
        return 0
    total = sum(s["score"] for s in students)
    return total / len(students)

# 测试
add_student("张三", 85)
add_student("李四", 92)
add_student("王五", 78)

print(find_student("李四"))
print(f"平均分：{average_score():.1f}")

# 按成绩排序
sorted_students = sorted(students, key=lambda s: s["score"], reverse=True)
for s in sorted_students:
    print(f"{s['name']}: {s['score']}分")
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 列表 | 创建、索引、append/remove/pop |
| 切片 | [start:end:step] |
| 列表推导式 | [expr for x in list if cond] |
| 字典 | 键值对、get()、增删改查 |
| 遍历 | for/enumerate/items |
| 元组 | 不可变、拆包 |
| 集合 | 去重、交并差 |

---

## 课后练习
完成练习（见 [exercises.md](exercises.md)）

---
