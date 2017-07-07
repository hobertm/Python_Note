# Crawler  

- 循环，判断  
for循环只能遍历容器  
没有switch，别忘了后面有冒号  
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
print(f(lambda x: x * 5, 100))  #x*5-->gf  100-->x

def f(gf, x, y):
    return gf(x, y) + 100
print(f(lambda x, y: x * y, 100, 200))
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

