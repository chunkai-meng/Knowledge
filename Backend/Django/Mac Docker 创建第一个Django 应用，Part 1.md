# Mac Docker 创建第一个Django 应用，Part 1
## 第一步：在Mac构建Django 容器

> 原文：[Quickstart: Compose and Django]("https://docs.docker.com/compose/django/#create-a-django-project")  
> 翻译整理：CK

这篇文章将指导你如何用Docker Compose 配置和启动一个简单的 Django + PostgreSQL 应用。请先确保您已安装Compose：
[Install Docker Compose]("https://docs.docker.com/compose/install/")

**定义您的项目组件**

您需要创建一个Dockerfile 和一个Python 依赖文件，以及一个docker-compose.yml文件
1. 创建一个项目目录
1. 创建一个心的Dockerfile在当前项目目录下
1. 添加内容到Dockerfile
```
FROM python:3
ENV PYTHONUNBUFFERED 1
RUN mkdir /code
WORKDIR /code
ADD requirements.txt /code/
RUN pip install -r requirements.txt
ADD . /code/
```

4. 保存Dockerfile
5. 创建一个 requirements.txt
Dockerfile 中的 `RUN pip install -r requirements.txt` 将会用到它
6. 添加所需的软件到requirements.txt
```
Django>=1.8,<2.0
psycopg2
```
7. 保存requirements.txt
8. 创建一个docker-compose.yml
docker-compose.yml文件里描述了您的app所需要的服务。compose一词我认为翻译为编制更恰当。在这里我们需要一个web服务器，一个数据服务器。编制文件指明了我们这些服务所用的镜像，他们如何连接，哪些卷要挂载到容器。最后定义服务端口。
```
version: '3'

services:
  db:
    image: postgres
  web:
    build: .
    command: python3 manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/code
    ports:
      - "8000:8000"
    depends_on:
      - db
```
9. 保存 docker-compose.yml

**创建一个Django项目**
1. 转到项目根目录
1. 用docker-compose 创建项目
```
docker-compose run web django-admin.py startproject composeexample .
```
docker将启动web容器，并在里面执行 django-admin.py startproject composeexample，因为web镜像不存在所以compose先从当前目录建立它，见 build: 因为挂在了当前目录，所以新创建的项目文件在`docker-compose run`执行完推出后可以看到
3. ls 项目目录
```
$ ls -l
 drwxr-xr-x 2 root   root   composeexample
 -rw-rw-r-- 1 user   user   docker-compose.yml
 -rw-rw-r-- 1 user   user   Dockerfile
 -rwxr-xr-x 1 root   root   manage.py
 -rw-rw-r-- 1 user   user   requirements.txt
 ```

 **连接数据库**
1. 打开composeexample/settings.py
1. 替换DATABASE = …项
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
这些参数是根据docker-compose.yml所指定的postgres Docker 镜像决定的。
3. 保存
4. 执行docker-compose up
```
$ docker-compose up
djangosample_db_1 is up-to-date
Creating djangosample_web_1 ...
Creating djangosample_web_1 ... done
Attaching to djangosample_db_1, djangosample_web_1
db_1   | The files belonging to this database system will be owned by user "postgres".
db_1   | This user must also own the server process.
db_1   |
db_1   | The database cluster will be initialized with locale "en_US.utf8".
db_1   | The default database encoding has accordingly been set to "UTF8".
db_1   | The default text search configuration will be set to "english".

. . .

web_1  | May 30, 2017 - 21:44:49
web_1  | Django version 1.11.1, using settings 'composeexample.settings'
web_1  | Starting development server at http://0.0.0.0:8000/
web_1  | Quit the server with CONTROL-C.
```
此时，你的Django app应该运行在8000端口上了。浏览器打开http://localhost:8000应该能看到
![image](https://docs.docker.com/compose/images/django-it-worked.png)
5. 列出所有容器：
```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
def85eff5f51        django_web          "python3 manage.py..."   10 minutes ago      Up 9 minutes        0.0.0.0:8000->8000/tcp   django_web_1
678ce61c79cc        postgres            "docker-entrypoint..."   20 minutes ago      Up 9 minutes        5432/tcp                 django_db_1
```
6. 关闭容器  
Ctrl-C  
或者新开一个terminal执行： `docker-compose down`

## **部署已有的项目到容器**

1. 将docker-compose.yml requirements.txt Dockerfile 拷贝到Django项目的根目录，应与manage.py同目录
2. 运行`docker-compose up`
