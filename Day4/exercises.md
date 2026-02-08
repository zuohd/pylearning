# Day 4 练习题：函数与文件操作
> 要求：独立完成以下练习，巩固函数和文件操作
>

---

## 练习 1：数学工具函数
```plain
编写以下函数：

1. is_even(n)：判断是否为偶数，返回 True/False
2. factorial(n)：计算阶乘（如 5! = 120）
3. fibonacci(n)：返回前 n 个斐波那契数列表
4. find_max(numbers)：找到列表中的最大值（不使用 max()）

测试代码：
print(is_even(4))        # True
print(is_even(7))        # False
print(factorial(5))      # 120
print(fibonacci(8))      # [1, 1, 2, 3, 5, 8, 13, 21]
print(find_max([3, 7, 2, 9, 1]))  # 9
```

---

## 练习 2：日记程序
```plain
编写一个简单的日记程序：

1. 支持以下功能：
   - write：写一篇日记（追加到 diary.txt）
   - read：读取所有日记内容
   - quit：退出程序

2. 每篇日记自动添加日期时间
3. 日记之间用分隔线隔开

示例交互：
> write
请输入日记内容：今天学习了 Python 函数
日记已保存

> read
--- 2025-01-15 14:30:00 ---
今天学习了 Python 函数
---

> quit

提示：
- 使用 from datetime import datetime
- datetime.now().strftime("%Y-%m-%d %H:%M:%S")
```

---

## 练习 3：JSON 配置管理
```plain
编写一个配置管理程序：

1. 使用 JSON 文件存储配置
2. 支持以下操作：
   - set <key> <value>：设置配置项
   - get <key>：获取配置项
   - list：列出所有配置
   - delete <key>：删除配置项
   - quit：退出

3. 配置自动保存到 config.json

示例交互：
> set username admin
已设置：username = admin
> set port 8080
已设置：port = 8080
> get username
username = admin
> list
username = admin
port = 8080
> delete port
已删除：port
> quit
```

---

## 完成检查
- [ ] 四个数学函数都能正确运行
- [ ] 日记程序能写入和读取
- [ ] 配置管理程序能正确读写 JSON
- [ ] 正确使用 with 语句操作文件

---
