# Day 5 练习题：面向对象编程
> 要求：独立完成以下练习，巩固类和异常处理
>

---

## 练习 1：银行账户类
```plain
编写 BankAccount 类：

1. 属性：
   - owner（户主姓名）
   - balance（余额，默认 0）

2. 方法：
   - deposit(amount)：存款，金额必须大于 0
   - withdraw(amount)：取款，余额不足时提示
   - get_balance()：返回当前余额
   - to_dict()：返回账户信息字典
   - __str__()：返回格式化字符串

3. 使用 raise ValueError 处理非法操作

测试代码：
account = BankAccount("张三", 1000)
account.deposit(500)
account.withdraw(200)
print(account)            # 张三的账户，余额：1300.00
print(account.to_dict())  # {'owner': '张三', 'balance': 1300}
account.withdraw(2000)    # ValueError: 余额不足
```

---

## 练习 2：图书管理类
```plain
编写 Book 类和 Library 类：

1. Book 类：
   - 属性：title、author、isbn、is_borrowed（默认 False）
   - 方法：to_dict()、__str__()

2. Library 类：
   - 属性：name、books（列表）
   - 方法：
     - add_book(book)：添加图书
     - find_book(title)：按书名查找
     - borrow_book(title)：借书（修改 is_borrowed）
     - return_book(title)：还书
     - list_available()：列出可借图书
     - list_all()：列出所有图书

3. 使用 try-except 处理异常情况

测试代码：
lib = Library("城市图书馆")
lib.add_book(Book("Python入门", "张三", "ISBN001"))
lib.add_book(Book("Flask实战", "李四", "ISBN002"))
lib.borrow_book("Python入门")
lib.list_available()  # 只显示 Flask实战
```

---

## 练习 3：重构通讯录
```plain
将 Day 4 的通讯录程序用面向对象方式重构：

1. 创建 Contact 类：
   - 属性：name、phone、email（可选）
   - 方法：to_dict()、__str__()

2. 创建 ContactBook 类：
   - 属性：contacts（列表）、filename（文件路径）
   - 方法：
     - load()：从 JSON 文件加载
     - save()：保存到 JSON 文件
     - add(name, phone, email)：添加联系人
     - find(name)：查找联系人
     - delete(name)：删除联系人
     - list_all()：列出所有联系人

3. 加入异常处理（文件不存在、联系人不存在等）
```

---

## 完成检查
- [ ] 银行账户类能正确处理存取款
- [ ] 图书管理类支持借还功能
- [ ] 通讯录使用 OOP 重构完成
- [ ] 正确使用 try-except 和 raise

---
