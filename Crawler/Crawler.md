# Crawler  

- OSI七层  
物理层：电器连接  
数据链路层：交换机，STP，帧中继  
网络层：路由器，IP协议  
传输层：TCP、UDP协议  
会话层：建立通信连接，网络拨号  
表示层：每次连接只处理一个请求  
应用层：HTTP、FTP  

- REQUEST 部分的 HTTP HEADER  
**Accept: text/plain**  
**Accept-Charset: utf-8**  
**Accept-Encoding: gzip, deflate**  
Accept-Language: en-US  
**Connection: keep-alive**  
Content-Length: 348  
Content-Type: application/x-www-form-urlencoded  
Date: Tue, 15 Nov 1994 08:12:31 GMT  
Host: en.wikipedia.org:80  
**User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:12.0) Gecko/20100101 Firefox/21.0**  
**Cookie: $Version=1; Skin=new;**  

- GET,POST对比  
GET的request没有body部分  
GET是idempotent幂等的，几次请求结果相同  

- HTTP 响应状态码  
3XX一般urllib2已经对重定向做了处理  
400 Bad Request客户端请求有语法错误，不能被服务器所理解  
401 Unauthorized 请求未经授权，这个状态代码必须和WWW-Authenticate报头域一起使用  
403 Forbidden服务器收到请求，但是拒绝提供服务  
404 Not Found请求资源不存在，eg：输入了错误的URL  
500 Internal Server Error 服务器发生不可预期的错误  
503 Server Unavailable服务器当前不能处理客户端的请求，一段时间后可能恢复正常  
解决方法  
400 Bad Request检查请求的参数或者路径  
401 Unauthorized 如果需要授权的网页，尝试重新登录  
403 Forbidden  
如果是需要登录的网站，尝试重新登录  
IP被封，暂停爬取，并增加爬虫的等待时间，如果拨号网络，尝试重新联网更改IP  
404 Not Found直接丢弃  
5XX服务器错误，直接丢弃，并计数，如果连续不成功，WARNING 并停止爬取  

- 选择策略  
重要的网页距离种子站点比较近  
万维网的深度并没有很深，一个网页有很多路径可以到达  
宽度优先有利于多爬虫并行合作抓取  
深度限制与宽度优先相结合  


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
selenium爬购物网站
```py
# coding:utf-8
from selenium import webdriver
import time

browser = webdriver.Firefox()
browser.set_page_load_timeout(30)
browser.get('http://www.17huo.com/search.html?sq=0&keyword=%E5%A4%A7%E8%A1%A3')
# 复制css选择器
page_info = browser.find_element_by_css_selector('body > div.wrap > div.pagem.product_list_pager > div')
# print(page_info.text) 打印有多少页，每页有多少个
pages = int((page_info.text.split('，')[0]).split(' ')[1])
for page in range(pages):
    if page > 2:  # 只抓前三页
        break
    url = 'http://www.17huo.com/?mod=search&sq=2&keyword=大衣&page=' + str(page + 1)
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
r = requests.get(url)  #发送请求，返回响应
print(r.text)
print(r.status_code)
print(r.encoding)

# 传递参数：拼接参数
params = {'k1':'v1', 'k2':'v2'}
r = requests.get('http://httpbin.org/get', params) #发送请求时拼接
print(r.url)

# 二进制数据，保存图片
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
print(type(soup.title)) #类型是tag
print(soup.title.name)
print(soup.title)

# String
print(type(soup.title.string))#NavigableString
print(soup.title.string)

# Comment
print(type(soup.a.string))#Comment
print(soup.a.string)

for item in soup.body.contents:
    print(item.name)

# CSS查询
print(soup.select('.sister'))
print(soup.select('#link1'))
print(soup.select('head > title'))

a_s = soup.select('a')  #soup.a 只会返回一个，select返回所有
for a in a_s:
    print(a)

```
HTMLParser
```py
from HTMLParser import HTMLParser
# markupbase 需要放到site-packages

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
宽度优先  
```py
import requests
import html5lib
import re
from bs4 import BeautifulSoup


s = requests.Session()
url_login = 'http://accounts.douban.com/login'

formdata = {
    'redir':'https://www.douban.com',
    'form_email': 't.t.panda@hotmail.com',
    'form_password': 'tp65536!',
    'login': u'登陆'
}
headers = {'user-agent': 'Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/44.0.2403.157 Safari/537.36'}

r = s.post(url_login, data = formdata, headers = headers)
content = r.text
soup = BeautifulSoup(content, 'html5lib')
captcha = soup.find('img', id = 'captcha_image')
if captcha:
    captcha_url = captcha['src']
    re_captcha_id = r'<input type="hidden" name="captcha-id" value="(.*?)"/'
    captcha_id = re.findall(re_captcha_id, content)
    print(captcha_id)
    print(captcha_url)
    captcha_text = input('Please input the captcha:')
    formdata['captcha-solution'] = captcha_text
    formdata['captcha-id'] = captcha_id
    r = s.post(url_login, data = formdata, headers = headers)
with open('contacts.txt', 'w+', encoding = 'utf-8') as f:
    f.write(r.text)
```

```py
import urllib2
from collections import deque
import json
from lxml import etree
import httplib
import hashlib
from pybloom import BloomFilter


class CrawlBSF:
    request_headers = {
        'host': "www.mafengwo.cn",
        'connection': "keep-alive",
        'cache-control': "no-cache",
        'upgrade-insecure-requests': "1",
        'user-agent': "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.95 Safari/537.36",
        'accept': "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",
        'accept-language': "zh-CN,en-US;q=0.8,en;q=0.6"
    }

    cur_level = 0
    max_level = 5
    dir_name = 'iterate/'
    iter_width = 50
    downloaded_urls = []

    du_md5_file_name = dir_name + 'download.txt'
    du_url_file_name = dir_name + 'urls.txt'

    download_bf = BloomFilter(1024 * 1024 * 16, 0.01)

    cur_queue = deque()
    child_queue = deque()

    def __init__(self, url):
        self.root_url = url
        self.cur_queue.append(url)
        self.du_file = open(self.du_url_file_name, 'a+')
        try:
            self.dumd5_file = open(self.du_md5_file_name, 'r')
            self.downloaded_urls = self.dumd5_file.readlines()
            self.dumd5_file.close()
            for urlmd5 in self.downloaded_urls:
                self.download_bf.add(urlmd5[:-2])
        except IOError:
            print ("File not found")
        finally:
            self.dumd5_file = open(self.du_md5_file_name, 'a+')

    def enqueueUrl(self, url):
        self.child_queue.append(url)

    def dequeuUrl(self):
        try:
            url = self.cur_queue.popleft()
            return url
        except IndexError:
            self.cur_level += 1
            if self.cur_level == self.max_level:
                return None
            if len(self.child_queue) == 0:
                return None
            self.cur_queue = self.child_queue
            self.child_queue = deque()
            return self.dequeuUrl()

    def getpagecontent(self, cur_url):
        print "downloading %s at level %d" % (cur_url, self.cur_level)
        try:
            req = urllib2.Request(cur_url, headers=self.request_headers)
            response = urllib2.urlopen(req)
            html_page = response.read()
            filename = cur_url[7:].replace('/', '_')
            fo = open("%s%s.html" % (self.dir_name, filename), 'wb+')
            fo.write(html_page)
            fo.close()
        except urllib2.HTTPError, Arguments:
            print Arguments
            return
        except httplib.BadStatusLine:
            print 'BadStatusLine'
            return
        except IOError:
            print 'IO Error at ' + filename
            return
        except Exception, Arguments:
            print Arguments
            return
        # print 'add ' + hashlib.md5(cur_url).hexdigest() + ' to list'
        dumd5 = hashlib.md5(cur_url).hexdigest()
        self.downloaded_urls.append(dumd5)
        self.dumd5_file.write(dumd5 + '\r\n')
        self.du_file.write(cur_url + '\r\n')
        self.download_bf.add(dumd5)

        html = etree.HTML(html_page.lower().decode('utf-8'))
        hrefs = html.xpath(u"//a")

        for href in hrefs:
            try:
                if 'href' in href.attrib:
                    val = href.attrib['href']
                    if val.find('javascript:') != -1:
                        continue
                    if val.startswith('http://') is False:
                        if val.startswith('/'):
                            val = 'http://www.mafengwo.cn' + val
                        else:
                            continue
                    if val[-1] == '/':
                        val = val[0:-1]
                    # if hashlib.md5(val).hexdigest() not in self.downloaded_urls:
                    if hashlib.md5(val).hexdigest() not in self.download_bf:
                        self.enqueueUrl(val)
                    else:
                        print 'Skip %s' % (val)
            except ValueError:
                continue

    def start_crawl(self):
        while True:
            url = self.dequeuUrl()
            if url is None:
                break
            self.getpagecontent(url)
        self.dumd5_file.close()
        self.du_file.close()


crawler = CrawlBSF("http://www.mafengwo.cn")
crawler.start_crawl()

```







