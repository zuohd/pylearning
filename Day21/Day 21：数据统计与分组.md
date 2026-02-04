# Day 21：数据统计与分组
> 学习目标：掌握 groupby 分组统计和排序操作
>

---

## 1. 基本统计函数
```python
import pandas as pd

df = pd.DataFrame({
    'name': ['张三', '李四', '王五', '赵六', '钱七'],
    'age': [20, 21, 22, 20, 23],
    'score': [85, 90, 88, 92, 78]
})

# 单列统计
print(df['score'].sum())     # 求和
print(df['score'].mean())    # 平均值
print(df['score'].median())  # 中位数
print(df['score'].max())     # 最大值
print(df['score'].min())     # 最小值
print(df['score'].std())     # 标准差
print(df['score'].var())     # 方差
print(df['score'].count())   # 计数

# 整体统计
print(df.describe())

# 多列统计
print(df[['age', 'score']].mean())
```

---

## 2. 排序
```python
import pandas as pd

df = pd.DataFrame({
    'name': ['张三', '李四', '王五', '赵六'],
    'age': [20, 21, 22, 20],
    'score': [85, 90, 88, 92]
})

# 按单列排序
df_sorted = df.sort_values('score')              # 升序
df_sorted = df.sort_values('score', ascending=False)  # 降序

# 按多列排序
df_sorted = df.sort_values(['age', 'score'], ascending=[True, False])

# 按索引排序
df_sorted = df.sort_index()

# 获取最大/最小的 n 行
print(df.nlargest(2, 'score'))   # 分数最高的2人
print(df.nsmallest(2, 'score'))  # 分数最低的2人
```

---

## 3. groupby 分组
```python
import pandas as pd

df = pd.DataFrame({
    'city': ['北京', '上海', '北京', '上海', '广州'],
    'category': ['A', 'A', 'B', 'B', 'A'],
    'sales': [100, 200, 150, 180, 120]
})

# 单列分组
grouped = df.groupby('city')
print(grouped['sales'].sum())
# city
# 上海    380
# 北京    250
# 广州    120

# 多列分组
grouped = df.groupby(['city', 'category'])
print(grouped['sales'].sum())

# 多种聚合
print(df.groupby('city')['sales'].agg(['sum', 'mean', 'count']))

# 不同列不同聚合
print(df.groupby('city').agg({
    'sales': ['sum', 'mean'],
    'category': 'count'
}))
```

---

## 4. 聚合函数 agg
```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'category': ['A', 'A', 'B', 'B', 'A'],
    'value1': [10, 20, 30, 40, 50],
    'value2': [100, 200, 300, 400, 500]
})

# 使用内置聚合函数
result = df.groupby('category').agg({
    'value1': 'sum',
    'value2': 'mean'
})

# 使用多个聚合函数
result = df.groupby('category').agg({
    'value1': ['sum', 'mean', 'max'],
    'value2': ['min', 'max']
})

# 自定义聚合函数
def range_func(x):
    return x.max() - x.min()

result = df.groupby('category')['value1'].agg(['sum', range_func])

# 使用 lambda
result = df.groupby('category')['value1'].agg([
    ('总和', 'sum'),
    ('平均', 'mean'),
    ('范围', lambda x: x.max() - x.min())
])
```

---

## 5. 透视表
```python
import pandas as pd

df = pd.DataFrame({
    'date': ['2024-01', '2024-01', '2024-02', '2024-02'],
    'city': ['北京', '上海', '北京', '上海'],
    'sales': [100, 200, 150, 250]
})

# 创建透视表
pivot = pd.pivot_table(
    df,
    values='sales',      # 值
    index='date',        # 行
    columns='city',      # 列
    aggfunc='sum'        # 聚合函数
)
print(pivot)
# city    北京   上海
# date
# 2024-01  100   200
# 2024-02  150   250

# 多值透视
pivot = pd.pivot_table(
    df,
    values='sales',
    index='date',
    columns='city',
    aggfunc=['sum', 'mean'],
    fill_value=0  # 填充空值
)
```

---

## 6. 数值统计实战
```python
import pandas as pd

# 假设已有豆瓣电影数据
df = pd.read_csv('douban_top250.csv')

# 基本统计
print('评分统计:')
print(f"  平均分: {df['rating'].mean():.2f}")
print(f"  最高分: {df['rating'].max()}")
print(f"  最低分: {df['rating'].min()}")

# 评分分布
print('\n评分分布:')
rating_counts = df['rating'].value_counts().sort_index()
print(rating_counts)

# 按评分分组
df['rating_level'] = pd.cut(
    df['rating'],
    bins=[0, 8.5, 9.0, 9.5, 10],
    labels=['8.5以下', '8.5-9.0', '9.0-9.5', '9.5以上']
)
print('\n评分等级分布:')
print(df['rating_level'].value_counts())

# 按导演统计
print('\n导演作品数量（前10）:')
director_counts = df['director'].value_counts().head(10)
print(director_counts)
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 统计函数 | sum、mean、max、min、std |
| 排序 | sort_values、nlargest、nsmallest |
| 分组 | groupby、聚合函数 |
| agg | 多列多函数聚合 |
| 透视表 | pivot_table |

---

## 课后练习
完成练习题 16-18（见 [exercises.md](exercises.md)）

---
