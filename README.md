# red_book

- 시작하기

```powershell
$ virtualenv venv
$ . venv/bin/activate
$ pip install django
$ django-admin startproject <프로젝트명>
$ python3 manage.py startapp <에플리케이션명>
$ python3 manage.py migrate
$ python3 manage.py createsuperuser
...
...
$ python3 manage.py runserver
```

127.0.0.1:8000/admin경로로 들어가서 로그인