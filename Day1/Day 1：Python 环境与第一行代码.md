# Day 1：Python 环境与第一行代码
> 学习目标：搭建 Python 开发环境，掌握变量、数据类型和运算符
>

---

## 1. Python 简介
Python 是一门通用编程语言，广泛应用于：
+ Web 后端开发（Flask、Django）
+ 数据分析与可视化（pandas、matplotlib）
+ 自动化脚本
+ 人工智能与机器学习

**为什么学 Python**：
+ 语法简洁，适合初学者
+ 生态丰富，第三方库众多
+ 就业需求大，应用场景广

---

## 2. 环境搭建
### 安装 Python
从官网下载 Python 3.x：https://www.python.org/downloads/

安装时勾选 **Add Python to PATH**。

验证安装：
```plain
python --version
```

### 安装 VS Code
下载地址：https://code.visualstudio.com/

安装 Python 扩展：
+ 打开 VS Code
+ 点击左侧扩展图标
+ 搜索 "Python"，安装微软官方扩展

### 运行第一个程序
创建文件 `hello.py`：
```python
print("Hello, Python!")
```

在终端运行：
```plain
python hello.py
```

---

## 3. print 函数
```python
# 输出文本
print("Hello, World!")

# 输出多个值
print("姓名:", "张三")

# 输出数字
print(100)
print(3.14)

# 使用 end 参数（不换行）
print("Hello", end=" ")
print("World")

# 使用 sep 参数（自定义分隔符）
print("2025", "01", "01", sep="-")
```

---

## 4. 变量与赋值
变量是存储数据的容器。
```python
# 变量赋值
name = "张三"
age = 20
height = 1.75

# 使用变量
print(name)
print(age)
print(height)

# 变量可以重新赋值
age = 21
print(age)
```

**命名规则**：
+ 只能包含字母、数字、下划线
+ 不能以数字开头
+ 不能使用 Python 关键字（如 if、for、class）
+ 建议使用小写字母和下划线（snake_case）

```python
# 合法命名
user_name = "张三"
age2 = 20
_count = 0

# 不合法命名（会报错）
# 2name = "张三"    # 数字开头
# my-name = "张三"  # 包含连字符
# class = "一班"    # 关键字
```

查看变量类型：
```python
name = "张三"
age = 20
print(type(name))  # <class 'str'>
print(type(age))   # <class 'int'>
```

---

## 5. 基本数据类型
```python
# 整数 int
age = 20
count = -5
print(type(age))  # <class 'int'>

# 浮点数 float
price = 9.99
pi = 3.14159
print(type(price))  # <class 'float'>

# 字符串 str
name = "张三"
greeting = 'Hello'
message = "它说：'你好'"
print(type(name))  # <class 'str'>

# 布尔值 bool
is_student = True
is_teacher = False
print(type(is_student))  # <class 'bool'>
```

### 类型转换
```python
# 字符串转整数
num_str = "123"
num = int(num_str)
print(num + 1)  # 124

# 整数转字符串
age = 20
age_str = str(age)
print("年龄：" + age_str)

# 字符串转浮点数
price_str = "9.99"
price = float(price_str)
print(price)  # 9.99

# 整数转浮点数
x = float(10)
print(x)  # 10.0
```

### input 函数
```python
# 获取用户输入（返回值始终是字符串）
name = input("请输入你的名字：")
print("你好，" + name)

# 输入数字需要类型转换
age = int(input("请输入你的年龄："))
print("明年你", age + 1, "岁")
```

---

## 6. 运算符
### 算术运算符
```python
a = 10
b = 3

print(a + b)   # 13   加法
print(a - b)   # 7    减法
print(a * b)   # 30   乘法
print(a / b)   # 3.33 除法（结果为浮点数）
print(a // b)  # 3    整除
print(a % b)   # 1    取余
print(a ** b)  # 1000 幂运算
```

### 比较运算符
```python
x = 10
y = 20

print(x == y)   # False  等于
print(x != y)   # True   不等于
print(x > y)    # False  大于
print(x < y)    # True   小于
print(x >= 10)  # True   大于等于
print(x <= 20)  # True   小于等于
```

### 赋值运算符
```python
x = 10
x += 5   # x = x + 5，结果 15
x -= 3   # x = x - 3，结果 12
x *= 2   # x = x * 2，结果 24
x /= 4   # x = x / 4，结果 6.0

print(x)
```

### 字符串运算
```python
# 字符串拼接
first = "Hello"
second = "World"
result = first + " " + second
print(result)  # Hello World

# 字符串重复
line = "-" * 20
print(line)  # --------------------

# f-string 格式化（推荐）
name = "张三"
age = 20
print(f"我叫{name}，今年{age}岁")
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 环境搭建 | Python 3.x + VS Code |
| print | 输出、end、sep 参数 |
| 变量 | 赋值、命名规则、type() |
| 数据类型 | int、float、str、bool |
| 类型转换 | int()、float()、str() |
| input | 用户输入，返回字符串 |
| 运算符 | 算术、比较、赋值、字符串运算 |

---

## 课后练习
完成练习（见 [exercises.md](exercises.md)）

---
