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
