# django??

- [1. Model](#1.-Model)
- [2. View](2.-View)
- [3. Template](#3.-Template)
- 

## 1. Model

**모델(model)은 부가적인 메카데이타를 가진 데이터베이스의 구조(layout)을 말한다.**
저장할 데이터의 필수적인 필드들과 동작들을 포함하고 있다. 
데이터베이스 모델을 한곳에 정의한다. 
`<앱이름>/model.py` 

```python
from django.db import models

class Question(models.Model):
    # 컬럼명 = 필드
    # 각 필드의 자료형을 Field클래스의 인스턴스로 나타낸다. (CharField, IntegerField)
    question_text = models.CharField(max_length=200) 
    pub_date = models.DateTimeField('date published')

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

해당 파일에 정의한 모델은 `django.db.models.Model`의 서브클래스이다. 
각 모델은 몇개의 클래스변수(프로퍼티)를 가지고 있다. 
ForeignKey는 각각의 Choice가 하나의 Question에 관계된다는 것을 django에게 알려준다. 

> [`ForeignKey`](https://docs.djangoproject.com/en/3.1/ref/models/fields/#django.db.models.ForeignKey). That tells Django each `Choice` is related to a single `Question`. Django supports all the common database relationships: many-to-one, many-to-many, and one-to-one.

정의한 모델 객체에 접근하기 위해서는 현재프로젝트에 해당 앱이 설치된 사실을 알려야 한다. 
`프로젝트명/settings.py` 의 INSTALLED_APPS에 추가하면 된다. 
앱이름이 jimin 이면, `jimin.apps.JiminConfig` 경로를 추가하면 된다. 

그리고 아래 명령을 통해 모델을 변경시킨 사실(=새로운 모델을 생성한 사실)과 이 변경사항(migration)을 저장하고 싶다는 것을 django에게 알린다. 

```powershell
$ python3 manage.py makemigrations <앱이름>
$ python3 manage.py migrate <앱이름>
```

- 앱을 만들고
- 모델을 만들고
- 앱을 프로젝트에 추가하고
- makemigrations 명령을 통해 변경사항에 대한 마이그레이션을 생성하고,
- migrate 명령으로 적용안된 마이그레이션(모델 변경사항)을 데이터베이스에 적용한다. 



위의 출력결과는 객체를 표현하는 것에 도움이 안된다. 모델에 `__str__()` 메서드를 추가한다. 

```python
import datetime

from django.db import models
from django.utils import timezone

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

    def __str__(self):
        return self.question_text

    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    # 각각의 Choice가 하나의 Question에 관계된다는 것을 django에게 알림.
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

    def __str__(self):
        return self.choice_text
```

관리자 만들기

```shell
$ python3 manage.py createsuperuser
Username: ...
Email address: ...
Password: ...

$ python3 manage.py runserver
# 127.0.0.1:8000/admin/ 에 접근하여 로그인
# django.contrib.auth 모듈에 의해 생성된 편집가능한 그룹과 사용자 등을 볼 수 있다. 
```

근데 poll app이 관리자 페이지에서 보이지 않는다. Question은 관리자가 편집가능해야한다. 
admin에게 Question object가 admin 인터페이스를 가진다는 것을 알려야한다. 

```python
# polls/admin.py
from django.contrib import admin
from .models import Question
admin.site.register(Question)
```

그럼 관리자 인덱스페이지에 표시가 된다.

## 3. view

뷰는 Django 어플리케이션이 특정 기능과 템플릿을 제공하는 웹페이지의 한 종류이다.
예를 들어 블로그에서는 홈페이지(최근글항목), 세부페이지(포스팅내용), 댓글기능, 월별페이지 등등이 있다. 

튜토리얼에서 만드는  투표앱에는 아래 4개의 view를 만들것이다. 

| 질문 index 페이지  | 최근의 질문들을 표시                    |
| ------------------ | --------------------------------------- |
| 질문 detail 페이지 | 질문 내용과, 투표 서식을 표시           |
| 질문 result 페이지 | 특정 질문에 대한 결과를 표시            |
| 투표기능           | 질문에 대해 특정 선택을 할 수 있는 기능 |

장고에서는 웹페이지와 콘텐츠가 view를 통해 전달이 된다. 
URL의 정보(도메인 이후 내용)에 따라 그에 연결된 view 함수가 리턴된다. 
URL로부터 뷰를 얻기위해 django는 `URLconfs` 라는 것을 사용한다. 

### 뷰 추가하기

- views.py에 함수(뷰) 추가하기
- path() 호출로 \<앱>.urls.py 모듈에 뷰를 연결하기

```python
# polls/views.py
# 뷰 추가하기
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```

```python
# polls/urls.py
# 패턴에 만든 뷰를 URL과 연결시키는 작업을 한다. 
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'), # ex: /polls/
    path('<int:question_id>/', views.detail, name='detail'), # ex: /polls/5/
    path('<int:question_id>/results/', views.results, name='results'), # ex: /polls/5/results/
    path('<int:question_id>/vote/', views.vote, name='vote'), # ex: /polls/5/vote/
]
```

이렇게 하고, 해당 URL을 요청하면 설정한 텍스트가 보인다.

뷰는 두가지 중 하나를 하게 되어있다. 

- HttpResponse객체를 반환.
- Http404 예외를 반환.

또한 뷰는 데이터 베이스를 읽고, 여러 템플릿을 사용하고, PDF, zip등의 파일생성, XML출력 등 파이썬의 어떤 라이브러리도 사용할 수 있다. 그렇지만 django에게 필요한 것은 HttpResponse객체 혹은 예외이다. 

뷰는 응답이지만 페이지 디자인 관련 내용은 포함하지 않는다. 
그런 내용은 템플릿에서 관리한다. (model, view, template)
앱디렉토리에 `templates` 라는 디렉토리를 만든다. 그럼 django는 여기서 템플릿을 찾게 된다. 
그리고 `templates/<앱이름>/index.html` 이렇게 파일을 만든다. 좀 이상해보일 수는 있다. 

```python
polls/templates/polls/index.html
```

근데 이게 app_directories 템플릿 로더가 동작하는 방식이다.  

그리고 아래 내용을 넣는다. 

```python
# polls/views.py
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
...
```

```html
<!-- polls/templates/polls/index.html -->
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

뷰를 보면, 로더를 통해 index템플릿을 불러오고, context에 전달한다. 
context는 템플릿에서 쓰이는 변수명과 파이썬 객체를 연결짓는 dictionary값이다. 

그리고 /polls/  페이지를 불러오면, 질문 리스트가 보이고, 링크로 디테일페이지가 잡혀있다. 

템플릿에 context를 채워넣어 표현한 결과를 HttpResponse객체로 반환하는 것을 더 간단하게 나타낼 수 있다. 

```python
from django.shortcuts import render

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
```

`shortcuts` 모듈의 `render` 함수를 반환하면 같은 결과를 만들 수 있다. 
따라서 전체 뷰를 render로 바꾸면 loader와 HttpResponse를 임포트하지 않아도 된다. 

```python
render(<request객체>, <템플릿이름>, <사전형context>(이건 없어도 됨.))
```

객체가 존재하지 않을 때는 404에러를 내야한다.

```python
from django.shortcuts import get_object_or_404, render

from .models import Question
# ...
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```





