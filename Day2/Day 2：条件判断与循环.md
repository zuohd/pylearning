# Day 2：条件判断与循环
> 学习目标：掌握条件判断和循环语句，能编写有逻辑的程序
>

---

## 1. 布尔值与比较
```python
# 布尔值
print(True)
print(False)

# 比较运算
print(10 > 5)    # True
print(10 == 5)   # False
print(10 != 5)   # True

# 逻辑运算
print(True and False)   # False
print(True or False)    # True
print(not True)         # False

# 实际应用
age = 20
is_adult = age >= 18
print(is_adult)  # True

# 组合条件
score = 85
passed = score >= 60 and score <= 100
print(passed)  # True
```

---

## 2. if 语句
```python
# 基本 if
age = 20
if age >= 18:
    print("成年人")

# if-else
age = 15
if age >= 18:
    print("成年人")
else:
    print("未成年人")

# if-elif-else
score = 85
if score >= 90:
    print("优秀")
elif score >= 80:
    print("良好")
elif score >= 60:
    print("及格")
else:
    print("不及格")
```

**注意缩进**：Python 使用缩进（4 个空格）表示代码块。
```python
# 缩进错误示例（会报错）
# if True:
# print("Hello")  # IndentationError

# 正确写法
if True:
    print("Hello")
```

### 嵌套 if
```python
age = 20
has_id = True

if age >= 18:
    if has_id:
        print("允许进入")
    else:
        print("请携带身份证")
else:
    print("未成年不允许进入")
```

---

## 3. while 循环
```python
# 基本 while
count = 1
while count <= 5:
    print(f"第 {count} 次")
    count += 1

# 计算 1 到 100 的和
total = 0
num = 1
while num <= 100:
    total += num
    num += 1
print(f"1 到 100 的和：{total}")  # 5050

# while + 用户输入
password = ""
while password != "123456":
    password = input("请输入密码：")
print("密码正确，登录成功")
```

**避免无限循环**：
```python
# 设置退出条件
count = 0
while count < 10:
    print(count)
    count += 1  # 忘记这行会导致无限循环
```

---

## 4. for 循环
```python
# 遍历 range
for i in range(5):
    print(i)  # 0, 1, 2, 3, 4

# range 带起始值
for i in range(1, 6):
    print(i)  # 1, 2, 3, 4, 5

# range 带步长
for i in range(0, 10, 2):
    print(i)  # 0, 2, 4, 6, 8

# 遍历字符串
for char in "Hello":
    print(char)

# 遍历列表
fruits = ["苹果", "香蕉", "橘子"]
for fruit in fruits:
    print(fruit)
```

---

## 5. 循环控制
### break
```python
# 找到第一个能被 7 整除的数
for i in range(1, 100):
    if i % 7 == 0:
        print(f"第一个能被 7 整除的数：{i}")
        break

# while + break
while True:
    cmd = input("输入命令（quit 退出）：")
    if cmd == "quit":
        print("退出程序")
        break
    print(f"执行命令：{cmd}")
```

### continue
```python
# 跳过偶数，只打印奇数
for i in range(1, 11):
    if i % 2 == 0:
        continue
    print(i)  # 1, 3, 5, 7, 9
```

---

## 6. 嵌套循环
```python
# 九九乘法表
for i in range(1, 10):
    for j in range(1, i + 1):
        print(f"{j}x{i}={i*j}", end="\t")
    print()

# 打印矩形
for i in range(3):
    for j in range(5):
        print("*", end=" ")
    print()
# * * * * *
# * * * * *
# * * * * *
```

---

## 7. 综合练习
```python
# 求 1 到 100 中所有偶数的和
even_sum = 0
for i in range(1, 101):
    if i % 2 == 0:
        even_sum += i
print(f"偶数和：{even_sum}")  # 2550

# 找出 1 到 50 中所有能被 3 整除的数
print("能被 3 整除的数：")
for i in range(1, 51):
    if i % 3 == 0:
        print(i, end=" ")
print()

# 倒计时
import time
for i in range(5, 0, -1):
    print(i)
    time.sleep(1)
print("时间到")
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 布尔值 | True/False、and/or/not |
| if 语句 | if/elif/else、缩进、嵌套 |
| while | 条件循环、计数器 |
| for | range()、遍历序列 |
| break | 提前退出循环 |
| continue | 跳过本次迭代 |
| 嵌套循环 | 循环中的循环 |

---

## 课后练习
完成练习（见 [exercises.md](exercises.md)）

---
