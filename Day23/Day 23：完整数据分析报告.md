# Day 23：完整数据分析报告
> 学习目标：综合运用所学知识，完成一个完整的数据分析项目
>

---

## 1. 数据分析流程
```
完整数据分析流程：

1. 明确问题
   - 分析目标是什么？
   - 需要回答哪些问题？

2. 数据获取
   - 数据来源
   - 数据格式

3. 数据清洗
   - 处理缺失值
   - 处理异常值
   - 数据类型转换

4. 数据探索
   - 基本统计
   - 分布分析
   - 相关性分析

5. 数据可视化
   - 选择合适图表
   - 美化图表

6. 结论与建议
   - 总结发现
   - 提出建议
```

---

## 2. 项目示例：电影数据分析
```python
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np

# 设置中文显示
plt.rcParams['font.sans-serif'] = ['SimHei', 'Arial Unicode MS']
plt.rcParams['axes.unicode_minus'] = False

class MovieAnalyzer:
    def __init__(self, filename):
        self.df = pd.read_csv(filename)
        print(f'加载数据: {len(self.df)} 条记录')

    def clean_data(self):
        """数据清洗"""
        # 删除空值
        self.df = self.df.dropna()

        # 转换数据类型
        self.df['rating'] = self.df['rating'].astype(float)
        self.df['rating_num'] = self.df['rating_num'].astype(int)

        print(f'清洗后: {len(self.df)} 条记录')

    def basic_stats(self):
        """基本统计"""
        print('\n=== 基本统计 ===')
        print(f"电影数量: {len(self.df)}")
        print(f"平均评分: {self.df['rating'].mean():.2f}")
        print(f"评分范围: {self.df['rating'].min()} - {self.df['rating'].max()}")
        print(f"平均评价人数: {self.df['rating_num'].mean():.0f}")

    def rating_distribution(self):
        """评分分布分析"""
        print('\n=== 评分分布 ===')

        # 评分统计
        rating_counts = self.df['rating'].value_counts().sort_index()
        print(rating_counts)

        # 绑制直方图
        plt.figure(figsize=(10, 6))
        plt.hist(self.df['rating'], bins=20, color='steelblue', edgecolor='black')
        plt.title('电影评分分布')
        plt.xlabel('评分')
        plt.ylabel('数量')
        plt.savefig('rating_distribution.png', dpi=100, bbox_inches='tight')
        plt.show()
```

---

## 3. 多图组合
```python
def create_dashboard(self):
    """创建数据看板"""
    fig, axes = plt.subplots(2, 2, figsize=(14, 12))

    # 图1: 评分分布直方图
    axes[0, 0].hist(self.df['rating'], bins=15, color='steelblue', edgecolor='black')
    axes[0, 0].set_title('评分分布')
    axes[0, 0].set_xlabel('评分')
    axes[0, 0].set_ylabel('数量')

    # 图2: 评分等级饼图
    rating_levels = pd.cut(self.df['rating'],
                          bins=[0, 8.5, 9.0, 9.5, 10],
                          labels=['8.5以下', '8.5-9.0', '9.0-9.5', '9.5以上'])
    level_counts = rating_levels.value_counts()
    axes[0, 1].pie(level_counts, labels=level_counts.index,
                   autopct='%1.1f%%', colors=['#ff9999', '#66b3ff', '#99ff99', '#ffcc99'])
    axes[0, 1].set_title('评分等级占比')

    # 图3: Top10 高分电影
    top10 = self.df.nlargest(10, 'rating')
    axes[1, 0].barh(top10['title'], top10['rating'], color='coral')
    axes[1, 0].set_xlabel('评分')
    axes[1, 0].set_title('评分最高的10部电影')
    axes[1, 0].invert_yaxis()

    # 图4: 评分与评价人数的关系
    axes[1, 1].scatter(self.df['rating'], self.df['rating_num'],
                       alpha=0.6, color='green')
    axes[1, 1].set_xlabel('评分')
    axes[1, 1].set_ylabel('评价人数')
    axes[1, 1].set_title('评分与评价人数关系')

    plt.tight_layout()
    plt.savefig('movie_dashboard.png', dpi=150, bbox_inches='tight')
    plt.show()
```

---

## 4. 代码封装
```python
class DataAnalyzer:
    """通用数据分析类"""

    def __init__(self, filename):
        self.df = self.load_data(filename)
        self.setup_plot()

    def load_data(self, filename):
        """加载数据"""
        if filename.endswith('.csv'):
            return pd.read_csv(filename)
        elif filename.endswith('.xlsx'):
            return pd.read_excel(filename)
        else:
            raise ValueError('不支持的文件格式')

    def setup_plot(self):
        """设置绑图参数"""
        plt.rcParams['font.sans-serif'] = ['SimHei', 'Arial Unicode MS']
        plt.rcParams['axes.unicode_minus'] = False

    def info(self):
        """数据基本信息"""
        print('=== 数据概览 ===')
        print(f'行数: {len(self.df)}')
        print(f'列数: {len(self.df.columns)}')
        print(f'列名: {list(self.df.columns)}')
        print('\n数据类型:')
        print(self.df.dtypes)

    def describe(self):
        """统计摘要"""
        print('\n=== 统计摘要 ===')
        print(self.df.describe())

    def missing_report(self):
        """缺失值报告"""
        missing = self.df.isnull().sum()
        missing_pct = (missing / len(self.df) * 100).round(2)

        report = pd.DataFrame({
            '缺失数': missing,
            '缺失比例(%)': missing_pct
        })

        print('\n=== 缺失值报告 ===')
        print(report[report['缺失数'] > 0])

    def save_report(self, filename='report.txt'):
        """保存分析报告"""
        with open(filename, 'w', encoding='utf-8') as f:
            f.write('数据分析报告\n')
            f.write('=' * 50 + '\n\n')
            f.write(f'数据行数: {len(self.df)}\n')
            f.write(f'数据列数: {len(self.df.columns)}\n')
            f.write('\n统计摘要:\n')
            f.write(self.df.describe().to_string())

        print(f'报告已保存: {filename}')
```

---

## 5. 完整分析流程
```python
def main():
    """完整分析流程"""
    # 1. 加载数据
    analyzer = MovieAnalyzer('douban_top250.csv')

    # 2. 数据清洗
    analyzer.clean_data()

    # 3. 基本统计
    analyzer.basic_stats()

    # 4. 分布分析
    analyzer.rating_distribution()

    # 5. 创建看板
    analyzer.create_dashboard()

    # 6. 保存报告
    analyzer.save_report()

    print('\n分析完成！')

if __name__ == '__main__':
    main()
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 分析流程 | 问题→获取→清洗→探索→可视化→结论 |
| 多图组合 | subplots、tight_layout |
| 代码封装 | 类封装、模块化 |
| 报告生成 | 保存图片、文本报告 |

---

## 课后练习
完成练习（见 [exercises.md](exercises.md)）

---
