# Python爬虫

爬虫是自动获取互联网信息的程序，分为三步，**采集**，**抽取**，**存储**。

### 所使用的Python库

* requests：通过URL获取网页的库，可以发送HTTP请求，获得response对象。参考：[requests详解](https://blog.csdn.net/shanzhizi/article/details/50903748)
* re：re 库是 Python 中处理正则表达式的标准库。参考：[re详解](https://blog.csdn.net/hihell/article/details/114648366#:~:text=re%20%E5%BA%93%E6%98%AF%20Python%20%E4%B8%AD%E5%A4%84%E7%90%86%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F%E7%9A%84%E6%A0%87%E5%87%86%E5%BA%93%EF%BC%8C%E6%9C%AC%E7%AF%87%E5%8D%9A%E5%AE%A2%E4%BB%8B%E7%BB%8D,re%20%E5%BA%93%E7%9A%84%E5%90%8C%E6%97%B6%EF%BC%8C%E4%BC%9A%E7%AE%80%E5%8D%95%E4%BB%8B%E7%BB%8D%E4%B8%80%E4%B8%8B%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F%E8%AF%AD%E6%B3%95%EF%BC%8C%E5%A6%82%E6%9E%9C%E6%83%B3%E6%B7%B1%E5%85%A5%E5%AD%A6%E4%B9%A0%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F%EF%BC%8C%E8%BF%98%E9%9C%80%E8%A6%81%E5%A5%BD%E5%A5%BD%E4%B8%8B%E4%B8%80%E7%95%AA%E5%8A%9F%E5%A4%AB%E3%80%82%2013.1.1%20%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F%E8%AF%AD%E6%B3%95)
* BeautifulSoup4：
* lxml：能够处理XML和HTML的库



### 正则表达式

[工具网站](https://regexr.com/)

正则表达式使用单个字符串来描述、匹配一系列符合某个句法规则的字符串。其实常常使用的规则是很简单的，只要记住几个语法，然后借助上面的测试网站，就能很方便的写出来。

常用的技巧：

* `.`：匹配除换行符 \n 之外的任何单字符
* `\S`：匹配非空格字符
* `\s`：匹配空格字符
* `?`：代表前面的字符可以出现0次、或1次
* `*`：代表前面的字符可以出现0次、1次、或多次
* `+`：代表前面的字符必须至少出现一次
* `[]`：匹配[]里面的所有字符，只要出现就要匹配，不要求相连
* `^`：取非
* `.*?`：用来匹配一个字符串
* `[\S\s]*?`：匹配有回车的串
* `\s*`：匹配任意个空格

### DOM

文档对象模型（Document Object Model，简称DOM），是一种处理HTML和XML文件的标准API，就是把文档当成对象。DOM提供了**对整个文档的访问模型，将文档作为一个树形结构**，树的每个结点表示了一个HTML标签或标签内的文本项。DOM树结构精确地描述了HTML文档中标签间的相互关联性。将HTML或XML文档转化为DOM树的过程称为解析(parse)。HTML文档被解析后，转化为DOM树，因此对HTML文档的处理可以通过对DOM树的操作实现。DOM模型不仅描述了文档的结构，还定义了结点对象的行为，利用对象的方法和属性，可以方便地访问、修改、添加和删除DOM树的结点和内容。

### XPath

[参考](https://zhuanlan.zhihu.com/p/29436838)

XPath即为XML路径语言（XML Path Language），它是一种用来确定XML文档中某部分位置的语言。XPath基于XML的树状结构，提供在数据结构树中找寻节点的能力。在Python中的库有lxml，BeautifulSoup。

Chrome里面有一个XPath helper的插件。

* `/`：从当前节点选取直接子节点
* `//`：从当前节点选取子孙节点
* `.`：选取当前节点
* `..`：选取当前节点的父节点
* `@`：选取属性

相比于正则表达式，这个路径语言明显更加方便。

```python
import requests
from lxml import html

douban_crawler = MyCrawler('douban.txt')
content = douban_crawler.download('https://book.douban.com/tag/%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C')
tree = html.fromstring(content)

book_infos = tree.xpath("//li[@class='subject-item']")
for book_info in book_infos:
    print(book_info)
    book_name_elem = book_info.xpath('.//h2/a')[0]
    book_name = book_name_elem.text.strip()
    book_url = book_name_elem.attrib['href']
    book_pub_info = book_info.xpath(".//div[@class='pub']")[0].text.strip()
    book_intro = ''
    book_intro_elem = book_info.xpath(".//div[@class='info']/p")
    if book_intro_elem:
        book_intro = book_intro_elem[0].text.strip()
    print(book_name, book_url, book_pub_info)
    print(book_intro)
```





### 爬虫框架

不同的爬虫又不同的要求，有的只要求爬一个网页并保存其中的信息，有的要求能完成一些操作，有的要能不断的往后爬，直到爬完为止。所以根据不同的要求，就可以设置不同的操作，但是这些操作可以共享一个框架。

#### 框架1

首先对于爬一个比较简单的网页，比如在这里我们爬取中关村在线上面的一个[手机排行榜](https://wap.zol.com.cn/top/cell_phone/hot.html)，我们想要获取的就是手机，以及价格。那么我们的需求就很明确了，首先是获取网页，然后解析网页里面的数据，最后保存下来。所以大致的框架如下：

```Python
import requests
import re

class MyCrawler:
    def __init__(self, filename):
        self.filename = filename
        
    def download(self, url):
        r = requests.get(url)
        return r.text
    
    def extract(self, content, pattern):
        result = re.findall(pattern, content)
        return result
    
    def save(self, info):
        with open(self.filename, 'a', encoding = 'utf-8') as f:
            for item in info:
                f.write('|||'.join(item) + '\n')
    
    def crawl(self, url, pattern):
        content = self.download(url)
        info = self.extract(content, pattern)
        self.save(info)
```

写好的正则表达式就是：`<p class="pro-info-name f28">(.*?)<\/p> [\S\s]*? <span class="pro-info-price f24">(.*?)<\/span>`，调用方式就是：

```Python
crawler = MyCrawler('mobile.txt')
crawler.crawl(
  'https://wap.zol.com.cn/top/cell_phone/hot.html',
  '<p class="pro-info-name f28">(.*?)<\/p> [\S\s]*? <span class="pro-info-price f24">(.*?)<\/span>'
)
```

抓取的数据就保存在mobile.txt里。



#### 框架2

有些网站是用curl来获取网页的，如爬[豆瓣读书](https://book.douban.com/tag/%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C)，仅仅使用get()是得不到页面的（需要更改一些信息），这时候就需要用到将curl命令自动转换到python代码的网站了，如[curl转换](https://curl.trillworks.com/)，其实本质上就是在`requests.get()`方法里面再多加两个参数headers和cookies，最低限度就是在headers里面的`User-Agent`字段填上浏览器的标志字段，模拟出Chrome浏览器的访问或者Safari浏览器。

```python
import requests
import re

class MyCrawler:
    def __init__(self, filename):
        self.filename = filename
        self.headers = {
            'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.72 Safari/537.36',
        }
        
    def download(self, url):
        r = requests.get(url, headers=self.headers)
        return r.text
    
    def extract(self, content, pattern):
        result = re.findall(pattern, content)
        return result
    
    def save(self, info):
        with open(self.filename, 'a', encoding = 'utf-8') as f:
            for item in info:
                f.write(' '.join(item) + '\n')
    
    def crawl(self, url, pattern, headers=None):
        if headers:
            self.headers.update(headers)
        content = self.download(url)
        info = self.extract(content, pattern)
        self.save(info)
```



#### 框架3

怎么能自动的翻页呢？比如说一个页面有好几页，那么就可以分析第一页和第二页之间的url的差别，归纳总结出url的规律，然后再获取页面，解析，存取。



### lxml

> 参考：[主页](https://lxml.de/)



### BeautifulSoup4

> 参考：[文档](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html)

Beautiful Soup 是一个可以从HTML或XML文件中提取数据的Python库。使用的示例如下：

```python
from bs4 import BeautifulSoup

html_doc = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""
soup = BeautifulSoup(html_doc)
```

`BeautifulSoup(html_doc)` 返回一个BeautifulSoup对象，这个对象内置了一些方法。

