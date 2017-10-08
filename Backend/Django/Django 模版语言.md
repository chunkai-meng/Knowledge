# Django 模版语言

> 参考原文：[Django Template language](https://docs.djangoproject.com/en/1.11/topics/templates/)  
> 翻译整理：CK

## 变量
变量形如`{{ and }}`，它从上下文context中取值输出
```
My first name is {{ first_name }}. My last name is {{ last_name }}.
```
如果上下文是：
```
{'first_name': 'John', 'last_name': 'Doe'}
```
则输出结果为：

**My first name is John. My last name is Doe.**

**字典**检索／**属性**检索／**列表-索引**检索都是用点标记

如果变量解析到一个可以可以被调用的对象，模版将自动调用它不传递任何参数，其结果作为变量的值。

## 标记
标记为渲染过程提供任意的逻辑
**{% csrf\_token %}**

设计者有意模糊了它的定义，例如它可以输出内容，也可以像个控制结构，例如：if语句，for循环，从数据库提取数据，甚至可以接入到其他模版标记.

很多标记接受实参
```
{% cycle 'odd' 'even' %}
```

部分标记需要开始和结束标记
```
{% if user.is_authenticated %}Hello, {{ user.username }}.{% endif %}
```

更多有关标记参考[内建模版标记和过滤器](https://docs.djangoproject.com/en/1.11/ref/templates/builtins/#ref-templates-builtins-tags)
