
* [欢迎来到爬虫笔记页面](#mulu)

<!-- /code_chunk_output -->
<!DOCTYPE html>
#<center><h1>爬虫</h1></center>
点击这里还可以去[我的博客主页](https://mrsyangjie.github.io/ "我的博客主页")o 
点击这里还可以去[大神的学习笔记](https://www.cnblogs.com/zhaof/tag/%E7%88%AC%E8%99%AB/default.html?page=2 "学习笔记")o 
<font face='宋体' > <span id='qianyan'><center>前言</center></span></font>
<center><p>本页只是为了方便本人以后复习爬虫用的笔记markdown</p></center>

<font color="#1c9999">纯属娱乐，如有雷同，打死不认*——*</font>

<center>
<img 
src='http://images2017.cnblogs.com/blog/764024/201802/764024-20180205162555654-1350503259.jpg'
width=500  / >
</center>

- - -

1. [什么是网络爬虫与爬虫实现原理](#1)
1. [Urllib简单的爬取网页](#2)
1. [编码问题](#3)
1. [requests库](#4)
1. [正则表达式和BeautifulSoup](#5)
1. [selenium](#6)
1. [cookie](#7)
1. [mongoDB](#8)
1. [py生成exe](#9)
- - - 
<font face='宋体' > <span id='1' color='#456526'><center>什么是网络爬虫与爬虫实现原理</center></span></font>
- 网络爬虫由 控制节点，爬虫节点，资源库构成，而在写爬虫中一定注意元组（）与列表的区别[]，字典{}和集合{}的区别
爬虫的流程
手动操作
分析页面数据
1查看源代码
2查看是否是 a jax异步加载
3解密 （非常熟悉前端，特别是js）
decode  解码 把其他编码转化为unicode decode('utf8')把utf-8转化为unicode
encode  编码  把unicod转化为其他编码  encode('gbk') 把unicode转化为gbk
可以这样理解unicode为中文 gbk为日文  utf-8为英文
转码只能转中文 所以就是   日文转英文 =就是 日转中转英  unicode就是像中间人
所以gbk怎样转为utf-8 ： decode('gbk').encode('utf8')
在这给自己补充下，我现在是主要用pycharm和vscode和jupytur notebook写python
pycharm的话，操作简单，写起来方便快捷，但是注意虚拟环境，这个可以参考Django.md，而且在pip时简单，如果错误就是原因是连接超时，所以需要自己设定安装源，
解决方法：
在 pip命令后自己设定收集源（-i +url）
eg：pip install requests -i http://pypi.douban.com/simple --trusted-host pypi.douban.com（通过豆瓣）就ok了

vscode的话轻便，需要创建一个文件夹，比如我的就是D:\新建文件夹，为什么在这个文件夹里面能写python因为已经在vscode里设置了详情自己百度，而且里面有一个D:\新建文件夹\.vscode\setting.josn，这就是关键
jupytur notebook 直接打开annocod但是也可以直接打开笔记本通过浏览器127.0.0.1:8888也可以
```
元组是一种序列类型，可以索引，而且可以用，和（）定义，当时元组是不可变的列表是可变的
列表是一种常见的数据结构，与元组相比，列表是可变的，len可以获得列表的长度，列表支持索引和切片，还可以append在末尾追加新的元素和extend在末尾追加新的序列（如元组或列表）方法，还支持pop,insert,remove,
字典是一种常用的数据结构，字典所以不同，使用（key）键进行索引，与键对应的值是值(value)，keys就是键和值，字典就是由若干键值对构成，并且键值都有引号，冒号分开
header={'host':www.adad .com'}
集合的特点是无序而且无重复元素，可以使用set()和大括号{}来初始化集合
s=set(['a','b'])
s={'a','b'}
enumerate是一种操作函数可以返回列表元组的索引和值
for ind ,val in enumerate([10,20,40])
print(ind,val)
...
0 10
1 20
2 40
...
在很长的代码中很有可能出问题，所以有异常处理机制
try:
    print('to do')
    a = 10/0
    print(a)
except Exception as e:
    print(e)
finally:
    print('我都执行')
    执行的结果为
    ---
to do
division by zero
我都执行
------
#那怎样主动抛出一个错误
来接着看：
class makeErr(ValueError):
    pass


def make_error(a):
    n = int(a)
    if n==0:
        raise makeErr('我是一个错误')
    return 1/n


make_error(0)
结果为
    make_error(0)
  File "D:/Python_code/pachong/pachong1.py", line 18, in make_error
    raise makeErr('我是一个错误')
__main__.makeErr: 我是一个错误
```
爬虫按照技术和结构分为通用网络爬虫，聚焦网络爬虫，增量式网络爬虫，深层网络爬虫等，实际网络爬虫通常是几种的组合体
一般都是由url集合，url队列，页面爬取，页面分析，页面数据库，链接过滤，内容评价，链接评价，想办法自动填写表单等
- 我们通过两种经典爬虫为例（通用爬虫和聚焦爬虫为例）分别记录他们的实现原理并且通过他们找到共同点
1. 通用网络爬虫
1获取初始的url，初始的url地址可以 由用户人为的指定，也可以由用户指定的某个或几个初始的网页决定
2根据初始的URL爬取页面并获得新的URL，将网页储蓄在原始的数据库中，并且在爬取网页中发现了新的url将已爬取的URL地址存放在一个URL列表中，再去重或者判断爬取的进程
3爬取新的URL列表里的URL并且有可能再爬取新URL中又再获得URL再加入列表，以此重复
4满足爬虫系统的停止条件后停止爬取
1. 聚焦网络爬虫
1由于其有目的的爬取，所以对于聚焦爬虫来说必须增加目标的定义和过滤机制还有下一句URL地址的选取等，所以第一步是做好爬取需求的定义和描述
2获取初始URL
3根据初始URL爬取并获得新的URL
4从新的URL过滤掉无用的，对有需要的放入URL列表，再去重和判断爬取的进程
5从URL列表中，根据搜索算法，确定URL优先级，并确定下一步要爬取的URL地址，在聚焦爬虫中，确定下一步的URL比较重要
然后又是从下一步的URL读取新的URL，并重复
6满足停止条件时停止

- - -
<font face='宋体' > <span id='2' color='#456526'><center>通过Urllib简单的爬取网页</center></span></font>

- - -

- 使用urllib库爬取网页的简单操作
```
在使用urllib前知道，当你获取一个URL你使用一个opener。在一般，我们都是使用的默认的opener，也就是urlopen。它是一个特殊的opener，可以理解成opener的一个特殊实例，传入的参数仅仅是url，data，timeout。
import urllib.request
url='http://www.baidu.com'
print('第一种')
response1=urllib.request.urlopen(url)
print (response1.getcode())
print (len(response1.read()))
结果为
第一种
200
153295
```
urllib思路也是大概一样
1首先，爬取一个网页并读出来赋给一个变量
2 以写入的方式打开一个文件，将值写入文件
3关闭文件
```
import urllib.request
file = urllib.request.urlopen('http://www.baidu.com')
data=file.read()
fhandle=open('D:/Python_code/pachong/handle/1.html','wb')
fhandle.write(data)
fhandle.close()
之前一直错因为我写错了
fhandle=open('D:D:\Python_code\pachong\handle\1.html','wb')
千万别复制路径，可能我的电脑这只原因，斜杠反的 一直错误
```
```
如果要进行编码和解码，比如我们对网址进行编码
import urllib.request
file = urllib.request.quote('http://www.baidu.com')
print(file)
quote=urllib.request.unquote('http%3A//www.baidu.com')
print(quote)
输出为
D:\Python_code\env_pacong\Scripts\python.exe D:/Python_code/pachong/pachong1.py
http%3A//www.baidu.com
http://www.baidu.com
下面再来看一个典型的我的错误
import urllib.request
url = 'http://www.baidu.com'
file=urllib.request.urlopen('url')
print(file.text)
错误一urlopen里面是参数时不用引号
错误二urlopen里没有text参数只有requests才有，别搞混了，urllib只有通过上面的read()才能读而
import requests
url = 'http://www.baidu.com'
file=requests.get(url)
print(file.text)
request这样就可以读
```
好了，大概的urllib流程也会了，现在来点不一样的吧
浏览器的模拟--Heathers属性
因为一些网站原因不能爬取，这时需要我们模拟网站访问
第一步打开你想要的网站点进去事件发生，然后检查或者f12找到user-Agent把它复制下来比如bing的就是
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36
```
import urllib.request
url ='http://blog.csdn.net/weiwei_pig/article/details/51178226'
req=urllib.request.Request(url)
req.add_header('user-agent','Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36')
data=urllib.request.urlopen(req).read()
fhandle=open('D:/Python_code/pachong/handle/3.html','wb')
fhandle.write(data)
fhandle.close()
然后打开3.html就有了
方式二
import urllib.request
url='http://blog.csdn.net/weiwei_pig/article/details/51178226'
headers=('user-agent','Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36')
opener=urllib.request.build_opener()
opener.addheaders=[headers]
data=opener.open(url).read()
fhandle=open('D:/Python_code/pachong/handle/4.html','wb')
fhandle.write(data)
fhandle.close()
一样有了
```
-简单的超时设置
file = urllib.request.urlopen('http://yum.iqianyue.com',timeout=1)

-http协议请求
1.GET请求
2.POST请求
3.PUT请求
4.DELETE请求
5.HEAD请求
- GET请求实例分析
keyworld = 'hello'
url = 'http://www.baidu.com/s?wd=' + keyworld
或者
kv={'wd':'python'}
r=urllib.request.urlopen('http://www.baidu.com/s',params=kv)
如果key是中文。编码可能带来问题，则需要解码
import urllib.request
url='http://www.baidu.com/s?wd='
key='阳'
key_code=urllib.request.quote(key)
url_all=url+key_code
req=urllib.request.Request(url_all)
data=urllib.request.urlopen(req).read()
print(data)或者可以写入文件也可以
- POST请求实例
import urllib.request
import urllib.parse
url='http://www.iqianyue.com/mypost/'
postdata=urllib.parse.urlencode({
    'name':'yangjie',
    'pass':'12345'
}).encode('utf-8')
req=urllib.request.Request(url,postdata)
req.add_header('user-agent','Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36')
data=urllib.request.urlopen(req).read()
fhand=open('D:/Python_code/pachong/handle/5.html','wb')
fhand.write(data)
fhand.close()

- -  -
<font face='宋体' > <span id='3' color='#456526'><center>编码问题</center></span></font>

>>>编码问题。
Python 默认脚本文件都是 ANSCII 编码的，当文件 中有非 ANSCII 编码范围内的字符的时候就要使用"编码指示"来修正一个 module 的定义中，如果.py文件中包含中文字符（严格的说是含有非anscii字符），则需要在第一行或第二行指定编码声明：# -*- coding=utf-8 -*- 或者 #coding=utf-8
python bytes和str两种类型转换的函数encode(),decode()
str通过encode()方法可以编码为指定的bytes
反过来，如果我们从网络或磁盘上读取了字节流，那么读到的数据就是bytes。要把bytes变为str，就需要用decode()方法：
是数据用decode不是连接url的response，
import urllib.request
file = urllib.request.urlopen('http://www.baidu.com')
data=file.read()
print(file.getcode())
结果为 200
我居然一开始打算用data.getcode()这个getcode()为状态码,只是用来检查你是否连接成功没有，而data是数据 。。。。。。。。。。。。。。。。
下面看经典的查看
import requests
r=requests.get('https://gs.amazon.cn/ref=as_cn_gs_topnav_reg?ld=AZCNAGSTopnav')
print(r.status_code)
回复
503
再用
print(r.encoding)
回复
'ISO-8859-1'
这时我们再用r.encoding=r.apparent_encoding
print(r.text)
就能看到返回提示错误，
：意外错误等
这就是网站不让我们爬虫浏览，所以就会有头部的修改
```
来一个错误
>>>import urllib.request
>>>file = urllib.request.urlopen('http://www.baidu.com')
data=file.read()
print(data)
结果为 这种错误，空格和换行被打印出了
b'<!DOCTYPE html>\n<!--STATUS OK-->\n\r\n\r\n\r\n\r\n\r\n\
\n\n\n<html>\n<head>\n    \n    <meta http-equiv="content-type" co
但是我改了一点之后
你看
import urllib.request
file = urllib.request.urlopen('http://www.baidu.com')
data=file.read()
print(data.decode())
就好了 ，没有了\n和\r,而且格式都整齐了如下
```
```
<html>
<head>
    <meta http-equiv="content-type" content="text/html;charset=utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=Edge">
   .....
后来经过百度
encode和decode分别指编码和解码
在python中，Unicode类型是作为编码的基础类型，即：

      decode                 encode
str ---------> str(Unicode) ---------> str
1
2
>>> u = '中文'                 # 指定字符串类型对象u 

>>> str1 = u.encode('gb2312')  # 以gb2312编码对u进行编码，获得bytes类型对象
>>> print(str1)
b'\xd6\xd0\xce\xc4'

>>> str2 = u.encode('gbk')     # 以gbk编码对u进行编码，获得bytes类型对象
>>> print(str2)
b'\xd6\xd0\xce\xc4'
>>> str3 = u.encode('utf-8')   # 以utf-8编码对u进行编码，获得bytes类型对象
>>> print(str3)
b'\xe4\xb8\xad\xe6\x96\x87'

>>> u1 = str1.decode('gb2312') # 以gb2312编码对字符串str进行解码，获得字符串类型对象
>>> print('u1')
'中文'

>>> u2 = str1.decode('utf-8')  # 报错，因为str1是gb2312编码的
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xd6 in position 0: invalid continuation byte
--------------------- 
可以看看这个
建立一个文件test.txt，文件格式用ANSI，内容为:"abc中文",用python来读取
# coding=gbk
print open("Test.txt").read()
结果：abc中文

把文件格式改成UTF-8：
结果：abc涓 枃

显然，这里需要解码：

# coding=gbk
import codecs
print open("Test.txt").read().decode("utf-8")
结果：abc中文
还有
with  open ('beautifulSoup_set.html','r',encoding='utf8') as f:
encoding为声明文件编码
 r.encoding=r.apparent_encoding一般这样就可以不用声明了
```
再来一个例子吧
```
逆水城殇
python爬虫解决gbk乱码问题
今天尝试了下爬虫，爬取一本小说，忘语的凡人修仙仙界篇，当然这样不好，大家要支持正版。

　　爬取过程中是老套路，先获取网页源代码　　

复制代码
# -*- coding:UTF-8 -*-
from bs4 import BeautifulSoup
import requests

if __name__ =='__main__':
    url='http://www.biquge.com.tw/18_18998/8750558.html'
    page_req=requests.get(url)
    html=page_req.text
    bf=BeautifulSoup( html)
    texts = bf.find_all('div',id='content')
    print(texts[0].text.replace('\xa0'*8,'\n\n'))
复制代码
　　结果：乱码



　　在浏览器看下代码，是gbk编码，需要进行转码，这方面不清楚，查了下资料。
　　PS：爬取的所有网页无论何种编码格式，都转化为utf-8格式进行存储，与源代码编码格式不同所以出现乱码

 

　　UTF-8通用性比较好，是用以解决国际上字符的一种多字节编码，它对英文使用8位（即一个字节），中文使用24位（三个字节）来编码。

　　UTF-8编码的文字可以在各国各种支持UTF8字符集的浏览器上显示，也就是必须两者都是utf-8才行。


　　gbk是是国家编码，通用性比UTF8差，GB2312之类的都算是gbk编码。

　　GBK包含全部中文字符；UTF-8则包含全世界所有国家需要用到的字符。


　　unicode是一种二进制编码，所有utf-8和gbk编码都得通过unicode编码进行转译，即utf-8和gbk编码之间不能直接转换。附图如下：



 

　　python中编码转换用到了两个函数decode（)和encode（）
　　比如：html=page_req.text.encode('iso-8859-1').decode('utf-8')
　　encode('iso-8859-1') 是将gbk编码编码成unicode编码
　　decode(‘gbk’) 是从unicode编码解码成gbk字符串

　　由于pycharm只能显示来自unicode的汉字，代码修改如下：

复制代码
# -*- coding:UTF-8 -*-
from bs4 import BeautifulSoup
import requests

if __name__ =='__main__':
    url='http://www.biquge.com.tw/18_18998/8750558.html'
    page_req=requests.get(url)
    html=page_req.text.encode('iso-8859-1')
    bf=BeautifulSoup( html)
    texts = bf.find_all('div',id='content')
    print(texts[0].text.replace('\xa0'*8,'\n\n'))
复制代码
最后解决：

最后再补充一点
raise_for_status()方法
理解Response类非常重要，Response这样的一个对象返回了所有的网页内容，那么它也提供了一个方法，叫raise_for_status()，这个方法是专门与异常打交道的方法，该方法有这样一个有趣的功能，它能够判断返回的Response类型状态是不是200。如果是200，他将表示返回的内容是正确的，如果不是200，他就会产生一个HttpError的异常。
那这个方法有什么用呢？
那我们来看一下我们的通用代码框架：
>>> def getHTMLText(url):
    try:
        r = requests.get(url, timeout = 30)
        r.raise_for_status()
        r.encoding = r.apparent_encoding
        return r.text
    except:
        return "产生异常"

这个代码中我们用r.raise_for_status()方法，它就可以有效的判断网络连接的状态。如果网连接出现错误，那么它就会用try-except来获取一个异常。而这个异常，在异常部分，我们用了一句 return “产生异常”  来表示，我们捕获到了这个异常，所以这样一个通用代码框架可以有效的处理，我们在访问或爬取网页过程中，它可能出现的一些错误，或者是网络不稳定造成的一些现象。
其他
我们就可以使用chardet.detect()方法，判断网页的编码方式了。至此，我们就可以编写一个小程序判断网页的编码方式了，新建文件名为chardet_test01.py：

# -*- coding: UTF-8 -*-
from urllib import request
import chardet

if __name__ == "__main__":
    response = request.urlopen("http://fanyi.baidu.com/")
    html = response.read()
    charset = chardet.detect(html)
    print(charset)
    结果是 utf-8
然后再解码就可以了
from urllib import request

if __name__ == "__main__":
    response = request.urlopen("http://www.fanyi.baidu.com/")
    html = response.read()
    html = html.decode("utf-8")
    print(html)

```
- - -
<font face='宋体' > <span id='4' color='#456526'><center>requests</center></span></font>
- - -
来看第一种requests库以及它怎样爬取一些基本的网页
>>> requests.request()构造一个请求
>>>requests.get()获取网页主要内容
requests.head()获取网页头部信息
还有post(),put(),delete()等
requests爬取的通用代码框架
import requests
def getHTMLText(url):
try:
  r=reqyests.get(url,timeout=30)
  r.raise_for_status()
  r.encoding=r.apparent_encoding
  return r.text
except:
  return '产生异常'
  (  r.raise_for_status()
  r.encoding=r.apparent_encoding
  后面有解释)
  详细解释下
  requests.request(method,url,**kwargs)
  **kwargs:控制访问的参数，为可选项
  params：字典或字节系列，作为参数增加到url中
  >>>kv = {'key1':'values1','key2':'values2'}
  >>>r=requests.request('GET','http://python123.io/ws',params=kv)
  print(r.url)
  结果为
https://python123.io/ws?key1=values1&key2=values2
如果要中文需要这样操作
body='主体内容'
body1=body.encode('utf8')
还可以读取文件
fs={'file':open('data.xls','rb')}
r=requests.request('POST','http://python123.io/ws',files=fs)
设定访问代理服务器
pxs={
  'http':'http://uesr:pass@10.10.10.1:1234'
  'http':'http://10.10.10.1:2323'}
  r-requests.request('GET','http://www.baidu.com',proxies=pxs)


```
看完了requests.request,来看下其他类型如get
def get_html(url):
    try:
        r=requests.get(url,timeout=20)
        r.raise_for_status()
        r.encoding=r.apparent_encoding
        return r.text
    except:
        print('错误')
    return ''
```
你也许经常想为 URL 的查询字符串(query string)传递某种数据。如果你是手工构建 URL，那么数据会以键/值对的形式置于 URL 中，跟在一个问号的后面。例如， httpbin.org/get?key=val。 Requests 允许你使用 params 关键字参数，以一个字符串字典来提供这些参数。举例来说，如果你想传递 key1=value1 和 key2=value2 到 httpbin.org/get ，那么你可以使用如下代码：

>>> payload = {'key1': 'value1', 'key2': 'value2'}
>>> r = requests.get("http://httpbin.org/get", params=payload)
通过打印输出该 URL，你能看到 URL 已被正确编码：

>>> print(r.url)
http://httpbin.org/get?key2=value2&key1=value1
注意字典里值为 None 的键都不会被添加到 URL 的查询字符串里。

你还可以将一个列表作为值传入：

>>> payload = {'key1': 'value1', 'key2': ['value2', 'value3']}
>>> r = requests.get('http://httpbin.org/get', params=payload)
>>> print(r.url)
http://httpbin.org/get?key1=value1&key2=value2&key2=value3
- 
>>> Requests 会自动解码来自服务器的内容。大多数 unicode 字符集都能被无缝地解码。
当你访问 r.text 之时，Requests 会使用其推测的文本编码。你可以找出 Requests 使用了什么编码，并且能够使用r.encoding 属性来改变它：

>>> r.encoding

'utf-8'
>>> r.encoding = 'ISO-8859-1'
如果你改变了编码，每当你访问 r.text ，Request 都将会使用 r.encoding 的新值。你可能希望在使用特殊逻辑计算出文本的编码的情况下来修改编码。比如 HTTP 和 XML 自身可以指定编码。这样的话，你应该使用 r.content 来找到编码，然后设置 r.encoding 为相应的编码。这样就能使用正确的编码解析 r.text 了。

 - Requests 中也有一个内置的 JSON 解码器，助你处理 JSON 数据：

>>> import requests

>>> r = requests.get('https://github.com/timeline.json')
>>> r.json()
[{u'repository': {u'open_issues': 0, u'url': 'https://github.com/...
如果 JSON 解码失败， r.json() 就会抛出一个异常。例如，响应内容是 401 (Unauthorized)，尝试访问 r.json() 将会抛出 ValueError: No JSON object could be decoded 异常。

需要注意的是，成功调用 r.json() 并**不**意味着响应的成功。有的服务器会在失败的响应中包含一个 JSON 对象（比如 HTTP 500 的错误细节）。这种 JSON 会被解码返回。要检查请求是否成功，请使用 r.raise_for_status() 或者检查 r.status_code 是否和你的期望相同。
>>> bad_r = requests.get('http://httpbin.org/status/404')
>>> bad_r.status_code
404
如果发送了一个错误请求(一个 4XX 客户端错误，或者 5XX 服务器错误响应)，我们可以通过 Response.raise_for_status() 来抛出异常：
>>> r.raise_for_status()
None

又如  一个网上的列子
1import requests
2 from bs4 import BeautifulSoup
3 
4 r = requests.get("http://www.baidu.com")
5 soup = BeautifulSoup(r.text,"html.parser")
6 
7 print(soup.title)

但是不知道出于什么原因，这段代码在同事电脑上运行良好，但是在我的电脑上运行 就会出现乱码。

然后又去请教，只要再补充上这段代码就可以完美运行了。

复制代码
1 import requests
2 from bs4 import BeautifulSoup
3 
4 r = requests.get("http://www.baidu.com")
5 r.raise_for_status()
6 r.encoding = r.apparent_encoding
7 soup = BeautifulSoup(r.text,"html.parser")
8 
9 print(soup.title)
这是因为r.apparent_ecoding使当前的encoding变得一致
详细可以参看后面
- - -

```
import requests
def get_content(url):
    resp=requests.get(url)
    return resp.text
url='http://jwc.sicnu.edu.cn/#'
content=get_content(url)
print('结果为',content)
````
或者加一个列表索引，以及其他操作
```
import requests
def get_content(url):
    respons=requests.get(url)
    return respons.text
    
url='https://www.baidu.com'
content=get_content(url)
content_len=len(content)
print('长度0-100',content[0:100])
print('长度为',content_len)
```
```
也可以通过同一个文件夹的形式调用其他文件的函数
如
import frist_get  //另一个文件
url='http://www.baidu.com.cn'
thor_content=frist_get.get_content(url)
但是最好引入if__name=='__main__':
避免重复或者其他问题
```
这个上面的都是requests的基本爬取网页内容，下面是将内容写入文件中
```
方式一
写入
f1=open('home_page.html','w',encoding='utf8')
f1.write(content)
f1.close
读取
f2=open(home_page.html','r',encoding='utf8')
content_read=f2.read()
f2.close()
方式二
写入
with open ('home_page.html','w',encoding='utf8')as f3:
    f3.write(content)
读取
with open('home_page.html','r',encoding='utf8')as f4:
    content_read_2=f4.read()
```
以上呢差别就是with语句会帮我们进行清理，而且不仅可以用于文件操作还能其他场景
下面再来看一个运用字典，列表，元组，集合，循环，等的爬取
```
import requests
url_dict={
    '电子工业出版社':'http://www.phei.com.cn/',
    '在线资源':'http://www.phei',
    'xyz':'www.baidu.com',
    '网上书店1':'http://phei.com.cn/module/goods/wssd_index.jsp',
    '网上书店2':'http://phei.com.cn/module/goods/wssd_index.jsp',
}
urls_list=[
    ('电子工业出版社','http://www.phei.com.cn/'),
     ('在线资源','http://www.phei'),
    ('xyz','www.baidu.com'),
    ('网上书店1','http://phei.com.cn/module/goods/wssd_index.jsp'),
    ('网上书店2','http://phei.com.cn/module/goods/wssd_index.jsp'),
]
crawled_url_for_dict=set()
for ind,name in enumerate(url_dict.keys()):
    name_url=url_dict[name]
    if name_url in crawled_url_for_dict:
        print(ind,name,'已结抓取过了')
    else:
        try:
            resp=requests.get(name_url)
        except Exception as e:
            print(ind,name,':',str(e)[0:50])
            continue
        content=resp.text
        crawled_url_for_dict.add(name_url)
        with open('bydict_'+name+',html','w',encoding='utf8')  as f:
            f.write(content)
            print('完成抓取:{} {},内容长度{}'.format(ind,name,len(content)))
for u in crawled_url_for_dict:
    print(u)
print('-'*50)

crawled_url_for_list=set()
for ind,tup in enumerate(urls_list):
    name=tup[0]
    name_url=tup[1]
    if name_url in crawled_url_for_list:
        print('已经打印过了')
    else:
        try:
            resp=requests.get(name_url)
        except Exception as e:
            print(ind,name,':',str(e)[0:50])
            continue
        content=resp.text
        crawled_url_for_list.add(name_url)
        with open('bylist'+name+'.html','w',encoding='utf8')as f:
            f.write(content)
            print('抓取完成:{}{},内容长度为{}'.format(ind,name,len(content)))
for u in crawled_url_for_list:
    print(u)
```
```
结果为
完成抓取:0 电子工业出版社,内容长度134
1 在线资源 : HTTPConnectionPool(host='www.phei', port=80): Max 
2 xyz : Invalid URL 'www.baidu.com': No schema supplied. P
3 网上书店1 : HTTPConnectionPool(host='phei.com.cn', port=80): M
4 网上书店2 : HTTPConnectionPool(host='phei.com.cn', port=80): M
http://www.phei.com.cn/
--------------------------------------------------
抓取完成:0电子工业出版社,内容长度为134
1 在线资源 : HTTPConnectionPool(host='www.phei', port=80): Max 
2 xyz : Invalid URL 'www.baidu.com': No schema supplied. P
3 网上书店1 : HTTPConnectionPool(host='phei.com.cn', port=80): M
4 网上书店2 : HTTPConnectionPool(host='phei.com.cn', port=80): M
http://www.phei.com.cn/
```
```
import requests
img_url='https://i0.hdslb.com/bfs/live/5934676.jpg@.webp?03132026'
response=requests.get(img_url)
print(response)
返回
<Response [200]>
```
定制请求头
如果你想为请求添加 HTTP 头部，只要简单地传递一个 dict 给 headers 参数就可以了。

例如，在前一个示例中我们没有指定 content-type:

>>> url = 'https://api.github.com/some/endpoint'
>>> headers = {'user-agent': 'my-app/0.0.1'}
>>> r = requests.get(url, headers=headers)
注意: 定制 header 的优先级低于某些特定的信息源，例如：

如果在 .netrc 中设置了用户认证信息，使用 headers= 设置的授权就不会生效。而如果设置了auth= 参数，``.netrc`` 的设置就无效了。
如果被重定向到别的主机，授权 header 就会被删除。
代理授权 header 会被 URL 中提供的代理身份覆盖掉。
在我们能判断内容长度的情况下，header 的 Content-Length 会被改写。
更进一步讲，Requests 不会基于定制 header 的具体情况改变自己的行为。只不过在最后的请求中，所有的 header 信息都会被传递进去。

注意: 所有的 header 值必须是 string、bytestring 或者 unicode。尽管传递 unicode header 也是允许的，但不建议这样做。

- 无法访问的网页可以通过设置报头
import requests
r=requests.get('https://gs.amazon.cn/ref=as_cn_gs_topnav_reg?ld=AZCNAGSTopnav')
print(r.status_code)
回复
503
再用
print(r.encoding)
回复
'ISO-8859-1'
这时我们再用r.encoding=r.apparent_encoding//这使得当前返回的代码变得是我们能看懂的代码返回，就算错误也会返回而且能看懂，如果是正确的就会返回规范正确的代码
print(r.text)
就能看到返回提示错误，
：意外错误等
这就是网站不让我们爬虫浏览，所以就会有头部的修改
修改添加如下
>>>kv={'uesr-agent':'Mozilla/5.0'}
url='https://gs.amazon.cn/ref=as_cn_gs_topnav_reg?ld=AZCNAGSTopnav'
r=requests.get(url,headers=kv)
r.status_code
返回200说明可以了
```
全部代码如下
import requests
url= 'http://www.amazon.cn/gp/product/B01M8L5Z3Y'
try:
    kv={'user-agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36'}
    r=requests.get(url,headers=kv)
    r.raise_for_status()
    r.encoding=r.apparent_encoding
    print(r.text[0:2000])
except:
    print('爬取失败')
    结果为
    
    <!doctype html><html class="a-no-js" data-19ax5a9jf="dingo">
    <head>
<script type="text/javascript">var ue_t0=ue_t0||+new Date();</script>
。。。。。。。等
连格式也是正确的
```
http协议请求
我们在浏览器用百度为例子输入hello
发现为https://www.baidu.com/s?wd=hello&rsv_spt=1&rsv_iqid=0xcbd3ca2000079cac&issp=1&f=8&rsv_bp=1&rsv_idx=2&ie=utf-8&tn=baiduhome_pg&rsv_enter=1&rsv_sug3=7&rsv_sug1=6&rsv_sug7=100
```
keyworld='hello'
 url='http://www.baidu.com/s?wd='+keyworld
 req=requests.get(url)
 print(req.text)
 或者这样
 import requests
kv={'wd':'python'}
r=requests.get('http://www.baidu.com/s',params=kv)
print(r.status_code)
print(r.request.url)
（requests模块发送请求有data、params两种携带参数的方法。
params在get请求中使用，data在post请求中使用。）
结果为
D:\Python_code\env_pacong\Scripts\python.exe D:/Python_code/pachong/pachong1.py
200
http://www.baidu.com/s?wd=python
```
下面将它写入文件 用requests
```
import requests
kv={'wd':'python'}
r=requests.get('http://www.baidu.com/s',params=kv)
r.encoding=r.apparent_encoding
with open('D:/Python_code/pachong/handle/2.html','wb')as f:
    f.write(r.text)
print('下载完成')
报错TypeError: a bytes-like object is required, not 'str'
 python bytes和str两种类型转换的函数encode(),decode()

str通过encode()方法可以编码为指定的bytes
反过来，如果我们从网络或磁盘上读取了字节流，那么读到的数据就是bytes。要把bytes变为str，就需要用decode()方法：
然而只需要这样就行了
import requests
kv={'wd':'python'}
r=requests.get('http://www.baidu.com/s',params=kv)
with open('D:/Python_code/pachong/handle/2.html','wb')as f:
    f.write(r.content)
print('下载完成')
然后打开2.html 是python 的百度页面  对的
   整个只需要把text换content
可能因为content拿到的是二进制的形式而text是字符串

```
用requests下载图片
```
import requests
import os
url='http://image.nationalgeographic.com.cn/2017/0211/20170211061910157.jpg'
root='D://pathon_code//pachong//'
文件根目录
path=root+url.split('/')[-1]
用最后命名
try:
    if not os.path.exists(root):
        os.madir(root)
        判断目录是否存在
    if not os.path.exists(path):
    判断路径是否存在
        r=requests.get(url)
        with open(path,'wb') as f:
            f.write(r.content)
            f.close()
            print('文件保持成功')
    else:
        print('文件已经存在')
except:
    print('爬取失败')
```
>>>下面将进行几个爬取例子
1.爬取京东商品信息
以下是我的代码
import requests
r=requests.get('https://item.jd.com/37230573868.html')
print(r.status_code)
print(r.encoding)
print(r.text[0:1000])
下面来看规范的代码
import requests
url='https://item.jd.com/37230573868.html'
try:
    r=requests.get(url)
    r.raise_for_status()
    r.encoding=r.apparent_encoding
    print(r.text[:1000])
except:
    print('爬取失败')

虽然结果都一样，但是这样通过 r.raise_for_status()排除了异常，如果返回的代码是200是不会有用的，否则就会产生异常，再通过try except这样程序健壮性增强许多，而且通过r.encoding=r.apparent_encoding使得返回的代码我们能看得懂，就算错误也会得到意外错误的代码返回

>>>下面来看爬取例子二
import requests
url= 'http://www.amazon.cn/gp/product/B01M8L5Z3Y'
try:
    kv={'user-agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36'}
    r=requests.get(url,headers=kv)
    r.raise_for_status()
    r.encoding=r.apparent_encoding
    print(r.text[0:2000])
except:
    print('爬取失败')
     

更加复杂的 POST 请求
通常，你想要发送一些编码为表单形式的数据——非常像一个 HTML 表单。要实现这个，只需简单地传递一个字典给 data 参数。你的数据字典在发出请求时会自动编码为表单形式：

>>> payload = {'key1': 'value1', 'key2': 'value2'}

>>> r = requests.post("http://httpbin.org/post", data=payload)
>>> print(r.text)
{
  ...
  "form": {
    "key2": "value2",
    "key1": "value1"
  },
  ...
}
你还可以为 data 参数传入一个元组列表。在表单中多个元素使用同一 key 的时候，这种方式尤其有效：

>>> payload = (('key1', 'value1'), ('key1', 'value2'))
>>> r = requests.post('http://httpbin.org/post', data=payload)
>>> print(r.text)
{
  ...
  "form": {
    "key1": [
      "value1",
      "value2"
    ]
  },
  ...
}
很多时候你想要发送的数据并非编码为表单形式的。如果你传递一个 string 而不是一个 dict，那么数据会被直接发布出去。

例如，Github API v3 接受编码为 JSON 的 POST/PATCH 数据：

>>> import json

>>> url = 'https://api.github.com/some/endpoint'
>>> payload = {'some': 'data'}
>>> r = requests.post(url, data=json.dumps(payload))
此处除了可以自行对 dict 进行编码，你还可以使用 json 参数直接传递，然后它就会被自动编码。这是 2.4.2 版的新加功能：

>>> url = 'https://api.github.com/some/endpoint'
>>> payload = {'some': 'data'}
>>> r = requests.post(url, json=payload)
POST一个多部分编码(Multipart-Encoded)的文件
Requests 使得上传多部分编码文件变得很简单：

>>> url = 'http://httpbin.org/post'
>>> files = {'file': open('report.xls', 'rb')}

>>> r = requests.post(url, files=files)
>>> r.text
{
  ...
  "files": {
    "file": "<censored...binary...data>"
  },
  ...
}
你可以显式地设置文件名，文件类型和请求头：

>>> url = 'http://httpbin.org/post'
>>> files = {'file': ('report.xls', open('report.xls', 'rb'), 'application/vnd.ms-excel', {'Expires': '0'})}

>>> r = requests.post(url, files=files)
>>> r.text
{
  ...
  "files": {
    "file": "<censored...binary...data>"
  },
  ...
}
如果你想，你也可以发送作为文件来接收的字符串：

>>> url = 'http://httpbin.org/post'
>>> files = {'file': ('report.csv', 'some,data,to,send\nanother,row,to,send\n')}

>>> r = requests.post(url, files=files)
>>> r.text
{
  ...
  "files": {
    "file": "some,data,to,send\\nanother,row,to,send\\n"
  },
  ...
}
如果你发送一个非常大的文件作为 multipart/form-data 请求，你可能希望将请求做成数据流。默认下 requests 不支持, 但有个第三方包 requests-toolbelt 是支持的。你可以阅读 toolbelt 文档来了解使用方法。

在一个请求中发送多文件参考 高级用法 一节。

警告
强烈建议你用二进制模式(binary mode)打开文件。这是因为 Requests 可能会试图为你提供 Content-Length header，在它这样做的时候，这个值会被设为文件的字节数（bytes）。如果用文本模式(text mode)打开文件，就可能会发生错误。

响应状态码
我们可以检测响应状态码：

>>> r = requests.get('http://httpbin.org/get')
>>> r.status_code
200
为方便引用，Requests还附带了一个内置的状态码查询对象：

>>> r.status_code == requests.codes.ok
True
如果发送了一个错误请求(一个 4XX 客户端错误，或者 5XX 服务器错误响应)，我们可以通过Response.raise_for_status() 来抛出异常：

>>> bad_r = requests.get('http://httpbin.org/status/404')
>>> bad_r.status_code

404

>>> bad_r.raise_for_status()

Traceback (most recent call last):
  File "requests/models.py", line 832, in raise_for_status
    raise http_error
requests.exceptions.HTTPError: 404 Client Error
但是，由于我们的例子中 r 的 status_code 是 200 ，当我们调用 raise_for_status() 时，得到的是：

>>> r.raise_for_status()
None
一切都挺和谐哈。

响应头
我们可以查看以一个 Python 字典形式展示的服务器响应头：

>>> r.headers
{
    'content-encoding': 'gzip',
    'transfer-encoding': 'chunked',
    'connection': 'close',
    'server': 'nginx/1.0.4',
    'x-runtime': '148ms',
    'etag': '"e1ca502697e5c9317743dc078f67693f"',
    'content-type': 'application/json'
}
但是这个字典比较特殊：它是仅为 HTTP 头部而生的。根据 RFC 2616， HTTP 头部是大小写不敏感的。

因此，我们可以使用任意大写形式来访问这些响应头字段：

>>> r.headers['Content-Type']
'application/json'

>>> r.headers.get('content-type')
'application/json'
它还有一个特殊点，那就是服务器可以多次接受同一 header，每次都使用不同的值。但 Requests 会将它们合并，这样它们就可以用一个映射来表示出来，参见 RFC 7230:

A recipient MAY combine multiple header fields with the same field name into one "field-name: field-value" pair, without changing the semantics of the message, by appending each subsequent field value to the combined field value in order, separated by a comma.

接收者可以合并多个相同名称的 header 栏位，把它们合为一个 "field-name: field-value" 配对，将每个后续的栏位值依次追加到合并的栏位值中，用逗号隔开即可，这样做不会改变信息的语义。

Cookie
如果某个响应中包含一些 cookie，你可以快速访问它们：

>>> url = 'http://example.com/some/cookie/setting/url'
>>> r = requests.get(url)

>>> r.cookies['example_cookie_name']
'example_cookie_value'
要想发送你的cookies到服务器，可以使用 cookies 参数：

>>> url = 'http://httpbin.org/cookies'
>>> cookies = dict(cookies_are='working')

>>> r = requests.get(url, cookies=cookies)
>>> r.text
'{"cookies": {"cookies_are": "working"}}'
Cookie 的返回对象为 RequestsCookieJar，它的行为和字典类似，但界面更为完整，适合跨域名跨路径使用。你还可以把 Cookie Jar 传到 Requests 中：

>>> jar = requests.cookies.RequestsCookieJar()
>>> jar.set('tasty_cookie', 'yum', domain='httpbin.org', path='/cookies')
>>> jar.set('gross_cookie', 'blech', domain='httpbin.org', path='/elsewhere')
>>> url = 'http://httpbin.org/cookies'
>>> r = requests.get(url, cookies=jar)
>>> r.text
'{"cookies": {"tasty_cookie": "yum"}}'
重定向与请求历史
默认情况下，除了 HEAD, Requests 会自动处理所有重定向。

可以使用响应对象的 history 方法来追踪重定向。

Response.history 是一个 Response 对象的列表，为了完成请求而创建了这些对象。这个对象列表按照从最老到最近的请求进行排序。

例如，Github 将所有的 HTTP 请求重定向到 HTTPS：

>>> r = requests.get('http://github.com')

>>> r.url
'https://github.com/'

>>> r.status_code
200

>>> r.history
[<Response [301]>]
如果你使用的是GET、OPTIONS、POST、PUT、PATCH 或者 DELETE，那么你可以通过 allow_redirects 参数禁用重定向处理：

>>> r = requests.get('http://github.com', allow_redirects=False)
>>> r.status_code
301
>>> r.history
[]
如果你使用了 HEAD，你也可以启用重定向：

>>> r = requests.head('http://github.com', allow_redirects=True)
>>> r.url
'https://github.com/'
>>> r.history
[<Response [301]>]
超时
你可以告诉 requests 在经过以 timeout 参数设定的秒数时间之后停止等待响应。基本上所有的生产代码都应该使用这一参数。如果不使用，你的程序可能会永远失去响应：

>>> requests.get('http://github.com', timeout=0.001)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
requests.exceptions.Timeout: HTTPConnectionPool(host='github.com', port=80): Request timed out. (timeout=0.001)
注意
timeout 仅对连接过程有效，与响应体的下载无关。 timeout 并不是整个下载响应的时间限制，而是如果服务器在 timeout 秒内没有应答，将会引发一个异常（更精确地说，是在timeout 秒内没有从基础套接字上接收到任何字节的数据时）If no timeout is specified explicitly, requests do not time out.

错误与异常
遇到网络问题（如：DNS 查询失败、拒绝连接等）时，Requests 会抛出一个 ConnectionError 异常。

如果 HTTP 请求返回了不成功的状态码， Response.raise_for_status() 会抛出一个 HTTPError异常。

若请求超时，则抛出一个 Timeout 异常。

若请求超过了设定的最大重定向次数，则会抛出一个 TooManyRedirects 异常。

所有Requests显式抛出的异常都继承自 requests.exceptions.RequestException 。



- - -


- - - 
<font face='宋体' > <span id='5' color='#456526'><center>正则和BeautifulSoup的解析</center></span></font>
-
                正则表达式
两个特殊的符号'^'和'$'。他们的作用是分别指出一个字符串的开始和结束。例子如下：

"^The"：表示所有以"The"开始的字符串（"There"，"The cat"等）；
"of despair$"：表示所以以"of despair"结尾的字符串；
"^abc$"：表示开始和结尾都是"abc"的字符串——呵呵，只有"abc"自己了；
"notice"：表示任何包含"notice"的字符串。

象最后那个例子，如果你不使用两个特殊字符，你就在表示要查找的串在被查找串的任意部分——你并
不把它定位在某一个顶端。

其它还有'*'，'+'和'?'这三个符号，表示一个或一序列字符重复出现的次数。它们分别表示“没有或
更多”，“一次或更多”还有“没有或一次”。下面是几个例子：

"ab*"：表示一个字符串有一个a后面跟着零个或若干个b。（"a", "ab", "abbb",……）；
"ab+"：表示一个字符串有一个a后面跟着至少一个b或者更多；
"ab?"：表示一个字符串有一个a后面跟着零个或者一个b；
"a?b+$"：表示在字符串的末尾有零个或一个a跟着一个或几个b。

你也可以使用范围，用大括号括起，用以表示重复次数的范围。

"ab{2}"：表示一个字符串有一个a跟着2个b（"abb"）；
"ab{2,}"：表示一个字符串有一个a跟着至少2个b；
"ab{3,5}"：表示一个字符串有一个a跟着3到5个b。

请注意，你必须指定范围的下限（如："{0,2}"而不是"{,2}"）。还有，你可能注意到了，'*'，'+'和
'?'相当于"{0,}"，"{1,}"和"{0,1}"。
还有一个'¦'，表示“或”操作：

"hi¦hello"：表示一个字符串里有"hi"或者"hello"；
"(b¦cd)ef"：表示"bef"或"cdef"；
"(a¦b)*c"：表示一串"a""b"混合的字符串后面跟一个"c"；

'.'可以替代任何字符：

"a.[0-9]"：表示一个字符串有一个"a"后面跟着一个任意字符和一个数字；
"^.{3}$"：表示有任意三个字符的字符串（长度为3个字符）；

方括号表示某些字符允许在一个字符串中的某一特定位置出现：
```
"[ab]"：表示一个字符串有一个"a"或"b"（相当于"a¦b"）；
"[a-d]"：表示一个字符串包含小写的'a'到'd'中的一个（相当于"a¦b¦c¦d"或者"[abcd]"）；
"^[a-zA-Z]"：表示一个以字母开头的字符串；
"[0-9]%"：表示一个百分号前有一位的数字；
",[a-zA-Z0-9]$"：表示一个字符串以一个逗号后面跟着一个字母或数字结束。
```
你也可以在方括号里用'^'表示不希望出现的字符，'^'应在方括号里的第一位。（如："%[^a-zA-Z]%"表示两个百分号中不应该出现字母）。

为了逐字表达，你必须在"^.$()¦*+?{\"这些字符前加上转移字符'\'。

请注意在方括号中，不需要转义字符。

2.正则表达式验证控制文本框的输入字符类型
```
1.只能输入数字和英文的： 
<input onkeyup="value=value.replace(/[\W]/g,'') " onbeforepaste="clipboardData.setData('text',clipboardData.getData('text').replace(/[^\d]/g,''))" ID="Text1" NAME="Text1">

2.只能输入数字的： 
<input onkeyup="value=value.replace(/[^\d]/g,'') " onbeforepaste="clipboardData.setData('text',clipboardData.getData('text').replace(/[^\d]/g,''))" ID="Text2" NAME="Text2">

3.只能输入全角的： 
<input onkeyup="value=value.replace(/[^\uFF00-\uFFFF]/g,'')" onbeforepaste="clipboardData.setData('text',clipboardData.getData('text').replace(/[^\uFF00-\uFFFF]/g,''))" ID="Text3" NAME="Text3">

4.只能输入汉字的： 
<input onkeyup="value=value.replace(/[^\u4E00-\u9FA5]/g,'')" onbeforepaste="clipboardData.setData('text',clipboardData.getData('text').replace(/[^\u4E00-\u9FA5]/g,''))" ID="Text4" NAME="Text4">
```

3.正则表达式的应用实例通俗说明
 
*******************************************************************************

//校验是否全由数字组成
```
/^[0-9]{1,20}$/
```
 

^ 表示打头的字符要匹配紧跟^后面的规则

$ 表示打头的字符要匹配紧靠$前面的规则

[ ] 中的内容是可选字符集

[0-9] 表示要求字符范围在0-9之间

{1,20}表示数字字符串长度合法为1到20，即为[0-9]中的字符出现次数的范围是1到20次。

 

/^ 和 $/成对使用应该是表示要求整个字符串完全匹配定义的规则，而不是只匹配字符串中的一个子串。

 

*******************************************************************************

//校验登录名：只能输入5-20个以字母开头、可带数字、“_”、“.”的字串
```
/^[a-zA-Z]{1}([a-zA-Z0-9]|[._]){4,19}$/
```
 
```
^[a-zA-Z]{1} 表示第一个字符要求是字母。
```
```
([a-zA-Z0-9]|[._]){4,19} 
```
表示从第二位开始（因为它紧跟在上个表达式后面）的一个长度为4到9位的字符串，它要求是由大小写字母、数字或者特殊字符集[._]组成。

 

*******************************************************************************

//校验用户姓名：只能输入1-30个以字母开头的字串
```
/^[a-zA-Z]{1,30}$/
```
 

*******************************************************************************

//校验密码：只能输入6-20个字母、数字、下划线
```
/^(\w){6,20}$/
```
 

\w：用于匹配字母，数字或下划线字符


//校验普通电话、传真号码：可以“+”或数字开头，可含有“-” 和 “ ”
```
/^[+]{0,1}(\d){1,3}[ ]?([-]?((\d)|[ ]){1,12})+$/
```
 

\d：用于匹配从0到9的数字；

“?”元字符规定其前导对象必须在目标对象中连续出现零次或一次

 

可以匹配的字符串如：+123 -999 999 ； +123-999 999 ；123 999 999 ；+123 999999等



//校验URL
```
/^http[s]{0,1}:\/\/.+$/ 或 /^http[s]{0,1}:\/\/.{1,n}$/ (表示url串的长度为length(“https://”) + n )
\ / ：表示字符“/”。
. 表示所有字符的集
+ 等同于{1,}，就是1到正无穷吧。

再来一个检验以.com .cn结尾url
import re
pattern='[a-zA-Z]+://[^\s]*[.com|.cn]'
string="<a href='http://www.baidu.com'>百度首页</a>"
result=re.search(pattern,string)
print(result)
\s为空白字符
[^\s]则为非空白字符
*匹配一次或多次前面的字符
```

下面再来看一个例子
```
import re
pattern1='p.*y' #贪婪模式
patton2='p.*?y' #懒惰模式
string ='abcdfphp124pythony_py'
result1=re.search(pattern1,string)
result2=re.search(patton2,string)
print(result1)
print(result2)
结果为
D:\Python_code\env_pacong\Scripts\python.exe D:/Python_code/pachong/pachong1.py
<re.Match object; span=(5, 21), match='php124pythony_py'>
<re.Match object; span=(5, 13), match='php124py'>
```
```
import re
string='adadjgdfsdkdasadfksskaasadadyfhlmbnlvzxxaewrtl'
pattern='.a.'
result=re.compile(pattern).findall(string)
print(result)
结果为
['dad', 'das', 'kaa', 'sad', 'xae']
```
再来看一个京东商品的经典例子
```
import urllib.request
import re
from bs4 import BeautifulSoup


def crawl(url,page):
    headers={'url-agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36'}
    req=urllib.request.Request(url=url,headers=headers)
    html=urllib.request.urlopen(req).read()
    html1=html.decode() #格式变得标准化。解码
    part1 = r'<div id="J_goodsList" .+?<div class="m-aside">'
    result1 = re.compile(part1,flags=re.DOTALL).findall(html1)
    result1=result1[0] #使数据变得工整而且格式化了 么以偶了/n等重复
    part2=r'<img width="220" height="220" class="err-product" data-img="1" source-data-lazy-img="//(.+?\.jpg)" />'
    imagelist=re.compile(part2).findall(result1)
    x=1
    for imageurl in imagelist:
        imagename="D:/python_code/pachong/handle/jd_image/"+str(page)+str(x)+'.jpg'
        imageurl='http://'+imageurl
        try:
            urllib.request.urlretrieve(imageurl,filename=imagename)
        except urllib.error.URLError as e:
            if hasattr(e,'code'):
                x+=1
            if hasattr(e,'reason'):
                x+=1
        x+=1

for i in range(1,5):
    url='https://search.jd.com/Search?keyword=iphonex&enc=utf-8&qrst=1&rt=1&stop=1&vt=2&suggest=1.his.0.0&page='+str(i)
    crawl(url,i)
    print('第i爬取成功')

经过艰难的终于写好这用第一个爬虫爬取图片
下面回顾下写的时候的问题
1正则表达式出现了问题
我在京东商品的页面源代码用ctrl+f也找了
下面的标志只有一个也就是只有一次出现正好满足条件
part1 = r'<div id="J_goodsList" .+?<div class="m-aside">'
result1 = re.compile(part1).findall(html1)
一开始我是这样写的，但是一直返回我一个[]然后百度很久终于知道为什么
这个问题一般出现在希望使用句点(.)来匹配任意字符，但是忘记了句点并不能匹配换行符时:
import re
comment = re.compile(r'/\*(.*?)\*/')  # 匹配C的注释
text1 = '/* this is a comment */'
text2 = """/*this is a 
    multiline comment */"""

comment.findall(text1)
Out[4]: [' this is a comment ']

comment.findall(text2)  # 由于text2文本换行了，没匹配到
Out[5]: []

解决方法1：添加对换行符的支持，(?:.|\n)指定了一个非捕获组(即，这个组只做匹配但不捕获结果，也不会分配组号)
comment = re.compile(r'\*((?:.|\n)*?)\*/') 
comment.findall(text2)
Out[7]: ['this is a \n    multiline comment ']

解决方法2：re.compile()函数可接受一个有用的标记--re.DOTALL。这使得正则表达式中的句点(.)可以匹配所有的字符，也包括换行符
comment = re.compile(r'/\*(.*?)\*/', flags=re.DOTALL)
comment.findall(text2)
Out[10]: ['this is a \n    multiline comment ']
很明显我就是用的第二种然后终于正则表达式成功的匹配了我想要的
爬取了想要的大概内容但是还不是我想要的图片url
但是发现每个页面的图片URL都在<img width="220" height="220" class="err-product" data-img="1" source-data-lazy-img="//img12.360buyimg.com/n7/jfs/t1/38769/36/65/76968/5cb83ffeEe5900ad2/ee8924e4c5a0fd03.jpg" />
这里面，我用ctrl+f搜索source-data-lazy-img=找到他们
但是怎么用正则表达式找出他们那，
经过排查发现所有的图片链接都一样除了img12.360buyimg.com/n7/jfs/t1/38769/36/65/76968/5cb83ffeEe5900ad2/ee8924e4c5a0fd03.jpg"
这一段，所以这一段用.+?代替，而且千万别这里也用flags=re.DOTALL
会多一些莫名其妙的数据
如果是这样
   part2=r'<img width="220" height="220" class="err-product" data-img="1" source-data-lazy-img="//(.+?).jpg" />'
    imagelist=re.compile(part2).findall(result1)
    print(imagelist)

那结果为
['img14.360buyimg.com/n7/jfs/t1/4528/10/3590/153299/5b997bf5E4a513949/45ab3dd6c35d981b', 'img13.360buyimg.com/n7/jfs/t10675/253/1344769770/66891/92d54ca4/59df2e7fN86c99a27',
这没有后面的.jpg，
所以我用这样
 part2=r'<img width="220" height="220" class="err-product" data-img="1" source-data-lazy-img="//(.+?\.jpg)" />'
    imagelist=re.compile(part2).findall(result1)
    print(imagelist)
结果就是正确的咯
    ['img13.360buyimg.com/n7/jfs/t10675/253/1344769770/66891/92d54ca4/59df2e7fN86c99a27.jpg', 'img13.360buyimg.com/n7/jfs/t10690/249/1626659345/69516/b3643998/59e4279aNff3d63ac.jpg',
但是需要转义\因为有特殊字符.而且需要一个（）把正则表达式和.jpg一起

```
```
爬取链接
import re
import urllib.request
def getlink(url):
    headers={'user-agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36'}
    req=urllib.request.Request(url=url,headers=headers)
    html=urllib.request.urlopen(req).read().decode()
    print('================================')
    part='(https?://[^\s)";]+\.(\w|/)*)'
    #?指https或者http
    #[^\s)";]指不可以出现空格\s不出现双引号，分号
    link=re.compile(part).findall(html)
    linklist=list(set(link))
    for link in linklist:
        print(link[0])
<!-- 修改
part='((https|http)://[^\s)";]+\.(\w|/)*)'
匹配更多链接 -->


url='http://blog.csdn.net/'
getlink(url)
```
```
爬取糗事百科
import urllib.request
import re
import urllib.request
def getcontent(url,i):
     headers={'user-agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36'}
     req=urllib.request.Request(url=url,headers=headers)
     html=urllib.request.urlopen(req).read().decode()
     print('================================')
     part1='<div class="content">(.*?)</span>'
     content=re.compile(part1,flags=re.DOTALL).findall(html)
     print(content)


for i in range(1,5):
    url='https://www.qiushibaike.com/text/page/'+str(i)
    getcontent(url,i)
```
           BeautifulSoup库解析html/xml文档
首先BeautifulSoup有四种解析库，
1.python标准库 使用方法BeautifulSoup(markup,'html.parser')
2.lxml HTML解析器 使用方法BeautifulSoup(markup,'lxml')速度快，文档容错能力强，需要安装c语言库（推荐使用）
3 . lxml XML解析器 使用方法 BeautifulSoup(markup,'xml;)唯一支持XML的解析器，需要安装c语言库
4.html5lib 使用方法BeautifulSoup(markup,'hml5lib')最后的容错性，以浏览器的方式解析文档，生成html5格式，速度慢

- 基本使用
建议使用lxml
from bs4 import BeautifulSoup
soup=BeautifulSoup(html,'lxml')
print(soup.prettify())
格式化和自动补齐
print(soup.title.string)
输出title 的内容
下面介绍常用的选择器
- - -
- 第一种 ---标签选择器
1 .获取标签
如选择标签a li ul class p等或者子节点或者父节点等
```
import requests
from bs4 import BeautifulSoup
r=requests.get('http://python123.io/ws/demo.html')
demo=r.text
soup =BeautifulSoup(demo,'html.parser')
这里指定解析器是html的parser
print(soup.prettify())
prettify()函数能为html做换行符也能为每一个标签做格式
结果为
<html>
 <head>
  <title>
   This is a python demo page
  </title>
 </head>
 <body>
  <p class="title">
   <b>
    The demo python introduces several python courses.
   </b>
  </p>
  <p class="course">
   Python is a wonderful general-purpose programming language. You can learn Python from novice to professional by tracking the following courses:
   <a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">
    Basic Python
   </a>
   and
   <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">
    Advanced Python
   </a>
   .
  </p>
 </body>
</html>

```
如果是打开文件的格式就为
soup2=BeautifulSoup(open('D://demo.html'),'html.parser')
```
下面详细看看BeautifulSoup的对于html的便签
import requests
from bs4 import BeautifulSoup
r=requests.get('http://python123.io/ws/demo.html')
demo=r.text
soup =BeautifulSoup(demo,'html.parser')
print(soup.title)
print('========================')
print(soup.a)
print('========================')
print(soup.a.name)
print('========================')
print(soup.a.parent.name)
print('========================')
print(soup.a.parent.parent.name)
结果为
<title>This is a python demo page</title>
========================
<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>
========================
a
========================
p
========================
body
```
```
BeautifulSoup实际上是以树形图解释
如
下行遍历
import requests
from bs4 import BeautifulSoup
r=requests.get('http://python123.io/ws/demo.html')
demo=r.text
soup =BeautifulSoup(demo,'html.parser')
print(soup.head)
结果为
<head><title>This is a python demo page</title></head>
若为
print(soup.head.contents)
结果为
[<title>This is a python demo page</title>]
平行遍历
import requests
from bs4 import BeautifulSoup
r=requests.get('http://python123.io/ws/demo.html')
demo=r.text
soup =BeautifulSoup(demo,'html.parser')
print(soup.a)
print('============')
print(soup.a.next_sibling)
print('=========')
print(soup.a.next_sibling.next_sibling)
结果为
<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>
============
 and 
=========
<a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>
```

 2. 获取属性
```
import requests
from bs4 import BeautifulSoup
r=requests.get('http://python123.io/ws/demo.html')
demo=r.text
soup =BeautifulSoup(demo,'html.parser')
tag=soup.a
print(tag.attrs)
结果为
{'href': 'http://www.icourse163.org/course/BIT-268001', 'class': ['py1'], 'id': 'link1'}
他是以字典返回
所以可以这样
print(soup.find_all(attrs={"id":"1"}))

在lxml也可以这样
print(soup.find_all(attrs={id="1"}))
如果是关键字得加_如print(soup.find_all(attrs={class_="1"}))
也可以使用[]
如
print(soup.p.attrs['name'])=print(soup.p['name'])

```
bs4的信息提取
```
说完了大概方式 下面来一个例子爬取所有的url链接
import requests
from bs4 import BeautifulSoup
r=requests.get('http://python123.io/ws/demo.html')
demo=r.text
soup =BeautifulSoup(demo,'html.parser')
for link in soup.find_all('a'):
    print(link.get('href'))
结果为
http://www.icourse163.org/course/BIT-268001
http://www.icourse163.org/course/BIT-1001870001
当然还可以结合re正则表达式
import requests
import re
from bs4 import BeautifulSoup
r=requests.get('http://python123.io/ws/demo.html')
demo=r.text
soup =BeautifulSoup(demo,'html.parser')
for tag in soup.find_all(re.compile('b')):
    print(tag.name)
结果为
body
b
完美配合找出b的标签名字
当然还可以通过这样来查找
print(soup.find_all(id='link1'))
结果为
[<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>]
```
最后 bs4的编码为utf8所以与python3.上完美符合就算中文也能无需解决编码问题

 lxml解析器中也支持css选择器
运用select选择‘
如
print(soup.select('ul li'))
中间一个空格隔开
若一个id='list-2'直接可以选择内容 --加#
print(soup.select('#list-2.element'))
也可以用中括号获得属性
print(soup.select(ul['id']))
获得内容也可以用get_text()
print(soup.select(li.get_text()))

也可以拖过这样查找
```
   soup = BeautifulSoup(response.text, "html.parser")
    student_number = soup.find("span", id="lblStdNum")
    源代码如下
     <td align="center" style="font-size: 12px;">
                        学&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 号：
                    </td>
                    <td>
                        <span id="lblStdNum" style="font-size:12px;"></span>
                    </td>
                    最后再student['number'] = student_number.text
   
                    就能过的学号
```
```
                           XPATH
首先，python 具有一些比较流行的解析库,例如 lxml , 使用的是 XPath 语法，是大众普遍认为的网页文本信息提取的爬虫利器之一。

一. 关于 XPath
XPath 是 XML路径语言（XML Path Language），支持 HTML，是一种用来确定XML文档中某部分位置的语言。XPath基于XML的树状结构，提供在数据结构树中查找节点的能力。Xpath 可以通过元素和属性进行导航，相比 正则表达式，它同样可以在 XML 文档中查询信息，甚至使用起来更加简单高效。
对于一个 XML 文件( HTML 文件可以通过 etree.HTML()方法 转为这种格式）
如：# 使用etree对html解析
    tree = etree.HTML(pageCode)
    表达式	描述
nodename	选取此节点的所有子节点。
/	从根节点选取。
//	从匹配选择的当前节点选择文档中的节点，而不考虑它们的位置。
.	选取当前节点。
..	选取当前节点的父节点。
@	选取属性
！
下面再来看看几个实例并理解xpath
import requests
from lxml import etree


# 设计模式 --》面向对象编程
class Spider(object):
    def __init__(self):
        # 反反爬虫措施，加请求头部信息
        self.headers = {
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36",
            "Referer": "https://www.mzitu.com/xinggan/"
        }


    def xpath_data(self, html):
        # 2. 抽取想要的数据 标题 图片 xpath
        src_list = html.xpath('//ul[@id="pins"]/li/a/img/@data-original')
        alt_list = html.xpath('//ul[@id="pins"]/li/a/img/@alt')
        for src, alt in zip(src_list, alt_list):
            file_name = alt + ".jpg"
            response = requests.get(src, headers=self.headers)

            # 3. 存储数据 jpg with open
            try:
                with open('D:/Python_code/pachong/handle/mzitu/'+file_name, "wb") as f:
                    f.write(response.content)
                    print("正在抓取图片：" + file_name)
            except:
                print("==========文件名有误！==========")

    def start_request(self):
        # 1. 获取整体网页的数据 requests
        for i in range(1, 5):
            print("==========正在抓取%s页==========" % i)
            response = requests.get("https://www.mzitu.com/page/" + str(i) + "/", headers=self.headers)
            html = etree.HTML(response.content.decode())
            self.xpath_data(html)

spider = Spider()
spider.start_request()
和
import requests
import threading
from bs4 import BeautifulSoup
import re
from lxml import etree

class tieba(object):
    #1初始化,传参作用等
    def __init__(self,query_string):
        self.query_string=query_string
        self.url='https://tieba.baidu.com/f'
        self.headers={'user-agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.131 Safari/537.36'}

    #2
    def parse_data(self,data,rule):
        html_data=etree.HTML(data)
        data_list=html_data.xpath(rule)
        return data_list


    #3
    def seve_data(self,data,name):
        path='D:/Python_code/pachong/handle/tieba/'+name
        with open (path,'wb') as f:
            f.write(data)
            print("%s爬取完成" %name )
    #4
    def params(self):
        para={
            'kw':self.query_string
        }
        return para

    def send_request(self,url,parms={}):
        response=requests.get(url,params=parms,headers=self.headers)
        return response.content


    def run(self):
        tieba_params=self.params()
        datas=self.send_request(self.url,tieba_params)
        #re
        html=datas.decode()
        url_list=re.compile('<a rel="noreferrer" href="(.+)" title.*?</a>').findall(html)
        # xpath
        # detail_rule="//div[@class='t_con cleafix']/div/div/div/a/@href"
        # url_list=self.parse_data(datas,detail_rule)
        for url in url_list:
            detail_url='https://tieba.baidu.com'+url
            detail_data=self.send_request(detail_url)
            image_url='//img[@class="BDE_Image"]/@src'
            image_url_list=self.parse_data(detail_data,image_url)
            for image_url_1 in image_url_list:
                image_data=self.send_request(image_url_1)
                image_name=image_url_1[-12:]
                self.seve_data(image_data,image_name)



if __name__ == '__main__':
    a=input('请输入你要的关键字')
    tieba=tieba(a)
    tieba.run()


```
<font face='宋体' > <span id='6' 
color='#456526'><center><b>selenium</b></center></span></font>
- - -
Selenium 是为了测试而出生的. 但是没想到到了爬虫的年代, 它摇身一变, 变成了爬虫的好工具. 让我试着用一句话来概括 Seleninm: 它能控制你的浏览器, 有模有样地学人类”看”网页.
那么你什么时候会要用到 Selenium 呢? 当你:
发现用普通方法爬不到想要的内容
网站跟你玩”捉迷藏”, 太多 JavaScript 内容
需要像人一样浏览的爬虫
我是通过Anaconda写selenium所以我只需要! pip install selenium
selenium的环境搭建需要浏览器的支持和浏览器driver的支持，我就同的chrome所以我必须下载chromedriver添加到path里即我加的是我的python的Script里就可以了
详情参考百度python selenium
或者https://selenium-python.readthedocs.io/
下面运行第一个selenium
```
from selenium import webdriver
import time
firefox=webdriver.Chrome()
url='http://www.phei.com.cn/'
firefox.get(url)
time.sleep(10)
with open('phei.html','w',encoding='utf8')as f:
    f.write(firefox.page_source)
firefox.quit()
就利用chrome打开了一个html
本地还有一个phei.html
```
```
查找单个元素
from selenium import webdriver
browser=webdriver.Chrome()
browser.get('https://www.taobao.com')
input_frist=browser.find_element_by_id('q')
input_second=browser.find_element_by_css_selector('#q')
print(input_frist,input_second)
browser.close()
结果为
<selenium.webdriver.remote.webelement.WebElement (session="fa3593cb1ecbf4e6e7b70570b9b23b27", element="0.7509356911593199-1")> <selenium.webdriver.remote.webelement.WebElement (session="fa3593cb1ecbf4e6e7b70570b9b23b27", element="0.7509356911593199-1")>
```
```
多个元素
from selenium import webdriver
browser=webdriver.Chrome()
browser.get('https://www.taobao.com')
lis=browser.find_elements_by_css_selector('.service-bd li')
print(lis)
browser.close()
里面的.service-bd li'是淘宝导航条的一个区域的名字
结果为
[<selenium.webdriver.remote.webelement.WebElement (session="38dc04f1721619c75b9957d0bcab5366", element="0.16647184484271205-1")>, <selenium.webdriver.remote.webelement.WebElement (session="38dc04f1721619c75b9957d0bcab5366", element="0.16647184484271205-2")>, <selenium.webdriver.remote.webelement.WebElement (session="38dc04f1721619c75b9957d0bcab5366", element="0.16647184484271205-3")>, <selenium.webdriver.remote.webelement.WebElement (session="38dc04f1721619c75b9957d0bcab5366", element="0.16647184484271205-4")>, <selenium.webdriver.remote.webelement.WebElement (session="38dc04f1721619c75b9957d0bcab5366", element="0.16647184484271205-5")>, <selenium.webdriver.remote.webelement.WebElement (session="38dc04f1721619c75b9957d0bcab5366", element="0.16647184484271205-6")>, <selenium.webdriver.remote.webelement.WebElement (session="38dc04f1721619c75b9957d0bcab5366", element="0.16647184484271205-7")>, <selenium.webdriver.remote.webelement.WebElement (session="38dc04f1721619c75b9957d0bcab5366", element="0.16647184484271205-8")>, <selenium.webdriver.remote.webelement.WebElement (session="38dc04f1721619c75b9957d0bcab5366", element="0.16647184484271205-9")>, <selenium.webdriver.remote.webelement.WebElement (session="38dc04f1721619c75b9957d0bcab5366", element="0.16647184484271205-10")>, <selenium.webdriver.remote.webelement.WebElement (session="38dc04f1721619c75b9957d0bcab5366", element="0.16647184484271205-11")>, <selenium.webdriver.remote.webelement.WebElement (session="38dc04f1721619c75b9957d0bcab5366", element="0.16647184484271205-12")>, <selenium.webdriver.remote.webelement.WebElement (session="38dc04f1721619c75b9957d0bcab5366", element="0.16647184484271205-13")>, <selenium.webdriver.remote.webelement.WebElement (session="38dc04f1721619c75b9957d0bcab5366", element="0.16647184484271205-14")>, <selenium.webdriver.remote.webelement.WebElement (session="38dc04f1721619c75b9957d0bcab5366", element="0.16647184484271205-15")>, <selenium.webdriver.remote.webelement.WebElement (session="38dc04f1721619c75b9957d0bcab5366", element="0.16647184484271205-16")>]
获取了导航条的一大推
```
当然还可以加载时控制加载 比如不加载图片如
```
# -*- coding:utf-8 -*-
from selenium import webdriver
'''
设置页面不加载图片,这样可以加快页面的渲染，减少爬虫的等待时间，提升爬取效率
固定配置如下：
'''
chrome_opt = webdriver.ChromeOptions()
prefs = {'profile.managed_default_content_settings.images': 2}
chrome_opt.add_experimental_option('prefs',prefs)
# webdriver.Chrome(executable_path='path')启动失败的话,可以指定ChromeDriver驱动的位置path路径
browser = webdriver.Chrome(chrome_options=chrome_opt)
# 启动淘宝测试结果
browser.get('https://www.taobao.com')
```
```
界面跳转
from selenium import webdriver
import time 

browser=webdriver.Chrome()
browser.get('https://www.taobao.com')
input=browser.find_element_by_id('q')
input.send_keys('iphone')
time.sleep(1)
input.clear()
input.send_keys('ipad')
button=browser.find_element_by_class_name('search-button')
button.click()
程序执行后淘宝自动点击进入了ipad的页面
search-button就是淘宝源代码里面的按钮
click()就是selenium提供的api的方法
同理还有许多 如click_and_hold点击之后不动double_click双击等相关api
```
```
获得标签内容，文本值
from selenium import webdriver
browser=webdriver.Chrome()
url='https://www.zhihu.com/explore'
browser.get(url)
input=browser.find_element_by_class_name('zu-top-add-question')
print(input.text)
print(input.id)
print(input.tag_name)
print(input.location)
输出 为
提问
0.7567384595049704-1
button
{'x': 759, 'y': 7}
源代码是<button class="zu-top-add-question" id="zu-top-add-question">提问</button>
```
```
前进和后退
from selenium import webdriver
import time
browser=webdriver.Chrome()
browser.get('https://www.bing.com/')
browser.get('https://www.taobao.com/')
browser.get('https://s.taobao.com/search?q=iphone&imgfile=&commend=all&ssid=s5-e&search')
time.sleep(1)
browser.back()
browser.back()
browser.forward()
browser.close()
```
```
选项卡
import time 
from selenium import webdriver
browser=webdriver.Chrome()
browser.get('https://www.bing.com/')
browser.execute_script('window.open()')
#运用js的脚本可以切换和多个窗口
print(browser.window_handles)
browser.switch_to_window(browser.window_handles[1])
browser.get('https://www.taobao.com')
time.sleep(2)
browser.switch_to_window(browser.window_handles[0])
browser.get('https://python.org')
多个页面，并且可以自动跳转
```
```
下面再来一个selenium的抓取淘宝信息

```

- - -
<font face='宋体' > <span id='7' color='#456526'><center>COOKIE</center></span></font>
1.Opener
当你获取一个URL你使用一个opener。在前面，我们都是使用的默认的opener，也就是urlopen。它是一个特殊的opener，可以理解成opener的一个特殊实例，传入的参数仅仅是url，data，timeout。

如果我们需要用到Cookie，只用这个opener是不能达到目的的，所以我们需要创建更一般的opener来实现对Cookie的设置。

2.Cookiejar
cookiejar模块的主要作用是提供可存储cookie的对象，以便于与urllib模块配合使用来访问Internet资源。Cookiejar模块非常强大，我们可以利用本模块的CookieJar类的对象来捕获cookie并在后续连接请求时重新发送，比如可以实现模拟登录功能。该模块主要的对象有CookieJar、FileCookieJar、MozillaCookieJar、LWPCookieJar。

它们的关系：CookieJar —-派生—->FileCookieJar  —-派生—–>MozillaCookieJar和LWPCookieJar
下面来看看书中的代码】
```
import urllib.request
import urllib.parse
import http.cookiejar
url='*********************'
postdata=urllib.paser.urlencode({
    
    'username':'weisuen',
    'password':'aA123456'
}).encode('utf-8')
req=urllib.request.Request(url,postdata)
req.add_header('user-agent','Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.131 Safari/537.36')
#创建一个cookiejar对象
cjar=http.cookiejar.CookieJar()
#使用HTTPCookieProcessor创建cookie处理器，并以其构建参数作为opener
opener =urllib.request.build_opener(urllib.request.HTTPCookieProcessor(cjar))
#将opener安装为全局
urllib.request.install_opener(opener)
file=opener.open(req)
data=file.read()
```
```
cookiejar主要步骤
1.导入Cookie处理模块http.cookiejar
2.使用http.cookiejar.CookieJar()创建CookieJar对象
3.使用HTTPCookieProcessor创建cookie处理器，并以其为参数构建opener对象
4.创建全局默认的opener对象
```
- - - 
```
1）获取Cookie保存到变量
首先，我们先利用CookieJar对象实现获取cookie的功能，存储到变量中，先来感受一下

from urllib import request
from urllib import parse
from http import cookiejar

#声明一个CookieJar对象实例来保存cookie
cookie = cookiejar.CookieJar()
#利用urllib库中的request的HTTPCookieProcessor对象来创建cookie处理器
handler=request.HTTPCookieProcessor(cookie)
#通过handler来构建opener
opener = request.build_opener(handler)
#此处的open方法同urllib的urlopen方法，也可以传入request
response = opener.open('http://www.baidu.com')
for item in cookie:
    print ('Name = '+item.name)
    print ('Value = '+item.value)



我们使用以上方法将cookie保存到变量中，然后打印出了cookie中的值，运行结果如下


Name = BAIDUID
Value = 5C63AF95C94F5EE96BC89EE5E9CE0188:FG=1
Name = BIDUPSID
Value = 5C63AF95C94F5EE96BC89EE5E9CE0188
Name = H_PS_PSSID
Value = 1431_24557_21123_24022_20928
Name = PSTM
Value = 1508901974
Name = BDSVRTM
Value = 0
Name = BD_HOME
Value = 0





2）保存Cookie到文件
在上面的方法中，我们将cookie保存到了cookie这个变量中，如果我们想将cookie保存到文件中该怎么做呢？这时，我们就要用到

FileCookieJar这个对象了，建立LWPCookieJar实例，可以存Set-Cookie3类型的文件。而MozillaCookieJar类是存为'.txt'格式的文件.在这里我们使用它的子类MozillaCookieJar来实现Cookie的保存


#!/usr/bin/env/python
#encoding: UTF-8
from urllib import request
from urllib import parse
from http import cookiejar

#设置保存cookie的文件，同级目录下的cookie.txt
filename = 'cookie.txt'
#声明一个MozillaCookieJar对象实例来保存cookie，之后写入文件
cookie = cookiejar.MozillaCookieJar(filename)
#利用urllib库的HTTPCookieProcessor对象来创建cookie处理器
handler = request.HTTPCookieProcessor(cookie)
#通过handler来构建opener
opener = request.build_opener(handler)
#创建一个请求，原理同urllib2的urlopen
response = opener.open("http://www.baidu.com")
#保存cookie到文件
cookie.save(ignore_discard=True, ignore_expires=True)


关于最后save方法的两个参数在此说明一下：

官方解释如下：

ignore_discard: save even cookies set to be discarded. 

ignore_expires: save even cookies that have expiredThe file is overwritten if it already exists

由此可见，ignore_discard的意思是即使cookies将被丢弃也将它保存下来，ignore_expires的意思是如果在该文件中cookies已经存在，则覆盖原文件写入，在这里，我们将这两个全部设置为True。运行之后，cookies将被保存到cookie.txt文件中，我们查看一下内容，附图如下



使用LWPCookieJar来保存cookie:

filename = 'cookie'

cookie = cookiejar.LWPCookieJar(filename)

使用的时候直接load。

 3）从文件中获取Cookie并访问
那么我们已经做到把Cookie保存到文件中了，如果以后想使用，可以利用下面的方法来读取cookie并访问网站，感受一下

#!/usr/bin/env/python
#encoding: UTF-8

from urllib import request
from urllib import parse
from http import cookiejar

cookie = cookiejar.MozillaCookieJar()
cookie.load('cookie.txt', ignore_discard=True, ignore_expires=True)
req= request.Request('http://www.baidu.com')
opener = request.build_opener(request.HTTPCookieProcessor(cookie))
response = opener.open(req)
print(response.read())


设想，如果我们的 cookie.txt 文件中保存的是某个人登录百度的cookie，那么我们提取出这个cookie文件内容，就可以用以上方法模拟这个人的账号登录百度。

 4）利用cookie模拟网站登录
下面我们以知乎为例，利用cookie实现模拟登录，来感受一下cookie大法吧！

#!/usr/bin/env/python
#encoding: UTF-8
 
import re
import requests
import http.cookiejar
from PIL import Image
import time
import json
 
headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 '
                         '(KHTML, like Gecko) Chrome/57.0.2987.98 Safari/537.36',
           "Host": "www.zhihu.com",
           "Referer": "https://www.zhihu.com/",
           }
# 建立一个会话，可以把同一用户的不同请求联系起来；直到会话结束都会自动处理cookies
session = requests.Session()
# 建立LWPCookieJar实例，可以存Set-Cookie3类型的文件。
# 而MozillaCookieJar类是存为'/.txt'格式的文件
session.cookies = http.cookiejar.LWPCookieJar("cookie")
# 若本地有cookie则不用再post数据了
try:
    session.cookies.load(ignore_discard=True)
except IOError:
    print('Cookie未加载！')
 
 
def get_xsrf():
    """
    获取参数_xsrf
    """
    response = session.get('https://www.zhihu.com', headers=headers)
    html = response.text
    get_xsrf_pattern = re.compile(r'<input type="hidden" name="_xsrf" value="(.*?)"')
    _xsrf = re.findall(get_xsrf_pattern, html)[0]
    return _xsrf
 
 
def get_captcha():
    """
    获取验证码本地显示
    返回你输入的验证码
    """
    t = str(int(time.time() * 1000))
    captcha_url = 'http://www.zhihu.com/captcha.gif?r=' + t + "&type=login"
    response = session.get(captcha_url, headers=headers)
    with open('cptcha.gif', 'wb') as f:
        f.write(response.content)
    # Pillow显示验证码
    im = Image.open('cptcha.gif')
    im.show()
    captcha = input('本次登录需要输入验证码： ')
    return captcha
 
 
def login(username, password):
    """
    输入自己的账号密码，模拟登录知乎
    """
    # 检测到11位数字则是手机登录
    if re.match(r'\d{11}$', username):
        url = 'http://www.zhihu.com/login/phone_num'
        data = {'_xsrf': get_xsrf(),
                'password': password,
                'remember_me': 'true',
                'phone_num': username
                }
    else:
        url = 'https://www.zhihu.com/login/email'
        data = {'_xsrf': get_xsrf(),
                'password': password,
                'remember_me': 'true',
                'email': username
                }
    # 若不用验证码，直接登录
    result = session.post(url, data=data, headers=headers)
    # 打印返回的响应，r = 1代表响应失败，msg里是失败的原因
    # loads可以反序列化内置数据类型，而load可以从文件读取
    if (json.loads(result.text))["r"] == 1:
        # 要用验证码，post后登录
        data['captcha'] = get_captcha()
        result = session.post(url, data=data, headers=headers)
        print((json.loads(result.text))['msg'])
        # 保存cookie到本地
    session.cookies.save(ignore_discard=True, ignore_expires=True)
 
 
def isLogin():
    # 通过查看用户个人信息来判断是否已经登录
    url = "https://www.zhihu.com/settings/profile"
    # 禁止重定向，否则登录失败重定向到首页也是响应200
    login_code = session.get(url, headers=headers, allow_redirects=False).status_code
    if login_code == 200:
        return True
    else:
        return False
 
 
if __name__ == '__main__':
    if isLogin():
        print('您已经登录')
    else:
        account = input('输入账号：')
        secret = input('输入密码：')
        login(account, secret)


以上程序的原理如下

创建一个带有cookie的opener，在访问登录的URL时，将登录后的cookie保存下来，然后利用这个cookie来访问其他网址。

如登录之后才能查看的成绩查询呀，本学期课表呀等等网址，模拟登录就这么实现啦，是不是很酷炫？


```
- - - 
<font face='宋体' > <span id='8' color='#456526'><center>mongoDB</center></span></font>
在这之前，通过Django学习了mysql也对他有了一点了解，下面来看看爬虫经常用的数据库mongoDB
首先是安装mongoDB，直接进入官网，然后下载相应版本的commiunity 社区版，然后一路install和next 切记，位置一定在根目录下，不然最后的时候如果出现错误然后就会有三个文件分别是bin data log，而且最好把mongo db的compass一起下载好，默认的的是http://localhost:27017下载完成后打开目录就有一个bin和log,data了，可以在bin里面cmd打开命令行输入mongo就进入了mongodb的数据库，如果没有进入则mongod --dbpath 位置 就OK了，在数据库里面输入db就可以看见有一个test就可以对它进行增删除之列的,就开启了mongodb 的学习之路
db.test.insert(('a':'b'))等基本操作忘了自己百度
1.安装也可以sudo apt - get install mongodb - server
最好还是官网，新版本除了后自动配置好了，只需要下载就行了
2.指定数据库位置即data，这个也是闪退后解决办法之一
mongod --dbpath 位置 就OK了
3. python为了连接mongo DB 需要pymongo, pip install pymongo
4.import pymongo
client=pymongo.MongoClient('127.0.0.1',27017)连接客户端
当然也可以client=pymongo.MongoClient('localhost',port=27017)
5.db=client['shujuku_name']创建一个数据库，有就用这个数据库，没有就自动创建
db['biaodan_name'].insert(result)

- - - 
<font face='宋体' > <span id='9' color='#456526'><center>py转exe</center></span></font>
1.准备好需要转换的py文件和一张用于做图标的照片，将他们存放于同一个文件夹中，文件的路径全部为英文路径
2.在网上将图标装换为.ico格式
3.利用命令窗口安装pyinstaller插件 pip install 
4.将命令窗口路径切换到需要处理的py文件的路径，使用cd命令来完成
5.执行命令 pyinstaller -F -i 1.ico 2.py(即ico和py的名字)
6.执行完命令后，需要的exe文件就在dist文件夹中
- - -
<font face='宋体' > <span id='9' color='#456526'><center>redis</center></span></font>
首先下载redis直接搜索redis安装，有一个菜鸟教程还可以，点击进入一个github，https://github.com/MSOpenTech/redis/releases。找到想要的发行版本，安装下载即可，然后在github搜索redis desktop安装可视化界面，强调下，我的都是在E盘的根目录。