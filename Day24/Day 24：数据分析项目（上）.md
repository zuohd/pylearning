# Day 24：数据分析项目（上）
> 学习目标：完成项目搭建、数据准备和清洗、基本统计分析
>

---

## 1. 项目介绍

今天和明天，我们将完成一个完整的数据分析项目。

```
项目目标：分析电影数据，回答以下问题
- 电影评分的整体分布如何？
- 哪些年份的电影评分较高？
- 不同类型电影的表现如何？
- 评分和评价人数有什么关系？

今天任务（Day 24）：
1. 搭建项目结构
2. 准备数据
3. 数据清洗
4. 基本统计分析

明天任务（Day 25）：
5. 数据可视化
6. 生成分析报告
```

---

## 2. 第一步：搭建项目结构

先创建项目文件夹，养成良好的项目组织习惯。

```python
import os

# 创建项目目录结构
os.makedirs('movie_analysis/data', exist_ok=True)
os.makedirs('movie_analysis/output', exist_ok=True)

print('项目结构创建完成！')
print('''
movie_analysis/
├── data/           <- 存放数据文件
├── output/         <- 存放输出结果（图表、报告）
└── analysis.py     <- 分析代码（稍后创建）
''')
```

---

## 3. 第二步：准备数据

如果你有 Day 18 爬取的豆瓣数据，可以直接使用。如果没有，我们用代码生成模拟数据。

```python
import pandas as pd
import numpy as np

# 设置随机种子，保证每次运行结果一致
np.random.seed(42)

# 生成 250 部电影的模拟数据
n = 250

# 创建数据
movies = pd.DataFrame({
    'rank': range(1, n + 1),
    'title': [f'电影{i}' for i in range(1, n + 1)],
    'rating': np.round(np.random.uniform(8.0, 9.8, n), 1),
    'rating_num': np.random.randint(10000, 2000000, n),
    'year': np.random.randint(1950, 2024, n),
    'genre': np.random.choice(['剧情', '喜剧', '动作', '科幻', '爱情'], n),
    'country': np.random.choice(['美国', '中国', '日本', '韩国', '英国', '法国'], n)
})

# 故意加入一些"脏数据"，模拟真实情况
# 添加一些缺失值
movies.loc[5, 'rating'] = np.nan
movies.loc[10, 'year'] = np.nan
movies.loc[15, 'genre'] = None

# 添加一些异常值
movies.loc[20, 'rating'] = 15.0    # 评分不可能超过 10
movies.loc[25, 'rating'] = -1.0    # 评分不可能为负数
movies.loc[30, 'year'] = 1800      # 电影 1895 年才发明

# 保存数据
movies.to_csv('movie_analysis/data/movies.csv', index=False)
print(f'数据已保存，共 {len(movies)} 条记录')

# 查看前几行
print('\n数据预览：')
print(movies.head())
```

---

## 4. 第三步：加载和查看数据

现在开始正式的分析工作。第一步永远是：了解你的数据。

```python
import pandas as pd

# 加载数据
df = pd.read_csv('movie_analysis/data/movies.csv')

# 查看基本信息
print('=== 数据基本信息 ===')
print(f'数据形状: {df.shape[0]} 行, {df.shape[1]} 列')
print(f'\n列名: {list(df.columns)}')

print('\n=== 数据类型 ===')
print(df.dtypes)

print('\n=== 前 5 行数据 ===')
print(df.head())

print('\n=== 数据统计摘要 ===')
print(df.describe())
```

**理解输出：**
- `shape` 告诉你数据有多少行和列
- `dtypes` 告诉你每列的数据类型
- `describe()` 给出数值列的统计信息（计数、均值、标准差等）

---

## 5. 第四步：检查数据质量

在分析之前，必须检查数据是否有问题。

```python
import pandas as pd
import numpy as np

df = pd.read_csv('movie_analysis/data/movies.csv')

print('=== 缺失值检查 ===')
missing = df.isnull().sum()
print(missing[missing > 0])  # 只显示有缺失的列

print('\n=== 评分范围检查 ===')
print(f'评分最小值: {df["rating"].min()}')
print(f'评分最大值: {df["rating"].max()}')
# 电影评分应该在 0-10 之间
abnormal_rating = df[(df['rating'] < 0) | (df['rating'] > 10)]
print(f'异常评分数量: {len(abnormal_rating)}')

print('\n=== 年份范围检查 ===')
print(f'年份最小值: {df["year"].min()}')
print(f'年份最大值: {df["year"].max()}')
# 电影 1895 年发明，不可能有更早的
abnormal_year = df[df['year'] < 1895]
print(f'异常年份数量: {len(abnormal_year)}')

print('\n=== 重复值检查 ===')
duplicates = df.duplicated().sum()
print(f'重复行数量: {duplicates}')
```

---

## 6. 第五步：数据清洗

发现问题后，逐一解决。

```python
import pandas as pd
import numpy as np

df = pd.read_csv('movie_analysis/data/movies.csv')
print(f'清洗前: {len(df)} 条数据')

# 1. 处理缺失值
# 方法：删除缺失值所在的行（数据量大时常用）
df = df.dropna()
print(f'删除缺失值后: {len(df)} 条')

# 2. 处理评分异常值
# 方法：将超出范围的值限制在合理范围内
df = df[(df['rating'] >= 0) & (df['rating'] <= 10)]
print(f'删除异常评分后: {len(df)} 条')

# 3. 处理年份异常值
# 方法：删除不合理的年份
df = df[df['year'] >= 1895]
print(f'删除异常年份后: {len(df)} 条')

# 4. 确保数据类型正确
df['year'] = df['year'].astype(int)
df['rating_num'] = df['rating_num'].astype(int)

print(f'\n清洗完成！最终数据: {len(df)} 条')

# 保存清洗后的数据
df.to_csv('movie_analysis/data/movies_cleaned.csv', index=False)
print('清洗后的数据已保存到 movies_cleaned.csv')
```

**小贴士：**
- 数据清洗没有标准答案，要根据实际情况决定
- 删除数据要谨慎，数据量少时可以考虑填充
- 清洗后最好保存一份，避免重复劳动

---

## 7. 第六步：基本统计分析

数据清洗完成，开始分析！

```python
import pandas as pd

# 加载清洗后的数据
df = pd.read_csv('movie_analysis/data/movies_cleaned.csv')

print('=' * 50)
print('电影数据基本统计分析')
print('=' * 50)

# 1. 整体统计
print('\n【1. 整体情况】')
print(f'电影总数: {len(df)} 部')
print(f'评分均值: {df["rating"].mean():.2f}')
print(f'评分中位数: {df["rating"].median():.2f}')
print(f'评分标准差: {df["rating"].std():.2f}')
print(f'最高评分: {df["rating"].max()}')
print(f'最低评分: {df["rating"].min()}')

# 2. 评分分布
print('\n【2. 评分分布】')
# 将评分分成几个等级
df['rating_level'] = pd.cut(
    df['rating'],
    bins=[0, 8.5, 9.0, 9.5, 10],
    labels=['8.5以下', '8.5-9.0', '9.0-9.5', '9.5以上']
)
print(df['rating_level'].value_counts().sort_index())

# 3. 按类型分组统计
print('\n【3. 各类型电影统计】')
genre_stats = df.groupby('genre').agg({
    'rating': ['count', 'mean'],
    'rating_num': 'mean'
}).round(2)
genre_stats.columns = ['数量', '平均评分', '平均评价人数']
print(genre_stats.sort_values('平均评分', ascending=False))

# 4. 按国家分组统计
print('\n【4. 各国家电影统计】')
country_stats = df.groupby('country').agg({
    'rating': ['count', 'mean']
}).round(2)
country_stats.columns = ['数量', '平均评分']
print(country_stats.sort_values('平均评分', ascending=False))

# 5. 按年代分组统计
print('\n【5. 各年代电影统计】')
df['decade'] = (df['year'] // 10) * 10  # 1994 -> 1990
decade_stats = df.groupby('decade').agg({
    'rating': ['count', 'mean']
}).round(2)
decade_stats.columns = ['数量', '平均评分']
print(decade_stats.tail(10))  # 最近 10 个年代
```

---

## 8. 第七步：保存分析结果

把今天的分析结果保存下来，明天继续。

```python
import pandas as pd

df = pd.read_csv('movie_analysis/data/movies_cleaned.csv')

# 添加计算列
df['rating_level'] = pd.cut(
    df['rating'],
    bins=[0, 8.5, 9.0, 9.5, 10],
    labels=['8.5以下', '8.5-9.0', '9.0-9.5', '9.5以上']
)
df['decade'] = (df['year'] // 10) * 10

# 保存带有新列的数据
df.to_csv('movie_analysis/data/movies_analyzed.csv', index=False)

# 保存统计结果
stats = {
    '电影总数': len(df),
    '平均评分': round(df['rating'].mean(), 2),
    '评分中位数': df['rating'].median(),
    '最高评分': df['rating'].max(),
    '最低评分': df['rating'].min(),
}

# 写入文本文件
with open('movie_analysis/output/day24_stats.txt', 'w', encoding='utf-8') as f:
    f.write('Day 24 统计结果\n')
    f.write('=' * 30 + '\n')
    for key, value in stats.items():
        f.write(f'{key}: {value}\n')

print('今日分析结果已保存！')
print('- 数据文件: movie_analysis/data/movies_analyzed.csv')
print('- 统计结果: movie_analysis/output/day24_stats.txt')
```

---

## 9. 常见问题解答

```
Q1: 为什么要做数据清洗？
A: 真实数据往往有缺失、错误、重复等问题。
   如果不清洗，分析结果会不准确。
   比如一个 -1 的评分会拉低平均分。

Q2: 删除数据会不会太浪费？
A: 如果数据量大（比如几万条），删除几条影响不大。
   如果数据量小，可以考虑用均值、中位数填充。

Q3: groupby 是什么意思？
A: 就像 Excel 的数据透视表，按某一列分组后计算。
   比如 df.groupby('genre')['rating'].mean()
   就是计算每个类型的平均评分。

Q4: agg 是什么？
A: agg 可以同时计算多个统计量。
   比如 agg(['count', 'mean']) 同时计算数量和均值。
```

---

## 今日小结
| 步骤 | 内容 | 代码要点 |
| --- | --- | --- |
| 1 | 搭建项目 | os.makedirs() |
| 2 | 准备数据 | pd.DataFrame, to_csv |
| 3 | 查看数据 | shape, dtypes, describe |
| 4 | 检查质量 | isnull, 范围检查 |
| 5 | 数据清洗 | dropna, 条件筛选 |
| 6 | 基本统计 | mean, groupby, agg |
| 7 | 保存结果 | to_csv, 写文件 |

---

## 课后练习
完成练习题（见 [exercises.md](exercises.md)）

---

## 明日预告
明天我们将完成：
- 数据可视化（4种图表）
- 组合成数据看板
- 撰写分析报告
- 整理代码

今天的代码和数据记得保存好！
