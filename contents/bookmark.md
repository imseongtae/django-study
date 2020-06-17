# Bookmark

pythonanywhere를 통해 배포한 
**[Bookmark Application](http://devhaemil.pythonanywhere.com/bookmark/) 링크**

## Table of Contents

  1. [Settings](#Settings)
  1. [BaseTemplate](#BaseTemplate)
  1. [Board Model](#Board-Model)

 

---

## Settings

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




**[⬆ back to top](#table-of-contents)**


