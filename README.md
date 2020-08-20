# django-project

- [polls app](#1.-polls-app)
- [blog app](#2.-blog-app)

## 1. polls app

- django 설치

```shell
$ virtualenv venv
$ . venv/bin/activate
$ pip install django
```

- 프로젝트, 앱 만들기
- 데이터베이스 파일, 슈퍼유저 생성

```shell
$ django-admin startproject <프로젝트명>
$ python3 manage.py startapp <에플리케이션명>
$ python3 manage.py migrate
$ python3 manage.py createsuperuser
```

- app을 project에 추가하기

```python
# <project>/settings.py
INSTALLED_APPS = [
    'polls.apps.PollsConfig', 
    # 앱이름.apps.앱이름Config
    # 앱 디렉토리에 apps.py에 앱이름이 있다. 그걸 넣어줘야함.
    ...
]

# urls.py
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')), # 추가한 앱으로 연결될 path를 추가한다.
    path('admin/', admin.site.urls),
]
```

- 앱 설정
  - `admin.py` 
  - `models.py`
  - `views.py`
  - `urls.py`
- **admin.py**
  - admin페이지에 만든 앱을 추가한다.
  - 기타 관리자 페이지의 UI설정도 할 수 있음.

```python
# <앱디렉토리>/admin.py
from django.contrib import admin

from .models import Question

admin.site.register(Question)
# /admin/ 에서 확인
```

- **models.py**
  - 모델(데이터베이스의 구조)를 만든다. 
  - 클래스가 모델명, 안에 변수가 컬럼(필드)가 된다.

```python
from django.db import models
from django.utils import timezone

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

    def __str__(self):
        return self.question_text

    def was_published_recently(self):
        now = timezone.now()
```

- **views.py**
  - 받은 요청에 대한 처리를 하는 로직이 정의되는 곳이다. 
  - 데이터베이스와 템플릿 등을 받은 요청에 맞게 응답한다.

```python
class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'

    def get_queryset(self):
        """
        Excludes any questions that aren't published yet.
        """
        return Question.objects.filter(pub_date__lte=timezone.now())
```

- **urls.py**
  - url의 구조를 만들고, view와 연결한다. 

```python
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
    path('<int:pk>/', views.DetailView.as_view(), name='detail'),
    path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

- 기타
  1. templates 디렉토리
  2. static 디렉토리
- 템플릿 파일
- `<앱이름>/templates/<앱이름>/index.html`
- `<앱이름>/templates/<앱이름>/detail.html`

```html
{% load static %}

<link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}">

{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <!-- <li><a href="/polls/{{question.id}}/">{{question.question_text}}</a></li> -->
        <li><a href="{% url 'polls:detail' question.id %}">{{question.question_text}}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

- css, image 파일
- `<앱이름>/static/<앱이름>/style.css`
- `<앱이름>/static/<앱이름>/<images>/test.jpg`

그리고 위와 같이 템플릿에 추가를 해줘야함.

```html
{% load static %}

<link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}">
```