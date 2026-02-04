# Day 16 练习题：requests + BeautifulSoup
> 要求：掌握 HTTP 请求和 HTML 解析基础
>

---

## 练习 1：基础请求
```plain
使用 requests 完成以下任务：

1. 发送 GET 请求到 https://httpbin.org/get
   - 打印状态码
   - 打印响应头
   - 打印响应内容

2. 发送带参数的 GET 请求
   - URL: https://httpbin.org/get
   - 参数: name=python, version=3.10
   - 打印完整 URL

3. 发送带请求头的请求
   - 添加自定义 User-Agent
   - 添加 Accept-Language: zh-CN
```

---

## 练习 2：HTML 解析
```plain
给定以下 HTML，使用 BeautifulSoup 提取数据：

html = '''
<html>
<body>
    <div class="products">
        <div class="product" id="p1">
            <h2 class="name">iPhone 15</h2>
            <span class="price">7999</span>
            <a href="/detail/1">详情</a>
        </div>
        <div class="product" id="p2">
            <h2 class="name">MacBook Pro</h2>
            <span class="price">14999</span>
            <a href="/detail/2">详情</a>
        </div>
        <div class="product" id="p3">
            <h2 class="name">iPad Air</h2>
            <span class="price">4799</span>
            <a href="/detail/3">详情</a>
        </div>
    </div>
</body>
</html>
'''

任务：
1. 提取所有产品名称
2. 提取所有价格
3. 提取所有详情链接
4. 将数据组织成字典列表
```

---

## 练习 3：爬取练习
```plain
爬取 http://quotes.toscrape.com/ 网站：

1. 获取首页所有名言
2. 提取每条名言的：
   - 内容
   - 作者
   - 标签
3. 打印格式化输出
4. 统计共有多少条名言
```

---

## 完成检查
- [ ] 能正确发送 GET 请求
- [ ] 能处理请求参数和请求头
- [ ] 能使用 BeautifulSoup 解析 HTML
- [ ] 能使用 CSS 选择器提取数据
- [ ] 能将提取的数据组织成结构化格式

---
