# Crawler  
### python  
循环，判断  
for循环只能遍历容器  
没有switch
if语句最后要加冒号  
```py
score = 80
if score > 90:
    print('A')
elif score > 70:
    print('B')
elif score >= 60:
    print('C')
else:
    print('D')
```

默认参数
```py
def hello(who = 'world'): 
    print('hello %s!' % (who)) #python 格式化输出

hello()
hello('sea')
```

函数作为参数传递
```py
def g(x):
    return x * 5
def f(gf, x):   #gf作为形式参数
    return gf(x) + 100
print(f(g, 100))
print(f(lambda x: x * 5, 100))  #x*5-->gf  100-->x，匿名函数

def f(gf, x, y):
    return gf(x, y) + 100
print(f(lambda x, y: x * y, 100, 200))
```
list的用法  
range包含头不含尾  
合并数组要用extend  
```py
li = [1, 2, 3, 4, 5] 
# 遍历
for i in li:
    # print(i)
    pass
# 用range模拟for (i = 0; i < x; ++i)
# range(x) => [0, x - 1]
# range(x, y) => [x, y - 1]
# range(x, y, z) => [x, x + z,..., < y]
for i in range(len(li)):
    # print(li[i])
    pass

for i in range(1, 10, 2):
    print(i)

# 负数索引
print(li[-1])
print(li[-2])

# 负数step的range => [x, x - z, ..., > z]
for i in range(3, -1, -1):
    print(i)

# 添加元素
li = []
li.append(1)
li.append(2)
li.append('abc')
li.append(['d', 'e', 'f'])
print(li)

# 按元素添加数组
li = [1, 2]
li_2 = [3, 4, 5]
# 我们想要[1, 2, 3, 4, 5]
# li.append(li_2) => [1, 2, [3, 4, 5]]
li.extend(li_2)
print(li)

# 删除元素
li.pop()    # => [1, 2, 3, 4]
print(li)
li.pop(2)   # => [1, 2, 4]
print(li)

li = [5, 8, 7, 4, 2, 3]
li.sort()
print(li)
# lambda帮助排序
li = [[5, 2], [3, 8], [2, 11], [7, 6]]
# li.sort(key = lambda x: x[0]) # 按照第一个元素排序
# 与lamda等价写法
def item_key(x):
    return x[0]
li.sort(key = item_key)
print(li)
```
字典  
```py
di = {'k1': 'v1', 'k2': 'v2'}
di['k3'] = 'v3'
di['k4'] = 'v4'

for k in di:
    print(di[k])

for k, v in di.items():
    print(k, v)
```
list切片  
```py
# [1, 2, 3, 4, 5]
#  => [1, 2, 3]
#  => [3, 4]
li = [1, 2, 3, 4, 5]
li_0_2 =li[0:3] # 0 <= ? < 3
# 等价li[:3]
print(li_0_2)
# [start, end, step] => [start, start + step, ..., < end]
# start默认是0，end默认-1，step默认1
li_last_3 = li[-1:-4:-1]
print(li_last_3)

# 直接用切片反转数组
print(li[::-1])
print(li[-2::-1])

# 切片是复制，原来的list不会变
li_0_2[-1] = 100
print(li)

```
字符串  
```py
s = 'abcdefg'
try:
    str[0] = 'x'
except Exception as e:
    print(e)

# 修改字符串
li = list(s)
# print(li)
li[0] = 'x'
s = ''.join(li)
print(s)
s = '-'.join(li)
print(s)

# 切割
s = 'abc,def,ghi'
p1, p2, p3 = s.split(',')
print(p1, p2, p3)

# 下标访问和切片
s = 'abcdefg'
print(s[0], s[-1])
print(s[2:5])
```
对象  
```py
# 用type查看对象类型
print(type([1, 2, 3, 4]))
print(type('abcd'))
print(type({1:2, 2:3}))

# 用dir查看属性和方法
print(dir(list))


class Clazz(object):
    # self参考C++的this指针！
    def __init__(self, x, y):
        self.x = x
        self.y = y
        
    # 声明成员函数的时候，第一个参数一定是self，不要忘记！
    def display(self):
        print(self.x, self.y)

print(type(Clazz))
clz = Clazz(100, 200)
clz.display()  # => display(clz)


class Base:
    def run(self):
        print('Base::run')

class Tom(Base):
    def run(self):
        print('Tom::run')

t = Tom()
print(isinstance(t, Base))
t.run()

def run(runner):
    runner.run()

class R1:
    def run(self):
        print('R1::run')

class R2:
    def run(self):
        print('R2::run')

run(R1())
run(R2())
```
file，with open就不用try exception  
```py
with open('text.txt') as f:
    for line in f.readlines():
        print(line)

with open('text.txt', 'rb') as f:
    print(f.read())

s = 'abcdefg'
b = bytes(s)
print(b)
```
异常处理
```py
try:
    r = 10 / 0
except ZeroDivisionError as e:
    print(type(e))
    print(e)
finally:
    # 主要防止资源泄露！
    print('Always come here.')
```
多线程  
```py
import threading

def thread_func(x):
    # 自己加sleep和其它复杂操作看效果
    print('%d\n' % (x * 100))

threads = []
for i in range(5):
    threads.append(threading.Thread(target = thread_func, args = (100, )))

for thread in threads:
    thread.start()

for thread in threads:
    thread.join()
```
json  字典转字符串  
```py
import json

obj = {'one': '一', 'two': '二'}
encoded = json.dumps(obj)
print(type(encoded))
print(encoded)
decoded = json.loads(encoded)
print(type(decoded))
print(decoded)
```
xml解析，dom和sax  
```py
#dom
from xml.dom import minidom

doc = minidom.parse('book.xml')
root = doc.documentElement
# print(dir(root))
print(root.nodeName)
books = root.getElementsByTagName('book')
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
from selenium import webdriver
import time

browser = webdriver.Chrome()
browser.set_page_load_timeout(30)
browser.get('http://www.17huo.com/search.html?sq=2&keyword=%E7%BE%8A%E6%AF%9B')
page_info = browser.find_element_by_css_selector('body > div.wrap > div.pagem.product_list_pager > div')
# print(page_info.text)
pages = int((page_info.text.split('，')[0]).split(' ')[1])
for page in range(pages):
    if page > 2:
        break
    url = 'http://www.17huo.com/?mod=search&sq=2&keyword=%E7%BE%8A%E6%AF%9B&page=' + str(page + 1)
    browser.get(url)
    browser.execute_script("window.scrollTo(0, document.body.scrollHeight);")
    time.sleep(3)   # 不然会load不完整
    goods = browser.find_element_by_css_selector('body > div.wrap > div:nth-child(2) > div.p_main > ul').find_elements_by_tag_name('li')
    print('%d页有%d件商品' % ((page + 1), len(goods)))
    for good in goods:
        try:
            title = good.find_element_by_css_selector('a:nth-child(1) > p:nth-child(2)').text
            price = good.find_element_by_css_selector('div > a > span').text
            print(title, price)
        except:
            print(good.text)
```






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

