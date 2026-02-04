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
