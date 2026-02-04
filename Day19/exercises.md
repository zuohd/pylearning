# Day 19 练习题：pandas 入门
> 要求：掌握 pandas 基础操作
>

---

## 练习 1：创建 DataFrame
```plain
完成以下任务：

1. 从字典创建 DataFrame，列名使用英文：
   - name: 张三、李四、王五、赵六
   - age: 20、21、22、20
   - score: 85、90、88、92
   - city: 北京、上海、广州、深圳

2. 打印以下信息：
   - 数据形状 (shape)
   - 列名 (columns)
   - 数据类型 (dtypes)
   - 前3行数据 (head)
   - 统计摘要 (describe)

3. 将 DataFrame 保存为 students.csv
```

---

## 练习 2：数据选择
```plain
使用练习 1 创建的 DataFrame，完成：

1. 选择 name 列
2. 选择 name 和 age 两列
3. 选择第 0 行
4. 选择第 0-2 行
5. 选择第 1 行的 name 列
6. 选择 age > 20 的行
7. 选择 score > 85 且 city == '上海' 的行
```

---

## 练习 3：添加和修改
```plain
继续使用上面的 DataFrame，完成：

1. 添加新列 passed，值为 score >= 60

2. 添加新列 grade：
   - score >= 90: 'A'
   - score >= 80: 'B'
   - score >= 60: 'C'
   - 其他: 'D'
   提示：使用 apply + lambda 或 pd.cut

3. 将所有 score 加 5 分

4. 添加新行：
   - name: 钱七
   - age: 23
   - score: 78
   - city: 杭州
   提示：使用 pd.concat 或 loc
```

---

## 完成检查
- [ ] 能创建 DataFrame
- [ ] 能查看数据基本信息
- [ ] 能选择行和列
- [ ] 能进行条件筛选
- [ ] 能添加和修改数据

---
