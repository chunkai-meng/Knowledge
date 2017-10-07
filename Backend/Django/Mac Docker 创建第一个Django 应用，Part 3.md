# Mac Docker 创建第一个Django 应用，Part 3

> 参考原文：[Writing your first Django app, part 3](https://docs.djangoproject.com/en/1.11/intro/tutorial03/)  
> 本文Python搭建在 Django Compose + Djang  执行Python需进入web server容器中，请参看[第一步：在Mac构建Django 容器]  
> 翻译整理：CK

## Part3：开发前端

### 1. 概述
一个View是一个为特定功能服务的一种网页，例如在本例中有4个View：
- Question “index” page – 展示最新的问题.
- Question “detail” page – 展示一个问题，提供答题表，无答案.
- Question “results” page – 展示一个特定问题的答案.
- Vote action – 处理针对一个特定问题的投票.

在Djanog中网页和其他内容通过View来展示，每个View相当于一个简单的Python函数（或方法，在基于类的View中），在URLconfs中做了URL到View的映射。

### 2. 编写更多的View
添加到polls/views.py
```
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```
添加到polls.urls

```
from django.conf.urls import url

from . import views

urlpatterns = [
    # ex: /polls/
    url(r'^$', views.index, name='index'),
    # ex: /polls/5/
    url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
    # ex: /polls/5/results/
    url(r'^(?P<question_id>[0-9]+)/results/$', views.results, name='results'),
    # ex: /polls/5/vote/
    url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
]
```

以 http://127.0.0.1:8000/polls/34/ 为例, URL之匹配除域名以外的部分，ROOT_URLCONF里的URL `url(r'^polls/', include('polls.urls')),` 匹配了 "polls/" 再将剩余部分 "34/" 发送到‘polls.urls’ URLconf 进一步处理，`(?P<question_id>[0-9]+)`圆括号用来捕获值，"?P<question_id>" 用来定义将要匹配到的模式的名字。[0-9]+ 匹配1到多个数字。最后将匹配到的34作为参数传给detail方法。

### 3. 写几个有用的View
每个View负责返回一个HttpResponse 对象，或者返回异常如Http404
列出最后5个问题：
```
from django.http import HttpResponse

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)

# Leave the rest of the views (detail, results, vote) unchanged
```
这样做的缺点时，页面内容写在View的代码里，如果要改变页面的样子，就得修改Python代码。可以用模版系统来把Python跟页面设计区分开来。在setting.py的TEMPLATES设置里描述了Django如何加载和渲染模版。习惯上Django会在每个安装的APP的目录下查找templates 文件夹。

创建**polls/templates/polls/index.html** 写入：

```
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```


修改**polls/views.py**
```
from django.http import HttpResponse
from django.template import loader

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    template = loader.get_template('polls/index.html')
    context = {
        'latest_question_list': latest_question_list,
    }
    return HttpResponse(template.render(context, request))
```
这段代码加载**polls/templates/polls/index.html** 并传入一个context。这个context是一个字典，将模版的变量名映射为Pyhon对象。

### 一个快捷的render() 方法，常见的做法是载入模版，填入上下文，返回一个包含了渲染后之模版的HttpResponse对象。Django提供一个捷径如下：
重写**polls/views.py**
```
from django.shortcuts import render

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
```

### 4. 报一个404错误
修改**polls/views.py**
```
from django.http import Http404
from django.shortcuts import render

from .models import Question
# ...
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, 'polls/detail.html', {'question': question})
```

创建**polls/templates/polls/detail.html**
```
{{ question }}
```

### 5. 捷径：`A shortcut: get_object_or_404()`
重写detail方法：
```
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```
之所以让model API 报Http404错误而不是ObjectDoesNotExist异常，是因为如果不这样会造成模型层跟视图层的耦合。*或者说views.py不应该关心模型层面的事情，而应该只专注于如何展示，取到什么内容就展示什么。去到的内容是正确的值还是错误信息，应该是model或者controller的事*

同样的**get_list_or_404()**函数像之前的一样，除了使用filter()而不是get()。当取回的列表为空的时候返回404错误

### 5. 使用模版系统
**polls/templates/polls/detail.html**

```
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
```
### 6. 去掉模版中的URL固定代码
修改**polls/index.html**
```
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
```
因为这是紧耦合的做法（当你修改项目的URL的时候，还需要去改动模版），可以使用模版标记来去掉模版对特定URL路径的依赖，改为：
```
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```
这样的好处是polls.urls模块中，名为'detail'的url已经定义好了，根据这个跳转就了，以后如果要修改指向，只要修改polls.urls就好，例如：
```
url(r'^specifics/(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
```

### 7. 使用带命名空间的URL名
如果项目中有多个**app** 不只一个app有**detail View**，如何使用模版标记`{% url %} template tag`?

在**polls/urls.py**中加入 `app_name = 'polls'` 变成
```
from django.conf.urls import url

from . import views

app_name = 'polls'
urlpatterns = [
    url(r'^$', views.index, name='index'),
    url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
    url(r'^(?P<question_id>[0-9]+)/results/$', views.results, name='results'),
    url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
]
```

修改**polls/templates/polls/index.html**

```
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```
这样Django就知道如何生成动态的超链接指向了，而去要修改app的URL也只需要修改对应**app/urls.py** ，无需修改模版。由于模版中往往包含超链接，这样的好处还是很大的。
