# 노트

### \__name__

```python
if __name__ == '__main__':
    main()
```

이게 뭔지 궁금했음.
모듈(파일)을 실행하는 방법은 직접 실행하거나, 임포트 하거나.

- 직접 실행하는 경우 => `__name__="__main__"` 이 된다. 
- 다른 모듈에서 임포트해서 쓰는 경우 => `__name__="executeThisModule"` 이 된다. 

### app vs Project

- app은 특정 기능을 하는 뭔가(데이터베이스, 웹로그 등등)
- project는 app과 설정들의 집합

### RDBMS, 관계형 데이터베이스 용어정리

- Database
  데이터베이스
- Table
  데이터베이스에 저장되는 가장 큰 단위인 테이블. 
  여러개의 테이블이 하나의 데이터베이스에 생성될 수 있음. 
- Column(=Field)
  테이블의 세로 줄 하나하나를 컬럼이라고 함. 
  날짜, 시간, 제목.. 등등 하나의 특징(속성)들이 컬럼으로 나눠짐.
- Row(=Record)
  여러개의 컬럼을 가지는 하나의 줄 단위를 Row라고 함.
  같은 테이블 내의 Row는 동일한 컬럼 값들을 가짐.
- Schema
  데이터베이스를 설계, 생성하는 과정에서 각각의 테이블에 필요한 컬럼의 타입과 네이밍을 결정하는 것을 데이터베이스 스키마라고 함.

### migrations

```powershell
$ python3 manage.py makemigrations <앱이름> => 커밋
$ python3 manage.py migrate <앱이름> => 배포
```

makemigrations는 <앱이름>/migrations/0001_initial.py 생성.
migrations에는 마치 git처럼 모델의 변화가 저장됨.

migrate는 migrations에 저장된 변경사항을 적용함.
마치 commit - poblish같은 개념이다. 

### \__init__.py

프로젝트 디렉토리의 `__init__.py`는 해당 디렉토리를 패키지처럼 다루라고 알려주기 위함이다. 빈파일임.

### URL, path

```python
# path( <route>(필수), <view>(필수), <name>(선택가능))
# mysite/urls.py
urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

include함수는 다른 URLconf를 참조할 수 있도록 도와준다. 
`include('<앱이름>.urls')`

- route = `https://naver.com/apple` 하면 `apple/` 부분이다. 
- view = 일치하는 route를 찾으면 HttpRequest 객체와 캡처된 키워드를 인자로 하여 view함수를 호출한다. 
- name = URL에 이름을 지으면, django어디서나 참조할 수 있다. 필수는 아니다.

### 모델

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

shell사용하기

```shell
>>> from polls.models import Choice, Question  # Import the model classes we just wrote.

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

# Now it has an ID.
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
<QuerySet [<Question: Question object (1)>]>
```

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



```shell
>>> from polls.models import Choice, Question

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

## Part 3 

### view

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

