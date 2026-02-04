# Python 进阶 + Golang 入门课程（Day 16-30）

---

# Day 16：requests + BeautifulSoup
> 学习目标：掌握 HTTP 请求基础，学会使用 requests 和 BeautifulSoup 抓取网页数据
>

---

## 1. 爬虫简介
爬虫是自动化获取网页数据的程序。应用场景：
+ 数据采集与分析
+ 价格监控
+ 舆情监测
+ 搜索引擎

**重要原则**：
+ 遵守 robots.txt
+ 控制请求频率
+ 尊重网站版权

---

## 2. requests 基础
```python
import requests

# 发送 GET 请求
response = requests.get('https://httpbin.org/get')
print(response.status_code)  # 状态码
print(response.text)         # 响应文本
print(response.json())       # JSON 解析

# 带参数请求
params = {'name': 'python', 'page': 1}
response = requests.get('https://httpbin.org/get', params=params)
print(response.url)  # 查看完整 URL

# 添加请求头
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
}
response = requests.get('https://httpbin.org/headers', headers=headers)
```

---

## 3. 响应处理
```python
import requests

url = 'https://httpbin.org/get'
response = requests.get(url)

# 常用属性
print(response.status_code)  # HTTP 状态码
print(response.headers)      # 响应头
print(response.encoding)     # 编码
print(response.text)         # 文本内容
print(response.content)      # 二进制内容

# 状态判断
if response.status_code == 200:
    print('请求成功')
elif response.status_code == 404:
    print('页面不存在')
else:
    print(f'其他状态: {response.status_code}')

# 简写
response.raise_for_status()  # 非 200 会抛异常
```

---

## 4. BeautifulSoup 解析 HTML
```python
from bs4 import BeautifulSoup

html = '''
<html>
<head><title>测试页面</title></head>
<body>
    <div class="content">
        <h1 id="title">欢迎学习爬虫</h1>
        <p class="intro">这是一段介绍文字</p>
        <ul>
            <li>Python</li>
            <li>requests</li>
            <li>BeautifulSoup</li>
        </ul>
        <a href="https://example.com">链接</a>
    </div>
</body>
</html>
'''

soup = BeautifulSoup(html, 'html.parser')

# 获取标签
print(soup.title)           # <title>测试页面</title>
print(soup.title.text)      # 测试页面
print(soup.h1)              # <h1 id="title">欢迎学习爬虫</h1>
print(soup.h1['id'])        # title
```

---

## 5. CSS 选择器
```python
from bs4 import BeautifulSoup

# 使用 select 方法
soup = BeautifulSoup(html, 'html.parser')

# 标签选择器
soup.select('h1')           # 所有 h1 标签

# class 选择器
soup.select('.content')     # class="content"
soup.select('.intro')       # class="intro"

# id 选择器
soup.select('#title')       # id="title"

# 层级选择器
soup.select('div h1')       # div 下的 h1
soup.select('ul li')        # ul 下的所有 li

# 获取内容
for li in soup.select('ul li'):
    print(li.text)

# 获取属性
link = soup.select_one('a')
print(link['href'])         # https://example.com
print(link.text)            # 链接
```

---

## 6. find 和 find_all
```python
from bs4 import BeautifulSoup

soup = BeautifulSoup(html, 'html.parser')

# find - 找第一个
h1 = soup.find('h1')
print(h1.text)

# find_all - 找所有
lis = soup.find_all('li')
for li in lis:
    print(li.text)

# 按属性查找
div = soup.find('div', class_='content')
p = soup.find('p', {'class': 'intro'})

# 按 id 查找
title = soup.find(id='title')
```

---

## 7. 完整爬虫示例
```python
import requests
from bs4 import BeautifulSoup

def crawl_quotes():
    """爬取名言网站"""
    url = 'http://quotes.toscrape.com/'
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
    }

    response = requests.get(url, headers=headers)
    response.raise_for_status()

    soup = BeautifulSoup(response.text, 'html.parser')

    quotes = []
    for quote in soup.select('.quote'):
        text = quote.select_one('.text').text
        author = quote.select_one('.author').text
        tags = [tag.text for tag in quote.select('.tag')]

        quotes.append({
            'text': text,
            'author': author,
            'tags': tags
        })

    return quotes

# 运行
if __name__ == '__main__':
    results = crawl_quotes()
    for q in results[:3]:
        print(f"「{q['text'][:50]}...」")
        print(f"  —— {q['author']}")
        print(f"  标签: {', '.join(q['tags'])}")
        print()
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| requests | GET 请求、参数、请求头 |
| 响应处理 | status_code、text、json |
| BeautifulSoup | HTML 解析、创建 soup 对象 |
| 选择器 | select、find、find_all |
| CSS 选择器 | 标签、class、id、层级 |

---

## 课后练习
完成练习题 1-3（见 [phase2_exercises.md](phase2_exercises.md#day-16-练习题requests--beautifulsoup)）

---

# Day 17：翻页爬取 + 数据存储
> 学习目标：掌握翻页爬取技巧，学会将数据保存到 CSV 文件
>

---

## 1. 分析翻页规律
```python
# 常见翻页 URL 规律

# 1. 页码参数
# https://example.com/list?page=1
# https://example.com/list?page=2

# 2. 偏移量参数
# https://example.com/list?offset=0
# https://example.com/list?offset=10

# 3. 路径中的页码
# https://example.com/list/page/1
# https://example.com/list/page/2

# 观察规律，构造 URL
base_url = 'https://example.com/list?page={}'
for page in range(1, 6):
    url = base_url.format(page)
    print(url)
```

---

## 2. 翻页爬取实现
```python
import requests
from bs4 import BeautifulSoup
import time

def crawl_all_pages(max_pages=5):
    """翻页爬取"""
    base_url = 'http://quotes.toscrape.com/page/{}/'
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
    }

    all_quotes = []

    for page in range(1, max_pages + 1):
        url = base_url.format(page)
        print(f'正在爬取第 {page} 页: {url}')

        response = requests.get(url, headers=headers)
        if response.status_code != 200:
            print(f'第 {page} 页请求失败')
            break

        soup = BeautifulSoup(response.text, 'html.parser')
        quotes = soup.select('.quote')

        if not quotes:
            print('没有更多数据了')
            break

        for quote in quotes:
            text = quote.select_one('.text').text
            author = quote.select_one('.author').text
            all_quotes.append({
                'text': text,
                'author': author
            })

        # 重要：延时，避免给服务器压力
        time.sleep(1)

    return all_quotes

# 运行
quotes = crawl_all_pages(3)
print(f'共爬取 {len(quotes)} 条名言')
```

---

## 3. 保存到 CSV
```python
import csv

def save_to_csv(data, filename):
    """保存数据到 CSV 文件"""
    if not data:
        print('没有数据可保存')
        return

    # 获取字段名
    fieldnames = data[0].keys()

    with open(filename, 'w', newline='', encoding='utf-8-sig') as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()  # 写入表头
        writer.writerows(data)  # 写入数据

    print(f'数据已保存到 {filename}')

# 使用
quotes = [
    {'text': '名言1', 'author': '作者1'},
    {'text': '名言2', 'author': '作者2'},
]
save_to_csv(quotes, 'quotes.csv')
```

---

## 4. 读取 CSV
```python
import csv

def read_csv(filename):
    """读取 CSV 文件"""
    data = []
    with open(filename, 'r', encoding='utf-8-sig') as f:
        reader = csv.DictReader(f)
        for row in reader:
            data.append(dict(row))
    return data

# 使用
quotes = read_csv('quotes.csv')
for q in quotes:
    print(q)
```

---

## 5. 完整爬虫 + 存储
```python
import requests
from bs4 import BeautifulSoup
import csv
import time

class QuotesCrawler:
    def __init__(self):
        self.base_url = 'http://quotes.toscrape.com/page/{}/'
        self.headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
        }
        self.data = []

    def crawl_page(self, page):
        """爬取单页"""
        url = self.base_url.format(page)
        response = requests.get(url, headers=self.headers)

        if response.status_code != 200:
            return False

        soup = BeautifulSoup(response.text, 'html.parser')
        quotes = soup.select('.quote')

        if not quotes:
            return False

        for quote in quotes:
            self.data.append({
                'text': quote.select_one('.text').text.strip('""'),
                'author': quote.select_one('.author').text,
                'tags': ','.join([t.text for t in quote.select('.tag')])
            })

        return True

    def crawl_all(self, max_pages=10):
        """爬取所有页"""
        for page in range(1, max_pages + 1):
            print(f'爬取第 {page} 页...')
            if not self.crawl_page(page):
                break
            time.sleep(1)

        print(f'共爬取 {len(self.data)} 条数据')

    def save(self, filename='quotes.csv'):
        """保存数据"""
        if not self.data:
            return

        with open(filename, 'w', newline='', encoding='utf-8-sig') as f:
            writer = csv.DictWriter(f, fieldnames=self.data[0].keys())
            writer.writeheader()
            writer.writerows(self.data)

        print(f'保存到 {filename}')

# 使用
if __name__ == '__main__':
    crawler = QuotesCrawler()
    crawler.crawl_all(5)
    crawler.save('quotes.csv')
```

---

## 6. 增量爬取
```python
import os
import csv

def load_existing_data(filename):
    """加载已有数据"""
    if not os.path.exists(filename):
        return set()

    existing = set()
    with open(filename, 'r', encoding='utf-8-sig') as f:
        reader = csv.DictReader(f)
        for row in reader:
            # 用某个唯一字段作为标识
            existing.add(row['text'])
    return existing

def save_new_data(new_data, filename):
    """追加新数据"""
    file_exists = os.path.exists(filename)

    with open(filename, 'a', newline='', encoding='utf-8-sig') as f:
        writer = csv.DictWriter(f, fieldnames=new_data[0].keys())
        if not file_exists:
            writer.writeheader()
        writer.writerows(new_data)

# 使用
existing = load_existing_data('quotes.csv')
new_quotes = []

for quote in crawled_quotes:
    if quote['text'] not in existing:
        new_quotes.append(quote)

if new_quotes:
    save_new_data(new_quotes, 'quotes.csv')
    print(f'新增 {len(new_quotes)} 条数据')
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 翻页规律 | page 参数、offset 参数、路径页码 |
| 延时 | time.sleep() 控制频率 |
| CSV 写入 | DictWriter、writeheader、writerows |
| CSV 读取 | DictReader |
| 增量爬取 | 去重、追加写入 |

---

## 课后练习
完成练习题 4-6（见 [phase2_exercises.md](phase2_exercises.md#day-17-练习题翻页爬取--数据存储)）

---

# Day 18：爬虫实战 - 豆瓣电影 Top250
> 学习目标：完成一个完整的爬虫项目，学会处理常见反爬措施
>

---

## 1. 分析目标网站
```
目标：豆瓣电影 Top250
网址：https://movie.douban.com/top250

分析步骤：
1. 打开网页，查看结构
2. F12 打开开发者工具
3. 查看网页源码，找到数据位置
4. 分析翻页规律
   - 第1页: ?start=0
   - 第2页: ?start=25
   - 第3页: ?start=50

需要提取：
- 电影名称
- 评分
- 评价人数
- 简介
```

---

## 2. 反爬处理
```python
import requests
import random
import time

# 1. 随机 User-Agent
USER_AGENTS = [
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36',
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:89.0) Gecko/20100101 Firefox/89.0',
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.114 Safari/537.36',
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.1 Safari/605.1.15',
]

def get_random_headers():
    return {
        'User-Agent': random.choice(USER_AGENTS),
        'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
        'Accept-Language': 'zh-CN,zh;q=0.9,en;q=0.8',
    }

# 2. 随机延时
def random_sleep():
    time.sleep(random.uniform(1, 3))

# 3. 重试机制
def request_with_retry(url, max_retries=3):
    for i in range(max_retries):
        try:
            response = requests.get(url, headers=get_random_headers(), timeout=10)
            if response.status_code == 200:
                return response
            print(f'状态码 {response.status_code}，重试...')
        except Exception as e:
            print(f'请求失败: {e}，重试...')

        time.sleep(2 ** i)  # 指数退避

    return None
```

---

## 3. 豆瓣爬虫实现
```python
import requests
from bs4 import BeautifulSoup
import csv
import time
import random
import re

class DoubanMovieCrawler:
    def __init__(self):
        self.base_url = 'https://movie.douban.com/top250'
        self.movies = []

    def get_headers(self):
        return {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36',
            'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
            'Accept-Language': 'zh-CN,zh;q=0.9',
            'Referer': 'https://movie.douban.com/',
        }

    def parse_page(self, html):
        """解析单页数据"""
        soup = BeautifulSoup(html, 'html.parser')
        items = soup.select('.item')

        for item in items:
            # 排名
            rank = item.select_one('.pic em').text

            # 标题
            title = item.select_one('.title').text

            # 评分
            rating = item.select_one('.rating_num').text

            # 评价人数
            star_span = item.select_one('.star span:last-child')
            num_text = star_span.text if star_span else '0人评价'
            num = re.search(r'(\d+)', num_text)
            rating_num = num.group(1) if num else '0'

            # 简介（可能没有）
            quote = item.select_one('.inq')
            intro = quote.text if quote else ''

            self.movies.append({
                'rank': rank,
                'title': title,
                'rating': rating,
                'rating_num': rating_num,
                'intro': intro
            })

    def crawl(self):
        """爬取所有页"""
        for start in range(0, 250, 25):
            url = f'{self.base_url}?start={start}'
            print(f'正在爬取: {url}')

            try:
                response = requests.get(url, headers=self.get_headers(), timeout=10)
                response.raise_for_status()
                self.parse_page(response.text)
                print(f'  已获取 {len(self.movies)} 部电影')
            except Exception as e:
                print(f'  爬取失败: {e}')

            # 随机延时
            time.sleep(random.uniform(2, 4))

        return self.movies

    def save_csv(self, filename='douban_top250.csv'):
        """保存到 CSV"""
        if not self.movies:
            print('没有数据')
            return

        with open(filename, 'w', newline='', encoding='utf-8-sig') as f:
            writer = csv.DictWriter(f, fieldnames=self.movies[0].keys())
            writer.writeheader()
            writer.writerows(self.movies)

        print(f'保存成功: {filename}')

# 运行
if __name__ == '__main__':
    crawler = DoubanMovieCrawler()
    movies = crawler.crawl()
    crawler.save_csv()

    # 打印前5部
    print('\n前5部电影:')
    for m in movies[:5]:
        print(f"{m['rank']}. {m['title']} - {m['rating']}分")
```

---

## 4. 异常处理增强
```python
import requests
from requests.exceptions import RequestException, Timeout, ConnectionError

def safe_request(url, headers, max_retries=3):
    """安全的请求函数"""
    for attempt in range(max_retries):
        try:
            response = requests.get(
                url,
                headers=headers,
                timeout=10
            )
            response.raise_for_status()
            return response

        except Timeout:
            print(f'超时，第 {attempt + 1} 次重试')
        except ConnectionError:
            print(f'连接错误，第 {attempt + 1} 次重试')
        except RequestException as e:
            print(f'请求异常: {e}')
            return None

        time.sleep(2 ** attempt)

    return None
```

---

## 5. 使用代理（了解）
```python
# 代理设置示例（实际使用需要有效代理）
proxies = {
    'http': 'http://127.0.0.1:7890',
    'https': 'http://127.0.0.1:7890',
}

response = requests.get(url, headers=headers, proxies=proxies)

# 代理池示例
proxy_pool = [
    'http://ip1:port',
    'http://ip2:port',
    'http://ip3:port',
]

def get_random_proxy():
    return {
        'http': random.choice(proxy_pool),
        'https': random.choice(proxy_pool),
    }
```

---

## 6. 爬虫伦理
```
爬虫注意事项：

1. 遵守 robots.txt
   - 查看 https://example.com/robots.txt
   - 尊重网站的爬取规则

2. 控制频率
   - 不要太快，模拟人类行为
   - 使用延时

3. 设置合理 User-Agent
   - 不要用默认的 python-requests

4. 处理好异常
   - 重试机制
   - 错误日志

5. 数据使用
   - 仅用于学习和研究
   - 不要用于商业用途
   - 不要侵犯版权
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 网页分析 | F12 开发者工具、查看源码 |
| 反爬处理 | User-Agent、延时、重试 |
| 异常处理 | try-except、重试机制 |
| 数据提取 | 正则表达式辅助 |
| 爬虫伦理 | robots.txt、频率控制 |

---

## 课后练习
完成练习题 7-9（见 [phase2_exercises.md](phase2_exercises.md#day-18-练习题爬虫实战)）

---

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
完成练习题 13-15（见 [phase2_exercises.md](phase2_exercises.md#day-20-练习题数据清洗与处理)）

---

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
完成练习题 16-18（见 [phase2_exercises.md](phase2_exercises.md#day-21-练习题数据统计与分组)）

---

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
完成练习题 22-23（见 [phase2_exercises.md](phase2_exercises.md#day-23-练习题完整数据分析报告)）

---

# Day 24-25：独立数据分析项目
> 学习目标：独立完成一个完整的数据分析项目
>

---

## 1. 项目要求
```
独立完成以下任务：

1. 选择数据源（二选一）
   - 使用 Day 18 爬取的豆瓣数据
   - 使用提供的 CSV 数据集

2. 数据清洗
   - 处理缺失值
   - 处理异常值
   - 数据类型转换

3. 数据分析
   - 基本统计分析
   - 分组统计
   - 相关性分析

4. 数据可视化
   - 至少 4 种不同图表
   - 组合成数据看板

5. 分析报告
   - 分析目标
   - 分析过程
   - 主要发现
   - 结论建议
```

---

## 2. 项目结构
```
project/
├── data/
│   └── movies.csv        # 数据文件
├── output/
│   ├── dashboard.png     # 数据看板
│   └── report.txt        # 分析报告
├── analysis.py           # 分析代码
└── README.md             # 项目说明
```

---

## 3. 代码模板
```python
"""
数据分析项目模板
"""
import pandas as pd
import matplotlib.pyplot as plt
import os

# 创建输出目录
os.makedirs('output', exist_ok=True)

# 设置中文
plt.rcParams['font.sans-serif'] = ['SimHei', 'Arial Unicode MS']
plt.rcParams['axes.unicode_minus'] = False


def load_data(filename):
    """加载数据"""
    # TODO: 实现数据加载
    pass


def clean_data(df):
    """数据清洗"""
    # TODO: 实现数据清洗
    pass


def analyze_data(df):
    """数据分析"""
    # TODO: 实现数据分析
    pass


def create_visualizations(df):
    """创建可视化"""
    # TODO: 实现可视化
    pass


def generate_report(df, findings):
    """生成报告"""
    # TODO: 实现报告生成
    pass


def main():
    """主函数"""
    # 1. 加载数据
    df = load_data('data/movies.csv')

    # 2. 数据清洗
    df = clean_data(df)

    # 3. 数据分析
    findings = analyze_data(df)

    # 4. 可视化
    create_visualizations(df)

    # 5. 生成报告
    generate_report(df, findings)

    print('项目完成！')


if __name__ == '__main__':
    main()
```

---

## 4. 评分标准
```
项目评分标准（总分 100 分）

1. 数据清洗（20分）
   - 正确处理缺失值
   - 正确处理异常值
   - 数据类型转换合理

2. 数据分析（30分）
   - 基本统计完整
   - 分组统计合理
   - 发现有价值的信息

3. 数据可视化（30分）
   - 图表类型选择合理
   - 图表美观清晰
   - 组合成完整看板

4. 代码质量（10分）
   - 代码规范
   - 注释清晰
   - 结构合理

5. 报告质量（10分）
   - 结论有据可依
   - 表达清晰
```

---

## 5. 参考数据集
```python
# 如果没有爬取数据，可以使用模拟数据
import pandas as pd
import numpy as np

np.random.seed(42)

# 生成模拟电影数据
n = 250
movies = pd.DataFrame({
    'rank': range(1, n + 1),
    'title': [f'电影{i}' for i in range(1, n + 1)],
    'rating': np.round(np.random.uniform(8.0, 9.8, n), 1),
    'rating_num': np.random.randint(10000, 2000000, n),
    'year': np.random.randint(1950, 2024, n),
    'genre': np.random.choice(['剧情', '喜剧', '动作', '科幻', '爱情'], n),
    'country': np.random.choice(['美国', '中国', '日本', '韩国', '英国', '法国'], n)
})

movies.to_csv('data/movies.csv', index=False)
print('模拟数据已生成')
```

---

## 今日任务
- Day 24：完成数据加载、清洗和基本分析
- Day 25：完成可视化和分析报告

---

## 课后练习
完成练习题 24-25（见 [phase2_exercises.md](phase2_exercises.md#day-24-25-练习题独立数据分析项目)）

---

# Day 26：Go 环境 + 基础语法
> 学习目标：搭建 Go 开发环境，掌握基础语法
>

---

## 1. Go 语言简介
```
Go 语言特点：
- Google 开发，2009 年发布
- 编译型语言，性能接近 C
- 语法简洁，上手快
- 原生支持并发（goroutine）
- 丰富的标准库
- 跨平台编译

适用场景：
- 后端服务
- 微服务
- 命令行工具
- 云原生应用
```

---

## 2. 环境搭建
```bash
# 1. 下载安装
# 访问 https://go.dev/dl/ 下载对应系统版本

# 2. 验证安装
go version
# go version go1.21.0 linux/amd64

# 3. 设置环境变量（Linux/Mac）
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin

# 4. 创建项目
mkdir hello && cd hello
go mod init hello

# 5. 创建文件 main.go
# 6. 运行
go run main.go
```

---

## 3. Hello World
```go
// main.go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

```bash
# 运行
go run main.go

# 编译
go build -o hello
./hello
```

---

## 4. 变量和常量
```go
package main

import "fmt"

func main() {
    // 变量声明方式
    var a int = 10
    var b = 20        // 类型推断
    c := 30           // 简短声明（最常用）

    // 多变量声明
    var x, y int = 1, 2
    m, n := 3, 4

    // 常量
    const PI = 3.14159
    const (
        StatusOK = 200
        StatusNotFound = 404
    )

    fmt.Println(a, b, c)
    fmt.Println(x, y, m, n)
    fmt.Println(PI, StatusOK)
}
```

---

## 5. 基本数据类型
```go
package main

import "fmt"

func main() {
    // 整数
    var i int = 42
    var i8 int8 = 127
    var i64 int64 = 9223372036854775807

    // 无符号整数
    var u uint = 42
    var u8 uint8 = 255

    // 浮点数
    var f32 float32 = 3.14
    var f64 float64 = 3.141592653589793

    // 布尔
    var b bool = true

    // 字符串
    var s string = "Hello"

    // 类型转换（必须显式）
    var x int = 10
    var y float64 = float64(x)

    fmt.Printf("int: %d\n", i)
    fmt.Printf("float: %f\n", f64)
    fmt.Printf("bool: %t\n", b)
    fmt.Printf("string: %s\n", s)
    fmt.Printf("转换后: %f\n", y)
}
```

---

## 6. 条件语句
```go
package main

import "fmt"

func main() {
    age := 20

    // if 语句
    if age >= 18 {
        fmt.Println("成年人")
    } else {
        fmt.Println("未成年")
    }

    // if-else if-else
    score := 85
    if score >= 90 {
        fmt.Println("优秀")
    } else if score >= 80 {
        fmt.Println("良好")
    } else if score >= 60 {
        fmt.Println("及格")
    } else {
        fmt.Println("不及格")
    }

    // if 带初始化语句
    if num := 10; num > 5 {
        fmt.Println("num 大于 5")
    }

    // switch
    day := 3
    switch day {
    case 1:
        fmt.Println("周一")
    case 2:
        fmt.Println("周二")
    case 3:
        fmt.Println("周三")
    default:
        fmt.Println("其他")
    }
}
```

---

## 7. 循环
```go
package main

import "fmt"

func main() {
    // 基本 for 循环
    for i := 0; i < 5; i++ {
        fmt.Print(i, " ")
    }
    fmt.Println()

    // while 风格
    j := 0
    for j < 5 {
        fmt.Print(j, " ")
        j++
    }
    fmt.Println()

    // 无限循环
    // for {
    //     fmt.Println("无限")
    // }

    // range 遍历
    nums := []int{1, 2, 3, 4, 5}
    for index, value := range nums {
        fmt.Printf("索引: %d, 值: %d\n", index, value)
    }

    // 只要值
    for _, v := range nums {
        fmt.Print(v, " ")
    }
    fmt.Println()

    // break 和 continue
    for i := 0; i < 10; i++ {
        if i == 3 {
            continue  // 跳过 3
        }
        if i == 7 {
            break     // 到 7 停止
        }
        fmt.Print(i, " ")
    }
}
```

---

## 8. 切片（Slice）
```go
package main

import "fmt"

func main() {
    // 创建切片
    var s1 []int                    // 声明
    s2 := []int{1, 2, 3}            // 字面量
    s3 := make([]int, 5)            // make 创建，长度 5
    s4 := make([]int, 3, 10)        // 长度 3，容量 10

    fmt.Println(s1, s2, s3, s4)

    // 追加元素
    s1 = append(s1, 1, 2, 3)
    fmt.Println(s1)

    // 切片操作
    nums := []int{0, 1, 2, 3, 4, 5}
    fmt.Println(nums[1:4])   // [1 2 3]
    fmt.Println(nums[:3])    // [0 1 2]
    fmt.Println(nums[3:])    // [3 4 5]

    // 长度和容量
    fmt.Printf("长度: %d, 容量: %d\n", len(nums), cap(nums))

    // 遍历
    for i, v := range nums {
        fmt.Printf("索引 %d: %d\n", i, v)
    }
}
```

---

## 9. Map
```go
package main

import "fmt"

func main() {
    // 创建 map
    m1 := make(map[string]int)
    m2 := map[string]int{
        "apple":  5,
        "banana": 3,
    }

    // 添加/修改元素
    m1["one"] = 1
    m1["two"] = 2

    // 获取元素
    fmt.Println(m2["apple"])

    // 检查键是否存在
    value, exists := m2["orange"]
    if exists {
        fmt.Println("orange:", value)
    } else {
        fmt.Println("orange 不存在")
    }

    // 删除元素
    delete(m2, "apple")

    // 遍历
    for key, value := range m2 {
        fmt.Printf("%s: %d\n", key, value)
    }

    // 长度
    fmt.Println("长度:", len(m2))
}
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 环境 | go mod init、go run、go build |
| 变量 | var、:=、const |
| 类型 | int、float64、bool、string |
| 条件 | if、switch |
| 循环 | for、range |
| 切片 | []T、append、切片操作 |
| Map | map[K]V、make |

---

## 课后练习
完成练习题 26-28（见 [phase2_exercises.md](phase2_exercises.md#day-26-练习题go-环境--基础语法)）

---

# Day 27：Go 函数 + Struct
> 学习目标：掌握 Go 函数定义和结构体使用
>

---

## 1. 函数基础
```go
package main

import "fmt"

// 基本函数
func greet() {
    fmt.Println("Hello!")
}

// 带参数
func add(a int, b int) int {
    return a + b
}

// 参数类型简写
func multiply(a, b int) int {
    return a * b
}

// 多返回值
func divide(a, b int) (int, int) {
    return a / b, a % b
}

// 命名返回值
func calculate(a, b int) (sum int, diff int) {
    sum = a + b
    diff = a - b
    return  // 自动返回命名的变量
}

func main() {
    greet()
    fmt.Println(add(3, 5))
    fmt.Println(multiply(4, 6))

    quotient, remainder := divide(10, 3)
    fmt.Printf("商: %d, 余数: %d\n", quotient, remainder)

    s, d := calculate(10, 3)
    fmt.Printf("和: %d, 差: %d\n", s, d)
}
```

---

## 2. 可变参数
```go
package main

import "fmt"

// 可变参数
func sum(nums ...int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

// 混合参数
func printf(format string, args ...interface{}) {
    fmt.Printf(format, args...)
}

func main() {
    fmt.Println(sum(1, 2, 3))
    fmt.Println(sum(1, 2, 3, 4, 5))

    // 切片展开
    nums := []int{1, 2, 3, 4, 5}
    fmt.Println(sum(nums...))

    printf("Name: %s, Age: %d\n", "Tom", 20)
}
```

---

## 3. 匿名函数和闭包
```go
package main

import "fmt"

func main() {
    // 匿名函数
    add := func(a, b int) int {
        return a + b
    }
    fmt.Println(add(3, 5))

    // 立即执行
    func(msg string) {
        fmt.Println(msg)
    }("Hello!")

    // 闭包
    counter := func() func() int {
        count := 0
        return func() int {
            count++
            return count
        }
    }()

    fmt.Println(counter())  // 1
    fmt.Println(counter())  // 2
    fmt.Println(counter())  // 3
}
```

---

## 4. 结构体基础
```go
package main

import "fmt"

// 定义结构体
type Person struct {
    Name string
    Age  int
    City string
}

func main() {
    // 创建结构体
    p1 := Person{"张三", 20, "北京"}
    p2 := Person{Name: "李四", Age: 21, City: "上海"}
    p3 := Person{Name: "王五"}  // 其他字段为零值

    // 指针
    p4 := &Person{Name: "赵六", Age: 22}

    // 访问字段
    fmt.Println(p1.Name, p1.Age)
    fmt.Println(p2)
    fmt.Println(p3)
    fmt.Println(p4.Name)  // Go 自动解引用

    // 修改字段
    p1.Age = 21
    fmt.Println(p1)
}
```

---

## 5. 结构体方法
```go
package main

import "fmt"

type Rectangle struct {
    Width  float64
    Height float64
}

// 值接收者方法
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

// 指针接收者方法（可修改结构体）
func (r *Rectangle) Scale(factor float64) {
    r.Width *= factor
    r.Height *= factor
}

// 构造函数（Go 没有构造函数，用普通函数代替）
func NewRectangle(w, h float64) *Rectangle {
    return &Rectangle{Width: w, Height: h}
}

func main() {
    rect := NewRectangle(10, 5)
    fmt.Printf("面积: %.2f\n", rect.Area())

    rect.Scale(2)
    fmt.Printf("缩放后: %+v\n", rect)
    fmt.Printf("新面积: %.2f\n", rect.Area())
}
```

---

## 6. 结构体嵌套
```go
package main

import "fmt"

type Address struct {
    City    string
    Street  string
    ZipCode string
}

type Employee struct {
    Name    string
    Age     int
    Address Address  // 嵌套结构体
}

// 匿名嵌套（组合）
type Manager struct {
    Employee        // 匿名嵌套
    Department string
}

func main() {
    emp := Employee{
        Name: "张三",
        Age:  30,
        Address: Address{
            City:    "北京",
            Street:  "长安街",
            ZipCode: "100000",
        },
    }
    fmt.Printf("员工: %s, 城市: %s\n", emp.Name, emp.Address.City)

    mgr := Manager{
        Employee: Employee{
            Name: "李四",
            Age:  40,
            Address: Address{City: "上海"},
        },
        Department: "技术部",
    }
    // 匿名嵌套可以直接访问
    fmt.Printf("经理: %s, 部门: %s, 城市: %s\n",
               mgr.Name, mgr.Department, mgr.City)
}
```

---

## 7. 接口基础
```go
package main

import (
    "fmt"
    "math"
)

// 定义接口
type Shape interface {
    Area() float64
}

// 实现接口（隐式）
type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

// 使用接口
func printArea(s Shape) {
    fmt.Printf("面积: %.2f\n", s.Area())
}

func main() {
    c := Circle{Radius: 5}
    r := Rectangle{Width: 10, Height: 5}

    printArea(c)
    printArea(r)

    // 接口切片
    shapes := []Shape{c, r}
    for _, s := range shapes {
        printArea(s)
    }
}
```

---

## 8. 错误处理
```go
package main

import (
    "errors"
    "fmt"
)

// 返回错误
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("除数不能为零")
    }
    return a / b, nil
}

// 自定义错误
type ValidationError struct {
    Field   string
    Message string
}

func (e ValidationError) Error() string {
    return fmt.Sprintf("%s: %s", e.Field, e.Message)
}

func validateAge(age int) error {
    if age < 0 {
        return ValidationError{Field: "age", Message: "不能为负数"}
    }
    if age > 150 {
        return ValidationError{Field: "age", Message: "不能超过150"}
    }
    return nil
}

func main() {
    // 处理错误
    result, err := divide(10, 0)
    if err != nil {
        fmt.Println("错误:", err)
    } else {
        fmt.Println("结果:", result)
    }

    // 自定义错误
    if err := validateAge(-5); err != nil {
        fmt.Println("验证失败:", err)
    }
}
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 函数 | 多返回值、命名返回值 |
| 可变参数 | ...T |
| 闭包 | 匿名函数、状态保持 |
| 结构体 | type struct |
| 方法 | 值接收者、指针接收者 |
| 接口 | 隐式实现 |
| 错误处理 | error 接口 |

---

## 课后练习
完成练习题 29-31（见 [phase2_exercises.md](phase2_exercises.md#day-27-练习题go-函数--struct)）

---

# Day 28：Gin 框架写 API
> 学习目标：使用 Gin 框架搭建 RESTful API
>

---

## 1. Gin 简介
```
Gin 框架特点：
- 高性能 HTTP Web 框架
- API 友好
- 中间件支持
- 路由分组
- JSON 验证
- 错误处理

安装：
go get -u github.com/gin-gonic/gin
```

---

## 2. Hello Gin
```go
package main

import "github.com/gin-gonic/gin"

func main() {
    // 创建引擎
    r := gin.Default()

    // 定义路由
    r.GET("/", func(c *gin.Context) {
        c.JSON(200, gin.H{
            "message": "Hello, Gin!",
        })
    })

    // 启动服务
    r.Run(":8080")  // 监听 8080 端口
}
```

```bash
# 运行
go run main.go

# 测试
curl http://localhost:8080
```

---

## 3. 路由和参数
```go
package main

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default()

    // 路径参数
    r.GET("/users/:id", func(c *gin.Context) {
        id := c.Param("id")
        c.JSON(http.StatusOK, gin.H{
            "id": id,
        })
    })

    // 查询参数
    r.GET("/search", func(c *gin.Context) {
        keyword := c.Query("keyword")
        page := c.DefaultQuery("page", "1")
        c.JSON(http.StatusOK, gin.H{
            "keyword": keyword,
            "page":    page,
        })
    })

    // POST 表单
    r.POST("/login", func(c *gin.Context) {
        username := c.PostForm("username")
        password := c.PostForm("password")
        c.JSON(http.StatusOK, gin.H{
            "username": username,
            "password": password,
        })
    })

    r.Run(":8080")
}
```

---

## 4. JSON 处理
```go
package main

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

// 请求结构体
type CreateUserRequest struct {
    Name  string `json:"name" binding:"required"`
    Email string `json:"email" binding:"required,email"`
    Age   int    `json:"age" binding:"gte=0,lte=150"`
}

// 响应结构体
type User struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
    Age   int    `json:"age"`
}

func main() {
    r := gin.Default()

    // 接收 JSON
    r.POST("/users", func(c *gin.Context) {
        var req CreateUserRequest

        // 绑定并验证
        if err := c.ShouldBindJSON(&req); err != nil {
            c.JSON(http.StatusBadRequest, gin.H{
                "error": err.Error(),
            })
            return
        }

        // 创建用户（模拟）
        user := User{
            ID:    1,
            Name:  req.Name,
            Email: req.Email,
            Age:   req.Age,
        }

        c.JSON(http.StatusCreated, user)
    })

    r.Run(":8080")
}
```

---

## 5. RESTful API 示例
```go
package main

import (
    "net/http"
    "strconv"
    "github.com/gin-gonic/gin"
)

// 用户模型
type User struct {
    ID   int    `json:"id"`
    Name string `json:"name"`
    Age  int    `json:"age"`
}

// 模拟数据库
var users = []User{
    {ID: 1, Name: "张三", Age: 20},
    {ID: 2, Name: "李四", Age: 21},
}
var nextID = 3

func main() {
    r := gin.Default()

    // 获取所有用户
    r.GET("/users", func(c *gin.Context) {
        c.JSON(http.StatusOK, gin.H{
            "data": users,
        })
    })

    // 获取单个用户
    r.GET("/users/:id", func(c *gin.Context) {
        id, _ := strconv.Atoi(c.Param("id"))
        for _, u := range users {
            if u.ID == id {
                c.JSON(http.StatusOK, gin.H{"data": u})
                return
            }
        }
        c.JSON(http.StatusNotFound, gin.H{"error": "用户不存在"})
    })

    // 创建用户
    r.POST("/users", func(c *gin.Context) {
        var user User
        if err := c.ShouldBindJSON(&user); err != nil {
            c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
            return
        }
        user.ID = nextID
        nextID++
        users = append(users, user)
        c.JSON(http.StatusCreated, gin.H{"data": user})
    })

    // 更新用户
    r.PUT("/users/:id", func(c *gin.Context) {
        id, _ := strconv.Atoi(c.Param("id"))
        var input User
        if err := c.ShouldBindJSON(&input); err != nil {
            c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
            return
        }
        for i, u := range users {
            if u.ID == id {
                users[i].Name = input.Name
                users[i].Age = input.Age
                c.JSON(http.StatusOK, gin.H{"data": users[i]})
                return
            }
        }
        c.JSON(http.StatusNotFound, gin.H{"error": "用户不存在"})
    })

    // 删除用户
    r.DELETE("/users/:id", func(c *gin.Context) {
        id, _ := strconv.Atoi(c.Param("id"))
        for i, u := range users {
            if u.ID == id {
                users = append(users[:i], users[i+1:]...)
                c.JSON(http.StatusOK, gin.H{"message": "删除成功"})
                return
            }
        }
        c.JSON(http.StatusNotFound, gin.H{"error": "用户不存在"})
    })

    r.Run(":8080")
}
```

---

## 6. 路由分组
```go
package main

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default()

    // API v1 分组
    v1 := r.Group("/api/v1")
    {
        v1.GET("/users", getUsers)
        v1.POST("/users", createUser)

        // 嵌套分组
        admin := v1.Group("/admin")
        {
            admin.GET("/stats", getStats)
        }
    }

    // API v2 分组
    v2 := r.Group("/api/v2")
    {
        v2.GET("/users", getUsersV2)
    }

    r.Run(":8080")
}

func getUsers(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{"version": "v1", "data": []string{}})
}

func createUser(c *gin.Context) {
    c.JSON(http.StatusCreated, gin.H{"message": "created"})
}

func getStats(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{"stats": "admin stats"})
}

func getUsersV2(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{"version": "v2", "data": []string{}})
}
```

---

## 7. 中间件
```go
package main

import (
    "fmt"
    "net/http"
    "time"
    "github.com/gin-gonic/gin"
)

// 日志中间件
func Logger() gin.HandlerFunc {
    return func(c *gin.Context) {
        start := time.Now()
        path := c.Request.URL.Path

        // 处理请求
        c.Next()

        // 记录日志
        latency := time.Since(start)
        status := c.Writer.Status()
        fmt.Printf("[%s] %s %d %v\n",
                   c.Request.Method, path, status, latency)
    }
}

// 认证中间件
func Auth() gin.HandlerFunc {
    return func(c *gin.Context) {
        token := c.GetHeader("Authorization")
        if token == "" {
            c.JSON(http.StatusUnauthorized, gin.H{
                "error": "未授权",
            })
            c.Abort()  // 中止后续处理
            return
        }

        // 设置用户信息
        c.Set("userID", 1)
        c.Next()
    }
}

func main() {
    r := gin.New()  // 不使用默认中间件

    // 全局中间件
    r.Use(Logger())

    // 公开路由
    r.GET("/", func(c *gin.Context) {
        c.JSON(http.StatusOK, gin.H{"message": "public"})
    })

    // 需要认证的路由
    auth := r.Group("/api")
    auth.Use(Auth())
    {
        auth.GET("/profile", func(c *gin.Context) {
            userID := c.GetInt("userID")
            c.JSON(http.StatusOK, gin.H{
                "userID": userID,
            })
        })
    }

    r.Run(":8080")
}
```

---

## 8. 统一响应格式
```go
package main

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

// 统一响应结构
type Response struct {
    Code    int         `json:"code"`
    Message string      `json:"message"`
    Data    interface{} `json:"data,omitempty"`
}

// 成功响应
func Success(c *gin.Context, data interface{}) {
    c.JSON(http.StatusOK, Response{
        Code:    0,
        Message: "success",
        Data:    data,
    })
}

// 错误响应
func Error(c *gin.Context, code int, message string) {
    c.JSON(http.StatusOK, Response{
        Code:    code,
        Message: message,
    })
}

func main() {
    r := gin.Default()

    r.GET("/users", func(c *gin.Context) {
        users := []map[string]interface{}{
            {"id": 1, "name": "张三"},
            {"id": 2, "name": "李四"},
        }
        Success(c, users)
    })

    r.GET("/users/:id", func(c *gin.Context) {
        id := c.Param("id")
        if id == "0" {
            Error(c, 404, "用户不存在")
            return
        }
        Success(c, map[string]interface{}{
            "id":   id,
            "name": "张三",
        })
    })

    r.Run(":8080")
}
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 基础 | gin.Default()、r.Run() |
| 路由 | GET/POST/PUT/DELETE |
| 参数 | Param、Query、PostForm |
| JSON | ShouldBindJSON、c.JSON |
| 分组 | r.Group |
| 中间件 | Use、Next、Abort |

---

## 课后练习
完成练习题 32-34（见 [phase2_exercises.md](phase2_exercises.md#day-28-练习题gin-框架写-api)）

---

# Day 29：项目整理
> 学习目标：整理项目代码，完善文档，提交到 GitHub
>

---

## 1. 项目检查清单
```
代码检查：
[ ] 代码能正常运行
[ ] 没有语法错误
[ ] 没有未使用的导入
[ ] 变量命名规范
[ ] 函数有适当注释

文件检查：
[ ] 项目结构清晰
[ ] 不包含敏感信息（密码、密钥）
[ ] 不包含临时文件
[ ] .gitignore 配置正确

文档检查：
[ ] README.md 完整
[ ] 有安装说明
[ ] 有使用说明
[ ] 有 API 文档（如果是后端项目）
```

---

## 2. README 模板
```markdown
# 项目名称

简短描述项目功能

## 功能特点

- 功能1
- 功能2
- 功能3

## 技术栈

- Python 3.x
- Flask / pandas / ...

## 安装

```bash
# 克隆项目
git clone https://github.com/username/project.git
cd project

# 安装依赖
pip install -r requirements.txt
```

## 使用

```bash
# 运行
python main.py
```

## 项目结构

```
project/
├── app/
│   ├── __init__.py
│   └── main.py
├── data/
├── requirements.txt
└── README.md
```

## API 文档

### 获取用户列表
```
GET /api/users
```

响应：
```json
{
  "code": 0,
  "data": [...]
}
```

## 作者

- 姓名 - email@example.com

## 许可证

MIT
```

---

## 3. .gitignore 配置
```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
.venv/

# IDE
.idea/
.vscode/
*.swp
*.swo

# 环境变量
.env
.env.local

# 数据库
*.db
*.sqlite

# 日志
*.log

# 系统文件
.DS_Store
Thumbs.db

# 临时文件
*.tmp
*.temp
```

---

## 4. 代码规范检查
```bash
# Python 代码检查
pip install flake8
flake8 app/

# 代码格式化
pip install black
black app/

# Go 代码格式化
go fmt ./...

# Go 代码检查
go vet ./...
```

---

## 5. Git 提交规范
```bash
# 提交信息格式
# <type>: <description>

# type 类型：
# feat: 新功能
# fix: 修复bug
# docs: 文档更新
# style: 代码格式
# refactor: 重构
# test: 测试
# chore: 构建/工具

# 示例
git commit -m "feat: 添加用户登录功能"
git commit -m "fix: 修复分页参数错误"
git commit -m "docs: 更新 README"
```

---

## 6. 提交到 GitHub
```bash
# 1. 检查状态
git status

# 2. 添加文件
git add .

# 3. 提交
git commit -m "完成项目开发"

# 4. 推送
git push origin main

# 如果是新仓库
git remote add origin https://github.com/username/project.git
git branch -M main
git push -u origin main
```

---

## 7. 项目展示准备
```
准备内容：
1. 项目截图/演示 GIF
2. 核心功能说明
3. 技术难点和解决方案
4. 学到了什么

展示要点：
- 项目背景和目标
- 主要功能演示
- 代码亮点
- 遇到的问题和解决方法
- 后续改进计划
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 代码检查 | flake8、black、go fmt |
| 文档 | README.md 规范 |
| Git | .gitignore、提交规范 |
| 展示 | 截图、演示、总结 |

---

## 课后练习
完成练习题 35-36（见 [phase2_exercises.md](phase2_exercises.md#day-29-练习题项目整理)）

---

# Day 30：交付自学资料 + 总结
> 学习目标：回顾学习内容，获取自学资源，规划后续学习
>

---

## 1. 30 天学习回顾
```
第一阶段（Day 1-15）：Python 后端基础
- Python 基础语法
- Flask Web 框架
- MySQL + SQLAlchemy
- JWT 认证
- RESTful API

第二阶段（Day 16-18）：爬虫入门
- requests + BeautifulSoup
- 翻页爬取
- 数据存储

第三阶段（Day 19-25）：数据分析
- pandas 数据处理
- matplotlib 可视化
- 完整数据分析项目

第四阶段（Day 26-28）：Go 入门
- Go 基础语法
- 结构体和方法
- Gin 框架

第五阶段（Day 29-30）：收尾
- 项目整理
- 自学资料
```

---

## 2. 技能清单
```
已掌握技能：

Python 基础
[ ] 变量、数据类型、运算符
[ ] 条件、循环、函数
[ ] 列表、字典、集合
[ ] 文件操作
[ ] 面向对象编程

Web 开发
[ ] Flask 框架
[ ] 路由和视图
[ ] 请求处理
[ ] 数据库操作
[ ] 用户认证

数据处理
[ ] requests 爬虫
[ ] pandas 数据分析
[ ] matplotlib 可视化

Go 入门
[ ] 基础语法
[ ] 函数和结构体
[ ] Gin API 开发
```

---

## 3. Python 自学资源
```
官方文档：
- https://docs.python.org/zh-cn/3/

在线教程：
- 廖雪峰 Python 教程
- Real Python (英文)
- Python 官方教程

书籍推荐：
- 《Python 编程：从入门到实践》
- 《流畅的 Python》
- 《Python Cookbook》

练习平台：
- LeetCode
- 牛客网
- Codewars
```

---

## 4. Go 自学资源
```
官方文档：
- https://go.dev/doc/
- https://tour.go-zh.org/ (Go 指南)

在线教程：
- Go by Example
- Go 语言圣经

书籍推荐：
- 《Go 程序设计语言》
- 《Go 语言实战》
- 《Go Web 编程》

框架学习：
- Gin: https://gin-gonic.com/
- Gorm: https://gorm.io/
```

---

## 5. 进阶学习路线
```
Python 进阶方向：

1. Web 开发深入
   - Django 框架
   - FastAPI 框架
   - 异步编程（asyncio）

2. 数据科学
   - NumPy 深入
   - 机器学习（scikit-learn）
   - 深度学习（PyTorch）

3. 自动化运维
   - Shell 脚本
   - Docker
   - Kubernetes

Go 进阶方向：

1. Web 开发
   - Gin 深入
   - GORM 数据库
   - 微服务

2. 并发编程
   - Goroutine
   - Channel
   - sync 包

3. 云原生
   - Docker
   - Kubernetes
   - gRPC
```

---

## 6. 考研 vs 就业建议
```
考研准备：
- 数据结构与算法
- 计算机网络
- 操作系统
- 数据库原理

建议：
- 保持代码练习
- 准备项目经验
- 学习算法

就业准备：
- 完善简历项目
- 刷题（LeetCode）
- 学习面试常考题
- 准备技术面试

建议：
- 有 2-3 个完整项目
- 熟悉至少一个技术栈
- 了解基础原理
```

---

## 7. 持续学习建议
```
1. 每日学习
   - 每天 30 分钟编程
   - 写代码比看视频重要

2. 项目驱动
   - 通过做项目学习
   - 解决实际问题

3. 参与社区
   - GitHub 开源项目
   - 技术博客
   - 技术群组

4. 总结复盘
   - 记笔记
   - 写博客
   - 教给别人

5. 保持好奇
   - 探索新技术
   - 不断尝试
```

---

## 8. 学习总结模板
```markdown
# 30 天学习总结

## 学到了什么
- ...
- ...

## 完成了哪些项目
1. ...
2. ...

## 遇到的困难和解决方法
- 问题1: ...
  解决: ...
- 问题2: ...
  解决: ...

## 还需要加强的地方
- ...

## 后续学习计划
- 短期（1个月）: ...
- 中期（3个月）: ...
- 长期（1年）: ...
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 回顾 | 30 天学习内容 |
| 资源 | Python、Go 自学资料 |
| 路线 | 进阶学习方向 |
| 建议 | 考研/就业准备 |

---

## 课后练习
完成练习题 37-38（见 [phase2_exercises.md](phase2_exercises.md#day-30-练习题交付自学资料--总结)）

---

## 恭喜完成 30 天 Python 进阶 + Golang 入门学习！

接下来：
1. 整理好所有项目代码
2. 完善 GitHub 主页
3. 制定后续学习计划
4. 保持学习热情

**祝你学业进步，前程似锦！**
