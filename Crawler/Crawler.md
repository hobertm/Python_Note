# Crawler  

- 爬虫工作流程  
将种子URL放入队列  
从队列中获取URL，抓取内容  
解析抓取内容，将需要进一步抓取的URL放入工作队列，存储解析后的内容  

- 抓取策略  
深度优先  
广度优先  
PageRank  
大站优先策略  

```python
# coding:utf-8
import requests
import xml.etree.ElementTree as ET
from xml.parsers.expat import ParserCreate

class DefaultSaxHandler(object):
    def __init__(self, provinces):
        self.provinces = provinces

    # 处理标签开始
    def start_element(self, name, attrs):
        if name != 'map':
            name = attrs['title']
            number = attrs['href']
            self.provinces.append((name, number))

    # 处理标签结束
    def end_element(self, name):
        pass

    # 文本处理
    def char_data(self, text):
        pass
    
def get_province_entry(url):
    # 获取文本，并用gb2312解码
    content = requests.get(url).content.decode('gb2312')
    # 确定要查找字符串的开始结束位置，并用切片获取内容。
    start = content.find('<map name=\"map_86\" id=\"map_86\">')
    end = content.find('</map>')
    content = content[start:end + len('</map>')].strip()
    provinces = []
    # 生成Sax处理器
    handler = DefaultSaxHandler(provinces)
    # 初始化分析器
    parser = ParserCreate()
    parser.StartElementHandler = handler.start_element
    parser.EndElementHandler = handler.end_element
    parser.CharacterDataHandler = handler.char_data
    # 解析数据
    parser.Parse(content)
    # 结果字典为每一页的入口代码
    return provinces

provinces = get_province_entry('http://www.ip138.com/post')
print(provinces)
```  
xml解析，dom和sax  
```py
#dom
from xml.dom import minidom

doc = minidom.parse('book.xml')
root = doc.documentElement
# print(dir(root))
print(root.nodeName)
books = root.getElementsByTagName('book') #返回数组
print(type(books))
for book in books:
    titles = book.getElementsByTagName('title')
    print(titles[0].childNodes[0].nodeValue)
```
```py
#sax
import string
from xml.parsers.expat import ParserCreate


class DefaultSaxHandler(object):
    def start_element(self, name, attrs):
        self.element = name
        print('element: %s, attrs: %s' % (name, str(attrs)))

    def end_element(self, name):
        print('end element: %s' % name)

    def char_data(self, text):
        if text.strip():
            print("%s's text is %s" % (self.element, text))

handler = DefaultSaxHandler()
parser = ParserCreate()
parser.StartElementHandler = handler.start_element
parser.EndElementHandler = handler.end_element
parser.CharacterDataHandler = handler.char_data
with open('book.xml', 'r') as f:
    parser.Parse(f.read())

```
爬17huo网站
```py
# coding:utf-8
from selenium import webdriver
import time
import os

iedriver = "C:\Program Files\Internet Explorer\IEDriverServer.exe"
os.environ["webdriver.ie.driver"] = iedriver
browser = webdriver.Ie(iedriver)
browser.set_page_load_timeout(30)
browser.get('http://www.17huo.com/search.html?sq=2&keyword=%E7%BE%8A%E6%AF%9B')
# 复制css选择器
page_info = browser.find_element_by_css_selector('body > div.wrap > div.pagem.product_list_pager > div')
# print(page_info.text) 打印有多少页，每页有多少个
pages = int((page_info.text.split('，')[0]).split(' ')[1])
for page in range(pages):
    if page > 2:  # 只抓前三页
        break
    url = 'http://www.17huo.com/?mod=search&sq=2&keyword=%E7%BE%8A%E6%AF%9B&page=' + str(page + 1)
    browser.get(url)
    # 拖动滚动条加载商品
    browser.execute_script("window.scrollTo(0, document.body.scrollHeight);")
    time.sleep(3)  # 不然会load不完整
    goods = browser.find_element_by_css_selector(
        'body > div.wrap > div:nth-child(2) > div.p_main > ul').find_elements_by_tag_name('li')
    print('%d页有%d件商品' % ((page + 1), len(goods)))
    for good in goods:
        try:
            title = good.find_element_by_css_selector('a:nth-child(1) > p:nth-child(2)').text
            price = good.find_element_by_css_selector('div > a > span').text
            print(title, price)
        except:
            print(good.text)

```
request  
```py
# coding:utf-8
import json
import requests

from PIL import Image
from io import BytesIO

# print(dir(requests))

url = 'http://www.baidu.com'
r = requests.get(url)
print(r.text)
print(r.status_code)
print(r.encoding)

# 传递参数：拼接参数
params = {'k1':'v1', 'k2':'v2'}
r = requests.get('http://httpbin.org/get', params)
print(r.url)

# 二进制数据
r = requests.get('图片地址')
image = Image.open(BytesIO(r.content)) 
image.save('图片.jpg')

# json处理
r = requests.get('https://github.com/timeline.json')
print(type(r.json))
print(r.text)

# 原始数据处理 以流的形式一段段写入
r = requests.get('http://i-2.shouji56.com/2015/2/11/23dab5c5-336d-4686-9713-ec44d21958e3.jpg', stream = True)
with open('meinv2.jpg', 'wb+') as f:
    for chunk in r.iter_content(1024):
        f.write(chunk)

# 提交表单
form = {'username':'user', 'password':'pass'}
r = requests.post('http://httpbin.org/post', data = form)
print(r.text)
r = requests.post('http://httpbin.org/post', data = json.dumps(form))
print(r.text)

# cookie
url = 'http://www.baidu.com'
r = requests.get(url)
cookies = r.cookies
for k, v in cookies.get_dict().items():
    print(k, v)
cookies = {'c1':'v1', 'c2': 'v2'}
r = requests.get('http://httpbin.org/cookies', cookies = cookies)
print(r.text)

# 重定向和重定向历史
r = requests.head('http://github.com', allow_redirects = True)
print(r.url)
print(r.status_code)
print(r.history)

# 代理
proxies = {'http': ',,,', 'https': '...'}
r = requests.get('...', proxies = proxies)
```
BeatifulSoup  
```py
from bs4 import BeautifulSoup

soup = BeautifulSoup(open('test.html'))
# print(soup.prettify())

# Tag
print(type(soup.title))
print(soup.title.name)
print(soup.title)

# String
print(type(soup.title.string))
print(soup.title.string)

# Comment
print(type(soup.a.string))
print(soup.a.string)

for item in soup.body.contents:
    print(item.name)

# CSS查询
print(soup.select('.sister'))
print(soup.select('#link1'))
print(soup.select('head > title'))

a_s = soup.select('a')
for a in a_s:
    print(a)

```
HTMLParser
```py
from HTMLParser import HTMLParser
# markupbase

class MyParser(HTMLParser):
    def handle_decl(self, decl):
        HTMLParser.handle_decl(self, decl)
        print('decl %s' % decl)

    def handle_starttag(self, tag, attrs):
        HTMLParser.handle_starttag(self, tag, attrs)
        print('<' + tag + '>')

    def handle_endtag(self, tag):
        HTMLParser.handle_endtag(self, tag)
        print('</' + tag + '>')

    def handle_data(self, data):
        HTMLParser.handle_data(self, data)
        print('data %s' % data)

    #<br/>
    def handle_startendtag(self, tag, attrs):
        HTMLParser.handle_startendtag(self, tag, attrs)

    def handle_comment(self, data):
        HTMLParser.handle_comment(self, data)
        print('data %s' % data)

    def close(self):
        HTMLParser.close(self)
        print('Close')

demo = MyParser()
demo.feed(open('test.html').read())
demo.close()

```










