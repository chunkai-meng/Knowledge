# Mac Docker 创建第一个Django 应用，Part 2

> 原文：[Writing your first Django app, part 2](https://docs.djangoproject.com/en/1.11/intro/tutorial02/)  
本文Python搭建在 Django Compose + Djang 执行Python需进入web server容器中，请参看[第一步：在Mac构建Django 容器]  
> 翻译整理：CK

## Part 2：配置后端

### 1. 配置数据库

已有建有的项目为mysite，Docker Django 的教程里建立了一个composeexample，这里不适用它。假设我们的真实项目为mysite，打开myiste/settings.py，默认配置的数据库是SQLite，SQLite包含在Python里，无需安装任何其他东西。如果第一次创建真实的项目，仍然希望用可伸缩性更强的数据库像PostgreSQL，免得将来又要切换数据库则：

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'postgres',
        'USER': 'postgres',
        'HOST': 'db',
        'PORT': 5432,
    }
}
```

其他数据库绑定参考：[Database Bindings](https://docs.djangoproject.com/en/1.11/topics/install/#database-installation)

由于使用了Docker容器来运行python，数据迁移`$ python manage.py migrate`也需要在容器里做，所以先查看容器

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
bf1ef3b74f1b        mysite_web          "python3 manage.py..."   About an hour ago   Up 13 minutes       0.0.0.0:8000->8000/tcp   mysite_web_1
09f6cebc8741        postgres            "docker-entrypoint..."   2 hours ago         Up 13 minutes       5432/tcp                 mysite_db_1
```
进入容器并执行migrate：
```
docker exec -it mysite_db_1 bash
$ python manage.py migrate
```
这条命令根据myiste/settings.py里的 INSTALLED_APPS 的需要安装必要数据库表格，安装完成后再连接PostgreSQL `psql DBNAME USERNAME`
```
root@09f6cebc8741:/# psql postgres postgres
```
输入`\dt` 应该能看到类似Tables

```
postgres=# \dt
                   List of relations
 Schema |            Name            | Type  |  Owner
--------+----------------------------+-------+----------
 public | auth_group                 | table | postgres
 public | auth_group_permissions     | table | postgres
 public | auth_permission            | table | postgres
 public | auth_user                  | table | postgres
 public | auth_user_groups           | table | postgres
 public | auth_user_user_permissions | table | postgres
 public | django_admin_log           | table | postgres
 public | django_content_type        | table | postgres
 public | django_migrations          | table | postgres
 public | django_session             | table | postgres
(10 rows)
```

退出Postgres
```
postgres-# \q
```

### 2. 创建数据模型

编辑`polls/models.py`
```python
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

### 3. 启动数据模型
1. 通知Prject，polls app 应用已安装

    编辑mysite/settings.py，添加一行`'polls.apps.PollsConfig',`

```
INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```


2. 到web容器中执行命令：

```
$ python manage.py makemigrations polls
```
makemigrations 告诉Django你对模型做了些修改，并并希望保存下来。Django会创建修改脚本polls/migrations/0001_initial.py，
应该会看到类似：

```
Migrations for 'polls':
  polls/migrations/0001_initial.py:
    - Create model Choice
    - Create model Question
    - Add field question to choice
```

到此数据库并未改变，真正执行变更并且帮助你管理数据库模式是`migration`命令，可以用如下命令查看将要执行的SQL命令
```
$ python manage.py sqlmigrate polls 0001
```

应该会看到类似（不同数据库会有所不同）：
```
BEGIN;
--
-- Create model Choice
--
CREATE TABLE "polls_choice" ("id" serial NOT NULL PRIMARY KEY, "choice_text" varchar(200) NOT NULL, "votes" integer NOT NULL);
--
-- Create model Question
--
CREATE TABLE "polls_question" ("id" serial NOT NULL PRIMARY KEY, "question_text" varchar(200) NOT NULL, "pub_date" timestamp with time zone NOT NULL);
--
-- Add field question to choice
--
ALTER TABLE "polls_choice" ADD COLUMN "question_id" integer NOT NULL;
CREATE INDEX "polls_choice_question_id_c5b4b260" ON "polls_choice" ("question_id");
ALTER TABLE "polls_choice" ADD CONSTRAINT "polls_choice_question_id_c5b4b260_fk_polls_question_id" FOREIGN KEY ("question_id") REFERENCES "polls_question" ("id") DEFERRABLE INITIALLY DEFERRED;
COMMIT;
```

保存数据库修改：
```
$ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Rendering model states... DONE
  Applying polls.0001_initial... OK
```
小结：
修改数据库分三步：
1. 修改模型 (in models.py).
1. 执行: `python manage.py makemigrations` 创建一个迁移
1. 执行：`python manage.py migrate` 保存变更

### 4. API
下面进入交互式的Python Shell跟API一起玩耍吧，别忘了先进入容器。
```
$ python manage.py shell
```
如需了解database API的相关内容[：database API](https://docs.djangoproject.com/en/1.11/topics/db/queries/)

可进行一下尝试了解API：
```
>>> from polls.models import Question, Choice   # Import the model classes we just wrote.

# No questions are in the system yet.
>>> Question.objects.all()
<QuerySet []>

# Create a new Question.
# Support for time zones is enabled in the default settings file, so
# Django expects a datetime with tzinfo for pub_date. Use timezone.now()
# instead of datetime.datetime.now() and it will do the right thing.
>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())

# Save the object into the database. You have to call save() explicitly.
>>> q.save()

# Now it has an ID. Note that this might say "1L" instead of "1", depending
# on which database you're using. That's no biggie; it just means your
# database backend prefers to return integers as Python long integer
# objects.
>>> q.id
1

# Access model field values via Python attributes.
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)

# Change values by changing the attributes, then calling save().
>>> q.question_text = "What's up?"
>>> q.save()

# objects.all() displays all the questions in the database.
>>> Question.objects.all()
<QuerySet [<Question: Question object>]>
```

为了让`<Question: Question object>`输出更可读
给模块Question and Choice (in the polls/models.py file) 添加 `__str__()` 方法
```
from django.db import models
from django.utils.encoding import python_2_unicode_compatible

@python_2_unicode_compatible  # only if you need to support Python 2
class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text

@python_2_unicode_compatible  # only if you need to support Python 2
class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
```

除了那些常见的Python方法，可以添加一个自定义的
```
import datetime

from django.db import models
from django.utils import timezone


class Question(models.Model):
    # ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
```

在次进入shell看看有哪些变化：
```
>>> from polls.models import Question, Choice

# Make sure our __str__() addition worked.
>>> Question.objects.all()
<QuerySet [<Question: What's up?>]>

# Django provides a rich database lookup API that's entirely driven by
# keyword arguments.
>>> Question.objects.filter(id=1)
<QuerySet [<Question: What's up?>]>
>>> Question.objects.filter(question_text__startswith='What')
<QuerySet [<Question: What's up?>]>

# Get the question that was published this year.
>>> from django.utils import timezone
>>> current_year = timezone.now().year
>>> Question.objects.get(pub_date__year=current_year)
<Question: What's up?>

# Request an ID that doesn't exist, this will raise an exception.
>>> Question.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Question matching query does not exist.

# Lookup by a primary key is the most common case, so Django provides a
# shortcut for primary-key exact lookups.
# The following is identical to Question.objects.get(id=1).
>>> Question.objects.get(pk=1)
<Question: What's up?>

# Make sure our custom method worked.
>>> q = Question.objects.get(pk=1)
>>> q.was_published_recently()
True

# Give the Question a couple of Choices. The create call constructs a new
# Choice object, does the INSERT statement, adds the choice to the set
# of available choices and returns the new Choice object. Django creates
# a set to hold the "other side" of a ForeignKey relation
# (e.g. a question's choice) which can be accessed via the API.
>>> q = Question.objects.get(pk=1)

# Display any choices from the related object set -- none so far.
>>> q.choice_set.all()
<QuerySet []>

# Create three choices.
>>> q.choice_set.create(choice_text='Not much', votes=0)
<Choice: Not much>
>>> q.choice_set.create(choice_text='The sky', votes=0)
<Choice: The sky>
>>> c = q.choice_set.create(choice_text='Just hacking again', votes=0)

# Choice objects have API access to their related Question objects.
>>> c.question
<Question: What's up?>

# And vice versa: Question objects get access to Choice objects.
>>> q.choice_set.all()
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
>>> q.choice_set.count()
3

# The API automatically follows relationships as far as you need.
# Use double underscores to separate relationships.
# This works as many levels deep as you want; there's no limit.
# Find all Choices for any question whose pub_date is in this year
# (reusing the 'current_year' variable we created above).
>>> Choice.objects.filter(question__pub_date__year=current_year)
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

# Let's delete one of the choices. Use delete() for that.
>>> c = q.choice_set.filter(choice_text__startswith='Just hacking')
>>> c.delete()
```

### 5. 介绍Django Admin

#### 1. 创建Admin用户
```
$ python manage.py createsuperuser
```
```
Username: admin
```

```
Email address: admin@example.com
```

验证两次密码：
```
Password: **********
Password (again): *********
Superuser created successfully.
```
注册成功

#### 2. 开始服务器开发
Django的Admin是默认激活的

启动服务器 `docker-compose up`  

打开： http://127.0.0.1:8000/admin/

![image](https://docs.djangoproject.com/en/1.11/_images/admin01.png)

输入账号密码

![image](https://docs.djangoproject.com/en/1.11/_images/admin02.png)

#### 3. 通过admin来管理poll应用
修改**polls/admin.py** 注册Question到admin site 来告诉admin Question 已经提供了admin的接口
```
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```

![image](https://docs.djangoproject.com/en/1.11/_images/admin03t.png)

可以看到之前创建的question

![image](https://docs.djangoproject.com/en/1.11/_images/admin04t.png)

可以修改

![image](https://docs.djangoproject.com/en/1.11/_images/admin05t.png)

如果选择时间**Now**的时候发现时间不对，那可能就是时区问题。
检查之前的设置`mysite/setting.py` 例如：
```
#TIME_ZONE = 'UTC'
TIME_ZONE = 'Pacific/Auckland'
```
好了，完成第二步了
