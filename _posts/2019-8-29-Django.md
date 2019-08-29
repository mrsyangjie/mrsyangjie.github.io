---
layout:     post
title:      Django Beginner
subtitle:   
date:       2018-09-25
author:     YJ
catalog: true
tags:
    - Markdown
---
* [欢迎来到Django笔记页面](#mulu)


<!-- /code_chunk_output -->
<!DOCTYPE html>

#<center><font color="#1c9999" >Django</font></center>
点击这里还可以去[我的博客主页](https://mrsyangjie.github.io/ "我的博客主页")o 


<font face='宋体' > <span id='qianyan'><center>总结</center></span></font>
学习django一段时间,
1.可以virtualenv[name]创建一个名字叫django_env 如果要选择自己想要的解释器可以virtualen -p C:\Users\E\AppData\Local\Programs\Python\Python37\python.exe+[name]，然后pip install django
- 进入虚拟activate后就执行django-admin startproject[name]
会自动生成几个配置文件 _init_ setting设置数据库 静态文件位置等相关设置,model设置ROM的映射相关 还有一个mannage.py:一些相关的命令都得使用它比如 python manage.py makemigrations
- 因为这个没用Pycharm所以要执行就得 cd python_code/django_env/[name]里面再执行python manage.py runsever就行啦
2.使用pycharm就更方便 
- pycharm的自己创建的虚拟环境只需要在创建django时选择你想呀的解释器，最好用一个总的解释器,比如我用的都在D:\Python_code\django_env\Lib\site-packages选择解释器时也都是用的D:\Python_code\django_env\Scripts\python.exe
然后生成一个django项目而且和命令行生成不一样在于多一个templates
不同在于setting 里面的路径问题
```
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
#指明绝对路径，如果不要这个在下面的'DIRS'就得改
#改为这种
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')]
        ,
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```
- pycharm中，右上角->项目配置->port就改成自己想要的端口号
让局域网中其他电脑访问本机的项目需要让项目运行的时候host为0.0.0.0
如在终端执行 python manage.py runserver 0.0.0.0:8000并且在setting.py的配置ALLOWLD_HOSTS中将本机的ip加进去如 (用ipconfig查看本机ip)
```
ALLOWED_HOSTS = [' 169.254.157.141']
```
-然后就是写项目的操作
<font face='宋体' > <span id='mulu'><center>目录</center></span></font>
1. [url与视图函数](#1)
1. [url传参，命名，反转](#2)
1. [路径问题](#3)
1. [url详解](#4)
1. [过滤器](#5)
1. [ORM&mysql](#6)
1. [表单和验证器](#7)
1. [](#8)
1. [项目开发介绍和过程](#9)


<font face='宋体' > <span id='1' color='#456526'><center>url视图函数</center></span></font>
1视图一般写在app中的views中，也可以叫做url发射器，并且第一个参数永远是request(一个HttpRequest)对象,直接返回字符串是会报错的
视图函数只会返回django.http.request.HttpResposeBase的对象而它的对象有HttpRequest,html等
```
from django.http import HttpResponse
def book_link(request):
    return HttpResponse('书籍列表')
```
然后需要在url.py中把路径加进去
```
from book import views
urlpatterns = [
    path('admin/', admin.site.urls),
    path('book/',views.book_link)
]

```
- 但是如果有两个app和以上就这样
```
from book import views
from front import views as front_views
urlpatterns = [
    path('admin/', admin.site.urls),
    path('book/',views.book_link)
    path('front/',front_views.index)
]
```
再访问首页不能进去，是因为django的默认首页为admin新加入了path就失效了改为这样就可以看到首页的
```
from django.http import HttpResponse
from book import views


def index(request):
    return HttpResponse('首页')
urlpatterns = [
    path('', index),
    path('book/',views.book_link)
]
```
再访问http://127.0.0.1:8000/book/就能看到放回'书籍列表'文字了
这就是基本的url与视图关系也是开启hello world一样的开端
<font face='宋体' > <span id='2' color='#456526'><center>url传参，命名，反转</center></span></font>
- 在url.py中设置好参数
```
urlpatterns = [
    path('', index),
    path('book/<book_id>',views.book)
]
```
- 在views中写好视图函数配合参数
```
def book(request,book_id):
    text='你输入的id是 %s' %book_id
    return HttpResponse(text)
```
就能在http://127.0.0.1:8000/book/1看到 '你输入的id是 1',关键是url中的<>必须与views里的参数一致book_id
同理
```
path('book/<book_id>/<price>',views.book)
```
```
def book(request,book_id,price):
    text='你输入的id是 %s,价格是%s' %(book_id,price)
    return HttpResponse(text)
```
在http://127.0.0.1:8000/book/1/111中显示:'你输入的id是 1,价格是111'
- - -
- 第二种方式就是查询字符串的方式,不需要再url中单独匹配查询字符串部分，只需要在views中用request.GET.get('')方法
url.py
```
path('book/author/',views.author)
```
views.py,记住id有单引号不然为错
```
def author(request):
    author_id=request.GET.get('id')
    text='作者id是: %s' % author_id
    return HttpResponse(text)
```
然后在http://127.0.0.1:8000/book/author/?id=2(问号不能少)中就是作者id是: 2
- - -
- url命名与反转url
先创建两个APP一个book一个font并且添加一个urls.py
在总urls中
```
from django.urls import path,include
urlpatterns = [
    path('',include('font.urls')),
    path('book/',include('book.urls')),
]
```
font的views.py
```
from django.http import HttpResponse


def index(request):
    return  HttpResponse('font首页')


def login(request):
    return HttpResponse('font登录')
```
font的urls中
```
from django.urls import  path
from . import views


urlpatterns = [
    path('',views.index),
    path('login',views.login)
]
```
book的views中
```
# Create your views here.
from django.http import HttpResponse
def index(request):
    return  HttpResponse('book首页')

def login(request):
    return HttpResponse('book登录')
```
book的urls中
```
from django.urls import  path
from . import views
urlpatterns = [
    path('',views.index),
    path('/login',views.login),
]
```
千万记住在总的urls里path('book/',include('book.urls'))指向app的book后面一定加/因为浏览器访问时自动加了不加就会错
然后就是在网站访问http://127.0.0.1:8000    为  ：  font首页
http://127.0.0.1:8000/login  为 : font登录
(这里就是我错的地方没加/访问不到)
http://127.0.0.1:8000/book/  为  ：book首页
访问出错因为上面book的login没有在后面加/改了之后
在book 的 urls中
```
path('login/',views.login),
```
访问http://127.0.0.1:8000/book/login/   为  ：book登录
 正确
 - - -
- 命名
```
 path('',views.index,name='index'),
    path('login',views.login,name='login')
```
- 反转
```
from django.http import redirect,reverse
 login_url= reverse('login')
        print('='*30)
        print('login_url')
        return redirect(login_url)
```
反转的是name所以以后就算改了path前面的浏览器名字一样有用
再引用html基本的网站已经形成了
```
return render(request,'index.html'
```
在此基本的

- - - 
<font face='宋体' > <span id='3' color='#456526'><center>路径问题</center></span></font>
- 在代码的执行中会引用静态模板等html,css,js,png等，这时就 需要考虑路径的问题

1 首先要确保 'django.contrib.staticfiles',已经在INSTALLED_APPS 中，一般创建django都有，如果没有就无法加载，
2 确保有
```
STATIC_URL = '/static/'
```
要有这个，如果在浏览器请求静态文件url就需要用这个127.0.0.1/static/xxx.jpng
   3 如果在app中创建static并添加静态文件，先把app放在INSTALLED_APPS然后
   ```
   <img src='/static/logo.jpg/' alt=''>
   ```
   这样就可以把静态照片加载出来了
   都是因为STATIC_URL = '/static/'
   如果改为STATIC_URL = '/abc/'
   则这样
   ```
   <img src='/abc/logo.jpg/' alt=''>
   ```
   也可以
   但是为了以后改名字等复杂操作也可以这样，用static便签，不过需要先下载load一下，然后他就会自动在static文件中找叫这个名字的如logo.fpg
 ```
 {% load static %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    {{ today|date:'y/m/d' }}
    <img src="{% static 'logo.jpg' %}" alt="">
</body>
</html>
```
当这些都行了。那么问题来了，如果我两个APP都有一个static，而且名字还一样，怎么区分和正确引用呢，可以这样
在static下面又创建一个文件，文件名和app一致这样再引用时要多加这个路径就不会重叠了如
```
<img src="{% static 'front/logo.jpg' %}" alt="">
```
这样就区别了
4告别了3的app里面的静态文件，但有些静态文件和app没关系，可以添加一个
```
STATICFILES_DIRS={
    os.path.join(BASE_DIR,'static')
}
```
base_dirs是配置文件有的绝对路径BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))，这样static在app里没有找到的就会在外面的static找，就有点像templates
5 templates一般是'DIRS': [os.path.join(BASE_DIR, 'templates')]
也可以这样用直接目录带入
- - -
<font face='宋体' > <span id='4' color='#456526'><center>url详解</center></span></font>

在前端的页面中经常跳转，一般的跳转是
在urls.py中有
```
path('book/',views.book)
```
```
<li><a href='/book/'>读书</a></li>
```
使用url后
```
path('book/',views.book,name='book')
```
```
<li><a href='{% url 'book' %}'>读书</a></li>
```
若是要传参的话
```
path('book/detail/<book_id>/',views.book_detail,name='detail')
```
```
def book_detail(request,book_id):
text='图书id:%s'%book_id
```
```
<li><a href='{% url 'book' book_id='1'%}'>读书</a></li>
```
- - -
<font face='宋体' > <span id='5' color='#045'><center>过滤器</center></span></font>
过滤器是模板在传参时方便使用如add,cut等
```
{{'hello world'|cut:''}}
```
这样就把所有空白字符剪掉
最好用莫过于date
{{today|date:'y/m/d}}
urls中
```
path('date/',views.date_views,name='date'),
```
views中
```
from datetime import datetime

def date_views(request):
    content={
        'today': datetime.now()
    }
    return render(request,'date.html',context=content)
```
html中
```
{{ today|date:'y/m/d' }}
```
最后访问http://127.0.0.1:8000/date/得到  : '19/04/19'

<font face='宋体' > <span id='6' color='#456526'><center>ORM&mysql</center></span></font>
ORM 一种数据库的映射使用可以更加方便,把数据库映射为一个类
放在app的models.py中
```
from django.db import models

# Create your models here.


class Book(models.Model):
    num=models.AutoField(primary_key=True)
    name=models.CharField(max_length=100,null=False)
    author=models.CharField(max_length=100,null=False)
    price=models.FloatField(null=False,default=0)

```
写完并且连接上数据库后‘就需要映射到数据库中
在终端(虚拟环境下)python manage.py makemigrations
再 python manage.py migrate
之后数据库就有了，表名为app名加class名字 如 front_book
- 连接数据库
因为python3.0以上不支持mysql 需要在项目的_init_中添加
```
import pymysql
pymysql.install_as_MySQLdb()
```
并且需要 pip install pymysql
然后在setting 中
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'book_manager',
        'USER':'root',
        'PASSWORD':'root',
        'HOST':'127.0.0.1',
        'PORT':'3306',
    }
}
```

连接在app的views中
```
from django.shortcuts import render,redirect,reverse
from django.db import connection

from django.shortcuts import render,redirect,reverse
from django.db import connection
# Create your views here.
from .models import Book


def get_cursor():
    return connection.cursor()
#连接数据库

def index(request):
    cursor=get_cursor()
    cursor.execute("select num,name,author from book")
    books=cursor.fetchall()
    #fetchall,fetchone都是获得数据
    #并且现在books.o就是num book.1就是name这样
    # {
    # #两种方法
    # cursor.execute("insert into book values(1,'python','yj')")
    # books=cursor.fetchall()

    # book1=Book(name='西游记',author='吴承恩',price='100')
    # book1.save()
    return render(request,'index.html',context={'books':books})
在对应的html中#
table>
        <thead>
            <tr>
                <th>序号</th>
                <th>书名</th>
                <th>作者</th>
            </tr>
        </thead>
        <tbody>
            {% for book in books %}
        <tr>
            <td>{{ forloop.counter }}</td>
            #forloop.count显示序号
            <td><a href="{%  url 'book_detail' book_id=book.0%}">{{ book.0 }}</a></td>
            <td>{{ book.1 }}</td>
            <td>{{ book.2 }}</td>
        </tr>
        {% endfor %}
        </tbody>
    </table>
#

def add_book(request):
    if request.method == 'GET':
        return render(request, 'add_book.html')
    else:
        print(request.POST)
        name = request.POST.get('name')
        author = request.POST.get('author')
        cursor = get_cursor()
        cursor.execute("insert into book(name,author) values('{0}','{1}') ".format(name, author))
        return redirect(reverse('index'))
对应的html为
 <form action="." method="post">
     <table>
         <tbody>
         <tr>
         <td>书名:</td>
             <td><input type="text" name="name"></td>
         </tr>
           <tr>
         <td>作者:</td>
             <td><input type="text" name="author"></td>
         </tr>
        <tr>
            <td></td>
            <td><input type="submit" value="提交"></td>
        </tr>
         </tbody>
     </table>


def book_detail(request,book_id):
    cursor=get_cursor()
    cursor.execute("select num,name,author from book where num=%s" % book_id)
    book=cursor.fetchone()
    return render(request,'book_detail.html',context={'book':book})
html为
   <p>书名:{{ book.1 }}</p>
    <p>作者:{{ book.2 }}</p>
    <form action="{% url 'book_delete' %}"method="post">
        <input type="hidden" name="'book_id" value="{{ book.0 }}">
        <input type="submit"value="删除">
    </form>
#
def delete_book(request):
    if request.method=='post':
        book_id=request.POST.get('book_id')
        cursor=get_cursor()
        cursor.execute('delete from book where num=%s' %book_id)
        return redirect(reverse('index'))
    else:
        raise RuntimeError('删除错误')
```
- - -
ORM映射流程
1创建一个数据库命名[name]
2在setting中的databases设置相关参数
3创建一个app python manage.py startapp [name]
4在app中的models连接数据库from django.db import models并且创建类class book(models.Model):
5映射到数据库python manange.py makemigrations
python manage.py migrate移植到数据库中
6再就是urls和views的操作
在views操作时记得加上
```
from .models import book
```
其他操作参考点击这里 [url和views](#2)
7一些基本操作
增

book1=book(name='三国演义',author='罗贯中',price=100)
book1.save()
查询(默认的查询都是通过objects查询)
#pk=primary key就不用管主键到底叫什么
book_1=book.objects.get(pk=1)
print(book_1)
就会在终端打印出了 book object [1]
#如果想要打印的时候好看就在models中的book类里写
```
class Book(models.Model):
    num=models.AutoField(primary_key=True)
    name=models.CharField(max_length=100,null=False)
    author=models.CharField(max_length=100,null=False)
    price=models.FloatField(null=False,default=0)
都是在这个类中写的
    def _str_(self):
        return print('<Book:{{name},{author},{price}}>'.format(name=self.name,author=self.author,price=self.price))
        还可以修改表名
        class Meta:
        db_table = '[name]'
        #
        在django的模型model中都继承django.db.models.Model类
        字段必须是models.XXXField类型
        Meta类的db_table为映射数据表的名字如果没有就默认应用名_模型名
```
然后再执行在终端返回的就是<book>
:三国演义，罗贯中，100这样的形式了
- 也可以
```
books=book.objects.filter(name=’西游记‘)(还可以这样books=book.objects.filter(name=’西游记‘).frist())
return render(request,'book_detail.html',context={'book':books})
```
filter是一个过滤器把所有西游记的都返回而render的context也是返回传过区内容

删除
首先查找到对应值在delete()
```
books=book.objects.get(pk=1)
books.delete()
```
修改
```
books=book.objects.get(pk=1)
books.price=200
books.save()
```
8 ORM的外键 ForeignKey
```
定义模型如下
from django.db import models

class User(models.Model):
    name = models.CharField(max_length=100)
    password = models.CharField(max_length=100)

class Article(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()
     author= models.ForeignKey('User',on_delete=models.CASCADE
操作如下
    article=Article(title='asd',content='asfa')
    user=User(name='sfa',password='sada')
    article.author=user
    article.save()
```
9 使用exact精确查找
```
article=Article.objects.get(id__exact=15)
article1=Article.objects.filter(id__exact=1)
```
终端返回的是filter 为一个QuerySet
10聚合函数
需要放在支持聚合函数的方法中比如：aggregate
```
from django.db.models import Avg
result=Book.objects.aggregate(Avg('price))
```
```
Student.objects.annotate(score_avg=Avg('score__number')).filter(score_avg__gt=60)
annotate 方法在底层调用了数据库的数据聚合函数，annotate可以做一个整合汇总的作用
```

<font face='宋体' > <span id='7' color='#456526'><center>表单和验证器</center></span></font>
- a表单
创建一个forms.py
```
from django import forms


class MesssageBoardForm(forms.Form):
    title = forms.CharField(max_length=100,min_length=2,label='标题',error_messages={'min_length':'不得少于一个字符'})
    coutent =forms.CharField(widget=forms.Textarea,label='内容')
    # 控件
    email=forms.EmailField(label='邮箱')
    reply=forms.BooleanField(required=False,label='是否回复')
```
```
还可以增加一些选项如
kind_chioce=(
    ('python','python')
    ('java',java')
    ('c','c')
)

    username=models.CharField(max_length=100)
    kind=models.CharField(max_length=100,choices=kend_chioce,default=king_chioce[0])
    #这样就可以形成选择填表，default是默认显示
```

views视图中
```
from .forms import  MesssageBoardForm


class IndexView(View):
    def get(self,request):
        form = MesssageBoardForm()
        return render(request,'liuyan.html',context={'form':form})
```
urls中
``
path('liuyan/',views.IndexView.as_view(),name='liuyan'),
```
liuyan.html中

<body>
 <form action=""method="post"></form>
 <table>
    {{ form.as_table }}
    <tr>
           <td></td>
        <td><input type="submit" value="提交"> </td>
        </tr>
    </table>
</body>
```
- - -
- - - 
<font face='宋体' > <span id='8' color='#456526'><center><h1>项目介绍和过程</h1></center></span></font>
- - -
1安装nvm，管理node
下载地址https://github.com/coreybutler/nvm-windows/releases
并添加进入环境变量
2 nvm install node下载最新版本node
nvm install [virsion]下载相应版本
nvm use [virsion]用相应版本
nvm uninstall[virsion]删除相应版本
cmd执行下面就会下载
```
nvm install 6.4.0
```
版本后会自动安装相应版本的npm，npm就是用来管理这些包的6.4.0对应10.3
3 安装包
先使用nvm use 6.4.0
本地安装
npm install express
执行后安装包放在node_modules下面即D:\nvm\nvm\v6.4.0\node_modules
可以通过require()引入本地包
express包还带有许多其他包
全局安装
npm install express -g
卸载包就是
npm uninstall [package]
如 npm uninstall express卸载express
npm uninstall express -g
还有更新和搜索包等
npm update [包名字]如 npm update express
npm search [package]

4 在本地中，将安装的包的写的文件放在一个文件夹中方便发送给别人你的项目
并且在你要写项目的文件夹目录cmd 中
nmp init
比如我的就是D:\nvm_demo\demo_1
使它生成一个package.json它记录你使用的包并方便看和管理，毕竟node_modules那么多包不是都要用，安装时参数一路enter最后yes就 ok了
5 安装一个前端自动化过程的包gulp
需要全局和本地都安装
npm install gulp --save.dev
--save。Dev是将安装的包放在package.josn下的devDependencies依赖中
npm install gulp -g 全局
6以后就可以通过npm install直接安装package.json里devDependencies有的包了，也就是项目用的包 ，所以发送项目时只用发这个package.json不用node_modules
7输出hello world过程
- - -
总项目文件 demo_1
要有node_modules放包的地方即
npm uninstall express
还需要一个pcakage.json即nmp init
在用gulp时才能调用包
在demo_1总下创建一个js 
```
 var gulp = require("gulp")

 gulp.task("greet",function(){
 	console.log("hello world");
 });
 ```
 在终端 Cd到该项目文件再
 gulp greet输出hello world
 - - -
 

