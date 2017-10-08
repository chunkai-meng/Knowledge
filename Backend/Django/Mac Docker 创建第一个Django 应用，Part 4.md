# Mac Docker 创建第一个Django 应用，Part 4

> 参考原文：[Writing your first Django app, part 4](https://docs.djangoproject.com/en/1.11/intro/tutorial04/)  
> 本文Python搭建在 Django Compose + Djang  执行Python需进入web server容器中，请参看[第一步：在Mac构建Django 容器]  
> 翻译整理：CK

简单的表单处理以及精简代码

## 1. 简单表单
**polls/templates/polls/detail.html**
```html
<h1>{{ question.question_text }}</h1>

{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

<form action="{% url 'polls:vote' question.id %}" method="post">
<!-- 防止跨站点的伪造请求 -->
{% csrf_token %}
{% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}" />
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br />
{% endfor %}
<input type="submit" value="Vote" />
</form>
```

POST方法可能会去修改数据，因此要防止伪造请求攻击，只要是POST到站内的URL都应该包含这条模版标记
**{% csrf\_token %}**


修改View中的Vote方法
**polls/views.py**
```Python
from django.shortcuts import get_object_or_404, render
from django.http import HttpResponseRedirect, HttpResponse
from django.urls import reverse

from .models import Choice, Question
# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

之前urls.py 通过模式匹配拆分提取内容赋值给变量，这里reverse()方法正好相反
reverse()将url名加参数重新组装回一个URL

## 2. 使用通用的Views 精简代码
detail() 和 results() 代码相似，是冗余的。这在网页开发中很有代表性，比如根据URL里的参数从数据库获取数据， 载入模版，在返回渲染后的模版。 因为这很常见所以Django提供了快捷方式，称为“generic views”系统。

Generic views 抽象出相同的模式以便你不需要为app写Python代码，要应用Generic views分三步：
- 修改URLconf
- 删除旧的views
- 采用新的基于Django generic views的新views

**polls/urls.py**

```Python
from django.conf.urls import url

from . import views

app_name = 'polls'
urlpatterns = [
    url(r'^$', views.IndexView.as_view(), name='index'),
    url(r'^(?P<pk>[0-9]+)/$', views.DetailView.as_view(), name='detail'),
    url(r'^(?P<pk>[0-9]+)/results/$', views.ResultsView.as_view(), name='results'),
    url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
]
```

**polls/views.py**

```Python
from django.shortcuts import get_object_or_404, render
from django.http import HttpResponseRedirect
from django.urls import reverse
from django.views import generic

from .models import Choice, Question


class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by('-pub_date')[:5]


class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'


class ResultsView(generic.DetailView):
    model = Question
    template_name = 'polls/results.html'


def vote(request, question_id):
    ... # same as above, no changes needed.
```

ListView 和 DetailView 抽象出两个概念“展示一个对象列表”和”展示一个特定类型对象的详情页“
- 每个generic view都需要知道他们操作的数据模型：通过**model** 属性指定。
- DetailView 需要从URL获得变量名为**pk**的主键值，所以要从urls.py中修改变量名Question_id为pk
模版默认自动生成在**<app name>/<model name>_detail.html**(以DetailView为例)。详情页和结果页都用DetailView，为了最终渲染出不同的页面，可以用**template_name**指定模版位置。

Django能够为上下文决定合适的变量名：<model name>作为变量名，对于DetailView: e.g. question。对于ListView 则会创建一个上下文变量<model name_list> e.g. question_list。因为我们在模版中用了latest_question_list，所以在这里用`context_object_name = 'latest_question_list'` 修改默认变量名。或者，也可以去模版里修改。看哪个方法更有意义吧。

For full details on generic views, see the [generic views documentation](https://docs.djangoproject.com/en/1.11/topics/class-based-views/).
