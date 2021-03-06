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

- 如何记录抓取历史？  
1.将访问过的URL保存到数据库效率太低  
2.用HashSet将访问过的URL保存起来。那只需接近O(1)的代价就可以查到一个URL是否被访问过了。消耗内存  
3.URL经过MD5或SHA-1等单向哈希后再保存到HashSet或数据库。  
4.Bit-Map方法。建立一个BitSet，将每个URL经过一个哈希函数映射到某一位。  

- Bloom Filter  
Bloom Filter使用了多个哈希函数，而不是一个。  
创建一个m位BitSet，先将所有位初始化为0，然后选择k个不同的哈希函数。  
第i个哈希函数对字符串str哈希的结果记为h（i，str），且h（i，str）的范围是0到m-1。  
只能插入，不能删除！  

- 网页的关键内容  
Title 网页的标题  
Content Title正文的标题  
Content 正文部分  
Anchor 内部的锚点  
Link 外部链接  
 
- 实现一个多线程爬虫  
1.创建一个线程池threads = []  
2.确认url队列线程安全Queue Deque  
3.从队列取出url，分配一个线程开始爬取pop()/get() threading.Thread  
4.如果线程池满了，循环等待，直到有线程结束t.is_alive()  
5.从线程池移除已经完成下载的线程threads.remove(t)  
6.如果当前级别的url已经遍历完成，t.join()函数等待所有现场结束，然后开始下一级别的爬取  

- 多进程爬虫评估  
目的：控制线程数量对线程进行隔离，减少资源竞争  
某些环境下，在单机上利用多个IP来伪装局限性：  
不能突破网络瓶颈  
单机单IP的情况下，变得没有意义  
数据交换的代价更大  

- 分布式爬虫的重要性   
解决目标地址对IP访问频率的限制  
利用更高的带宽，提高下载速度  
大规模系统的分布式存储和备份  
数据的扩展能力  

- 将多进程爬虫部署到多台主机上  
将数据库地址配置到统一的服务器上  
数据库设置仅允许特定IP来源的访问请求  
1.GRANT ALL PRIVILEGES ON \*.* TO 'root'@'%' IDENTIFIED BY  
'password' WITH GRANT OPTION; FLUSH PRIVILEGES;  
2.my.cnf#bind-address = 127.0.0.1  
设置防火墙，允许端口远程连接  
iptables-A INPUT -ieth0 -p tcp-m tcp--dport3306 -j ACCEPT  


- HDFS  
Distributed, Scalable, Portable File System  
Written in Java  
Not fully POSIX-compliant  
Replication: 3 copies by default  
Designed for immutable files  
Files are cached and chunked, chunk size 64MB  

- HBASE  
On top of HDFS  
Column-oriented database  
Can store huge size raw data  
KEY-VALUE  

- MongoDB  

    RDBMS|MongoDB 
-|-
Database|Database 
Table|Collection
Tuple/Row|Document
column|Field
Table Join|Embedded Document
Primary Key|Primary Key (Default by mongodb )  

- Redis Overview  
基于KEY VALUE模式的内存数据库  
支持复杂的对象模型（MemoryCached仅支持少量类型）  
支持Replication，实现集群（MemoryCached不支持分布式部署）  
所有操作都是原子性（MemoryCached多数操作都不是原子的）  
可以序列化到磁盘（MemoryCached不能序列化）  

- Master-Slave 结构  
有一个主机，对所有的服务器进行管理。绝大多数分布式系统，  
都是Master-Slave的主从模式。而之前我们的爬虫，是完全独立的,  
依次从url队列里获取url，进行抓取。  
当爬虫服务器多的时候，必须能通过一个中心节点对从节点进行管理  
能对整体的爬取进行控制 
爬虫之间信息共享的桥梁  
shutdown connection: server call close(), clietnrecv() returns 0  

- Master及Slave工作  
Master  
1.管理爬虫  
2.动态重拍  
3.间隔地状态检查  
Slave  
1.注册  
2.获取并执行命令  
3.同步状态  
4.爬取网页，保存到各个分布式数据库  

- 结束服务器客户端通信  
fixed length message: whiletotalsent< MSGLEN:  
delimited: some message \0  
indicates message length in beginning: LEN: 50;  
shutdown connection: server call close(), clietnrecv() returns 0  

- Log 系统基本用途  
多线程情况下，debug调试非常困难  
错误出现可能有一些随机性  
性能分析
错误记录与分析
运行状态的实时监测  

- PageRank 算法优缺点  
优点：  
一个与查询无关的静态算法，所有网页的PageRank值通过离线计  
算获得；有效减少在线查询时的计算量，极大降低了查询响应时间。  
缺点：  
人们的查询具有主题特征，PageRank忽略了主题相关性，导致结  
果的相关性和主题性降低。   
旧的页面等级会比新页面高。因为即使是非常好的新页面也不会  
有很多上游链接，除非它是某个站点的子站点。

- 服务器处理Web请求流程  
1.到达防火墙，对访问频次进行检查  
2.根据端口映射，到达对应的服务，例如Apache  
3.到达Apache，通过virtualhost 查找根目录  
4.查找.htaccess伪静态设置，映射实际目录及文件  
5.执行脚本或提取文件  
6.确认cookie信息，查找用户  
7.用户权限检查  
8.执行命令，返回数据  

- Virtual Host
一台服务器主机可以处理多个域名或IP  
将不同的域名及Ip映射到不同的网站根目录  
不同的域名指向同一个服务器，更容易被服务器识别为爬虫（异常访问）而禁止  
对于一个域名映射多台服务器的网站，在单机上不能并发抓取不同服务器的数据  

- 网站如何发现爬虫？  
单一IP非常规的访问频次  
单一IP非常规的数据流量  
大量重复简单的网站浏览行为  
只下载网页，没有后续的js、css请求  
通过一些陷阱来发现爬虫，例如一些通过CSS对用户隐藏的链接，只有爬虫才会访问  

- 网站如何进行反爬  
服务器日志分析  
UserAgent  
动态网页  
基于流量  
基于ip  
iptables  

- 好的规避反爬虫检查的方法  
1.多主机策略  
2.爬的慢一点，不要攻击主机，如果发现被阻止，立即降低访问频次，  
设计得smart一点，来找到访问频次限制的临界点  
3.通过变换IP或者代理服务器来掩饰  
4.把爬虫放到访问频繁的主站IP的子网下，例如教育网  
5.频繁改变自己的User-Agent  
6.探测陷阱，比如nofollow的tag，或者display:none的CSS  
7.如果使用了规则(pattern)来批量爬取，需要对规则进行组合  
8.如果可能，按照Robots.txt定义的行为去文明抓取  


- 表单类型  
form-data  
http请求中的multipart/form-data，它会将表单的数据处理为一条消息，  
以标签为单元，用分隔符分开。既可以上传键值对，也可以上传文件。当  
上传的字段是文件时，会有Content-Type来表名文件类型。由于有boundary  
隔离，所以multipart/form-data既可以上传文件，也可以上传键值对，它  
采用了键值对的方式，所以可以上传多个文件。  
-x-www-form-urlencoded  
application/x-www-from-urlencoded，会将表单内的数据转换为键值对  

- 登录  
1.根据chrome inspector 里检查到的参数，来设置登录方式  
最常用的是x-www-form-urlencoded和json方式，两种方式只是body编码不同  
如果是form-data，按照form-data的方式组合body  
body部分经常需要按照特定的方式编码，比如base64，或者md5  
对于非常复杂的登录协议，利用第5章的动态网页方式登录  
2.request = urllib2.Request(url, data, headers = loginheaders)  
url是访问的地址  
data 是body部分  
headers = loginheaders是HTTP请求的HEADER数据  

- 登录方式  
表单登录  
见request_login.py  
获取并设置Cookie  
获取并设置Cookie登录成功后，HEADER会有设置cookie的相关信息  
此时我们需要把服务器返回的Cookie信息，写入到我们后续请求的  
HEADER的Cookie里  
- cookie意外  
当遇到网页登录后，返回302跳转的情况下，urllib2的Response会  
丢失Set-Cookie 的信息，导致登录不成功  
更通常的情况下，我们需要一个通用的能处理Cookie的工具自动处理：  
Set-Cookie请求  
自动处理管理过期Cookie  
自动在对应的域下发送特殊Cookie   
解决方法：cookie jar  

- Python Web  
Selenium：一个自动化的Web测试工具，可以支持包括Firefox、chrome、  
PhatomJS、IE等多种浏览器的连接与测试  
PhantonJs：一个基于Webkit的Headless的Web引擎，支持JavaScript  

- Useful Methods & Properties  
Selenium 通过浏览器的驱动，支持大量的HTML及Javascript的操作，常用的可以包括：  
page_source: 获取当前的html 文本  
title：HTML的title  
current_url：当前网页的URL  
get_cookie()&get_cookies()：获取当前的cookie  
delete_cookie() & delete_all_cookies()：删除所有的cookie  
add_cookie()：添加一段cookie  
set_page_load_timeout()：设置网页超时  
execute_script()：同步执行一段javascript命令  
execute_async_script()：异步执行javascript命令  
 
- PhantomJS配置  
ignore-ssl-errors=[true|false]  
load-images=[true|false]  
disk-cache=[true|false]  
cookies-file=/path/to/cookies.txt  
debug=[true|false]  
config  

- 动态页面的抓取技巧  
Javascript的执行是需要时间的，为了等待所有的内容都完成加载，  
需要设置一个等待time.sleep()   
使用--ignore-images=true来避免加载图片  
每次初始化webdriver每次都会创建一个新的PhantomJS的进程，所以  
webdriver是可以重用的，一个webdriverinstance 相当于一个chrome   
的标签页，每次只需调用get(url)即可打开一个新的页面  
PhantomJS运行需要占用比较多的系统资源，所以并发数需要视计算机  
性能以及实际测试情况而定，建议不要设置太大  
程序退出后，考虑调用shell来杀掉所有phantomjs进程  
subprocess.call('pgrep phantomjs| xargs kill')    

- Scrapy Core Features  
模板化创建工程、启动爬虫的脚本  
面向对象的编程模型  
内容、数据抽取的各类接口  
封装了Xpath及CSS选择器  
一个shell的控制台(Ipythonaware），用来对下载的网页实验CSS 及Xpath表达式以进行数据提取及debug  
可以将数据转化为JSONCSVXML等多种格式，并支持FTP/S3/Local FS 的存储  
支持多种文本编码格式，并支持自动识别编码格式  
强大的插件支持，可以自己挂载插件到Scray中来做功能增强  
Telnet Console 进入到Pythonconsole来监控和调试爬虫  
自动的图片异步下载  

- Commands -genspider  
genspider[-t template] <name> <domain>  
在当前文件夹内，利用模板创建一个新的spider，或者在当前工程的spiders  
文件夹里（参考startporject创建的工程的文件夹架构）创建一个新的spider  
一个spider 就是继承了scrapy.Spider，并包含了start_url以及基本规则的python文件  
Template:  
basic 几乎空的spider，只下载domain，不保存不解析  
crawl 创建基于scrapy.spiders.CrawlSpider对象的模板，定义了如何提取  
link以及Items的信息，详情参考Items及样例spider rule  
csvfeed创建包含了可以按行解析的模板pass row()  
xmlfeed创建包含解析xml文件节点的模板parse node()  

- Commands 运行
scracpy runspider<spider-file>  
运行一个已经存在的spider  
scracpy crawl <spider-name>  
运行一个已经存在的spider，通过spider 名字来运行  
scracpy fetch url  
下载一个网页并在命令行里输出，用于简单实验  
--headers 打印HTTP的header  
--spider=SPIDER 用指定的爬虫来运行，指定的爬虫的参数会覆盖默认参数，例如user-agent  
scrapy bench  
一个快速的benchmark测试，会显示下载流量、时间、响应数、内存消耗、responsecode等  

- Spider work flow  
1.start request() start urls: 指定开始抓取的网页  
2.parse():网页下载完成后后，调用parse开始处理。parse可以同时返回解析后的数据，  
比如dict，Items对象，同时也可以返回Request 对象。必须以yield 方式返回一个可以  
遍历的对象。scrapy会根据返回的iterable里的对象的类型来处理，例如取到的如果是item，  
会作为解析对象放回结果里；如果是Request，会放入scheduler，进入下载队列  
3.对于解析出来的Items，可以通过Item Pipeline 写入数据库，或者Feed exports以JSON、XML等方式写入文件  

- CrawlSpider 定义规则  
针对一些通用的网页抓取所设计的基础类，提供了比较简单的方式来处理  
外链以及定义一系列规则。通过scrapygenspider–t crawl 创建的Spider  
会使用CrawlSpider的baseclass
需要重写的方法及属性：  
rule:定一个了抓取的规则，是一个元组，应用rule的优先级是按照rule在Rules元组的顺序来的  
parse start url(response)：特定针对第一个网页的处理  

- XMLFeed  
直接抓取并分析xml文件，可以用于sitemap  
iterator: iternodesxml html 三种方式来调用xpath或re 解析node  
itertag：node 名称  
parse node:解析回调  
使用response.selector.remove namespaces()方法去除namespace，然后再使用xpath  
例如response.xpath("//sitemap")   

- File and Image Pipelines  
Enabling your Media Pipeline  
ITEM_PIPELINES = {'scrapy.pipelines.images.ImagesPipeline': 1}  
Set the FILES_STORE or IMAGES_STOREsetting:FILES_STORE = '/path/to/valid/dir'  
Items 里定义images_urls和images，这些field会被传入pipelines 处理  

- 完整做一个scrapyspider 任务  
1.scrapy start project chdemo创建新的工程  
2.scrapygenspidermfwmafengwo.com创建mfw的spider  
3.编辑Settings，设置ITEM_PIPELINES 以及IMAGES_STORE  
4.scrapyshell http://www.mafengwo.cn/i/1082510.html 调试这个网页，重点测试xpath的选择  
5.编辑spider/mfw.py设置start_url  
6.修改parse文件  
--yield iterable对象，用于提取  
--提取外链yield Request  
7.scrapy crawl mfw  

- 网页排重过程  
1.建立10个分块存储区  
2.计算一篇文章的SimHash，对于主题抓取，可以对特定主题词增加权重  
simhash:分词、Hash、加权、合并、降维  
3.把SimHash值分为16、12、12、12、12这样的5块，计算出10个组合  
4.将这10个组合，分别插入到各自对应的分块中，通过二分法插入  
5.重复2~4直到所有的文档都计算完SimHash并插入到索引表  
6.扫描索引表，进行压缩，将每一个块的第一个SimHash保存到单独索引表  
7.对于一个要检查的文档，计算SimHash，分块后计算出10个查找Key，通过  
SimHash的block索引表，利用插补法进行查找，找到AB区相同的block  
8.解压block，再一一比较计算海明距离，找出距离小于k的文档  

- 正文提取  
1.去除Javascript及CSS  
利用lxml的clean类，能删除HTML里所包含css及script  
2.去除所有HTMLTAG  
reg= re.compile("<[^>]*>")  
content = reg.sub('', content)  
3.去除噪声  
基于文本长度或聚类的方法  
```py
# coding:utf-8
import requests
import xml.etree.ElementTree as ET
from xml.parsers.expat import ParserCreate

class DefaultSaxHandler(object):
    def __init__(self, provinces):
        self.provinces = provinces


indicates message length in beginning: LEN: 50;  
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
lxml包xpath
```py
# coding:utf-8
import lxml
from lxml import html
from lxml import etree

from bs4 import BeautifulSoup

with open('jd.com_2131674.html', 'r', -1, 'utf-8', 'ignore') as f:
    content = f.read()
tree = etree.HTML(content)

print('--------------------------------------------')
print('# different quote //*[@class="p-price J-p-2131674"')
print('--------------------------------------------')
print(tree.xpath("//*[@class='p-price J-p-2131674']"))  # *指所有标签
print('')

print('--------------------------------------------')
print('# partial match ' + "//*[@class='J-p-2131674']")
print('--------------------------------------------')
print(tree.xpath("//*[@class='J-p-2131674']"))
print('')

print('--------------------------------------------')
print('# exactly match class string ' + '//*[@class="p-price J-p-2131674"]')
print('--------------------------------------------')
print(tree.xpath('//*[@class="p-price J-p-2131674"]'))
print('')

print('--------------------------------------------')
print('# use contain ' + "//*[contains(@class, 'J-p-2131674')]")
print('--------------------------------------------')
print(tree.xpath("//*[contains(@class, 'J-p-2131674')]"))
print('')

print('--------------------------------------------')
print('# specify tag name ' + "//strong[contains(@class, 'J-p-2131674')]")
print('--------------------------------------------')
print(tree.xpath("//strong[contains(@class, 'J-p-2131674')]"))  # 在strong标签中查找
print('')

print('--------------------------------------------')
print('# css selector with tag' + "cssselect('strong.J-p-2131674')")
print('--------------------------------------------')
htree = lxml.html.fromstring(content)
print(htree.cssselect('strong.J-p-2131674'))
print('')

print('--------------------------------------------')
print('# css selector without tag, partial match' + "cssselect('.J-p-2131674')")
print('--------------------------------------------')
htree = lxml.html.fromstring(content)
elements = htree.cssselect('.J-p-2131674')
print(elements)
print('')

print('--------------------------------------------')
print('# attrib and text')
print('--------------------------------------------')
for element in tree.xpath("//strong[contains(@class, 'J-p-2131674')]"):
    print(element.text)
    print(element.attrib)
print('')

print('--------------------------------------------')
print('########## use BeautifulSoup ##############')
print('--------------------------------------------')
print('# loading content to BeautifulSoup')
soup = BeautifulSoup(content, 'html.parser')
print('# loaded, show result')
print(soup.find(attrs={'class': 'J-p-2131674'}).text)

f.close()

```
豆瓣电影  
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
宽度优先爬取马蜂窝
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
爬取马蜂窝游记
```py
# coding=utf-8

import os
import re
import httplib
import urllib2
from pybloom import BloomFilter


request_headers = {
    'host': "www.mafengwo.cn",
    'connection': "keep-alive",
    'cache-control': "no-cache",
    'upgrade-insecure-requests': "1",
    'user-agent': "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.95 Safari/537.36",
    'accept': "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",
    'accept-language': "zh-CN,en-US;q=0.8,en;q=0.6"
}

city_home_pages = []
city_ids = []
dirname = 'mafengwo_notes/'

# 创建 Bloom Filter
download_bf = BloomFilter(1024 * 1024 * 16, 0.01)


def download_city_notes(id):
    for i in range(1, 999):
        url = 'http://www.mafengwo.cn/yj/%s/1-0-%d.html' % (id, i)
        if url in download_bf:
            continue
        print(('open url %s' % url))
        download_bf.add(url)
        req = urllib2.Request(url, headers=request_headers)
        response = urllib2.urlopen(req)
        htmlcontent = response.read()
        city_notes = re.findall('href="/i/\d{7}.html', htmlcontent)

        # 如果导航页错误，该页的游记数为0，则意味着 1-0-xxx.html 已经遍历完，结束这个城市
        if len(city_notes) == 0:
            return
        for city_note in city_notes:
            try:
                city_url = 'http://www.mafengwo.cn%s' % (city_note[6:])
                if city_url in download_bf:
                    continue
                print('download %s' % (city_url))
                req = urllib2.Request(city_url, headers=request_headers)
                response = urllib2.urlopen(req)
                html = response.read()
                filename = city_url[7:].replace('/', '_')
                fo = open("%s%s" % (dirname, filename), 'wb+')
                fo.write(html)
                fo.close()
                download_bf.add(city_url)
            except Exception as Arguments:
                print(Arguments)
                continue


# 检查用于存储网页文件夹是否存在，不存在则创建
if not os.path.exists(dirname):
    os.makedirs(dirname)

try:
    # 下载目的地的首页
    req = urllib2.Request('http://www.mafengwo.cn/mdd/', headers=request_headers)
    response = urllib2.urlopen(req)
    htmlcontent = response.read()

    # 利用正则表达式，找出所有的城市主页
    city_home_pages = re.findall('/travel-scenic-spot/mafengwo/\d{5}.html', htmlcontent)

    # 通过循环，依次下载每个城市下的所有游记
    for city in city_home_pages:
        city_ids.append(city[29:34])
        download_city_notes(city[29:34])
except urllib2.HTTPError as Arguments:
    print(Arguments)
except httplib.BadStatusLine:
    print('BadStatusLine')
except Exception as Arguments:
    print(Arguments)

```





