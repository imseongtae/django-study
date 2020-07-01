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
    'bookmark',
]
```


```python
TEMPLATES = [
    {
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
    }
]
```


### URL and View Mapping

```python
from django.contrib import admin
from django.urls import path, include


urlpatterns = [
    path('bookmark/', include('bookmark.urls')),
]
```

### app urls setting

```python
from django.urls import path
from .views import *

# 패스랑 뷰를 임포트 한다.
urlpatterns = [
    path('', BookmarkList.as_view(), name='list'),
    path('add/', BookmarkCreateView.as_view(), name='add'),
    path('detail/<int:pk>/', BookmarkDetailView.as_view(), name='detail'),
    path('_update/<int:pk>/', BookmarkUpdateView.as_view(), name='update'),
    # path('delete/<int:pk>/', BookmarkDeleteView.as_view(), name='delete'),
    path('delete/<int:pk>/', BookmarkDeleteView.as_view(), name='delete'),
]
```

### admin 

`list_display`필드에는 타이틀만 나오게끔 설정  
그리고 보드어드민 연결

```python
from django.contrib import admin
from .models import Bookmark
# Register your models here.

admin.site.register(Bookmark) # 북마크를 넣어주기만 하면 된다.
```

## Model

```python
from django.db import models

# Create your models here.
# URL, NAME
class Bookmark(models.Model):
    site_name = models.CharField(max_length=130)
    url = models.URLField('Site URL')

    def __str__(self):
        return self.site_name +':' + self.url

```

## View

```python
from django.shortcuts import render
from django.views.generic import ListView, CreateView, DetailView, UpdateView, DeleteView
from django.urls import reverse_lazy
from .models import Bookmark

# Create your views here.

class BookmarkList(ListView):
    model = Bookmark
    # context_object_name = 'latest_list'
    paginate_by = 5
    # 한 페이지에 보여줄 개수를 정해준다.

class BookmarkCreateView(CreateView):
    model = Bookmark
    fields = ['site_name', 'url'] # 모델 속성
    success_url = reverse_lazy('list')

class BookmarkDetailView(DetailView):
    model = Bookmark

class BookmarkUpdateView(UpdateView):
    model = Bookmark
    fields = ['site_name', 'url']
    success_url = reverse_lazy('list') # 정상적으로 진행이 되면 리스트로 간다.
    template_name_suffix = '_update'

# class BookmarkDeleteView(DeleteView):
#     model = Bookmark
#     # template_name = 'bookmark_confirm_delete'
#     # fields = ['site_name', 'url']
#     success_url = reverse_lazy('list') # 정상적으로 진행이 되면 리스트로 간다.

class BookmarkDeleteView(DeleteView):
    model = Bookmark
    fields = ['site_name', 'url']
    success_url = reverse_lazy('list')
```


## Template

{% raw %}
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{% block title %} {% endblock %} </title>

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">
    <script src="https://code.jquery.com/jquery-3.4.1.slim.min.js" integrity="sha384-J6qa4849blE2+poT4WnyKhv5vZF5SrPo0iEjwBvKU7imGFAV0wwj1yYfoRSJoZ+n" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js" integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/js/bootstrap.min.js" integrity="sha384-wfSDF2E50Y2D1uUdj0O3uMBJnjuUD4Ih7YwaYd1iqfktj0Uod8GCExl3Og8ifwB6" crossorigin="anonymous"></script>


</head>
<body>
<nav class="navbar navbar-expand-lg navbar-light bg-light">
  <a class="navbar-brand" href="#">Bookmark</a>
  <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
    <span class="navbar-toggler-icon"></span>
  </button>
  <div class="collapse navbar-collapse" id="navbarSupportedContent">
    <ul class="navbar-nav mr-auto">
      <li class="nav-item active">
        <a class="nav-link" href="#">Home <span class="sr-only">(current)</span></a>
      </li>
    </ul>
  </div>
</nav>

<div class="container">
  {% block content %}

    {% endblock %}

</div>

</body>
</html>
```
{% endraw %}



**[⬆ back to top](#table-of-contents)**


