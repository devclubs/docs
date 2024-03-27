# 设计模式和模板

## 1. 设计模式

- MVC 模式 :代表 Model-View-Controller（模型-视图-控制器） 模式。
    - 作用: 降低模块间的耦合度(解耦)
    - M 模型层(Model), 主要用于对数据库层的封装
    - V 视图层(View), 用于向用户展示结果
    - C 控制(Controller），用于处理请求、获取数据、返回结果(重要)
    ![](./imgs/mvc.png)

- MTV 模式: 代表 Model-Template-View（模型-模板-视图） 模式。这种模式用于应用程序的分层开发
    - 作用: 降低模块间的耦合度(解耦)
    - M -- 模型层(Model)  负责与数据库交互
    - T -- 模板层(Template)  负责呈现内容到浏览器
    - V -- 视图层(View)  是核心，负责接收请求、获取数据、返回结果
    ![](./imgs/mvc.png)


- MVP 模式: 代表 Model-Presenter-View（模型-控制器-视图） 模式。这种模式用于应用程序的分层开发
    - 作用: 降低模块间的耦合度(解耦)
    - M -- 模型层(Model)  负责与数据库交互
    - P -- 控制层(Presenter)  负责与视图层交互，获取数据，返回结果
    - V -- 视图层(View)  是核心，负责接收请求、获取数据、返回结果

## 2. 模板Templates

模板是可以根据字典数据动态变化的html网页,模板可以根据视图中传递的字典数据动态生成相应的HTML网页。

### 2.1 模板配置

- 在项目目录下创建templates文件夹，用来存放模板文件
- 在settings.py中配置模板路径
```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates', # 指定模板的引擎
        'DIRS': ['templates'],  # 模板的搜索目录(可以是一个或多个)
        'APP_DIRS': True, # 是否索引各app里的templates目录
        'OPTIONS': { # 模板引擎的选项
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

### 2.2 模板的加载

Django使用render()函数直接加载模板，并返回渲染好的HTML页面。

```python
from django.shortcuts import render

def xx_view(request):
    return render(request, 'templates/xx.html', locals()) # locals()为数据字典
```

## 3. 模板语言

模板语言是django用来渲染模板的语法，django内置了django模板语言，使用django模板语言可以快速开发网站。

### 3.1 模板传参

模板传参是指把数据形成字典，传参给模板，为模板渲染提供数据

### 3.2 模板的变量

模板变量是指模板中可以获取到的变量，模板变量可以通过字典传参给模板，也可以通过模板的context_processors获取到。

1. 在模板中使用变量语法

- `{{ 变量名 }}`

- `{{ 变量名.index }}`

- `{{ 变量名.key}}`

- `{{ 对象.方法 }}`

- `{{ 函数名 }}`


2. 视图函数中必须将变量封装到字典中才允许传递到模板上

```python
def xxx_view(request)
    dic = {
        "变量1":"值1",
        "变量2":"值2",
    }
    return render(request, 'xxx.html', dic)
```

3.  如果变量过多，可以使用 locals() 将局部变量自动生成字典

```python
def xxx_view(request)
	变量1 = 值1
    变量2 = 值2
    ...
    return render(request, 'xxx.html', locals())
```
```
def tmpl(request):
    data = {
        "show_info": True,
        "name": "dengyouf",
        "age": 18,
        "hobby": ["read", "play", "eating"],
        "books": {
            "java": 20,
            "golang": 30,
            "python": 40
        }
    }
    return render(request, "tmpl.html", {"data": data})
```

### 3.3 模板的标签

> [官网](https://docs.djangoproject.com/en/3.2/ref/templates/builtins/#built-in-tag-reference): `https://docs.djangoproject.com/en/3.2/ref/templates/builtins/#built-in-tag-reference`

标签作用于将服务端的功能嵌入到模板中

- 标签语法
```python
{% tag %}
{% endtag %}
```

- if 标签
    - if 条件表达式里可以用的运算符 ==, !=, <, >, <=, >=, in, not in, is, is not, not、and、or
    -在if标记中使用实际括号是无效的语法。 如果您需要它们指示优先级，则应使用嵌套的if标记
```
{% if 条件表达式1 %}
...
{% elif 条件表达式2 %}
...
{% elif 条件表达式3 %}
...
{% else %}
...
{% endif %}
```

```html
<body>
  <h2>模板渲染</h2>

  {% if data.show_info %}
   {{ data.name }}-{{ data.age }}
  {% else %}
    查无此人，请添加数据
  {% endif %}
</body>
```

- for标签
```
{% for 变量 in 可迭代对象 %}
    ... 循环语句
{% empty %}
    ... 可迭代对象无数据时填充的语句
{% endfor %}
```
```html
 {% for k, v in data.books.items %}
  {{ k }}-{{ v }}
 {% endfor%}
```

### 3.4 模板过滤器
> [官网](https://docs.djangoproject.com/en/3.2/ref/templates/builtins/#built-in-filter-reference): `https://docs.djangoproject.com/en/3.2/ref/templates/builtins/#built-in-filter-reference`

过滤器作用于对模板变量进行过滤，过滤器可以改变模板变量的值，也可以改变模板变量的输出格式。

### 3.5 模板继承

### 3.6 模板缓存

### 3.7 模板内置函数

###  3.8 模板内置过滤器

### 3.9 模板内置标签

## 4. 静态文件

静态文件是指不需要经过服务器处理的文件，比如图片、CSS、JS等。
静态文件可以通过静态文件服务器提供，也可以通过django提供的静态文件功能直接提供。

### 4.1 静态文件配置

- 在settings.py中配置静态文件路径
    - STATIC_URL = '/static/'： 通过哪个url地址找静态文件
    - STATICFILES_DIR: 配置静态文件的存储路径
    - ```python
      # file: setting.py
      STATIC_URL = '/static/'
      STATICFILES_DIRS = (
          os.path.join(BASE_DIR, "static"),
      )
      ```

### 4.2 静态文件加载

```python
# 方式1
<img src="/static/images/1.jpg">
# 方式2
<img src="http://127.0.0.1:8000/static/images/1.jpg">

# 方式3
{% load static %}
<img src="{% static 'images/1.jpg' %}">
```















