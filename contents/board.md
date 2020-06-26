# Board 




## Table of Contents

1. [Project Setting](#project-setting)
2. [Application Setting](#application-setting)



---



## Project Setting


### config setting

`config` 뒤에 `.`을 찍지 않으면 config 폴더 아래에 config 폴더가 생기므로  `config .`로 프로젝트를 생성할 수 있도록 주의한다

```
django-admin startproject config  # X
django-admin startproject config . # O
```

### Django server 실행

프로젝트가 서버가 올바르게 생성되었는지 확인하기 위하여!


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

```python3
INSTALLED_APPS = [
  ...
  'polls.apps.PollsConfig',
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


### SQLitebrowser
**Homebrew**  
If you prefer using Homebrew for macOS, our latest release can be installed via Homebrew Cask:

```
brew cask install db-browser-for-sqlite
```



## Application Setting

### Folder structure

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
    path('list', views.board_list, name='board_list'),
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

자동으로 등록이 되는 것 같은데?


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