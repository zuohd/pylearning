# Day 19：pandas 入门
> 学习目标：掌握 pandas 基础，学会创建和操作 DataFrame
>

---

## 1. pandas 简介
```python
# 安装
# pip install pandas

import pandas as pd
import numpy as np

# pandas 两种核心数据结构
# Series - 一维数据
# DataFrame - 二维表格数据
```

---

## 2. Series 基础
```python
import pandas as pd

# 创建 Series
s1 = pd.Series([1, 2, 3, 4, 5])
print(s1)
# 0    1
# 1    2
# 2    3
# 3    4
# 4    5

# 指定索引
s2 = pd.Series([90, 85, 88], index=['语文', '数学', '英语'])
print(s2)
# 语文    90
# 数学    85
# 英语    88

# 访问元素
print(s2['语文'])    # 90
print(s2[0])        # 90
print(s2.values)    # array([90, 85, 88])
print(s2.index)     # Index(['语文', '数学', '英语'])
```

---

## 3. DataFrame 创建
```python
import pandas as pd

# 从字典创建
data = {
    'name': ['张三', '李四', '王五'],
    'age': [20, 21, 22],
    'score': [85, 90, 88]
}
df = pd.DataFrame(data)
print(df)
#   name  age  score
# 0   张三   20     85
# 1   李四   21     90
# 2   王五   22     88

# 从列表创建
data = [
    ['张三', 20, 85],
    ['李四', 21, 90],
    ['王五', 22, 88]
]
df = pd.DataFrame(data, columns=['name', 'age', 'score'])

# 从 CSV 读取
df = pd.read_csv('data.csv')

# 从 Excel 读取
df = pd.read_excel('data.xlsx')
```

---

## 4. 基本属性和方法
```python
import pandas as pd

df = pd.DataFrame({
    'name': ['张三', '李四', '王五', '赵六'],
    'age': [20, 21, 22, 20],
    'score': [85, 90, 88, 92]
})

# 基本属性
print(df.shape)      # (4, 3) 行数和列数
print(df.columns)    # 列名
print(df.index)      # 行索引
print(df.dtypes)     # 数据类型

# 预览数据
print(df.head())     # 前5行
print(df.head(2))    # 前2行
print(df.tail())     # 后5行

# 基本信息
print(df.info())     # 数据类型、非空值数量
print(df.describe()) # 统计摘要
```

---

## 5. 数据选择
```python
import pandas as pd

df = pd.DataFrame({
    'name': ['张三', '李四', '王五', '赵六'],
    'age': [20, 21, 22, 20],
    'score': [85, 90, 88, 92],
    'city': ['北京', '上海', '广州', '深圳']
})

# 选择列
print(df['name'])           # 单列，返回 Series
print(df[['name', 'age']])  # 多列，返回 DataFrame

# 选择行 - 使用 loc（标签）
print(df.loc[0])            # 第0行
print(df.loc[0:2])          # 第0-2行（包含2）

# 选择行 - 使用 iloc（位置）
print(df.iloc[0])           # 第0行
print(df.iloc[0:2])         # 第0-1行（不包含2）

# 选择行和列
print(df.loc[0, 'name'])           # 第0行的name列
print(df.loc[0:2, ['name', 'age']]) # 0-2行的name和age列
print(df.iloc[0:2, 0:2])           # 0-1行，0-1列
```

---

## 6. 条件筛选
```python
import pandas as pd

df = pd.DataFrame({
    'name': ['张三', '李四', '王五', '赵六'],
    'age': [20, 21, 22, 20],
    'score': [85, 90, 88, 92]
})

# 单条件筛选
print(df[df['age'] > 20])
print(df[df['score'] >= 90])

# 多条件筛选（与）
print(df[(df['age'] > 20) & (df['score'] >= 88)])

# 多条件筛选（或）
print(df[(df['age'] == 20) | (df['score'] >= 90)])

# isin 筛选
print(df[df['name'].isin(['张三', '李四'])])

# 字符串包含
df['city'] = ['北京', '上海', '广州', '深圳']
print(df[df['city'].str.contains('北')])
```

---

## 7. 添加和修改列
```python
import pandas as pd

df = pd.DataFrame({
    'name': ['张三', '李四', '王五'],
    'score': [85, 90, 88]
})

# 添加新列
df['passed'] = df['score'] >= 60
df['grade'] = ['B', 'A', 'B']

# 基于计算添加列
df['score_plus'] = df['score'] + 10

# 条件添加列
df['level'] = df['score'].apply(lambda x: '优秀' if x >= 90 else '良好')

# 修改列
df['score'] = df['score'] * 1.1

print(df)
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| Series | 一维数据，带索引 |
| DataFrame | 二维表格，行列结构 |
| 创建 | 字典、列表、CSV、Excel |
| 选择 | []、loc、iloc |
| 筛选 | 条件表达式、& | isin |

---

## 课后练习
完成练习题 10-12（见 [phase2_exercises.md](phase2_exercises.md#day-19-练习题pandas-入门)）

---
