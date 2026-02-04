# Day 25：数据分析项目（下）
> 学习目标：完成数据可视化、生成分析报告、整理项目代码
>

---

## 1. 今日任务

```
昨天我们完成了：
✓ 项目搭建
✓ 数据准备和清洗
✓ 基本统计分析

今天要完成：
□ 创建 4 种可视化图表
□ 组合成数据看板
□ 撰写分析报告
□ 整理完整代码
```

---

## 2. 准备工作

先加载昨天处理好的数据。

```python
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np

# 设置中文显示（重要！）
plt.rcParams['font.sans-serif'] = ['SimHei', 'Arial Unicode MS', 'DejaVu Sans']
plt.rcParams['axes.unicode_minus'] = False

# 加载昨天清洗好的数据
df = pd.read_csv('movie_analysis/data/movies_cleaned.csv')
print(f'加载数据: {len(df)} 条')

# 如果昨天没有添加这些列，现在添加
if 'rating_level' not in df.columns:
    df['rating_level'] = pd.cut(
        df['rating'],
        bins=[0, 8.5, 9.0, 9.5, 10],
        labels=['8.5以下', '8.5-9.0', '9.0-9.5', '9.5以上']
    )
if 'decade' not in df.columns:
    df['decade'] = (df['year'] // 10) * 10

print('数据准备完成！')
```

---

## 3. 图表1：评分分布直方图

直方图用来展示数据的分布情况。

```python
import pandas as pd
import matplotlib.pyplot as plt

# 加载数据
df = pd.read_csv('movie_analysis/data/movies_cleaned.csv')

# 创建图形
plt.figure(figsize=(10, 6))

# 绑制直方图
plt.hist(df['rating'], bins=20, color='steelblue', edgecolor='black', alpha=0.7)

# 添加标题和标签
plt.title('电影评分分布', fontsize=16)
plt.xlabel('评分', fontsize=12)
plt.ylabel('电影数量', fontsize=12)

# 添加网格线（更容易读数）
plt.grid(axis='y', alpha=0.3)

# 添加均值线
mean_rating = df['rating'].mean()
plt.axvline(mean_rating, color='red', linestyle='--', label=f'均值: {mean_rating:.2f}')
plt.legend()

# 保存图片
plt.savefig('movie_analysis/output/chart1_rating_dist.png', dpi=100, bbox_inches='tight')
plt.show()

print('图表1 保存完成！')
```

**代码解释：**
- `figsize=(10, 6)` 设置图片大小（宽10英寸，高6英寸）
- `bins=20` 将数据分成20个区间
- `alpha=0.7` 设置透明度
- `axvline` 添加垂直线标记均值
- `bbox_inches='tight'` 保存时去掉多余空白

---

## 4. 图表2：类型分布饼图

饼图用来展示各部分占比。

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv('movie_analysis/data/movies_cleaned.csv')

# 统计各类型数量
genre_counts = df['genre'].value_counts()

# 创建图形
plt.figure(figsize=(10, 8))

# 颜色列表
colors = ['#ff9999', '#66b3ff', '#99ff99', '#ffcc99', '#c2c2f0']

# 绘制饼图
plt.pie(
    genre_counts,
    labels=genre_counts.index,
    autopct='%1.1f%%',      # 显示百分比，保留1位小数
    colors=colors,
    startangle=90,          # 从90度开始
    explode=[0.05] * len(genre_counts)  # 每块稍微分离
)

plt.title('电影类型分布', fontsize=16)

# 保存
plt.savefig('movie_analysis/output/chart2_genre_pie.png', dpi=100, bbox_inches='tight')
plt.show()

print('图表2 保存完成！')
```

**代码解释：**
- `autopct='%1.1f%%'` 自动显示百分比
- `startangle=90` 从正上方开始绘制
- `explode` 让每块饼稍微分离，更美观

---

## 5. 图表3：各类型平均评分柱状图

柱状图用来比较不同类别的数值。

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv('movie_analysis/data/movies_cleaned.csv')

# 计算各类型平均评分
genre_rating = df.groupby('genre')['rating'].mean().sort_values(ascending=False)

# 创建图形
plt.figure(figsize=(10, 6))

# 绘制柱状图
bars = plt.bar(genre_rating.index, genre_rating.values, color='coral', edgecolor='black')

# 在柱子上添加数值
for bar, value in zip(bars, genre_rating.values):
    plt.text(
        bar.get_x() + bar.get_width()/2,  # x 位置：柱子中心
        bar.get_height() + 0.02,           # y 位置：柱子顶部稍上
        f'{value:.2f}',                    # 显示的文字
        ha='center',                       # 水平居中
        fontsize=10
    )

plt.title('各类型电影平均评分', fontsize=16)
plt.xlabel('电影类型', fontsize=12)
plt.ylabel('平均评分', fontsize=12)

# 设置 y 轴从 8 开始，更能看出差异
plt.ylim(8, 10)

plt.savefig('movie_analysis/output/chart3_genre_rating.png', dpi=100, bbox_inches='tight')
plt.show()

print('图表3 保存完成！')
```

**代码解释：**
- `sort_values(ascending=False)` 按评分从高到低排序
- `plt.text()` 在柱子上方添加数值标签
- `plt.ylim(8, 10)` 设置 y 轴范围，让差异更明显

---

## 6. 图表4：评分与评价人数散点图

散点图用来展示两个变量之间的关系。

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv('movie_analysis/data/movies_cleaned.csv')

# 创建图形
plt.figure(figsize=(10, 6))

# 绘制散点图
plt.scatter(
    df['rating'],
    df['rating_num'],
    alpha=0.5,           # 透明度，防止点重叠看不清
    c='green',           # 颜色
    s=50                 # 点的大小
)

plt.title('评分与评价人数关系', fontsize=16)
plt.xlabel('评分', fontsize=12)
plt.ylabel('评价人数', fontsize=12)

# 添加网格
plt.grid(alpha=0.3)

plt.savefig('movie_analysis/output/chart4_scatter.png', dpi=100, bbox_inches='tight')
plt.show()

print('图表4 保存完成！')
```

**思考：** 从散点图中你能看出评分和评价人数有什么关系吗？

---

## 7. 组合成数据看板

把 4 个图表组合到一张图上，形成数据看板。

```python
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np

# 设置中文
plt.rcParams['font.sans-serif'] = ['SimHei', 'Arial Unicode MS', 'DejaVu Sans']
plt.rcParams['axes.unicode_minus'] = False

df = pd.read_csv('movie_analysis/data/movies_cleaned.csv')

# 创建 2x2 的子图
fig, axes = plt.subplots(2, 2, figsize=(14, 12))

# 添加总标题
fig.suptitle('电影数据分析报告', fontsize=20, fontweight='bold')

# ===== 图1: 评分分布直方图 =====
axes[0, 0].hist(df['rating'], bins=20, color='steelblue', edgecolor='black', alpha=0.7)
axes[0, 0].set_title('评分分布')
axes[0, 0].set_xlabel('评分')
axes[0, 0].set_ylabel('数量')
mean_rating = df['rating'].mean()
axes[0, 0].axvline(mean_rating, color='red', linestyle='--', label=f'均值: {mean_rating:.2f}')
axes[0, 0].legend()

# ===== 图2: 类型分布饼图 =====
genre_counts = df['genre'].value_counts()
colors = ['#ff9999', '#66b3ff', '#99ff99', '#ffcc99', '#c2c2f0']
axes[0, 1].pie(genre_counts, labels=genre_counts.index, autopct='%1.1f%%', colors=colors)
axes[0, 1].set_title('类型分布')

# ===== 图3: 各类型平均评分柱状图 =====
genre_rating = df.groupby('genre')['rating'].mean().sort_values(ascending=False)
bars = axes[1, 0].bar(genre_rating.index, genre_rating.values, color='coral', edgecolor='black')
axes[1, 0].set_title('各类型平均评分')
axes[1, 0].set_xlabel('类型')
axes[1, 0].set_ylabel('平均评分')
axes[1, 0].set_ylim(8, 10)
# 添加数值标签
for bar, value in zip(bars, genre_rating.values):
    axes[1, 0].text(bar.get_x() + bar.get_width()/2, bar.get_height() + 0.02,
                    f'{value:.2f}', ha='center', fontsize=9)

# ===== 图4: 评分与评价人数散点图 =====
axes[1, 1].scatter(df['rating'], df['rating_num'], alpha=0.5, c='green', s=30)
axes[1, 1].set_title('评分与评价人数关系')
axes[1, 1].set_xlabel('评分')
axes[1, 1].set_ylabel('评价人数')
axes[1, 1].grid(alpha=0.3)

# 调整布局
plt.tight_layout()
plt.subplots_adjust(top=0.92)  # 为总标题留出空间

# 保存数据看板
plt.savefig('movie_analysis/output/dashboard.png', dpi=150, bbox_inches='tight')
plt.show()

print('数据看板保存完成: output/dashboard.png')
```

---

## 8. 撰写分析报告

最后，把分析结果写成文字报告。

```python
import pandas as pd
from datetime import datetime

df = pd.read_csv('movie_analysis/data/movies_cleaned.csv')

# 计算统计数据
total = len(df)
mean_rating = df['rating'].mean()
median_rating = df['rating'].median()
max_rating = df['rating'].max()
min_rating = df['rating'].min()

top_genre = df.groupby('genre')['rating'].mean().idxmax()
top_country = df.groupby('country')['rating'].mean().idxmax()

# 生成报告
report = f'''
{'='*60}
电影数据分析报告
{'='*60}

生成时间: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}

一、数据概况
-----------
分析电影总数: {total} 部
数据来源: 豆瓣电影 Top250（模拟数据）

二、评分分析
-----------
平均评分: {mean_rating:.2f} 分
评分中位数: {median_rating:.1f} 分
最高评分: {max_rating} 分
最低评分: {min_rating} 分

三、类型分析
-----------
电影类型分布:
{df['genre'].value_counts().to_string()}

平均评分最高的类型: {top_genre}

四、国家分析
-----------
电影国家分布:
{df['country'].value_counts().to_string()}

平均评分最高的国家: {top_country}

五、主要发现
-----------
1. 电影平均评分为 {mean_rating:.2f} 分，整体质量较高
2. {df['genre'].value_counts().index[0]} 类型电影数量最多
3. {top_genre} 类型电影平均评分最高
4. {top_country} 国家的电影平均评分最高

六、结论与建议
-----------
1. 该数据集电影整体评分偏高，说明是精选的优质电影
2. 建议重点关注 {top_genre} 类型电影
3. {top_country} 电影值得推荐

{'='*60}
报告生成完毕
{'='*60}
'''

# 保存报告
with open('movie_analysis/output/report.txt', 'w', encoding='utf-8') as f:
    f.write(report)

print(report)
print('\n报告已保存到: output/report.txt')
```

---

## 9. 整理完整代码

把所有代码整合成一个完整的分析脚本。

```python
"""
电影数据分析项目
完整代码 - analysis.py
"""
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
import os
from datetime import datetime

# 设置中文显示
plt.rcParams['font.sans-serif'] = ['SimHei', 'Arial Unicode MS', 'DejaVu Sans']
plt.rcParams['axes.unicode_minus'] = False


def setup_project():
    """创建项目目录"""
    os.makedirs('movie_analysis/data', exist_ok=True)
    os.makedirs('movie_analysis/output', exist_ok=True)


def load_data(filepath):
    """加载数据"""
    df = pd.read_csv(filepath)
    print(f'加载数据: {len(df)} 条')
    return df


def clean_data(df):
    """数据清洗"""
    print(f'清洗前: {len(df)} 条')

    # 删除缺失值
    df = df.dropna()

    # 删除评分异常值
    df = df[(df['rating'] >= 0) & (df['rating'] <= 10)]

    # 删除年份异常值
    df = df[df['year'] >= 1895]

    print(f'清洗后: {len(df)} 条')
    return df


def analyze_data(df):
    """数据分析"""
    print('\n=== 数据分析 ===')
    print(f'电影总数: {len(df)}')
    print(f'平均评分: {df["rating"].mean():.2f}')
    print(f'评分范围: {df["rating"].min()} - {df["rating"].max()}')

    print('\n各类型统计:')
    print(df.groupby('genre')['rating'].agg(['count', 'mean']).round(2))


def create_dashboard(df, output_path):
    """创建数据看板"""
    fig, axes = plt.subplots(2, 2, figsize=(14, 12))
    fig.suptitle('电影数据分析报告', fontsize=20, fontweight='bold')

    # 图1: 直方图
    axes[0, 0].hist(df['rating'], bins=20, color='steelblue', edgecolor='black', alpha=0.7)
    axes[0, 0].set_title('评分分布')
    axes[0, 0].axvline(df['rating'].mean(), color='red', linestyle='--')

    # 图2: 饼图
    genre_counts = df['genre'].value_counts()
    axes[0, 1].pie(genre_counts, labels=genre_counts.index, autopct='%1.1f%%')
    axes[0, 1].set_title('类型分布')

    # 图3: 柱状图
    genre_rating = df.groupby('genre')['rating'].mean().sort_values(ascending=False)
    axes[1, 0].bar(genre_rating.index, genre_rating.values, color='coral')
    axes[1, 0].set_title('各类型平均评分')
    axes[1, 0].set_ylim(8, 10)

    # 图4: 散点图
    axes[1, 1].scatter(df['rating'], df['rating_num'], alpha=0.5, c='green')
    axes[1, 1].set_title('评分与评价人数关系')

    plt.tight_layout()
    plt.subplots_adjust(top=0.92)
    plt.savefig(output_path, dpi=150, bbox_inches='tight')
    print(f'\n数据看板已保存: {output_path}')


def generate_report(df, output_path):
    """生成分析报告"""
    report = f'''电影数据分析报告
生成时间: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}

数据概况:
- 电影总数: {len(df)} 部
- 平均评分: {df['rating'].mean():.2f} 分
- 最高评分: {df['rating'].max()} 分
- 最低评分: {df['rating'].min()} 分

主要发现:
1. 电影整体评分较高，平均 {df['rating'].mean():.2f} 分
2. {df['genre'].value_counts().index[0]} 类型电影数量最多
3. {df.groupby('genre')['rating'].mean().idxmax()} 类型平均评分最高
'''

    with open(output_path, 'w', encoding='utf-8') as f:
        f.write(report)
    print(f'报告已保存: {output_path}')


def main():
    """主函数"""
    print('='*50)
    print('电影数据分析项目')
    print('='*50)

    # 1. 加载数据
    df = load_data('movie_analysis/data/movies.csv')

    # 2. 数据清洗
    df = clean_data(df)

    # 3. 数据分析
    analyze_data(df)

    # 4. 创建可视化
    create_dashboard(df, 'movie_analysis/output/dashboard.png')

    # 5. 生成报告
    generate_report(df, 'movie_analysis/output/report.txt')

    print('\n' + '='*50)
    print('分析完成！')
    print('='*50)


if __name__ == '__main__':
    main()
```

---

## 10. 项目最终结构

```
movie_analysis/
├── data/
│   ├── movies.csv           # 原始数据
│   └── movies_cleaned.csv   # 清洗后数据
├── output/
│   ├── dashboard.png        # 数据看板
│   └── report.txt           # 分析报告
├── analysis.py              # 完整分析代码
└── README.md                # 项目说明（选做）
```

---

## 今日小结
| 步骤 | 内容 | 关键代码 |
| --- | --- | --- |
| 1 | 直方图 | plt.hist() |
| 2 | 饼图 | plt.pie() |
| 3 | 柱状图 | plt.bar() |
| 4 | 散点图 | plt.scatter() |
| 5 | 组合图 | plt.subplots() |
| 6 | 报告 | 字符串格式化 |
| 7 | 整合 | 函数封装 |

---

## 课后练习
完成练习题（见 [exercises.md](exercises.md)）

---

## 恭喜完成！

你已经独立完成了一个完整的数据分析项目！

这个项目包含了数据分析的核心流程：
1. ✅ 数据获取
2. ✅ 数据清洗
3. ✅ 数据分析
4. ✅ 数据可视化
5. ✅ 报告撰写

这个流程在实际工作中非常常用，无论是金融分析、市场研究还是学术研究，都会用到！
