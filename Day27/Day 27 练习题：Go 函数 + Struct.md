# Day 27 练习题：Go 函数 + Struct
> 要求：掌握函数和结构体
>

---

## 练习 29：函数
```plain
1. 基本函数
   - add(a, b int) int: 返回两数之和
   - max(a, b int) int: 返回较大值
   - swap(a, b int) (int, int): 交换两个数

2. 多返回值
   - divide(a, b int) (int, int, error)
     返回商、余数、错误（除数为0时）

3. 可变参数
   - sum(nums ...int) int: 计算任意个数的和
   - 测试: sum(1, 2, 3), sum(1, 2, 3, 4, 5)
```

---

## 练习 30：结构体
```plain
1. 定义 Student 结构体
   - Name string
   - Age int
   - Scores []int

2. 添加方法
   - (s Student) Average() float64: 计算平均分
   - (s *Student) AddScore(score int): 添加分数

3. 创建并使用
   - 创建学生实例
   - 添加几个分数
   - 计算并打印平均分
```

---

## 练习 31：综合练习
```plain
创建一个简单的学生管理系统：

1. 定义 Student 结构体
   - ID, Name, Age, Score

2. 定义 StudentManager 结构体
   - students []Student

3. 实现方法
   - Add(s Student): 添加学生
   - GetByID(id int) *Student: 按 ID 查找
   - List() []Student: 获取所有学生
   - Average() float64: 计算平均分

4. 测试所有功能
```

---

## 完成检查
- [ ] 能定义和调用函数
- [ ] 理解多返回值
- [ ] 能使用可变参数
- [ ] 能定义结构体和方法
- [ ] 理解值接收者和指针接收者

---
