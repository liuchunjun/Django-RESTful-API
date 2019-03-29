# Django开发RESTful API教程001

## 项目设置

- 创建一个名为的新Django项目`tutorial`，然后启动一个名为的新应用程序`quickstart`。

```
# Create the project directory
mkdir tutorial
cd tutorial

# Create a virtualenv to isolate our package dependencies locally
virtualenv env
source env/bin/activate  # On Windows use `env\Scripts\activate`

# Install Django and Django REST framework into the virtualenv
pip install django
pip install djangorestframework

# Set up a new project with a single application
django-admin startproject tutorial .  # Note the trailing '.' character
cd tutorial
django-admin startapp quickstart
cd ..
```

### 项目布局应如下所示

```
$ pwd
<some path>/tutorial
$ find .
.
./manage.py
./tutorial
./tutorial/__init__.py
./tutorial/quickstart
./tutorial/quickstart/__init__.py
./tutorial/quickstart/admin.py
./tutorial/quickstart/apps.py
./tutorial/quickstart/migrations
./tutorial/quickstart/migrations/__init__.py
./tutorial/quickstart/models.py
./tutorial/quickstart/tests.py
./tutorial/quickstart/views.py
./tutorial/settings.py
./tutorial/urls.py
./tutorial/wsgi.py
```

### 在项目目录中创建应用程序可能看起来很不寻常。使用项目的命名空间可以避免与外部模块的名称冲突

- 现在首次同步您的数据库

```
python manage.py migrate
```

- 创建超级用户

```
python manage.py createsuperuser
```

- 一旦你设置了一个数据库并创建了初始用户并准备就绪，打开应用程序的目录，我们就会得到编码......



## 序列化

- 首先，我们将定义一些序列化器。让我们创建一个名为的新模块`tutorial/quickstart/serializers.py`，我们将用于数据表示。
- 在此之前,用Pycharm打开我们的项目,并设置我们的Python环境为开始创建的virtualenv环境
- serializers.py

```python
from django.contrib.auth.models import User, Group
from rest_framework import serializers


class UserSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = User
        fields = ('url', 'username', 'email', 'groups')


class GroupSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Group
        fields = ('url', 'name')
```

- 请注意，我们在这种情况下使用的是超链接关系`HyperlinkedModelSerializer`。您还可以使用主键和其他各种关系，但超链接是一种很好的RESTful设计。



## 视图

- 是的，我们最好先写一些视图函数。打开`tutorial/quickstart/views.py`并打字。

```python
from django.contrib.auth.models import User, Group
from rest_framework import viewsets
from .serializers import UserSerializer, GroupSerializer


class UserViewSet(viewsets.ModelViewSet):
    """
    API endpoint that allows users to be viewed or edited
    运行用户查看和编辑的API接口
    """
    queryset = User.objects.all().order_by('-date_joined')
    serializer_class = UserSerializer


class GroupViewSet(viewsets.ModelViewSet):
    """
    API endpoint that allows groups to be viewed or edited
    允许组员查看或编辑的API接口
    """
    queryset = Group.objects.all()
    serializer_class = GroupSerializer

```

- 我们不是编写多个视图，而是将所有常见行为组合在一起调用`ViewSets`。
- 如果需要，我们可以轻松地将它们分解为单独的视图，但使用视图集可以使视图逻辑组织良好，并且非常简洁。



## 路由

- 好的，现在让我们连接API URL。到`tutorial/urls.py`......

```python
from django.urls import include, path
from rest_framework import routers
from quickstart import views

urls = routers.DefaultRouter()
urls.register(r'users', views.UserViewSet)
urls.register(r'groups', views.GroupViewSet)

urlpatterns = [
    path('', include(urls.urls)),
    path('api-auth/', include('rest_framework.urls', namespace='rest_framework'))
]
```

- 因为我们使用视图集而不是视图，所以我们可以通过简单地使用路由器类注册视图集来自动为我们的API生成URL conf。
- 同样，如果我们需要更多地控制API URL，我们可以简单地使用常规的基于类的视图，并明确地编写URL conf。
- 最后，我们将包含默认登录和注销视图，以便与可浏览API一起使用。这是可选的，但如果您的API需要身份验证并且您想要使用可浏览的API，则非常有用。



## 分页

- 分页允许您控制每页返回的对象数。要启用它，请添加以下行`tutorial/settings.py`

```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10
}
```



## 添加App

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'quickstart.apps.QuickstartConfig',
    'rest_framework',
]
```



## 测试我们的API

- 我们现在准备测试我们构建的API。让我们从命令行启动服务器。

```
python manage.py runserver
```

- 打开postman 没有下载的同学可以先自行百度下载
- 以get方式请求

```
http://127.0.0.1:8000/users/
```

- 如果每一步你都跟上了,这个时候API会返回你创建的超级用户
