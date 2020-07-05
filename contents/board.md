# Board 




## Table of Contents

1. [Project Setting](#project-setting)
2. [Application Setting](#application-setting)

---


## Project Setting

```bash
mkdir board-using-login
cd board-using-login
```

```bash
.Root folder
├── admin.py
├── apps.py
├── forms.py
├── migrations
│   ├── 0001_initial.py
├── models.py
├── templates
│   ├── base.html
│   ├── board_detail.html
│   ├── board_list.html
│   └── board_write.html
├── tests.py
├── urls.py
└── views.py
```


### config setting

`config` 뒤에 `.`을 찍지 않으면 config 폴더 아래에 config 폴더가 생기므로  `config .` 명령어를 입력하여 프로젝트를 생성할 수 있도록 주의한다

```
django-admin startproject config  # X
django-admin startproject config . # O
```

### Mac의 경우 가상환경 설정

```bash
python3 -m venv venv
```

터미널에서 가상 환경을 적용하는 파일

```
source venv/bin/activate
```


### djangp 설치

```bash
pip install django
```


### Django server 실행
프로젝트가 서버가 올바르게 생성되었는지 확인하기 위해 서버 실행!

```bash
python3 manage.py runserver # mac
python manage.py runserver # windows
```


### ALLOWED_HOSTS
`DEBUG`가 `True`이면 개발 모드 `DEBUG`가 `False`이면 운영모드로 장고가 인식하므로  
운영모드로 설정할 경우에는 서버의 IP나 도메인을 지정해야 한다.  
개발모드는 값을 따로 설정하지 않아도 `['localhost']`로 인식함

```python
DEBUG = True
ALLOWED_HOSTS = []
```

### INSTALLED_APPS
장고에서는 `INSTALLED_APPS`에 애플리케이션이 등록되어야 하는데, 이 때 간단하게 애플리케이션의 이름만 지정할 수 있지만, 더 정확하게는 설정 클래스로 등록해야 한다. 설정 클래스는 'polls' 앱의 `apps.py`에 생성된 클래스 'PollsConfig'를 말한다.

```python
INSTALLED_APPS = [
  ...
  'user', # App 이름에 맞게 설정하기
  'board'
]
```

### DATABASES Engine

```python
DATABASES = {
  'default': {
    'ENGINE': 'django.db.backends.sqlite3',
    'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
  }
}
```


### TIME_ZONE
`UTC` 세계 표준시를 한국시간으로 변경

```python
TIME_ZONE = 'Asia/Seoul'
```

### static 설정

```python
# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/3.0/howto/static-files/

STATIC_URL = '/static/'

STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static')
]
```


### SQLitebrowser
데이터가 DB에 쌓이는 내역을 확인하기 위해 SQLitebrowser를 설치합니다. 
Windows 운영체제 사용자는 알아서 설치하세요

**Homebrew**  
If you prefer using Homebrew for macOS, our latest release can be installed via Homebrew Cask:

```
brew cask install db-browser-for-sqlite
```

## Application Setting

### App 생성

```python
python manage.py startapp board # Windows
python3 manage.py startapp board # mac
python3 manage.py startapp user # user app도 생성해주세요

```


### migrate 진행하기

- `migrate` 진행
- `makemigrations`은 앱의 모델(Model)을 작성하고 적용하기

```python
python manage.py migrate
```


### Folder structure

- Mac OS의 경우 가상환경 설정하기!

```
<Root folder>
│
├<community_project>
│ ├── config
│ ├── board
│ ├── db.sqlite3
│ └── manage.py
└venv (가상환경 설정)
```

### Add Application 
`settings.py`에서 사용할 애플리케이션 등록  
본문의 [INSTALLED_APPS](#installed_apps) 를 참고하여 설정해도 됩니다!


```python
INSTALLED_APPS = [
    'board',
]
```

### URL and View Mapping

```python
from django.contrib import admin
from django.urls import path, include
from user.views import home

urlpatterns = [
    path('board/', include('board.urls')),
]
```

```python
from django.urls import path
from . import views

urlpatterns = [
  path('list/', views.board_list, name='board_list'),
  path('detail/<int:pk>/', views.board_detail),
  path('write/', views.board_write)
]
```

### admin 

`list_display`필드에는 타이틀만 나오게끔 설정  
그리고 보드어드민 연결

```python
from django.contrib import admin
from .models import Board

# Register your models here.
class BoardAdmin(admin.ModelAdmin):
    list_display = ('title', )

admin.site.register(Board, BoardAdmin) 
```

### apps
자동으로 등록이 되는 것 같은데 한 번 확인 부탁드립니다.

### Static Files
`static` 폴더를 생성하고 그 안에 `CSS, JavaScript` 파일을 정리한다. 
`STATIC_URL`은 파일에 접근했을 때 어느 폴더에 있는지 알려주기 위한 변수값
`STATICFILES_DIRS` 안에 리스트로 경로를 넣어주면 된다.
`BASE_DIR`은 기본 프로젝트를 가리킨다.

```
<Root folder>
│
├<community_project>
│ └── static
└venv (가상환경 설정)
```

```python
# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/3.0/howto/static-files/
STATIC_URL = '/static/'

STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static')
]
```

**[⬆ back to top](#table-of-contents)**



## Board Model

writer 필드에는 ForeignKey를 통해 User 테이블과 연결한 값을 넣는다.
ForeignKey 메소드의 인자값으로 전달하는 `on_delete`는 사용자가 작성한 글들에 대한 정책을 설정할 수 있다.
`on_delete`의 값으로 `CASCADE`를 전달하면 user가 삭제되었을 때 user가 작성한 글들도 함께 삭제된다. 

- 아래 내용을 작성 후 
- `python manage.py makemigrations` 진행

```python
from django.db import models

class Board(models.Model):
    title = models.CharField(max_length=128, verbose_name='제목')
    contents = models.TextField(verbose_name='내용') # 길이에 제한이 없음
    writer = models.ForeignKey('user.User', on_delete=models.CASCADE, verbose_name='작성자')

    registered_dttm = models.DateTimeField(auto_now_add=True, verbose_name='등록시간')

    def __str__(self):
        return self.title

    class Meta:
        db_table = 'board'
        verbose_name = '게시글'
        verbose_name_plural = '게시글'
```

