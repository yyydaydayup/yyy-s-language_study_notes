[toc]

# Django模版

## 01.模板简介

在之前的章节中，视图函数只是直接返回文本，而在实际生产环境中其实很少这样用，因为实际的页面大多是带有样式的HTML代码，这可以让浏览器渲染出非常漂亮的页面。目前市面上有非常多的模板系统，其中最知名最好用的就是DTL和Jinja2。DTL是Django Template Language三个单词的缩写，也就是Django自带的模板语言。当然也可以配置Django支持Jinja2等其他模板引擎，但是作为Django内置的模板语言，和Django可以达到无缝衔接而不会产生一些不兼容的情况。因此建议大家学习好DTL。

### 1.1 DTL与普通的HTML文件的区别
DTL模板是一种带有特殊语法的HTML文件，这个HTML文件可以被Django编译，可以传递参数进去，实现数据动态化。在编译完成后，生成一个普通的HTML文件，然后发送给客户端。

### 1.2 渲染模板
渲染模板有多种方式。这里讲下两种常用的方式。

1. `render_to_string`：找到模板，然后将模板编译后渲染成Python的字符串格式。最后再通过HttpResponse类包装成一个HttpResponse对象返回回去。示例代码如下：
```python
from django.template.loader import render_to_string
from django.http import HttpResponse
def book_detail(request,book_id):
    html = render_to_string("detail.html")
    return HttpResponse(html)
```
2. 以上方式虽然已经很方便了。但是django还提供了一个更加简便的方式，直接将模板渲染成字符串和包装成HttpResponse对象一步到位完成。示例代码如下：
```python
 from django.shortcuts import render
 def book_list(request):
     return render(request,'list.html')
```

### 1.3 模版查找路径

在项目的 **settings.py** 文件中。有一个 `TEMPLATES` 配置，这个配置包含了模板引擎的配置，模板查找路径的配置，模板上下文的配置等。模板路径可以在两个地方配置。

* `DIRS`：这是一个列表，在这个列表中可以存放所有的模板路径，以后在视图中使用render或者render_to_string渲染模板的时候，会在这个列表的路径中查找模板。

* `APP_DIRS`：默认为True，这个设置为True后，会在INSTALLED_APPS的安装了的APP下的templates文件加中查找模板。

* 查找顺序：比如代码render('list.html')。先会在DIRS这个列表中依次查找路径下有没有这个模板，如果有，就返回。如果DIRS列表中所有的路径都没有找到，那么会先检查当前这个视图所处的app是否已经安装，如果已经安装了，那么就先在当前这个app下的templates文件夹中查找模板，如果没有找到，那么会在其他已经安装了的app中查找。如果所有路径下都没有找到，那么会抛出一个TemplateDoesNotExist的异常。
  * ==项目templates  ->  当前app的templates   ->  其他app的templates==



## 02.模版变量

* 在模版中使用变量，需要将变量放到`{{ 变量 }}`中。

* 如果想要访问 **对象** 的属性，那么可以通过`对象.属性名`来进行访问。

```python
class Person(object):
    def __init__(self,username):
        self.username = username

context = {
    'person': p
}
```
​	以后想要访问`person`的`username`，那么就是通过`person.username`来访问。

* 如果想要访问一个 **字典** 的key对应的value，那么只能通过`字典.key`的方式进行访问，不能通过`中括号[]`的形式进行访问。

```python
context = {
    'person': {
        'username':'zhiliao'
    }
}
```
​	那么以后在模版中访问`username`。就是以下代码`person.username`

​	因为在访问字典的`key`时候也是使用`点.`来访问，因此不能在字典中定义字典本身就有的属性名当作`key`，否则字典的那个属性将变成字典中的key了。

```python
context = {
    'person': {
        'username':'zhiliao',
        'keys':'abc'
    }
}
```
以上因为将`keys`作为`person`这个字典的`key`了。因此以后在模版中访问`person.keys`的时候，返回的不是这个字典的所有key，而是对应的值。

* 如果想要访问 **列表或元组** ，那么也是通过`点.`的方式进行访问，不能通过`中括号[]`的形式进行访问。这一点和python中是不一样的。示例代码如下：

```python
{{ persons.1 }}
```

## 03.模板标签

[django官方文档-模板标签](https://docs.djangoproject.com/zh-hans/3.0/ref/templates/builtins/#ref-templates-builtins-tags) 

### 3.1 if标签

1. 所有的标签都是在`{%%}`之间。
2. if标签有闭合标签。就是`{% endif %}`。
3. if标签的判断运算符，就跟python中的判断运算符是一样的。`==、!=、<、<=、>、>=、in、not in、is、is not`这些都可以使用。
4. 还可以使用`elif`以及`else`等标签。

```python
# ====================views.py=======================
from django.shortcuts import render

def label_if(request):
    context = {
        'age': 20,
    }
    return render(request, "label_if_index.html", context=context)

# =====================label_if_index.html==============
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Title</title>
</head>
<body>
    {% if age < 18 %}
        <p>您的年龄小于18岁</p>
    {% elif age == 18 %}
        <p>您的年龄等于18岁</p>
    {% else %}
        <p>您的年龄大于18岁</p>
    {% endif %} 
</body>
</html>
```



### 3.2 for标签

#### 1、`for...in...`标签：
`for...in...`类似于`Python`中的`for...in...`。可以遍历列表、元组、字符串、字典等一切可以遍历的对象。示例代码如下：

```python
{% for person in persons %}
<p>{{ person.name }}</p>
{% endfor %}
```

如果想要反向遍历，那么在遍历的时候就加上一个`reversed`。示例代码如下：

```python
{% for person in persons reversed %}
	<p>{{ person.name }}</p>
{% endfor %}
```

遍历字典的时候，需要使用`items`、`keys`和`values`等方法。在`DTL`中，执行一个方法**不能**使用圆括号的形式。遍历字典示例代码如下：

```python
# ===================views.py===============
from django.shortcuts import render
def index(request):
    person_var = {
        'name': 'yyy',
        'age': '18',
        'height': 185,
    }
    context = {
        'person': person_var,
    }
    return render(request, "index.html", context=context)
# ===================index.html==============
<body>
    {% for key,value in person.items %}
        <p>key：{{ key }}</p>
        <p>value：{{ value }}</p>
    {% endfor %}
</body>
```

在`for`循环中，`DTL`提供了一些变量可供使用。这些变量如下：

* `forloop.counter`：当前循环的下标。以1作为起始值。
* `forloop.counter0`：当前循环的下标。以0作为起始值。
* `forloop.revcounter`：当前循环的反向下标值。比如列表有5个元素，那么第一次遍历这个属性是等于5，第二次是4，以此类推。并且是以1作为最后一个元素的下标。
* `forloop.revcounter0`：类似于forloop.revcounter。不同的是最后一个元素的下标是从0开始。
* `forloop.first`：是否是第一个元素。
* `forloop.last`：是否是最后一个元素。
* `forloop.parentloop`：如果有多个循环嵌套，那么这个属性代表的是上一级的for循环。
* 模板中的for...in...==没有==continue和break语句，这一点和Python中有很大的不同，一定要记清楚！ 

#### 2、 `for...in...empty`标签：
这个标签使用跟`for...in...`是一样的，只不过是在遍历的对象如果没有元素的情况下，会执行`empty`中的内容。示例代码如下：

```django
{% for person in persons %}
	<li>{{ person }}</li>
{% empty %}
	暂时还没有任何人
{% endfor %}
```

####3、详细例子

```python
# ===========================views.py============================
from django.shortcuts import render

def label_if(request):
    context = {
        'age': 20,
    }
    return render(request, "label_if_index.html", context=context)


def label_for(request):
    context = {
        'fruits': [
            '苹果',
            '柿子',
            '梨',
            '西瓜',
        ],
        'person': {
            'username': '杨阳羊',
            'age': 3,
            'height': 185,
        },
        'books': [
            {
                'name': '三国演义',
                'author': '罗贯中',
                'price': '43',
            },
            {
                'name': '水浒传',
                'author': '施耐庵',
                'price': '44',
            },
            {
                'name': '红楼梦',
                'author': '曹雪芹',
                'price': '50',
            },
            {
                'name': '西游记',
                'author': '吴承恩',
                'price': '45',
            },
        ],
        'comments': [
            '好',
            '真好',
            '确实好',
        ]
    }
    return render(request, "label_for_index.html", context=context)
```

```django
{# ========================DTL:label_for_index.html=======================#}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    {#  遍历列表  #}
    <ul>
        {% for fruit in fruits reversed %}
            <li>{{ fruit }}</li>
        {% endfor %}
    </ul>
    {#  遍历字典  #}
    <ul>
        {% for key, value in person.items  %}
            <li>{{ key }} : {{ value }}</li>
        {% endfor %}
    </ul>
    {#高级操作#}
    <table>
        <thead>
            <tr>
                <td>序号</td>
                <td>书名</td>
                <td>作者</td>
                <td>价格</td>
            </tr>
        </thead>
        <tbody>
            {% for book in books %}
                {% if forloop.first %}
                    <tr style="background: pink">
                {% elif forloop.last %}
                    <tr style="background: red">
                {% else %}
                    <tr>
                {% endif %}
                    <td>{{ forloop.counter }}</td> {# 加入序号 #}
                    <td>{{ book.name }}</td>
                    <td>{{ book.author }}</td>
                    <td>{{ book.price }}</td>
                </tr>
            {% endfor %}

        </tbody>
    </table>
    {# 使用empty #}
    <ul>
        {% for comment in comments %}
            <li>{{ comment }}</li>
        {% empty %}
            <li>暂无评论</li>
        {% endfor %}
    </ul>

    
</body>
</html>
```



### 3.3 with标签

1. 在模板中，想要定义变量，可以通过`with`语句来实现。
2. `with`语句有两种使用方式，第一种是`with xx=xxx`的形式，第二种是`with xxx as xxx`的形式。
3. 定义的变量只能在with语句块中使用，在with语句块外面使用取不到这个变量。
示例代码如下：
```django
    {% with zs=persons.0 %}
        <p>{{ zs }}</p>
        <p>{{ zs }}</p>
    {% endwith %}

    下面这个因为超过了with语句块，因此不能使用
    <p>{{ zs }}</p>

    {% with persons.0 as zs %}
        <p>{{ zs }}</p>
    {% endwith %}
```

### 3.4 url标签

`url`标签：在模版中，我们经常要写一些`url`，比如某个`a`标签中需要定义`href`属性。当然如果通过硬编码的方式直接将这个`url`写死在里面也是可以的。但是这样对于以后项目维护可能不是一件好事。因此建议使用这种反转的方式来实现，类似于`django`中的`reverse`一样。示例代码如下：

```django
<a href="{% url 'book:list' %}">图书列表页面</a>
```

如果`url`反转的时候需要传递参数，那么可以在后面传递。但是参数分位置参数和关键字参数。位置参数和关键字参数不能同时使用。示例代码如下：

```django
# path部分
path('detail/<book_id>/',views.book_detail,name='detail')

# url反转，使用位置参数
<a href="{% url 'book:detail' 1 %}">图书详情页面</a>

# url反转，使用关键字参数
<a href="{% url 'book:detail' book_id=1 %}">图书详情页面</a>
```

如果想要在使用`url`标签反转的时候要传递查询字符串的参数，那么必须要手动在在后面添加。示例代码如下：

```django
<a href="{% url 'book:detail' book_id=1 %}?page=1">图书详情页面</a>
```

如果需要传递多个参数，那么通过空格的方式进行分隔。示例代码如下：

```django
<a href="{% url 'book:detail' book_id=1 page=2 %}">图书详情页面</a>
```

### 3.5 autoescape标签

1. DTL中默认已经开启了自动转义。会将那些特殊字符进行转义。比如会将`<`转义成`&lt;`等。
2. 如果你不知道自己在干什么，那么最好是使用DTL的自动转义。这样网站才不容易出现XSS漏洞。
3. 如果变量确实是可信任的。那么可以使用 `autoescape` 标签来关掉自动转义。示例代码如下：
```django
{% autoescape off %}
{{ info }}
{% endautoescape %}
```

### 3.6 verbatim标签

`verbatim`标签：默认在`DTL`模板中是会去解析那些特殊字符的。比如`{%`和`%}`以及`{{`等。如果你在某个代码片段中不想使用`DTL`的解析引擎。那么你可以把这个代码片段放在`verbatim`标签中。示例代码下：

```django
{% verbatim %}
	{{if dying}}Still alive.{{/if}}
{% endverbatim %}
```

## 04.模板过滤器

### 4.1 为什么需要过滤器？

因为在DTL中，不支持函数的调用形式`()`，因此不能给函数传递参数，这将有很大的局限性。而过滤器 **其实就是一个函数** ，可以对需要处理的参数进行处理，并且还可以额外接收一个参数（也就是说，最多只能有 **2个参数** ）。

### 4.2 Django常见内置过滤器
#### 4.2.1 add过滤器
将传进来的参数添加到原来的值上面。这个过滤器会尝试将`值`和`参数`转换成整形然后进行相加。如果转换成整形过程中失败了，那么会将`值`和`参数`进行拼接。如果是字符串，那么会拼接成字符串，如果是列表，那么会拼接成一个列表。示例代码如下：

```django
{{ value|add:"2" }}
```

如果`value`是等于4，那么结果将是6。如果`value`是等于一个普通的字符串，比如`abc`，那么结果将是`abc2`。`add`过滤器的源代码如下：

```python
@register.filter(is_safe=False)
def add(value, arg):
    """Add the arg to the value."""
    try:
        return int(value) + int(arg)
    except (ValueError, TypeError):
        try:
            return value + arg
        except Exception:
            return ''
```

#### 4.2.2 cut过滤器
移除值中所有指定的字符串。类似于`python`中的`replace(args,"")`。示例代码如下：

```python
{{ value|cut:" " }}
```

以上示例将会移除`value`中所有的空格字符。`cut`过滤器的源代码如下：

```python
@register.filter
@stringfilter
def cut(value, arg):
    """Remove all values of arg from the given string."""
    safe = isinstance(value, SafeData)
    value = value.replace(arg, '')
    if safe and arg != ';':
        return mark_safe(value)
    return value
```

#### 4.2.3 date过滤器
将一个日期按照指定的格式，格式化成字符串。示例代码如下：

```python
# 数据
context = {
	"today": datetime.datetime.now()
}

# 模版
{{ today|date:"Y-m-d H:i:s" }} #我的标准时间格式
# 输出：2020-06-06 14:19:06
```

那么将会输出`2020-06-06 14:19:06` 。其中`Y`代表的是四位数字的年份，`m`代表的是两位数字的月份，`d`代表的是两位数字的日。  
还有更多时间格式化的方式。见下表。

| 格式字符 | 描述                                 | 示例  |
| -------- | ------------------------------------ | ----- |
| Y        | 四位数字的年份                       | 2018  |
| m        | 两位数字的月份                       | 01-12 |
| n        | 月份，1-9前面没有0前缀               | 1-12  |
| d        | 两位数字的天                         | 01-31 |
| j        | 天，但是1-9前面没有0前缀             | 1-31  |
| g        | 小时，12小时格式的，1-9前面没有0前缀 | 1-12  |
| h        | 小时，12小时格式的，1-9前面有0前缀   | 01-12 |
| G        | 小时，24小时格式的，1-9前面没有0前缀 | 1-23  |
| H        | 小时，24小时格式的，1-9前面有0前缀   | 01-23 |
| i        | 分钟，1-9前面有0前缀                 | 00-59 |
| s        | 秒，1-9前面有0前缀                   | 00-59 |



#### 4.2.4 default

如果值被评估为`False`。比如`[]`，`""`，`None`，`{}`等这些在`if`判断中为`False`的值，都会使用`default` 过滤器提供的默认值。示例代码如下：

```django
{{ value|default:"nothing" }}
```

如果`value`是等于一个空的字符串。比如`""`，那么以上代码将会输出`nothing`。

#### 4.2.5 default\_if\_none

如果值是`None`，那么将会使用`default_if_none`提供的默认值。这个和`default`有区别，`default`是所有被评估为`False`的都会使用默认值。而`default_if_none`则只有这个值是等于`None`的时候才会使用默认值。示例代码如下：

```django
{{ value|default_if_none:"nothing" }}
```

如果`value`是等于`""`也即空字符串，那么以上会输出空字符串。如果`value`是一个`None`值，以上代码才会输出`nothing`。

#### 4.2.6 first

返回列表/元组/字符串中的第一个元素。示例代码如下：

```django
{{ value|first }}
```

如果`value`是等于`['a','b','c']`，那么输出将会是`a`。

#### 4.2.7 last

返回列表/元组/字符串中的最后一个元素。示例代码如下：

```django
{{ value|last }}
```

如果`value`是等于`['a','b','c']`，那么输出将会是`c`。

#### 4.2.8 floatformat

使用 **四舍五入** 的方式格式化一个浮点类型。如果这个过滤器没有传递任何参数。那么只会在小数点后保留 **一个** 小数，如果小数后面全是0，那么只会保留整数。当然也可以传递一个参数，标识具体要保留几个小数。

1. 如果没有传递参数：

| value    | 模版代码                  | 输出 |
| -------- | ------------------------- | ---- |
| 34.23234 | `{{ value|floatformat }}` | 34.2 |
| 34.000   | `{{ value|floatformat }}` | 34   |
| 34.260   | `{{ value|floatformat }}` | 34.3 |

2. 如果传递参数：

| value    | 模版代码                  | 输出   |
| -------- | ------------------------- | ------ |
| 34.23234 | `{{value|floatformat:3}}` | 34.232 |
| 34.0000  | `{{value|floatformat:3}}` | 34.000 |
| 34.26000 | `{{value|floatformat:3}}` | 34.260 |

#### 4.2.9 join

类似与`Python`中的`join`，将列表/元组/字符串用指定的字符进行拼接。示例代码如下：

```django
{{ value|join:"/" }}
```

如果`value`是等于`['a','b','c']`，那么以上代码将输出`a/b/c`。

#### 4.2.10 length

获取一个列表/元组/字符串/字典的长度。示例代码如下：

```django
{{ value|length }}
```

如果`value`是等于`['a','b','c']`，那么以上代码将输出`3`。如果`value`为`None`，那么以上将返回`0`。

#### 4.2.11 lower

将值中所有的字符全部转换成小写。示例代码如下：

```django
{{ value|lower }}
```

如果`value`是等于`Hello World`。那么以上代码将输出`hello world`。

#### 4.2.12 upper

类似于`lower`，只不过是将指定的字符串全部转换成大写。

#### 4.2.13 random:随机选择

在被给的列表/字符串/元组中随机的选择一个值。示例代码如下：

```django
{{ value|random }}
```

如果`value`是等于`['a','b','c']`，那么以上代码会在列表中随机选择一个。

#### 4.2.14 safe：关闭自动转义

标记一个字符串是安全的。也即会 **关掉这个字符串的自动转义** 。

相当于  `{% autoscape off%}` 的缩写形式。

示例代码如下:

```django
{{value|safe}}
```

如果`value`是一个不包含任何特殊字符的字符串，比如`<a>`这种，那么以上代码就会把字符串正常的输入。如果`value`是一串`html`代码，那么以上代码将会把这个`html`代码渲染到浏览器中。

#### 4.2.15 slice：切片

类似于`Python`中的切片操作。示例代码如下：

```django
{{ some_list|slice:"2::2" }}  {# 以2为步长获取从2开始往后的所有元素 #}
```

以上代码将会给`some_list`从`2`开始做切片操作。

#### 4.2.16 striptags

删除字符串中所有的`html`标签。只留下纯文本进行展示。常用，比如博客摘要的显示。

示例代码如下：

```django
{{ value|striptags }}
```

如果`value`是`<strong>hello world</strong>`，那么以上代码将会输出`hello world`。

#### 4.2.17 truncatechars

如果给定的字符串长度超过了过滤器指定的长度。那么就会进行切割，并且会拼接三个点来作为省略号。示例代码如下：

```django
{{ value|truncatechars:5 }}
```

如果`value`是等于`北京欢迎您~`，那么输出的结果是`北京...`。可能你会想，为什么不会`北京欢迎您...`呢。因为三个点也占了三个字符，所以`北京`+三个点的字符长度就是5。

#### 4.2.18 truncatechars\_html

类似于`truncatechars`，只不过是不会切割`html`标签。示例代码如下：

```django
{{ value|truncatechars:5 }}
```

如果`value`是等于`<p>北京欢迎您~</p>`，那么输出将是`<p>北京...</p>`。

### 4.3 自定义过滤器笔记

#### 步骤

1. 首先在某个app中，创建一个python **包** ，叫做`templatetags`，注意，这个包的名字 **必须** 为`templatetags`，不然就找不到。

2. 在这个`templatetags`包下面，创建一个python文件用来存储过滤器。

3. 在新建的python文件中，定义过滤器（也就是函数），这个函数的第一个参数永远是被过滤的那个值，并且如果在使用过滤器的时候传递参数，那么还可以定义另外一个参数。但是过滤器最多只能有2个参数。

4. 在写完过滤器（函数）后，要使用`django.template.Library().filter(self, name=None, filter_func=None):`进行注册。或者使用装饰器。

```python
# ===========my_filter.py============
# ====方法一====
from django import template
register = template.Library()
# 过滤器最多有两个参数，且第一个永远是被过滤的那个值
def greet(value, word):
return value + word
register.filter("greet", greet)
# ===方法二：使用装饰器===
from django import template
register = template.Library()
@register.filter()  # 可以指定过滤器名称参数，如果不指定，将使用函数名作为过滤器的名字
def greet(value, word):
return value + word
```


5. 还要把这个过滤器所在的这个app添加到`settings.INSTALLED_APS`中，不然Django也找不到这个过滤器。

6. 在模板中使用 `load` 标签加载过滤器所在的python文件名。如：`{% load my_filter %}` 。

7. 可以使用过滤器了。

#### 实战：自定义时间过滤器
有时候经常会在朋友圈、微博中可以看到一条信息发表的时间，并不是具体的时间，而是距离现在多久。比如`刚刚`，`1分钟前`等。这个功能DTL是没有内置这样的过滤器的，因此我们可以自定义一个这样的过滤器。

具体要求如下：

​	time距离现在的时间间隔：
1. 如果时间间隔小于1分钟以内，那么就显示“刚刚”
2. 如果是大于1分钟小于1小时，那么就显示“xx分钟前”
3. 如果是大于1小时小于24小时，那么就显示“xx小时前”
4. 如果是大于24小时小于30天以内，那么就显示“xx天前”
5. 否则就是显示具体的时间 2017/10/20 16:15



示例代码如下：

​```python
# =========time_filter.py===========
from datetime import datetime
from django import template

register = template.Library()
@register.filter(name="time_since")  # 注册过滤器
def time_since(value):
    if isinstance(value, datetime):
        now = datetime.now()
        timestamp = (now - value).total_seconds()  # 作差之后返回的是timedelay对象
        if timestamp < 60:
            return "刚刚"
        elif timestamp >= 60 and timestamp < 60*60:
            minutes = int(timestamp / 60)
            return "%s分钟前" % minutes
        elif timestamp >= 60*60 and timestamp < 60*60*24:
            hours = int(timestamp / (60*60))
            return "%s小时前" % hours
        elif timestamp >= 60*60*24 and timestamp < 60*60*24*30:
            days = int(timestamp / (60*60*24))
            return "%s天前" % days
        else:
            return value.strftime("%Y/%m/%d %H:%M")
    else:
        return value
```

在模版中使用的示例代码如下：

```django
{% load time_filter %}
...
{% value|time_since %}
...
```

为了更加方便的将函数注册到模版库中当作过滤器。也可以使用装饰器来将一个函数包装成过滤器。示例代码如下：

```python
from django import template
register = template.Library()

@register.filter(name='mycut')
def mycut(value,mystr):
    return value.replace(mystr,"")
```

## 05.模板结构优化
### 5.1 引入模板：include标签

1. 有些模版代码是重复的。因此可以单独抽取出来，以后哪里需要用到，就直接使用`include`进来就可以了。
2. 如果想要在`include`子模版的时候，传递一些参数，那么可以使用`with xxx=xxx`的形式。示例代码如下：
    ```python
    {% include 'header.html' with username='zhiliao' %}
    ```

### 5.2 模版继承：extends标签

在前端页面开发中。有些代码是需要重复使用的。这种情况可以使用`include`标签来实现。也可以使用另外一个比较强大的方式来实现，那就是模版继承。模版继承类似于`Python`中的类，在父类中可以先定义好一些变量和方法，然后在子类中实现。

模版继承也可以在父模版中先定义好一些子模版需要用到的代码，然后子模版直接继承就可以了。并且因为子模版肯定有自己的不同代码，因此可以在父模版中定义一个**block接口**，然后子模版再去实现。以下是父模版的代码：

```django
{# ====================base.html=============#}
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>{% block title %}我的站点{% endblock %}</title>
    </head>
    <body>
        <div id="content">
            {% block content %}{% endblock %}
        </div>
    </body>
</html>
```

这个模版，我们取名叫做`base.html`，定义好一个简单的`html`骨架，然后定义好两个`block`接口，让子模版来根据具体需求来实现。子模板然后通过`extends`标签来实现，示例代码如下：

```django
{# ===============parent1.html=================#}
{% extends "base.html" %}

{% block title %}博客列表{% endblock %}

{% block content %}
    {% for entry in blog_entries %}
        <h2>{{ entry.title }}</h2>
        <p>{{ entry.body }}</p>
    {% endfor %}
{% endblock %}
```

* **需要注意的是：extends标签必须是子模板的第一个标签**
* **子模板中的代码必须放在block中，否则将不会被渲染。**
* 如果在某个`block`中需要使用父模版的内容，那么可以使用`{{block.super}}`来继承。比如上例，`{%block title%}`，如果想要使用父模版的`title`，那么可以在子模版的`title block`中使用`{{ block.super }}`来实现。
* 父模板可以直接使用子模板中的变量。
* 在定义`block`的时候，除了在`block`开始的地方定义这个`block`的名字，还可以在`block`结束的时候定义名字。比如`{% block title %}{% endblock title %}`。这在大型模版中显得尤其有用，能让你快速的看到`block`包含在哪里。





## 06.加载静态文件

在一个网页中，不仅仅只有一个`html`骨架，还需要`css`样式文件，`js`执行文件以及一些图片等。因此在`DTL`中加载静态文件是一个必须要解决的问题。在`DTL`中，使用`static`标签来加载静态文件。由于`static` 标签并不是Django的内置标签，所以要使用`static`标签，首先需要`{% load static %}`。加载静态文件的步骤如下：

1. 首先确保`django.contrib.staticfiles`已经添加到`settings.INSTALLED_APPS`中。

2. 确保在`settings.py`中设置了`STATIC_URL`。

3. 在已经安装了的`app`下创建一个文件夹叫做`static`，然后再在这个`static`文件夹下创建一个当前`app`的名字的文件夹，再把静态文件放到这个文件夹下。例如你的`app`叫做`book`，有一个静态文件叫做`zhiliao.jpg`，那么路径为`book/static/book/zhiliao.jpg`。（为什么在`app`下创建一个`static`文件夹，还需要在这个`static`下创建一个同`app`名字的文件夹呢？原因是如果直接把静态文件放在`static`文件夹下，那么在模版加载静态文件的时候就是使用`zhiliao.jpg`，如果在多个`app`之间有同名的静态文件，这时候可能就会产生混淆。而在`static`文件夹下加了一个同名`app`文件夹，在模版中加载的时候就是使用`app/zhiliao.jpg`，这样就可以避免产生混淆。）

4. 如果有一些静态文件是不和任何 `app` 挂钩的。那么可以在`settings.py`中添加`STATICFILES_DIRS` （此变量名 **必须** 是这个，一点也不能改！！！），以后`DTL`就会在这个列表的路径中查找静态文件。比如可以设置为:

```python
STATICFILES_DIRS = [
	os.path.join(BASE_DIR,"static")
]
```

5. 在模版中使用`load`标签加载`static`标签。比如要加载在项目的`static`文件夹下的`style.css`的文件。那么示例代码如下：

```html
{% load static %}
<link rel="stylesheet" href="{% static 'style.css' %}">
```

6. 如果不想每次在模版中加载静态文件都使用`load`加载`static`标签，那么可以在`settings.py`中的`TEMPLATES/OPTIONS`添加`'builtins':['django.templatetags.static']`，这样以后在模版中就可以直接使用`static`标签，而不用手动的`load`了。

7. 如果没有在`settings.INSTALLED_APPS`中添加`django.contrib.staticfiles`。那么我们就需要手动的将请求静态文件的`url`与静态文件的路径进行映射了。示例代码如下：

```python
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
# 其他的url映射
] + static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```