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
完成练习题 1-3（见 [exercises.md](exercises.md)）

---
