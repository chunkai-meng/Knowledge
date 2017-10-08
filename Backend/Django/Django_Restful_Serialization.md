# Django 序列化

> 参考原文：[Tutorial 1: Serialization](http://www.django-rest-framework.org/tutorial/1-serialization/)  
> 本文Python搭建在 Django Compose + Djang  执行Python需进入web server容器中，请参看[第一步：在Mac构建Django 容器]  
> 翻译整理：CK

## 1. 配置环境：
```
virtualenv env
# 进入环境
source env/bin/activate
```

安装需要的包
```
pip install django
pip install djangorestframework
pip install pygments  # We'll be using this for the code highlighting
```

## 2. 开始

创建项目

```
cd ~
django-admin.py startproject tutorial
cd tutorial
```

创建app

```
python manage.py startapp snippets
```

添加例子app和rest框架app到`INSTALL_APPS`

**tutorial/settings.py**
```
INSTALLED_APPS = (
    ...
    'rest_framework',
    'snippets.apps.SnippetsConfig',
)
```

## 3. 创建数据模型

```Python
from django.db import models
from pygments.lexers import get_all_lexers
from pygments.styles import get_all_styles

LEXERS = [item for item in get_all_lexers() if item[1]]
LANGUAGE_CHOICES = sorted([(item[1][0], item[0]) for item in LEXERS])
STYLE_CHOICES = sorted((item, item) for item in get_all_styles())


class Snippet(models.Model):
    created = models.DateTimeField(auto_now_add=True)
    title = models.CharField(max_length=100, blank=True, default='')
    code = models.TextField()
    linenos = models.BooleanField(default=False)
    language = models.CharField(choices=LANGUAGE_CHOICES, default='python', max_length=100)
    style = models.CharField(choices=STYLE_CHOICES, default='friendly', max_length=100)

    class Meta:
        ordering = ('created',)
```

创建起始的数据迁移并同步数据库

```
python manage.py makemigrations snippets
python manage.py migrate
```

## 4.创建序列化类

- 首先要提供方法来序列化和反序列化方一个实例到表现层（例如：json）。可以声明一个序列化器，这有点类似Django的forms。
创建snippets/serializers.py.

```Python
from rest_framework import serializers
from snippets.models import Snippet, LANGUAGE_CHOICES, STYLE_CHOICES


class SnippetSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(required=False, allow_blank=True, max_length=100)
    code = serializers.CharField(style={'base_template': 'textarea.html'})
    linenos = serializers.BooleanField(required=False)
    language = serializers.ChoiceField(choices=LANGUAGE_CHOICES, default='python')
    style = serializers.ChoiceField(choices=STYLE_CHOICES, default='friendly')

    def create(self, validated_data):
        """
        Create and return a new `Snippet` instance, given the validated data.
        """
        return Snippet.objects.create(**validated_data)

    def update(self, instance, validated_data):
        """
        Update and return an existing `Snippet` instance, given the validated data.
        """
        instance.title = validated_data.get('title', instance.title)
        instance.code = validated_data.get('code', instance.code)
        instance.linenos = validated_data.get('linenos', instance.linenos)
        instance.language = validated_data.get('language', instance.language)
        instance.style = validated_data.get('style', instance.style)
        instance.save()
        return instance
```

- 序列化器类首先定义需要序列化和反序列化的字段。
- create() 和 update() 定义当调用`serializer.save()` 的时候，填满值的实例如何被创建和修改。
- 序列化类很想Django的Form类，包含各个字段的类似的校验标记，例如：`required`, `max_length` 和 `default`。
- 字段标记还能控制特定条件下序列化器应该如何显示。例如渲染到HTML。
	`{'base_template': 'textarea.html'}` 标记等同于 Django Form 中到`widget=widgets.Textarea` 类。
	这对可以被浏览器浏览到API尤其有用。

使用序列化器

登录：

```
python manage.py shell
```

创建几个 snippets
```Python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser

snippet = Snippet(code='foo = "bar"\n')
snippet.save()

snippet = Snippet(code='print "hello, world"\n')
snippet.save()
```

我们已经有几个snippet 实例了，试试序列化它们：

```
serializer = SnippetSerializer(snippet)
serializer.data
```

这时我们已经将模型实例转化成Pyton原声数据类型了。通过渲染数据成json格式完成序列化。

```Python
content = JSONRenderer().render(serializer.data)
content
# '{"id": 2, "title": "", "code": "print \\"hello, world\\"\\n", "linenos": false, "language": "python", "style": "friendly"}'
```

同样的要反序列化：先现将数据流发到Python原生数据类型分析

```
from django.utils.six import BytesIO

stream = BytesIO(content)
data = JSONParser().parse(stream)
```

再将它们存进一个完全填满的对象实例。

```Python
serializer = SnippetSerializer(data=data)
serializer.is_valid()
# True
serializer.validated_data
# OrderedDict([('title', ''), ('code', 'print "hello, world"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')])
serializer.save()
# <Snippet: Snippet object>
```

也可以序列化查询结婚而不是模型实例，这样仅需要增加一个`many=Ture` 标记

```
serializer = SnippetSerializer(Snippet.objects.all(), many=True)
serializer.data
# [OrderedDict([('id', 1), ('title', u''), ('code', u'foo = "bar"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')]), OrderedDict([('id', 2), ('title', u''), ('code', u'print "hello, world"\n'), ('linenos', False), ('language', 'python'), ('style', 'friendly')]), OrderedDict([('id', 3), ('title', u''), ('code', u'print "hello, world"'), ('linenos', False), ('language', 'python'), ('style', 'friendly')])]
```

## 5. 使用模型序列化


