# Day 20：数据清洗与处理
> 学习目标：掌握数据清洗技巧，处理缺失值和数据类型
>

---

## 1. 检查缺失值
```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'name': ['张三', '李四', None, '赵六'],
    'age': [20, None, 22, 20],
    'score': [85, 90, np.nan, 92]
})

# 检查缺失值
print(df.isnull())       # 每个元素是否为空
print(df.isnull().sum()) # 每列空值数量
print(df.isnull().any()) # 每列是否有空值

# 检查非空
print(df.notnull())
```

---

## 2. 处理缺失值
```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'name': ['张三', '李四', None, '赵六'],
    'age': [20, None, 22, 20],
    'score': [85, 90, np.nan, 92]
})

# 删除包含空值的行
df_dropped = df.dropna()

# 删除全为空的行
df_dropped = df.dropna(how='all')

# 删除某列有空值的行
df_dropped = df.dropna(subset=['name'])

# 填充缺失值
df_filled = df.fillna(0)                  # 用0填充
df_filled = df.fillna({'age': 0, 'score': 60})  # 指定列填充

# 用均值填充
df['age'] = df['age'].fillna(df['age'].mean())

# 用前一个值填充
df_filled = df.fillna(method='ffill')

# 用后一个值填充
df_filled = df.fillna(method='bfill')
```

---

## 3. 数据类型转换
```python
import pandas as pd

df = pd.DataFrame({
    'id': ['1', '2', '3'],
    'price': ['10.5', '20.3', '15.8'],
    'date': ['2024-01-01', '2024-01-02', '2024-01-03']
})

print(df.dtypes)

# 转换数据类型
df['id'] = df['id'].astype(int)
df['price'] = df['price'].astype(float)

# 转换日期
df['date'] = pd.to_datetime(df['date'])

# 转换为数值（带错误处理）
df['value'] = pd.to_numeric(df['id'], errors='coerce')  # 无法转换变成 NaN

print(df.dtypes)
```

---

## 4. 字符串处理
```python
import pandas as pd

df = pd.DataFrame({
    'name': ['  张三  ', 'LISI', 'wang wu'],
    'email': ['zhang@test.com', 'li@test.com', 'wang@test.com']
})

# 去除空白
df['name'] = df['name'].str.strip()

# 大小写转换
df['name_upper'] = df['name'].str.upper()
df['name_lower'] = df['name'].str.lower()
df['name_title'] = df['name'].str.title()

# 替换
df['name'] = df['name'].str.replace('zhang', '张', regex=False)

# 分割
df['email_name'] = df['email'].str.split('@').str[0]

# 包含判断
df['is_test'] = df['email'].str.contains('test')

# 长度
df['name_len'] = df['name'].str.len()
```

---

## 5. 重复值处理
```python
import pandas as pd

df = pd.DataFrame({
    'name': ['张三', '李四', '张三', '王五'],
    'age': [20, 21, 20, 22]
})

# 检查重复
print(df.duplicated())           # 每行是否重复
print(df.duplicated().sum())     # 重复行数

# 删除重复行
df_unique = df.drop_duplicates()

# 基于某列去重
df_unique = df.drop_duplicates(subset=['name'])

# 保留最后一个
df_unique = df.drop_duplicates(subset=['name'], keep='last')

# 不保留任何重复
df_unique = df.drop_duplicates(subset=['name'], keep=False)
```

---

## 6. 数据替换
```python
import pandas as pd

df = pd.DataFrame({
    'grade': ['A', 'B', 'C', 'D', 'F'],
    'status': ['pass', 'pass', 'pass', 'pass', 'fail']
})

# 单值替换
df['grade'] = df['grade'].replace('F', 'E')

# 多值替换
df['grade'] = df['grade'].replace({'A': '优', 'B': '良', 'C': '中'})

# 使用字典批量替换
grade_map = {'A': 4, 'B': 3, 'C': 2, 'D': 1, 'E': 0}
df['score'] = df['grade'].map(grade_map)
```

---

## 7. 异常值处理
```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'score': [85, 90, 150, 88, -10, 92, 200]
})

# 查看分布
print(df['score'].describe())

# 使用条件筛选异常值
outliers = df[(df['score'] < 0) | (df['score'] > 100)]
print(f'异常值: {len(outliers)} 个')

# 替换异常值
df['score'] = df['score'].clip(lower=0, upper=100)

# 或用 np.where
df['score'] = np.where(df['score'] > 100, 100, df['score'])
df['score'] = np.where(df['score'] < 0, 0, df['score'])

# 使用 IQR 方法检测异常
Q1 = df['score'].quantile(0.25)
Q3 = df['score'].quantile(0.75)
IQR = Q3 - Q1
lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

outliers = df[(df['score'] < lower_bound) | (df['score'] > upper_bound)]
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 缺失值检查 | isnull、notnull、sum |
| 缺失值处理 | dropna、fillna |
| 类型转换 | astype、to_datetime、to_numeric |
| 字符串处理 | str.strip、replace、split |
| 重复值 | duplicated、drop_duplicates |
| 异常值 | clip、IQR 方法 |

---

## 课后练习
完成练习（见 [exercises.md](exercises.md)）

---
