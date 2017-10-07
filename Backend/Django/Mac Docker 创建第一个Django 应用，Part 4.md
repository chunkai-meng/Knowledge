# Mac Docker 创建第一个Django 应用，Part 4

> 参考原文：[Writing your first Django app, part 4](https://docs.djangoproject.com/en/1.11/intro/tutorial04/)  
> 本文Python搭建在 Django Compose + Djang  执行Python需进入web server容器中，请参看[第一步：在Mac构建Django 容器]  
> 翻译整理：CK

简单的表单处理以及精简代码

## 1. 简单表单
**polls/templates/polls/detail.html**
```
<h1>{{ question.question_text }}</h1>

{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
{% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}" />
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br />
{% endfor %}
<input type="submit" value="Vote" />
</form>
```
