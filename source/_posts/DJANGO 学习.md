---
title : Django学习笔记
categories : python
tags : DJANGO
index_img: /pages/淘宝店铺名素材当铺25.jpg
date: 2022-10-29 10:00:00
---
*DJANGO* 学习

## 1.安装django

```python
pip install django
```

## 2.创建项目

django项目中会有一些默认的文件和默认的文件夹

## 2.1在终端

在项目路径下

```
django-admin startproject projectname
```

```
│  manage.py 	[项目的管理，启动项目，创建app，数据管理]
│
└─mydj
        asgi.py		[接受网络请求]
        settings.py		[项目配置 ]
        urls.py		[路径函数的对应关系]
        wsgi.py		[接受网络请求]
        __init__.py
```

## 3.APP

```
-项目
	-app,	用户管理【表结构、函数、HTML模板、CSS】
	-app,	订单管理【表结构、函数、HTML模板、CSS】
	-app,	后台管理【表结构、函数、HTML模板、CSS】
	-app,	网站【表结构、函数、HTML模板、CSS】
	-app,	API【表结构、函数、HTML模板、CSS】
	..
	注意 我们开发变焦简介，用不到多app
```

创建app

~~~shell
python manage.py startapp appname
~~~



```python
python manage.py startapp appname
#文件结构如下图
│  manage.py
│
├─app01
│  │  admin.py 【django默认提供了后台管理】
│  │  apps.py
│  │  models.py
│  │  tests.py	【单元测试文件】
│  │  views.py	【视图函数】
│  │  __init__.py
│  │
│  └─migrations
│          __init__.py
│
└─mydj
    │  asgi.py
    │  settings.py
    │  urls.py
    │  wsgi.py
    │  __init__.py

```

## 4.快速上手

### 在setting.py文件当中注册app

![](https://cdn.jsdelivr.net/gh/mrthere3/typora_note/img/js/202208051521176.png)



### 编写url和视图函数关系

![](https://cdn.jsdelivr.net/gh/mrthere3/typora_note/img/js/202208051528185.png)

1. 路由系统复杂多变，也可以通过include（“app.urls”）,将连接通往子路由系统
2.  re_path()可以帮助我们匹配更加复杂的路由规则包含正则表达式。

### 编写视图函数views.index

![](https://cdn.jsdelivr.net/gh/mrthere3/typora_note/img/js/202208051530473.png)

### 项目启动命令

+ 命令行启动

  ```python
  python manage.py runserver
  ```

### 模板文件templates目录

```python
def user_list(request):
	return render(request,"user_list.html") #在app目录下的templates目录下寻找该HTML文件
#优先在templates目录下寻找
#根据app的注册顺序，在每个app下的templates目录中去寻找
```

![](https://cdn.jsdelivr.net/gh/mrthere3/typora_note/img/js/202208051602276.png)

### 静态文件路径static目录下

+ CSS
+ 图片
+ JS

默认存放在static目录下，在进行调用时候，进行/static加完整路径进行调用

也可在setting.py文件中自行设置

![](https://cdn.jsdelivr.net/gh/mrthere3/typora_note/img/js/202208051712055.png)

## 5. *DJANGO* 模板语法

本质上：在html中写一些占位符，由数据对这些占位符进行替换和处理。 

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <ul>
        {% for i in data_list %}
            <li>{{ i }}</li>
        {% endfor %}
    </ul>

</body>
</html>
```

模板中使用特定的模板语法

```python
from django.shortcuts import render,HttpResponse

# Create your views here.
def index(request):
    return render(request,'index.html',{'data_list':['发票', '海关缴款书', '代扣代缴', '农产品加计扣除发票信息', '农产品加计扣除海关文书', '异常发票']})
```

`在视图函数，使用json格式进行前后端的交互`

~~~javascript
{#<p>过滤器:(|)  将管道符左边的数据当做第一个参数传入过滤器中如果需要第二个参数 则名称后冒号</p>#}
<p>统计长度:{{ l|length }}</p>
<p>加法运算:{{ i|add:123 }}</p>
<p>默认值:{{ b|default:'布尔值为False' }}</p>
<p>时间格式:{{ ctime|date:'Y-m-d H:i:s' }}</p>
<p>截取字符(包含三个点):{{ ss|truncatechars:10 }}</p>
<p>截取单词(不包含三个点):{{ ss|truncatewords:3 }}</p>
<p>文件大小:{{ file_size|filesizeformat }}</p>
<p>转义:{{ s1|safe }}</p>
<p>标签:其实就是流程控制 if判断  for循环</p>
{% for foo in s %}
    {% if forloop.first %}
        <p>这是第一次循环</p>
    {% elif forloop.last %}
        <p>这是最后一次</p>
    {% else %}
        <p>继续!!!</p>
    {% endif %}
    {% empty %}
        <p>传递过来的数据是空的</p>
{% endfor %}
~~~

以下三张

![](https://img-blog.csdnimg.cn/65ead6b1b4b74f5f8450fbb495ffc2fc.png)

![](https://img-blog.csdnimg.cn/c847a5f7347745bfbaab0cf3f2e91e37.png)

![](https://img-blog.csdnimg.cn/c64b5070d7714ea59433c874987e2a82.png)

## 6.交互与响应

### request交互对象详解

~~~tcl
Request
　　我们知道当URLconf文件匹配到用户输入的路径后，会调用对应的view函数，并将 HttpRequest对象 作为第一个参数传入该函数。

我们来看一看这个HttpRequest对象有哪些属性或者方法：
1 HttpRequest.scheme 　 请求的协议，一般为http或者https，字符串格式(以下属性中若无特殊指明，均为字符串格式)

2 HttpRequest.body 　　 http请求的主体，二进制格式。

3 HttpRequest.path 所请求页面的完整路径(但不包括协议以及域名)，也就是相对于网站根目录的路径。

4 HttpRequest.path_info 获取具有 URL 扩展名的资源的附加路径信息。相对于HttpRequest.path，使用该方法便于移植。

if the WSGIScriptAlias for your application is set to “/minfo”, then path might be “/minfo/music/bands/the_beatles/” and path_info would be “/music/bands/the_beatles/”.
5 HttpRequest.method 获取该请求的方法，比如： GET POST …

6 HttpRequest.encoding 获取请求中表单提交数据的编码。

7 HttpRequest.content_type 获取请求的MIME类型(从CONTENT_TYPE头部中获取)，django1.10的新特性。

8 HttpRequest.content_params 获取CONTENT_TYPE中的键值对参数，并以字典的方式表示，django1.10的新特性。

9 HttpRequest.GET 返回一个 querydict 对象(类似于字典，本文最后有querydict的介绍)，该对象包含了所有的HTTP GET参数

10 HttpRequest.POST 返回一个 querydict ，该对象包含了所有的HTTP POST参数，通过表单上传的所有 字符 都会保存在该属性中。

11 HttpRequest.COOKIES 　 返回一个包含了所有cookies的字典。

12 HttpRequest.FILES 　　 返回一个包含了所有的上传文件的 querydict 对象。通过表单所上传的所有 文件 都会保存在该属性中。

key的值是input标签中name属性的值，value的值是一个UploadedFile对象

13 HttpRequest.META 返回一个包含了所有http头部信息的字典

View Code
14 HttpRequest.session 中间件属性

15 HttpRequest.site　　 中间件属性

16 HttpRequest.user　　 中间件属性，表示当前登录的用户。
~~~

### 响应的方式

```python
from django.shortcuts import render,HttpResponse

# Create your views here.
def index(request):
    return HttpResponse ("返回内容")
#HttpResponse 将后面的内容返回

def index(request):
    return redirect ("https://baidu.com")
#redirect 重定向到别的网址
def _get_queryset(klass):
	"""
	从模型、管理器或QuerySet中返回一个QuerySet。为了使get_object_or_404和get_list_or_404更加DRY(代码不重复)。
	如果klass不是Modle、Manager或QuerySet，那么就会产生一个ValueError。
	"""
def get_object_or_404(klass, *args, **kwargs):
	"""
	使用get()返回一个对象如果对象不存在返回Http404异常。
	klass可能是一个Model、Manager或QuerySet对象; get()查询中使用了参数和关键字参数。
	注意: 就像get()一样，如果有多个返回对象，则返回多个对象。
	"""
def get_list_or_404(klass, *args, **kwargs):
	"""
	使用filter()返回一个对象列表,如果列表为空则返回一个Http404异常
	klass可能是一个Model、Manager或QuerySet对象。
	filter()查询中使用了参数和关键字参数。
	"""
def resolve_url(to, *args, **kwargs):
	"""
	返回一个与所传递参数相对应的URL。
	参数可以是:
	*模型:模型的get_absolute_url()函数将被调用。
	*视图名称，可能带有参数:' urlresolvers.reverse()将用于反向解析名称。
	*一个URL，它将按原样返回。
	"""
```

### 登录案例

登陆界面

~~~python
def login(request):
    if request.method == "GET":
        return render(request,"login.html")
    else:
        print(request.POST)
        return HttpResponse("登陆成功")
~~~

~~~html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    {% csrf_token %}
    <form  method="post" action="/login/" >
        <input name="用户名" type="text" placeholder="用户名">
        <input name="密码" type="password" placeholder="密码">
        <input value="提交" type="submit" >
    </form>

</body>
</html>
~~~

`django和flask不同，在进行表单校验，多一个{% csrf_token %}安全机制校验，不加校验，无法实现登陆成功的响应。`

## 7.*DJANGO* ORM

~~~python
pip install mysqlclient==1.4.1 
~~~

#### 1修改配置文件

~~~python
#在sentting.py
DATABASES = {
    'default' : {
        'ENGINE' : 'django.db.backends.mysql',
        'NAME' : 'django_db',
        'USER' : 'root',
        'PASSWORD' : '123456',
        'HOST' : '127.0.0.1',
        'POST' : '3306',
    }
}
~~~

#### 2.在models.py创建表类

~~~python

#django底层会根据类，来生成对应的表和字段from django.db import models #继承models.Model
class UserInfo(models.Model):
    name = models.CharField(max_length=32)                        # primary_key = True
    password = models.CharField(max_length=64)                    # 修改表的时候，可以直接添加一行或去掉。 
    age = models.IntegerField(default=2)                          # 默认为2
    # data = models.IntegerField(null=True, blank=True)           # 允许为空
    # account = models.DecimalField(max_digits=10, decimal_places=2， default = 0)  # 总长度10位， 小数2位, 默认0.
    # create_time = models.DateTimeField()                        # 时间 
    # on_delete=models.CASCADE 级联删除， models.SET_NULL 设置为空。
    # depart = models.ForeignKey(to='Department', to_field='id')  # 生成depart_id字段，和Department的to_field关联
    # gender_choices = (
    #    (1, "男"),
    #    (2, "女"),)
    # gender = models.SmallIntegerField(verbose_name="性别", choices=gender_choices)  # 选着约束
#django底层会根据类，来生成对应的表和字段
~~~

#### 3.执行数据初始化脚本

~~~python
python manage.py makemigrations
python manage.py migrate
~~~

#### 4.使用orm操作数据库表的数据

##### 1.orm用法与字段类型

~~~python
 Meta 选项. 没有一个选项是必需的. 是否添加 class Meta 到你的 model 完全是可选的。

1、db_table 
本模块在数据库中对应的表的名字:

db_table = "pizza_orders"

若不提供该参数, Django 会使用 app_label + '_' + module_name 作为表的名字。

若表的名字是一个 SQL 保留字, 或包含 Python 变量名不允许的字符--特别是连字符 --没关系. Django 会自动在幕后替你将列名字和表名字用引号引起来.

2、get_latest_by 

一个 DateField 或 DateTimeField 字段的名字. 若提供该选项, 该模块将拥有一个 get_latest() 函数以得到 "最新的" 对象(依据那个字段):

get_latest_by = "order_date"。

3、order_with_respect_to 

将该对象标记为可按给定字段排序. This is almost always used with related objects to allow them to be ordered with respect to a parent object. 举例来说, 如果一个 PizzaToppping 关联到一个 Pizza 对象, 你可以使用:

order_with_respect_to = 'pizza'

这就允许 toppings 被排序to be ordered 关于相关的 pizza 对象

4、ordering 

默认排序字段及排序方式, 用于得到一个对象列表的任何场合:

ordering = ['-order_date']

这是一个 tuple 或一个字符串列表. 每个字符串是一个字段名带及一个可选的前缀 "-" , 这个前缀表示按降序排序(递减). 若没有这个前缀,则表示按升序排序.字符串 "?" 表示随机排序.

举个例子, 要对 pub_date 字段以升序排列, 这样做:

ordering = ['pub_date']

要降序排列, 这样:

ordering = ['-pub_date']

要对 pub_date 降序,然后对 author 升序, 这样:

ordering = ['-pub_date', 'author']

注意一点,不论你使用了多少个字段排序, admin 只使用第一个字段.

5、permissions 

要创建一个对象所需要的额外的权限. 如果一个对象有 admin 设置, 则每个对象的添加,删除和改变权限会人(依据该选项)自动创建.下面这个例子指定了一个附加权限: can_deliver_pizzas:

permissions = (("can_deliver_pizzas", "Can deliver pizzas"),)

这是一个2-元素 tuple 的tuple或列表, 其中两2-元素 tuple 的格式为:(permission_code, human_readable_permission_name). 

6、unique_together 

unique_together = (("driver", "restaurant"),)

这是一个字段列表的列表,这些字段的综合值必须是唯一的. 这会在 Django admin 层和数据库层同时做出限制(也就是相关的 UNIQUE 语句会被包括在 CREATE TABLE 语句中).

7、verbose_name 

是该对象的一个可读性更好的唯一名字:

verbose_name = "pizza"

若未提供该选项, Django 则会用一个类名字的 munged 版本来代替: CamelCase becomes camel case.

8、erbose_name_plural 

对象名字的复数:

verbose_name_plural = "stories"

若未提供该选项, Django 会使用 verbose_name + "s".
~~~



~~~python
1.orm用法：属性名 = models.字段类型()
 
2.常见字段类型
    (1)AutoField：    自动增长的IntegerField, 不指定时Django会自动创建属性名为id的自动增长属性
    (2)IntegerField： 整数
    (3)FloatField（）：浮点数
    (4)DecimalField(max_digits=None, decimal_places=None)：可以指定精度的十进制浮点数
                                                        max_digits：总位数
                                                        decimal_places：小数位数
    (5)CharField(max_length=20)：字符串（max_length：最大字符个数）
                                 必须要指定max_length参数 不指定会直接报错
    (6)TextFiled：         大文本字段，一般超过4000个字符时使用
    (7)BooleanField：      布尔字段，值为True或False
    (8)NullBooleanField：  支持Null、True、False三种值                                       
    (9)DateField(auto_now=False, auto_now_add=False)：日期
                参数auto_now：每次保存对象时，自动设置该字段为当前时间，
                              用于"最后一次修改"的时间戳，它总是使用当前日期，默认为false
                参数auto_now_add：当对象第一次被创建时自动设置当前时间，
                              用于创建的时间戳，它总是使用当前日期，默认为false
                注意：参数auto_now_add和auto_now是相互排斥的，组合将会发生错误
    (10)TimeField(auto_now=False, auto_now_add=False)：时间
    (11)DateTimeField：日期时间
    (12)FileField：上传文件字段，以二进制的形式
    (13)ImageField：继承于FileField，对上传的内容进行校验，确保是有效的图片
 
 
3.常见字段属性(选项)
    (1)null：如果为True，表示允许为空，默认值是False
    (2)blank：如果为True，则该字段允许为空白，默认值是False    
             对比：null是数据库范畴的概念，blank是表单验证范畴的
    (3)db_column：字段的名称，如果未指定，则使用属性的名称
              （只限于数据库表中的名字，操作数据库还是类属性的名字）
    (4)db_index：若值为True, 则在表中会为此字段创建索引，默认值是False（为了优化查询速度 ）
    (5)default：默认值，这可以是值或可调用对象。如果可调用，则每次创建新对象时都会调用它。
    (6)primary_key：若为True，则该字段会成为模型的主键字段，默认值是False，一般作为AutoField的选项使用
    (7)unique：如果为True, 这个字段在表中必须有唯一值，这个值不能重复，默认值是False
    (8)verbose_name:该参数是所有字段都有的 就是用来对字段的解释
 
 
4.关系字段类型(用法与普通字段类型一样)
    关系型数据库的关系包括三种类型：
        ForeignKey：        一对多，将字段定义在多的一端中
        ManyToManyField：   多对多，将字段定义在任意一端中
        OneToOneField：     一对一，将字段定义在任意一端中
 
 
5.字段的的增删改(记得执行数据库迁移的两条命令)：
    (1)字段的增加
	    a.可以在终端内直接给出默认值
        b.该字段可以为空
            info = models.CharField(max_length=32,verbose_name='个人简介',null=True)
        c.直接给字段设置默认值
            hobby = models.CharField(max_length=32,verbose_name='兴趣爱
好',default='study')
        然后执行数据库迁移的两条命令
    (2)字段的修改
	   直接修改代码然后执行数据库迁移的两条命令即可！
    (3)字段的删(慎重)
	   直接注释对应的字段然后执行数据库迁移的两条命令即可！
       注意：执行完毕之后字段对应的数据也都没有了
6. values和 values_list
        user_info = UserInfo.objects.filter(id=request.GET.get("id")).values("字段") 
  # 规定 字段名的值返回为字典元素
   valuise_list 规定的为元组
                                     
                                     
~~~

##### 2.ORM之查询操作(查询集、过滤器)

~~~python
1.基本概念
    (1)查询集：表示从数据库中获取的模型对象集合(objects.),在管理器上调用过滤器方法会返回查询集
               查询集可以含有0个、一个或多个过滤器
       查询集特点:a.惰性执行：创建查询集不会访问数据库，直到在模板中调用数据时，才会访问数据库
                             调用数据的情况包括迭代、序列化、与if合用
                 b.缓存：查询集的结果被存下来之后，再次查询相同数据时会使用之前缓存的数据
    (2)过滤器：基于所给的参数限制查询的结果
 
 
 
2.过滤器分两类
    (1)返回列表数据的过滤器：
        all()：       返回所有的数据(以对象形式)
        filter()：    返回满足条件的数据，括号内可以携带多个参数，参数与参数之间默认是and关系
        exclude()：   返回满足条件之外的数据，相当于sql语句中where部分的not关键字
        order_by()：  返回排序后的数据
        注意：返回的是列表对象，即[数据对象1,数据对象2...]，
              获取对象，它支持索引取值和切片操作，但是不支持负数索引
              获取第一个对象filter().first()
    (2)返回单个对象的过滤器:
        get()：       返回单个满足条件的对象
                      # 如果未找到会引发"模型类.DoesNotExist"异常
                      # 如果多条被返回，会引发"模型类.MultipleObjectsReturned"异常
        count()：     返回当前查询的总条数
        aggregate()： 聚合
        exists()：    判断查询集中是否有数据，如果有则返回True，没有则返回False
    (3)常见过滤器查询：
        a.查询id为1的用户名(用exact：判断是否相等)
            usernames = models.User.objects.filter(id__exact=1)
 
        b.查询姓名中包含'张'的名字(用contains：是否包含)
            usernames = models.User.objects.filter(username__contains='张')
 
        c.查询姓名中以‘欣’结尾的名字(用startswith/endswith：以什么开头/以什么结尾)
            usernames = models.User.objects.filter(username__endswith='欣')
 
        d.查询姓名不为空的成员(用isnull : 是否为null)
            usernames = models.User.objects.filter(username__isnull=False)
 
        e.查询编号为2或4的成员(pk:主键/id) (用in：是否包含在范围内)
            usernames = models.User.objects.filter(pk__in=[2,4])
 
        f.查询编号大于2的成员（gt:大于、gte:大于等于、lt:小于、lte:小)
            usernames = models.User.objects.filter(id__gt=2)
 
        g.查询id不等于３的成员(exclude:条件以外的数据)
            usernames = models.User.objects.exclude(id=3)
 
        h.查询2020年注册的成员(year、month、day、week_day、hour、minute、second：对日期时间类型的属性进行运算)
            usernames = models.User.objects.filter(pub_date__year=2020)
 
        i.查询2020年1月1日后注册的成员
            from datetime import date
            usernames = models.User.objects.filter(pub_date__gt=date(2020,1,1))
        h. 多条件查询
        data = {"id"="3","moible"="123"}
        usernames = models.User.objects.filter(**data)
~~~

~~~python
在views.py中对数据库数据的增删改查
from app01 import models
1.查数据
    # 方式一
        res = models.User.objects.filter(username=username)
            """
            返回值你先看成是列表套数据对象的格式
            它也支持索引取值，切片操作，但是不支持负数索引
            它也不推荐你使用索引的方式取值
            user_obj = models.User.objects.filter(username=username).first()
            filter括号内可以携带多个参数 参数与参数之间默认是and关系
            """
        return render(request,'userlist.html',{'user_queryset':user_queryset})    # 指定返回
    # 方式二
        user_queryset = models.User.objects.all()
        return render(request,'userlist.html',locals())    # 全部返回
 
 
2.增数据
    # 方法一：
    res = models.User.objects.create(username=username,password=password)  # 返回值就是当前被创建的对象本身
    # 方法二：
    user_obj = models.User(username=username,password=password)
    user_obj.save()              # 保存数据/对象调用save方法
 
 
3.删数据
    models.User.objects.filter(id_user=1).delete()    # 指定删除id_user=1的数据
 
 
4.改数据
    # 方式一：
    models.User.objects.filter(id=edit_id).update(username=username,password=password)
        # 将filter查询出来的列表中所有的对象全部更新，批量更新操作，只修改被修改的字段
    # 方式二：
    edit_obj.username = username
    edit_obj.password= password
    edit_obj.save()
        # 用重新赋值方式，全部逐一更换当字段特别多的时候效率会非常的低
        # 因为从头到尾，无论该字段是否被修改都将数据的所有字段全部更新一边 
~~~

#### 5.案例

~~~python
from django.db import models
class Department(models.Model):
    title = models.CharField(verbose_name="部门名称",max_length=32)

class UserInfo(models.Model):
    name = models.CharField(max_length=32,verbose_name="姓名")                        # primary_key = True
    password = models.CharField(verbose_name="密码",max_length=64)                    # 修改表的时候，可以直接添加一行或去掉。
    age = models.IntegerField(verbose_name="年龄")    # 默认为2
    count = models.DecimalField(verbose_name="余额",max_digits=10,decimal_places=2,default=0)
    create_time = models.DateTimeField(validators="入职时间")
    # depart_id = models.BigIntegerField(verbose_name="部门id")
    #设置外键
    deaprt = models.ForeignKey(to="Department",to_fields="id",on_delete=models.CASCADE)
    #on_delete 这是数据库外键定义的一个可选项，用来设置当主键表中的被参考列的数据发生变化时，外键表中响应字段的变换规则的。
	"""
on_update 和 on_delete 后面可以跟的词语有四个
1. no action 表示 不做任何操作，
2. set null 表示在外键表中将相应字段设置为null
3. set default 表示设置为默认值
4. cascade 表示级联操作，就是说，如果主键表中被参考字段更新，外键表中也更新，主键表中的记录被删除，外键表中改行也相应删除
	"""
~~~

#####  7.5.1部门列表

~~~html
      <link rel="stylesheet"href="mydj/static/bootstrap/dist/css/bootstrap.min.css">
      <script src="mydj/static/jquery/dist/jquery.js"></script>
      <script src="mydj/static/bootstrap/dist/js/bootstrap.min.js"></script>
~~~

+ 静态文件路径如图所示

![](https://cdn.jsdelivr.net/gh/mrthere3/typora_note/img/js/202208161405343.png)

+ 配置文件路径如下
  
  ~~~html
  STATIC_URL = '/static/'
  
  STATICFILES_DIRS = [(os.path.join(BASE_DIR, 'static'))]
  ~~~
  
+ 导入静态文件
  
  ~~~html
  {% load static %}
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <link rel="stylesheet" type="text/css" href="{% static 'bootstrap/dist/css/bootstrap.css' %}"/>
      <style>
          .navbar {
              border-radius: 0;
          }
      </style>
  
  </head>
  <body>
  <nav class="navbar navbar-default">
      <div class="container">
          <!-- Brand and toggle get grouped for better mobile display -->
          <div class="navbar-header">
              <button type="button" class="navbar-toggle collapsed" data-toggle="collapse"
                      data-target="#bs-example-navbar-collapse-1" aria-expanded="false">
                  <span class="sr-only">Toggle navigation</span>
                  <span class="icon-bar"></span>
                  <span class="icon-bar"></span>
                  <span class="icon-bar"></span>
              </button>
              <a class="navbar-brand" href="#">联通用户管理 </a>
          </div>
  
          <!-- Collect the nav links, forms, and other content for toggling -->
          <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
              <ul class="nav navbar-nav">
                  <li class="active"><a href="#">部门管理 <span class="sr-only">(current)</span></a></li>
                  <li><a href="#">用户管理</a></li>
              </ul>
              <ul class="nav navbar-nav navbar-right">
                  <li><a href="#">登录</a></li>
                  {#                    <li><a href="#">登出</a></li>#}
                  {#                    <li><a href="#">Link</a></li>#}
                  <li class="dropdown">
                      <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true"
                         aria-expanded="false">李洛克<span class="caret"></span></a>
                      <ul class="dropdown-menu">
                          <li><a href="#">个人资料 </a></li>
                          <li><a href="#">我的信息</a></li>
                          <li role="separator" class="divider"></li>
                          <li><a href="#">注销</a></li>
                      </ul>
                  </li>
              </ul>
          </div><!-- /.navbar-collapse -->
      </div><!-- /.container-fluid -->
  </nav>
  <div>
      <div class="container">
          <div style="margin-bottom: 10px">
              <a class="btn btn-primary btn-success " data-toggle="modal" data-target="#exampleModal">
                  <span class="glyphicon glyphicon-eject" aria-hidden="true"></span>
                  新建部门</a>
              <div class="modal fade" id="exampleModal" tabindex="-1" role="dialog"
                   aria-labelledby="exampleModalLabel">
                  <div class="modal-dialog" role="document">
                      <div class="modal-content">
                          <div class="modal-header">
                              <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span
                                      aria-hidden="true">&times;</span></button>
                              <h4 class="modal-title" id="exampleModalLabel">新建部门</h4>
                          </div>
                          <div class="modal-body">
                              <form action="/depart/add/" method="post" onsubmit="return isempty('#deparment-name')">
                                  {% csrf_token %}
                                  <div class="form-group">
                                      <label for="recipient-name" class="control-label">部门名称:</label>
                                      <input type="text" class="form-control" id="deparment-name" name="depart_name">
                                  </div>
                                  <div class="form-group">
                                      <label for="message-text" class="control-label">部门人员</label>
                                      <textarea class="form-control" id="message-text" name="depart_length"></textarea>
                                  </div>
                                  <div class="modal-footer">
                                      <button type="button" class="btn btn-default" data-dismiss="modal">关闭</button>
                                      <button type="submit" class="btn btn-primary">保存</button>
                                  </div>
                              </form>
                          </div>
                      </div>
                  </div>
              </div>
          </div>
          <div class="panel panel-default">
              <!-- Default panel contents -->
              <div class="panel-heading">
                  <span class="glyphicon glyphicon-list" aria-hidden="true"></span>
                  部门列表
              </div>
  
              <!-- Table -->
              <table class="table table-bordered">
                  <thead>
                  <tr>
                      <th>ID</th>
                      <th>名称</th>
                      <th>操作</th>
                  </tr>
                  </thead>
                  <tbody>
                  {% for i in depart_list %}
                      <tr>
                          <th scope="row">{{ start_index|add:forloop.counter }}</th>
                          <td>{{ i.title }}</td>
                          <td>
                              <a class="btn-primary btn-sm" data-toggle="modal" data-target="#exampleModal2"
                                 data-whatever="{{ i.title }}">编辑</a>
                              <a class="btn-danger btn-sm" href="/depart/delete/?nid={{ i.id }}" target="_parent">删除</a>
                          </td>
                      </tr>
                  {% endfor %}
                  <div class="modal fade" id="exampleModal2" tabindex="-1" role="dialog"
                       aria-labelledby="exampleModalLabel">
                      <div class="modal-dialog" role="document">
                          <div class="modal-content">
                              <div class="modal-header">
                                  <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                                      <span aria-hidden="true">&times;</span></button>
                                  <h4 class="modal-title" id="exampleModalLabel">部门名称编辑</h4>
                              </div>
                              <div class="modal-body">
                                  <form method="post" action="/depart/edit/" id="deaprt_edit" onsubmit="return isempty('#deparment-name1')">
                                      {% csrf_token %}
                                      <div class="form-group">
                                          <label for="recipient-name" class="control-label">部门名称</label>
                                          <input type="text" class="form-control" id="deparment-title"  readonly name="old_depart_name">
                                      </div>
                                      <div class="form-group">
                                          <label for="message-text" class="control-label">新部门名称</label>
                                          <input class="form-control" id="deparment-name1" name="depart_name">
                                      </div>
                                  </form>
                                  <div class="modal-footer">
                                      <button type="button" class="btn btn-default" data-dismiss="modal">关闭
                                      </button>
                                      <button type="submit" class="btn btn-primary" form="deaprt_edit">提交</button>
                                  </div>
                              </div>
  
                          </div>
                      </div>
                  </div>
                  </tbody>
              </table>
          </div>
      </div>
  </div>
  </body>
  <script src="{% static '/jquery/dist/jquery.js' %}"></script>
  <script src="{% static '/bootstrap/dist/js/bootstrap.js' %}"></script>
  <link rel="stylesheet" type="text/css" href="https://www.huangwx.cn/css/sweetalert.css">
  <script type="text/javascript" src="https://www.huangwx.cn/js/sweetalert-dev.js"></script>
  <script>
      function isempty(id) {
          try {
              var res = $(id).val();
              var res2 = $("#message-text").val();
              console.log(res, res2)
              if ((res.length == 0) && (res2.length == 0)) {
                  swal({title: "数据不能为空", type: "error", confirmButtonText: "确定"})
                  return false
              }
              return true
          } catch (error) {
              console.log(error)
          }
      }
  </script>
  <script>
      try {
          $('#exampleModal2').on('show.bs.modal', function (event) {
              var button = $(event.relatedTarget) // Button that triggered the modal
              var recipient = button.data('whatever') // Extract info from data-* attributes
              console.log(recipient)
              // If necessary, you could initiate an AJAX request here (and then do the updating in a callback).
              // Update the modal's content. We'll use jQuery here, but you could use a data binding library or other methods instead.
              var modal = $(this)
              {#modal.find('.modal-title').text('New message to ' + recipient)#}
              modal.find('.modal-body input:eq(1)').val(recipient)
          })
      }catch(error) {
          console.log(error)
      }
  </script>
  </html>
  ~~~
  
+ 例如下列视图函数
  
  ~~~python
  import time
  
  from django.shortcuts import render,HttpResponse,redirect
  from app01.models import Department
  from django.contrib import messages
  from django.core.paginator import Paginator
  
  Create your views here.
  def index(request):
      return render(request,'index.html',{'data_list':['发票', '海关缴款书', '代扣代缴', '农产品加计扣除发票信息', '农产品加计扣除海关文书', '异常发票']})
  #
  def depart_index(request):
      index =1
      page_size = 10
      if request.method == "GET":
          index = int(request.GET.get("index",1))
          page_size = int(request.GET.get("pagesize",3))
      query_set = Department.objects.all()
      paginator = Paginator(query_set, page_size)
      try:
          pages = paginator.page(index) # 可能请求的页数大于 实际分页数目
      except:
          index = paginator.num_pages #针对超出直接返回到最后一页数据
          last = paginator.num_pages
          pages = paginator.page(last)
      # print(query_set)
      startindex = page_size*(index-1)
      return render(request,"depart_list.html",{'depart_list':pages,"start_index":startindex})
  
  
  def depart_add(request):
      if request.method == "POST":
          depaer_ment=request.POST.get("depart_name")
          depart_query = Department.objects.filter(title=depaer_ment)
          if not depart_query:
              Department.objects.create(title=depaer_ment)
      # time.sleep(2)
      return redirect("/depart/list/")
  
  
  def depart_delete(request):
      print(request.path)
      nid = request.GET.get("nid")
      Department.objects.filter(id=nid).delete()
      return redirect("/depart/list/")
  
  
  def depart_edit(request):
      if request.method =="POST":
          print(request.POST)
          odepart = request.POST.get("old_depart_name")
          ndepart = request.POST.get("depart_name")
          Department.objects.filter(title=odepart).update(title=ndepart)
  
      return redirect("/depart/list/")
  
  
  ~~~

##### 7.5.2模板继承

~~~html
<!定义模板>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    {% block css %} #css模板
        
    {%endblock%}
</head>
<body>
	<div>
        {% block content %}
        
        {%endblock%}
    </div>

</body>
    {% block js %} #js模板
        
    {%endblock%}
</html>
~~~

​		继承模板

~~~html
{%extneds 'layout.html'%}

{% block css %}  自己的css{%endblock%}
{% block content %}

{%endblock%}
{% block js %}  自己的js{%endblock%}
<!--继承或者自己重载相关-->
~~~

+ ~~~python
  deaprt = models.ForeignKey(to="Department", to_field="id", on_delete=models.CASCADE,null=True)
  #depart 为两表关键外键
  title = depart.object.filter(depart).title #获取关联表的属性
  depart.object.filter(depart) #会直接帮你关联到另外一张表的对象
  #django为我们封装好了 
  ~~~

##### 7.5.3 django 组件

###### 	7.5.3.1django组件

1. form组件
2. modelform组件

###### 	7.5.3.2 form组件

~~~python
class myform(django.form):
    user = form.charField(weight=form.input)
    
def depart_edit(request):
    form = myform() #实例化自己的表单对象
    return render(requests,"/depart/list/",{"form":form})
~~~

前端

~~~html
<form>
    {% for i in form %}
    	{i} 或者 {i.user}
    {endfor}i
    
</form>
~~~

$\color{red}{form组件会操成与model大量重复代码 不推荐使用}$

###### 7.5.3.3   *modelform*组件

~~~python
#前端用法与form组件相同
from django.forms import widgets as wid  #因为重名，所以起个别名
        widgets = {
            "name":wid.TextInput(attrs={'class':'form-control'}),
            "age":wid.NumberInput(attrs={'class':'form-control'}),
            "email":wid.EmailInput(attrs={'class':'form-control'})
        }
  #简洁写法
def __init__(self,*args,**kwargs):
    super()__init(*args,**kwargs)
    for name,field in self.fields.items():
        field.wid.attrs = {'class':'form-control'})
 #添加样式
class userform(ModelForm):
    class Meta:
        model = UserInfo
        fields ="__all__"
        exclude = ["password"] 

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        for name, field in self.fields.items():
            field.widget.attrs = {"class":"form-control"}
def depart_list(request):

    if request.method == 'GET':
        student_list = userform()
        return render(request,'student.html',{'student_list':student_list})
    else:
        student_list = StudentList(request.POST)
        if student_list.is_valid():
            student_list.save()
        return redirect(request,'student_list.html',{'student_list':student_list})
  #保存数据的时候，不用挨个取数据了，只需要save一下。
#在视图函数当中的 使用
def user_add(request):
    if request.method == "POST":
        myform = userform(data=request.POST)
        if myform.is_valid():
            myform.save()
            return JsonResponse({"msg":"处理完成"})
        else:
            print(myform.errors)
            return JsonResponse({"msg":"处理失败 ","error_message":myform.errors})
        #直接存储数据 或者修改数据 
        myform = userform(data=request.POST, instance=user_info.first())
        myform.save()
        
~~~

~~~html
<form>
    {% for i in form %}
    	{{i}} 或者 {{i.user}}
    	{{i.label}} 显示数据库里面的verbose_name
    {endfor}i
    
</form>

~~~

##### 7.6. 用户登录验证

~~~python
#正则校验手机号码
from django.core.validators import RegexValidator
class myfrom(modelform):
    xx= form.chidel("") # 定义model没有的字段
    moible = forms.CharField(label = "手机号码"，validators=[RegexValidator(r"^159[0-9]+$",'号码必须以159开头 ')])
"""    
	1.字段规则校验，字符长度，是否必填等基本校验
    2.validators校验（RegexValidator校验器或自定义校验函数）
    3.局部钩子（类中定义的以clean_字段名命名的函数，校验正常必须返回该字段的值self.cleaned_data.get(‘name’)）
    4.全局钩子（类中定义的函数名clean，校验正常必须返回该对象的校验结果值return self.cleaned_data）
    5.每一步通过校验单结果都以字典形式保存在类对象的cleaned_data属性中
"""
      def clean_city(self): # 局部钩子定义在modelform初始化的时候
        """局部钩子判断城市必须是北京/上海/深圳其中一个"""
        city_val = self.cleaned_data.get('city', '')
        if city_val in ["北京", "上海", "深圳"]:
            return city_val
        else:
            raise forms.ValidationError('城市只能选:北京/上海/深圳')
~~~

~~~python
#分页操作 
from django.core.paginator import Paginator #导入分页类
    query_set = Department.objects.all()
    paginator = Paginator(query_set, page_size) #query_set model对象，page_size 每页数量
    pages = paginator.page(index) # index 代表 pages是显示第几页
    
#json 格式化model对象
from django.core import serializers
from django.http import JsonResponse # 
	user_info = UserInfo.objects.filter(id=request.GET.get("id")).values("字段")
    user_info = UserInfo.objects.filter(id=request.GET.get("id"))
    user_json = serializers.serialize("json",user_info)
      		
        #会将查询到的modesl对象转换成json
        return JsonResponse({"msg":"处理完成","user_info":user_json})
    #这边也可以不用serializers方法 用first()可以直接放进json中
~~~

###### cookie和session 设置

1. 使用Django框架自带的"request.user.is_authenticated()"方法来判断用户是否登录

~~~python
if request.user.is_authenticated():
	# 如果登录返回用户想去的页面
	return render(request, "用户想去页面")
else:
	# 未登录, 重定向到用户登录页面
	return redirect(reverse("登录页面"))
~~~

2. 使用Django自带的login_required装饰器(内部就是封装了is_authenticated())判断用户是否登录

~~~python
#1.需要在Django的settings配置文件中添加如果验证登录未通过需要跳转的路径
from django.contrib.auth.decorators import login_required
@login_required
LOGIN_URL = "/login/"
~~~

cookie 和session设置

~~~python
request.session[] = "" #相当于是字典操作
~~~

前端ajax如何绕过ajdngo  csrf认证

1. 读取浏览器cookie当中的csrf字段

2. 忽略部分view 的csrf

   ~~~python
   from django.views.decorators.csrf import csrf_exempt
   @csrf_exempt
   def task(request)
   #被装饰的函数将不会进行csrf认证
   ~~~


## 8. *Django*路由与视图详解

### 8.1 path 与 re_path 路由函数

re_path函数对比path，可以实现更加复杂的正则表达式路由规则

~~~python
path（'app2/<uuid:id>',view.show_uuid,name="show_uuid"）
"""
path 正则参数
 str : 任意非空字符，不包含“/”,默认类型
 int : 匹配0和任意整数
 slug ：匹配任意ASCⅡ字符、连接线和下划线
 uuid ： 匹配一个uuid格式的字符串，该对象必须包含“-”，所有字母必须小写。比如22221111-abcd-3cww-3321-123456789123
"""

re_path('app2/list/(?<year>\d{4})/',view.article.list)
"""
所用规则与正则表达式无差别
"""
此上两种方法关键参数都是在<>中
path(<type:type>)
re_path('(?<name>\d{4})')

~~~

### 8.2 反向路由解析器

~~~python
path（'app2/<uuid:id>',view.show_uuid,name="show_uuid"）
~~~

name可以在HTML中使用，根据name得到url地址。好处是name不变，url可以随意修改。

reverse() 方法 可以根据name 反向解析出这个name代表的URL

### 8.3 视图函数的底层处理

1. HttpRequest 和 HttpResponse 对象
   ~~~python
   # HttpRequest对象
   path  字符串，便是请求页面的路径，不包含域名
   method 字符串，表示页面请求的方式。必须以大写方式
   encoding 字符串，表示提交的数据的编码方式
   GET 字典类型，包含GET请求方法中的所有参数
   POSt 字典类型，包含POSt请求方法中的所有参数
   FILE 字典类型，包含上传文件的信息
   COOKIES 字典类型，包含所有的cookie对象
   session 字典类型，表示当前会话
   META 字典类型，包含所有的HTTP头部信息，如HTTP_USER_AGENT、REMOTE_ADDR
   user 表示当前登录用户
   
   ~~~

2. HttpResposne 
   ~~~python
   """
   常见参数
   content 返回的内容
   status_code 返回的HTTP响应状态吗
   content-type 返回的数据的MIME类型，默认为text/html
   """
   
   ~~~

### 8.4 视图处理函数的使用

1.  render()函数
   ~~~python
   from django.shortcts import render
   render(request,template_name,content=None,content_type=None,status=None，using=None)
   """
   content: 字典类型，保存要传到html的变量
   status: 表示响应的状态代码，默认为200
   content_type: 用于生成文档的MIME类型，默认为text/html
   using：设置末班引擎，用于解析模板文件
   """
   ~~~

2. redict()函数
   重定向的三种方式

   1. 通过调用模板的get_absolute_url()函数
      ~~~python
      class UserInfo(models.Model):
          name=models.CharField(max_length=32)                        # primary_key = True
          password = models.CharField(max_length=64)                    # 修改表的时候，可以直接添加一行或去掉。 
          age = models.IntegerField(default=2)  
          def get_absolute_url(self):
              return reverse("url",kwargs={'id':"self.id"})
      ~~~

      

   2. 通过路由反向解析进行重定向
      ~~~python
      def test_redict_view(request,id):
          return redirect("url",id)
      ~~~

   3. 通过绝对的或者相对的url进行重定向
      ~~~python
      return redirect("https://")
      ~~~

      











