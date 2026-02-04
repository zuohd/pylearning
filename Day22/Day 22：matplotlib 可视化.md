# Day 22：matplotlib 可视化
> 学习目标：掌握 matplotlib 绑制常用图表
>

---

## 1. matplotlib 基础
```python
# 安装
# pip install matplotlib

import matplotlib.pyplot as plt
import numpy as np

# 解决中文显示问题
plt.rcParams['font.sans-serif'] = ['SimHei', 'Arial Unicode MS']
plt.rcParams['axes.unicode_minus'] = False

# 简单绑图
x = [1, 2, 3, 4, 5]
y = [2, 4, 6, 8, 10]

plt.plot(x, y)
plt.title('简单折线图')
plt.xlabel('X轴')
plt.ylabel('Y轴')
plt.show()
```

---

## 2. 折线图
```python
import matplotlib.pyplot as plt

# 数据
months = ['1月', '2月', '3月', '4月', '5月', '6月']
sales = [100, 120, 140, 135, 160, 180]

# 创建图形
plt.figure(figsize=(10, 6))
plt.plot(months, sales, marker='o', linewidth=2, color='blue')

# 添加标题和标签
plt.title('月度销售趋势', fontsize=16)
plt.xlabel('月份', fontsize=12)
plt.ylabel('销售额（万元）', fontsize=12)

# 添加网格
plt.grid(True, linestyle='--', alpha=0.7)

# 保存图片
plt.savefig('line_chart.png', dpi=100, bbox_inches='tight')
plt.show()
```

---

## 3. 柱状图
```python
import matplotlib.pyplot as plt
import numpy as np

# 数据
categories = ['A', 'B', 'C', 'D', 'E']
values = [23, 45, 56, 78, 32]

# 创建柱状图
plt.figure(figsize=(10, 6))
bars = plt.bar(categories, values, color='steelblue', edgecolor='black')

# 在柱子上添加数值
for bar, value in zip(bars, values):
    plt.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 1,
             str(value), ha='center', va='bottom')

plt.title('各类别数据对比')
plt.xlabel('类别')
plt.ylabel('数值')
plt.show()

# 横向柱状图
plt.figure(figsize=(10, 6))
plt.barh(categories, values, color='coral')
plt.title('横向柱状图')
plt.xlabel('数值')
plt.ylabel('类别')
plt.show()
```

---

## 4. 分组柱状图
```python
import matplotlib.pyplot as plt
import numpy as np

# 数据
categories = ['Q1', 'Q2', 'Q3', 'Q4']
product_a = [20, 35, 30, 35]
product_b = [25, 32, 34, 20]

x = np.arange(len(categories))
width = 0.35

# 创建分组柱状图
fig, ax = plt.subplots(figsize=(10, 6))
bars1 = ax.bar(x - width/2, product_a, width, label='产品A', color='steelblue')
bars2 = ax.bar(x + width/2, product_b, width, label='产品B', color='coral')

ax.set_xlabel('季度')
ax.set_ylabel('销量')
ax.set_title('产品季度销量对比')
ax.set_xticks(x)
ax.set_xticklabels(categories)
ax.legend()

plt.tight_layout()
plt.show()
```

---

## 5. 饼图
```python
import matplotlib.pyplot as plt

# 数据
labels = ['北京', '上海', '广州', '深圳', '其他']
sizes = [30, 25, 20, 15, 10]
colors = ['#ff9999', '#66b3ff', '#99ff99', '#ffcc99', '#c2c2f0']
explode = (0.05, 0, 0, 0, 0)  # 突出第一块

# 创建饼图
plt.figure(figsize=(10, 8))
plt.pie(sizes, explode=explode, labels=labels, colors=colors,
        autopct='%1.1f%%', shadow=True, startangle=90)

plt.title('各城市销售占比', fontsize=16)
plt.axis('equal')  # 保证是圆形
plt.show()
```

---

## 6. 散点图
```python
import matplotlib.pyplot as plt
import numpy as np

# 生成数据
np.random.seed(42)
x = np.random.randn(100)
y = 2 * x + np.random.randn(100) * 0.5

# 创建散点图
plt.figure(figsize=(10, 6))
plt.scatter(x, y, c='blue', alpha=0.6, edgecolors='black', s=50)

plt.title('散点图示例')
plt.xlabel('X')
plt.ylabel('Y')
plt.grid(True, alpha=0.3)
plt.show()
```

---

## 7. 直方图
```python
import matplotlib.pyplot as plt
import numpy as np

# 生成数据
np.random.seed(42)
data = np.random.randn(1000)

# 创建直方图
plt.figure(figsize=(10, 6))
plt.hist(data, bins=30, color='steelblue', edgecolor='black', alpha=0.7)

plt.title('数据分布直方图')
plt.xlabel('值')
plt.ylabel('频数')
plt.show()
```

---

## 8. 子图
```python
import matplotlib.pyplot as plt
import numpy as np

# 创建 2x2 子图
fig, axes = plt.subplots(2, 2, figsize=(12, 10))

# 子图1 - 折线图
x = np.linspace(0, 10, 100)
axes[0, 0].plot(x, np.sin(x))
axes[0, 0].set_title('正弦曲线')

# 子图2 - 柱状图
categories = ['A', 'B', 'C', 'D']
values = [3, 7, 2, 5]
axes[0, 1].bar(categories, values, color='coral')
axes[0, 1].set_title('柱状图')

# 子图3 - 散点图
axes[1, 0].scatter(np.random.randn(50), np.random.randn(50))
axes[1, 0].set_title('散点图')

# 子图4 - 饼图
sizes = [30, 20, 25, 25]
axes[1, 1].pie(sizes, labels=['A', 'B', 'C', 'D'], autopct='%1.1f%%')
axes[1, 1].set_title('饼图')

plt.tight_layout()
plt.savefig('subplots.png', dpi=100)
plt.show()
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 基础设置 | figure、rcParams、中文支持 |
| 折线图 | plot、marker、linewidth |
| 柱状图 | bar、barh、分组柱状图 |
| 饼图 | pie、explode、autopct |
| 散点图 | scatter、alpha、s |
| 子图 | subplots、axes |

---

## 课后练习
完成练习题 19-21（见 [phase2_exercises.md](phase2_exercises.md#day-22-练习题matplotlib-可视化)）

---
